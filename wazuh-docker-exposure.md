# My First Wazuh Alert: Finding Exposed Docker Ports

*Posted November 3, 2026 · Also on [cyberhippo.ca](https://cyberhippo.ca/writeups/wazuh-docker-exposure)*

I installed Wazuh, a free and open-source SIEM, on my Proxmox host and connected my Mac mini as the first agent. The first alert I worked through looked harmless. Reading the full alert instead of just the change led me to two services that were open to my whole network when they didn't need to be: the Docker API and a leftover DNS server.

**Result:** both ports closed. The dashboard that needed Docker access now reaches it only over an internal Docker network, and it no longer has the Docker socket mounted directly.

## 1. Setting up Wazuh

Wazuh collects logs and security events from my machines, matches them against detection rules, and shows the results in a web dashboard. It's also close to what a junior SOC analyst works with day to day, which is why I picked it.

| Piece       | Setup                                                             | Why                                                                                                              |
|-------------|-------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| Host        | Ubuntu Server 24.04 VM on Proxmox, 4 vCPU, 10 GB RAM, 200 GB disk | Ubuntu 24.04 is on Wazuh's supported list, and the specs are above the minimum for up to 25 agents               |
| Clock       | Confirmed NTP sync before installing                              | A SIEM orders events by time, and its certificates depend on the right time                                      |
| Install     | Wazuh's all-in-one installer                                      | Puts the indexer, server and dashboard on one host and generates their certificates and passwords                |
| Upgrades    | Disabled the Wazuh apt source after installing                    | Stops routine Ubuntu updates from upgrading Wazuh's parts out of step with each other, as Wazuh's docs recommend |
| First agent | The Mac mini, deployed from the dashboard                         | It runs the most services, so it's the most interesting machine to watch                                         |

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
sudo sed -i "s/^deb /#deb /" /etc/apt/sources.list.d/wazuh.list
sudo apt update
```

The admin password went into my password manager. The installer's file of internal passwords and certificates stays off GitHub.

A few things confused me on day one:

| What I saw                        | What it meant                                                                                                                           |
|-----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| Agent stuck on "Pending"          | The agent had registered but hadn't sent data yet. It went Active a minute later; the first check-in was just slow.                     |
| A wall of text in the events view | The default view shows every field in one column. Adding `agent.name`, `rule.level` and `rule.description` as columns made it readable. |
| Lots of "failed" results          | Configuration checks against a hardening benchmark, run when an agent first connects. A hardening to-do list, not attacks.              |

## 2. The first alert

The first alert I worked through was rule 533, level 7: **"Listened ports status (netstat) changed"** on the Mac mini.

The alert includes the full list of listening ports now (`full_log`) and at the last check (`previous_log`). Comparing the two, the only change was the printing service's port (631, local only) closing. That part was benign.

But the alert also gave me the full list of open ports, so I went through it and matched each one to something I run. Two didn't fit:

| Port     | What it was                                                                      | Why it mattered                                                                                                                                |
|----------|----------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| TCP 2375 | A Docker socket proxy, published to the network                                  | Anyone who can reach the Docker API can see every container, and container details can include environment variables, which often hold secrets |
| 53       | A Pi-hole container left over from before I moved DNS filtering onto my firewall | A DNS server nothing used anymore, still answering on the network                                                                              |

## 3. Tracing the ports

On the Mac, both ports belonged to Docker Desktop, which opens ports on the host on behalf of containers:

```bash
sudo lsof -nP -iTCP:2375 -sTCP:LISTEN
sudo lsof -nP -i :53
```

Then I matched them to containers:

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}" | grep -E "2375|:53"
```

## 4. Fixing the Docker API

The proxy exists so my Homepage dashboard can show container status without full Docker access. It was already locked down well: it blocks any request that changes something, and it mounts the Docker socket read-only. The only problem was that it was published to the network, because Homepage reached it through the Mac's IP address.

The fix was to keep the traffic inside Docker:

1.  Moved the proxy onto Homepage's Docker network and removed its `ports:` section, so it's no longer published on the Mac.
2.  Turned off the Docker Swarm API sections I don't use.
3.  Pointed Homepage at the proxy by container name instead of IP address.
4.  Removed Homepage's own direct mount of the Docker socket, since it now goes through the proxy.

Three things tripped me up:

- **Containers couldn't find each other by name.** Each Docker Compose project gets its own default network, and containers on different networks can't see each other. The proxy was in a separate Compose project, so I connected it to Homepage's network as an external network.
- **Homepage broke after I closed the port.** Every container widget showed a "connection refused" error to port 2375 on the Mac's IP address. Running a request from inside the Homepage container straight to the proxy by name returned Docker's version info, so the network was fine and the problem had to be config. Reading the config file from inside the container showed it still pointed at the Mac's IP: my edit had never saved. Checking the file itself, instead of assuming from a search result, found it in one step.
- **The mount change didn't apply at first.** Mounts are set when a container is created, so `docker compose restart` keeps the old ones. `docker compose up -d` recreates the container with the new settings.

## 5. The leftover Pi-hole

Before turning it off, I checked that nothing still depended on it: the firewall's DHCP doesn't hand it out as a DNS server, and the firewall's resolver doesn't forward to it. Then I stopped the container, keeping it around for a few days in case anything broke.

## 6. Verifying the fix

- `docker ps` shows the proxy as `2375/tcp` with no host mapping, so it's only reachable inside its Docker network.
- `docker inspect` shows no Docker socket mount on Homepage.
- Homepage's container widgets work again, now through the proxy.
- Ports 2375 and 53 no longer appear on the Mac's network interfaces.

## What I learned

- **Read the whole alert, not just the change.** The change in this alert was harmless. The useful part was the full port list that came with it.
- **Know what every open port is for.** Both findings were things I'd set up myself and forgotten about.
- **A well-configured service can still be exposed.** The proxy's own settings were good. The risk came from how it was published.
- **Test the network and the config separately.** Once a request from inside the container worked, I knew the problem was the config, not the network.
- **Check what depends on something before removing it.** Confirming nothing used the old DNS server first meant stopping it broke nothing.

## Next steps

- Add agents for my Windows laptop and the Proxmox host
- Send the firewall's logs to Wazuh
- Move the lab onto its own VLAN

## References

- [Wazuh 4.14: quickstart (all-in-one install)](https://documentation.wazuh.com/4.14/quickstart.html)
- [Docker: protect the Docker daemon socket](https://docs.docker.com/engine/security/protect-access/)
- [Docker Compose: networking](https://docs.docker.com/compose/how-tos/networking/)

---

[All write-ups](README.md) · [cyberhippo.ca](https://cyberhippo.ca)
