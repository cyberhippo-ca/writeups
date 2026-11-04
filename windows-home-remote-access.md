# Remote Access to a Windows Home Laptop: SSH Keys and VNC Through a Tunnel

*Posted November 4, 2026 · Also on [cyberhippo.ca](https://cyberhippo.ca/writeups/windows-home-remote-access)*

I wanted to manage the always-on Windows laptop in my lab from my MacBook, and install a Wazuh agent on it without walking over to it. The laptop runs Windows 11 Home, which can't accept Remote Desktop connections. So I set up SSH for the command line and VNC for the screen, with VNC only reachable through the SSH connection.

**Result:** SSH with key-only login, passwords turned off, screen sharing that never opens a port to the network, and the laptop reporting to Wazuh as my second agent.

## The plan

| Piece            | Choice                                                      | Why                                                                                                                                         |
|------------------|-------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| Command line     | OpenSSH Server, built into Windows as an optional feature   | Works on Windows Home, and it's how I already manage my Linux machines                                                                      |
| Login            | SSH keys only                                               | Nothing to guess or brute-force                                                                                                             |
| Screen           | TightVNC, accepting connections from the laptop itself only | Works on Windows Home                                                                                                                       |
| Security for VNC | An SSH tunnel                                               | VNC passwords max out at 8 characters and VNC traffic isn't encrypted. The tunnel fixes both, and the VNC port is never open to the network |

## 1. Installing the SSH server

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

The install downloads from Windows Update and sat on "Operation running" for a long time with no progress bar. Checking from a second window showed it had actually finished:

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

## 2. The wrong IP address

The first address I found was a 172.x address. It belonged to the virtual adapter that WSL2 adds, which is internal to the laptop. This listed only the real adapters:

```powershell
Get-NetIPAddress -AddressFamily IPv4 | Where-Object InterfaceAlias -notmatch 'vEthernet|Loopback' | Select-Object InterfaceAlias, IPAddress
```

## 3. SSH timed out

From the Mac, the connection hung, and a port test on 22 timed out. Two things were in the way:

- **The Wi-Fi was set to Public.** Windows applies its strictest firewall profile to Public networks. I changed the home network to Private, which is the right setting for a network you trust.
- **A VPN was connected on the laptop.** VPN clients can route or block local network traffic. I disconnected it.

I changed both before testing again, so I can't say which one was blocking. Next time I'd change one at a time.

## 4. Key-only login

For administrator accounts, Windows' SSH server only reads keys from one shared file, not the user's own folder. The file also has to be locked down, or the server ignores it:

```powershell
Add-Content -Path C:\ProgramData\ssh\administrators_authorized_keys -Value '<public key>' -Encoding ascii
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
```

I tested the key with password login disabled on the client side, then turned off passwords on the server and restarted it. A password-only attempt now gets `Permission denied`.

One catch: when `ssh-keygen` offered to overwrite my existing key, I said no. Overwriting a key breaks everything else that key is used for.

## 5. VNC through the tunnel

```text
Mac localhost:5901  ──SSH tunnel──▶  laptop 127.0.0.1:5900 (VNC server)
```

The VNC server only accepts connections from the laptop itself, and there's no firewall exception for it. The tunnel delivers the connection to the laptop, so from the server's side it arrives locally:

```bash
ssh -L 5901:127.0.0.1:5900 user@laptop
```

Getting it to work took five fixes:

| Symptom                                                             | Cause                                                                                                                                                        | Fix                                                      |
|---------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------|
| The tunnel reached nothing, though the server said it was listening | Likely `localhost` resolving to the IPv6 address on the Windows side, while the server listened on IPv4                                                      | Tunnel to `127.0.0.1` instead of `localhost`             |
| macOS Screen Sharing: "incompatible with this version"              | Apple's viewer doesn't support the options this VNC server uses                                                                                              | TigerVNC Viewer, which is free and open source           |
| "No matching security types"                                        | The newest TightVNC (4.x) didn't offer a login method other viewers support. Its download page says encrypted connections need its paid version at both ends | Switched to the older TightVNC 2.8 release, which the project still supports |
| "Couldn't run server in application mode"                           | I'd opened the app-mode shortcut, which I'd disabled at install                                                                                              | Opened the service's own settings instead                |
| "Authentication failed"                                             | Password mismatch                                                                                                                                            | Reset an 8-character password and restarted the service  |

## 6. Checking it's not exposed

From the Mac, without the tunnel, a test on the VNC port times out. The only way to reach the screen is through an SSH connection, which needs my key.

## 7. The Wazuh agent

With SSH working, I installed the Wazuh agent from the commands the Wazuh dashboard generates. The silent installer returns before it finishes, so the "start service" step can run too early. Checking the service and starting it fixed that, and the laptop showed up as Active. The agent covers Windows only, not the WSL2 Linux environment on the same laptop.

## What I learned

- **Network profile matters on Windows.** The same firewall behaves very differently on a Public network.
- **Change one thing at a time.** I didn't, and I still don't know which of two settings blocked SSH.
- **Layer it.** VNC on its own is weak. Loopback only plus an SSH tunnel plus key-only SSH makes it fine.
- **Newest isn't always compatible.** The current TightVNC wouldn't work with standard viewers; the older supported release did.
- **Read the error, then check each half.** Each failure moved one step further along: network, tunnel, server, viewer, password.

## References

- [Microsoft: Get started with OpenSSH for Windows](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse)
- [Microsoft: Key-based authentication in OpenSSH for Windows](https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement)
- [TightVNC downloads](https://www.tightvnc.com/download/)
- [TigerVNC](https://tigervnc.org/)

---

[All write-ups](README.md) · [cyberhippo.ca](https://cyberhippo.ca)
