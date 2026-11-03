# Tracking Down a 100 Mbps Bottleneck

*Posted November 3, 2026 · Also on [cyberhippo.ca](https://cyberhippo.ca/writeups/slow-internet)*

After rewiring my network, the whole house was stuck at about 84 Mbps. The speed test numbers pointed to a link running at 100 Mbps somewhere. Testing one piece at a time found a damaged network cable. The same day, a TV that couldn't stream from my Plex server turned out to be plugged in on the wrong side of the firewall.

**Result:** a new cable brought the connection back to full speed, and the TV now streams from Plex on the local network.

## The layout

The house's fibre box (ONT) has only one active port. An unmanaged switch on that port feeds two things: the household router, which I leave alone, and my firewall, which runs the lab network. The ISP gives each its own public IP.

```text
Fibre ONT
  └─ Unmanaged switch
       ├─ Household router
       └─ Firewall (OPNsense)
            └─ Lab switch ── lab devices
```

## 1. Reading the speed test

Speed tests showed **84 Mbps down and 81 up**, and it affected the whole house.

Two things stood out:

- **Download and upload were almost equal.** A slow connection from the ISP doesn't usually look that even.
- **Both were just under 100.** A 100 Mbps network link can't deliver the full 100 once overhead is taken out, so a result in the 80s and low 90s is what a 100 Mbps link looks like.

Since the whole house was affected, I started with what everything shares: the switches and cables near the ONT.

## 2. One change at a time

My first move was to unplug the firewall. That changed two things at once, the firewall and its cables, so the result couldn't tell me which one mattered.

I started over and tested one switch and one link at a time, with the firewall unplugged. The lab switch's port lights gave the answer: on the link to the unmanaged switch, the port showed **one light (100 Mbps) instead of two (gigabit)**.

## 3. The cause: a damaged cable

Gigabit Ethernet uses all four wire pairs in a cable. 100 Mbps only needs two. If a cable has a damaged pair, the devices on each end don't report an error. They quietly agree on 100 Mbps instead, and everything still works, just slower.

A new cable brought the link back to gigabit and the speed tests back to full speed.

## 4. The TV that couldn't stream

The same day, a TV I'd wired in gave a "not fast enough" playback error when streaming from my Plex server on the lab network.

The TV was plugged into the **unmanaged switch at the ONT**. That put it directly on the internet, outside both the household network and the lab. To Plex, the TV looked like a remote device, so instead of streaming over the local network it fell back to Plex's relay, which is heavily speed-limited.

| Step                    | What I found                                                           | Fix                                                                    |
|-------------------------|------------------------------------------------------------------------|------------------------------------------------------------------------|
| Where it was plugged in | The switch in front of the firewall, so it was outside the lab network | Moved it behind the firewall, onto the same network as the Plex server |
| Its wired port          | 100 Mbps only                                                          | Switched it to 5 GHz Wi-Fi, which is faster than its wired port        |

A switch in front of a firewall is outside the network, even when it's sitting in the same rack.

## What I learned

- **The numbers can point to the cause.** Download and upload both just under 100 meant a 100 Mbps link, not a slow ISP.
- **Change one thing at a time.** My first test changed two things and told me nothing.
- **Check the port lights.** They showed the link speed right away, with no tools.
- **A bad cable doesn't always fail.** It can drop to 100 Mbps and keep working, which makes it easy to miss.
- **Know which side of the firewall a device is on.** The TV was plugged into the right rack but the wrong network.

## References

- [Wikipedia: Gigabit Ethernet](https://en.wikipedia.org/wiki/Gigabit_Ethernet)
- [Wikipedia: Fast Ethernet (100 Mbps)](https://en.wikipedia.org/wiki/Fast_Ethernet)

---

[All write-ups](README.md) · [cyberhippo.ca](https://cyberhippo.ca)
