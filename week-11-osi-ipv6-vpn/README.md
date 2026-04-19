# Week 11: OSI Model, IPv6 & VPN

## Overview

Over the past several weeks, you've built up networking knowledge one layer at a time: MAC addresses and IP in Week 5, TCP in Week 5's Wireshark captures, HTTP in Week 8, TLS in Week 8 and 9, encryption in Week 10. This week we zoom out and look at the big picture.

First, the **OSI model** — a 7-layer framework that organizes everything you've learned. Once you see it, you'll realize you already know most of it; you just didn't have names for the layers.

Then we'll revisit **IPv6**, which we only previewed back in Week 5. With IPv4 addresses practically exhausted, IPv6 is the future — and you'll see why its 128-bit addresses fundamentally change how networks work.

Finally, **VPNs** — a practical application that combines everything: networking (Weeks 5, 8, 9) and encryption (Week 10) come together to create private tunnels across public networks.

## Learning Objectives

By the end of this week, you should be able to:

1. **Explain** the seven layers of the OSI model and what each layer does
2. **Map** the protocols and tools you've already used (Ethernet, IP, TCP, HTTP, TLS) to their OSI layers
3. **Compare** the OSI model to the TCP/IP model and explain why both exist
4. **Read and write** IPv6 addresses, including the shorthand notation rules
5. **Identify** different IPv6 address types (global, link-local, loopback, unique local)
6. **Explain** what a VPN is, how it works, and what it protects against
7. **Configure** an SSH tunnel and a WireGuard VPN, then verify the traffic is encrypted with Wireshark

## Time Allocation

| Phase | Duration | Focus |
|-------|----------|-------|
| Pre-class | 1-2 hours | Reading about OSI, IPv6, VPN + mapping commands to layers |
| Class | 3.5 hours | OSI scavenger hunt, IPv6 hands-on, SSH tunneling, WireGuard setup |
| Post-class | 1-2 hours | Deploy WireGuard to your cloud server, IPv6 deployment audit |

## Prerequisites

- Completed Week 5 (networking basics, Wireshark, TCP/UDP)
- Completed Week 8 (HTTP/HTTPS, Wireshark captures)
- Completed Week 9 (DNS, certificates)
- Completed Week 10 (symmetric/asymmetric encryption, SSH keys)
- Docker installed and running
- Wireshark installed and working

## Videos (pick any — each is self-contained)

All videos live at **[tek2.apps.tobiasgrundtvig.dk/week-11](https://tek2.apps.tobiasgrundtvig.dk/#week-11)**.

| 🎬 Video | Duration | What it covers |
|---|---|---|
| [The OSI Model](https://tek2.apps.tobiasgrundtvig.dk/week-11/osi-model/) | 10:45 | A single map for everything you already know about networking |
| [IPv6 fundamentals — why the internet needed a bigger phone book](https://tek2.apps.tobiasgrundtvig.dk/week-11/ipv6-fundamentals/) | 3:48 | IPv4 gave us 4 billion addresses |
| [VPN fundamentals — encrypted tunnels](https://tek2.apps.tobiasgrundtvig.dk/week-11/vpn-fundamentals/) | 3:03 | A VPN wraps your network traffic in an encrypted tunnel, making it look like one point-… |

## Materials

| File | Description |
|------|-------------|
| [quick-reference.md](quick-reference.md) | One-page cheat sheet (print this!) |
| [🎬 OSI Model video (10 min)](https://tek2.apps.tobiasgrundtvig.dk/week-11/) | Pre-class video — start here |
| [pre-class/reading.md](pre-class/reading.md) | OSI model, IPv6 addressing, VPN fundamentals |
| [pre-class/exercises.md](pre-class/exercises.md) | Map commands to OSI layers, explore IPv6 on your machine |
| [class/exercises.md](class/exercises.md) | OSI scavenger hunt, IPv6 in Docker, SSH tunneling, WireGuard VPN |
| [post-class/advanced-tasks.md](post-class/advanced-tasks.md) | Deploy a real VPN, audit IPv6 adoption, design a network |
| [post-class/hints.md](post-class/hints.md) | Hints for advanced tasks |

## Key Concepts

- **OSI Model**: A 7-layer reference model (Physical, Data Link, Network, Transport, Session, Presentation, Application) used to describe how networks work
- **TCP/IP Model**: A simpler 4-layer model that matches real-world protocols — what the internet actually uses
- **Encapsulation**: How data gets wrapped with headers as it travels down the layers (and unwrapped on the way up)
- **IPv6**: The successor to IPv4, with 128-bit addresses (~340 undecillion possible addresses)
- **Dual-stack**: Running IPv4 and IPv6 side by side during the transition period
- **VPN**: Virtual Private Network — an encrypted tunnel that makes a public network behave like a private one
- **WireGuard**: A modern, fast, simple VPN protocol built on well-known cryptographic primitives
- **SSH tunnel**: A simpler, VPN-like encrypted channel that forwards specific ports through an SSH connection

## What's Next

Week 11 is the last "theory" week. In Week 12 you'll do a one-week AI project, and then Week 13 is dedicated to exam preparation — where everything from Weeks 1-11 comes together. After that, Weeks 14-16 are your final group project.

By now you should see the through-line of this course: you can build software (Weeks 1-3), deploy it to a real server (Week 4), understand every layer of how it talks to the world (Weeks 5-9), secure it (Week 10), and tunnel it privately (Week 11). That's the full DevOps and networking foundation.
