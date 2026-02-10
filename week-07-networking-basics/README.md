# Week 7: Networking Basics

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
