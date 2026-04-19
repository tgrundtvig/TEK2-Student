# Week 9: DNS & Certificates

## Overview

In Week 8, you saw HTTP requests and responses traveling inside TCP packets. You used `curl`, Wireshark, and browser dev tools to inspect traffic. But we glossed over two important questions: how does your browser know that `google.com` is at `142.250.74.46`? And how does it know the server is *really* Google, not an imposter?

This week answers both questions. **DNS** (Domain Name System) translates human-readable names into IP addresses. **Certificates** prove a server's identity and enable the encryption you saw in HTTPS.

## Learning Objectives

By the end of this week, you should be able to:

1. **Explain** how DNS translates domain names into IP addresses
2. **Use** `dig` and `nslookup` to query DNS records and interpret the results
3. **Identify** different DNS record types (A, AAAA, CNAME, MX, NS, TXT) and their purposes
4. **Describe** the DNS resolution process from root servers to authoritative nameservers
5. **Inspect** TLS certificates using browser tools and `openssl`
6. **Explain** the certificate chain of trust (root CA → intermediate CA → server certificate)
7. **Connect** DNS and certificates to the full HTTPS flow from Week 8

## Time Allocation

| Phase | Duration | Focus |
|-------|----------|-------|
| Pre-class | 1-2 hours | Reading about DNS and certificates + `dig` and certificate inspection |
| Class | 3.5 hours | DNS deep dive, Wireshark DNS capture, certificate chain analysis |
| Post-class | 1-2 hours | Advanced DNS analysis + certificate chain exploration |

## Prerequisites

- Completed Week 8 (HTTP/HTTPS — `curl`, Wireshark, browser dev tools)
- Wireshark installed and working
- Docker installed and running

## Videos (pick any — each is self-contained)

All videos live at **[tek2.apps.tobiasgrundtvig.dk/week-09](https://tek2.apps.tobiasgrundtvig.dk/#week-09)**.

| 🎬 Video | Duration | What it covers |
|---|---|---|
| [What DNS is — the internet's phone book](https://tek2.apps.tobiasgrundtvig.dk/week-09/what-is-dns/) | 2:53 | You type google.com, not 142.250.74.46 |
| [The DNS hierarchy — root, TLD, authoritative](https://tek2.apps.tobiasgrundtvig.dk/week-09/dns-hierarchy/) | 3:04 | DNS isn't one giant server in a basement somewhere |
| [DNS record types — A, AAAA, CNAME, MX, NS, TXT](https://tek2.apps.tobiasgrundtvig.dk/week-09/dns-record-types/) | 3:34 | DNS doesn't just map a name to one IP |
| [dig — reading DNS output like a pro](https://tek2.apps.tobiasgrundtvig.dk/week-09/dig-essentials/) | 2:46 | dig is the practical tool for inspecting DNS |
| [DNS TTL and caching — why changes take time](https://tek2.apps.tobiasgrundtvig.dk/week-09/dns-ttl-and-caching/) | 3:28 | Every DNS record has a TTL — time to live — that says how long answers can be cached |
| [TLS certificate anatomy — what's actually inside the padlock](https://tek2.apps.tobiasgrundtvig.dk/week-09/tls-certificate-anatomy/) | 3:09 | Click the padlock and you see 'Issued to: github.com' |
| [The chain of trust — root, intermediate, server](https://tek2.apps.tobiasgrundtvig.dk/week-09/chain-of-trust/) | 3:33 | A server cert alone isn't trusted — your browser has never heard of the issuing company |
| [Let's Encrypt and ACME — free automated certificates](https://tek2.apps.tobiasgrundtvig.dk/week-09/lets-encrypt-and-acme/) | 3:44 | Ten years ago, HTTPS certificates cost money and had to be renewed by hand |
| [The full HTTPS chain — DNS, TCP, TLS, HTTP](https://tek2.apps.tobiasgrundtvig.dk/week-09/full-https-chain/) | 4:04 | Type github.com |

## Materials

| File | Description |
|------|-------------|
| [quick-reference.md](quick-reference.md) | One-page cheat sheet (print this!) |
| [pre-class/reading.md](pre-class/reading.md) | DNS resolution, record types, certificate chain of trust |
| [pre-class/exercises.md](pre-class/exercises.md) | Hands-on with `dig`, `nslookup`, and certificate inspection |
| [class/exercises.md](class/exercises.md) | DNS deep dive, Wireshark DNS capture, certificate chain analysis |
| [post-class/advanced-tasks.md](post-class/advanced-tasks.md) | Advanced DNS and certificate challenges |
| [post-class/hints.md](post-class/hints.md) | Hints for advanced tasks |

## Key Concepts

- **DNS**: Domain Name System — translates domain names (like `google.com`) into IP addresses (like `142.250.74.46`)
- **DNS Record**: An entry in the DNS database (A = IPv4 address, AAAA = IPv6, CNAME = alias, MX = mail server)
- **Nameserver**: A server that answers DNS queries — either recursively (asks other servers) or authoritatively (knows the answer)
- **TTL**: Time To Live — how long a DNS response can be cached before asking again
- **TLS Certificate**: A digital document that proves a server's identity and contains its public key
- **Certificate Authority (CA)**: A trusted organization that issues and signs certificates
- **Chain of Trust**: The path from a server's certificate through intermediate CAs to a trusted root CA

## What's Next

In Week 10, we'll explore the encryption that powers all of this — symmetric vs asymmetric encryption, SSH keys, and Linux security. You'll understand *how* the scrambling actually works inside that TLS handshake you saw in Wireshark.
