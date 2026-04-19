# Week 5: Networking Basics

## Overview

Since Week 1, you've been using networks without thinking about it: port mapping with `-p 8080:80`, SSH into cloud servers, firewall rules with `ufw`. This week, we pull back the curtain and understand how computer networks actually work.

By the end of this week, you will understand IP addresses, ports, TCP vs UDP, and how Docker's networking connects to these fundamental concepts.

## Learning Objectives

By the end of this week, you should be able to:

1. **Explain** how devices on a network find and communicate with each other
2. **Identify** MAC addresses, IP addresses, and ports on your own machine and in containers
3. **Compare** TCP and UDP and explain when each is used
4. **Use** networking tools (ping, ip addr, netcat, ss, Wireshark) to explore network behavior
5. **Analyze** basic network traffic using Wireshark
6. **Connect** networking concepts to Docker port mapping, SSH, and firewall rules from earlier weeks

## Time Allocation

| Phase | Duration | Focus |
|-------|----------|-------|
| Pre-class | 1-2 hours | Reading + Wireshark installation + network exploration |
| Class | 3.5 hours | Hands-on networking exercises + Docker networks + Wireshark |
| Post-class | 1-2 hours | Advanced analysis + cloud server investigation |

## Prerequisites

- Completed Weeks 1-4 (Docker, Linux commands, CI/CD, cloud deployment)
- Docker installed and running
- A computer with administrator/sudo access (for Wireshark installation)

## Videos (pick any — each is self-contained)

All videos live at **[tek2.apps.tobiasgrundtvig.dk/week-05](https://tek2.apps.tobiasgrundtvig.dk/#week-05)**.

| 🎬 Video | Duration | What it covers |
|---|---|---|
| [IP vs MAC — why you need both](https://tek2.apps.tobiasgrundtvig.dk/week-05/ip-vs-mac/) | 2:23 | Your laptop has two addresses |
| [Private vs public IPs and NAT](https://tek2.apps.tobiasgrundtvig.dk/week-05/private-vs-public-ip/) | 3:19 | Your laptop says 192.168.1.42 |
| [Ports — which app gets the data](https://tek2.apps.tobiasgrundtvig.dk/week-05/ports-explained/) | 2:56 | An IP address gets a packet to the right machine |
| [TCP vs UDP — reliable vs fast](https://tek2.apps.tobiasgrundtvig.dk/week-05/tcp-vs-udp/) | 2:32 | Two transport protocols |
| [The TCP three-way handshake](https://tek2.apps.tobiasgrundtvig.dk/week-05/tcp-handshake/) | 2:45 | Before any data flows over TCP, the client and server shake hands |
| [docker run -p 8080:80 — what's actually happening](https://tek2.apps.tobiasgrundtvig.dk/week-05/docker-port-mapping/) | 2:22 | The -p flag says: forward host port 8080 to container port 80 |
| [Docker networks — bridge, custom, and DNS](https://tek2.apps.tobiasgrundtvig.dk/week-05/docker-networks/) | 2:33 | Docker runs its own tiny DNS server, but only on user-defined networks |
| [Wireshark — your first capture](https://tek2.apps.tobiasgrundtvig.dk/week-05/wireshark-first-capture/) | 2:57 | Open Wireshark, pick an interface, start capturing |

## Materials

| File | Description |
|------|-------------|
| [quick-reference.md](quick-reference.md) | One-page cheat sheet (print this!) |
| [pre-class/reading.md](pre-class/reading.md) | Conceptual introduction to networking |
| [pre-class/exercises.md](pre-class/exercises.md) | Wireshark installation and first network exploration |
| [class/exercises.md](class/exercises.md) | Guided hands-on networking exercises |
| [post-class/advanced-tasks.md](post-class/advanced-tasks.md) | Challenges to deepen understanding |
| [post-class/hints.md](post-class/hints.md) | Hints for advanced tasks |

## Key Concepts

- **IP Address**: A logical address that identifies a device on a network (like a postal address)
- **MAC Address**: A hardware address burned into a network card (like a serial number)
- **TCP**: Transmission Control Protocol — reliable, ordered delivery of data
- **UDP**: User Datagram Protocol — fast, no-guarantee delivery of data
- **Port**: A number (0-65535) that identifies which application receives the data
- **Packet**: A small chunk of data sent across a network

## What's Next

In Week 8, we'll use Wireshark to look inside the TCP connections you learned about this week, exploring the HTTP and HTTPS protocols that power the web.
