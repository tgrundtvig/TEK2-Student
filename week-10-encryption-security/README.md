# Week 10: Encryption + Security

## Overview

In Week 8, you saw that HTTPS traffic appears as "encrypted noise" in Wireshark. In Week 9, you inspected TLS certificates and learned that they contain a "public key" — but what does that actually mean? And when you used SSH keys back in Week 4 to connect to your server, we called it "cryptographic proof" without explaining how that works.

This week pulls back the curtain. You'll understand **how encryption actually works** — symmetric and asymmetric algorithms, hashing, and how TLS combines them all. By the end, those SSH keys and TLS handshakes won't be magic anymore.

## Learning Objectives

By the end of this week, you should be able to:

1. **Explain** the difference between symmetric and asymmetric encryption
2. **Encrypt and decrypt** files using `openssl` (symmetric AES and asymmetric RSA)
3. **Describe** how TLS uses both symmetric and asymmetric encryption together
4. **Generate** and use GPG key pairs to encrypt messages for others
5. **Explain** what hashing is and how it differs from encryption
6. **Connect** SSH keys (Week 4) and TLS certificates (Week 9) to asymmetric cryptography
7. **Apply** basic Linux security practices (file permissions, open ports, sudo)

## Time Allocation

| Phase | Duration | Focus |
|-------|----------|-------|
| Pre-class | 1-2 hours | Reading about encryption concepts + hands-on with `openssl` |
| Class | 3.5 hours | GPG encrypted messaging, SSH key anatomy, Linux security, security challenge |
| Post-class | 1-2 hours | GPG-signed git commits, SSH hardening, deeper exploration |

## Prerequisites

- Completed Week 4 (SSH keys to connect to your VM)
- Completed Week 8 (HTTP/HTTPS, Wireshark TLS handshake)
- Completed Week 9 (TLS certificates, chain of trust)
- Docker installed and running

## Materials

| File | Description |
|------|-------------|
| [quick-reference.md](quick-reference.md) | One-page cheat sheet (print this!) |
| [pre-class/reading.md](pre-class/reading.md) | Symmetric vs asymmetric encryption, hashing, how TLS uses both |
| [pre-class/exercises.md](pre-class/exercises.md) | Hands-on with `openssl` — encrypt, decrypt, hash |
| [class/exercises.md](class/exercises.md) | GPG messaging, SSH key anatomy, Linux security, security challenge |
| [post-class/advanced-tasks.md](post-class/advanced-tasks.md) | GPG-signed commits, SSH hardening, certificate pinning |
| [post-class/hints.md](post-class/hints.md) | Hints for advanced tasks |

## Key Concepts

- **Symmetric encryption**: One key encrypts and decrypts (AES). Fast, used for bulk data.
- **Asymmetric encryption**: A key *pair* — public key encrypts, only the private key decrypts (RSA, Ed25519). Slower, used for key exchange and authentication.
- **Hashing**: A one-way function that produces a fixed-size fingerprint (SHA-256). Not encryption — you can't reverse it.
- **TLS handshake**: Uses asymmetric encryption to safely exchange a symmetric key, then switches to symmetric for speed.
- **Digital signature**: The reverse of encryption — sign with your private key, anyone can verify with your public key.
- **File permissions**: Linux controls who can read, write, and execute files using owner/group/others and `chmod`.

## What's Next

In Week 11, we'll zoom out and look at the big picture — the OSI model that organizes everything you've learned (TCP, IP, HTTP, TLS) into layers. We'll also dive deeper into IPv6 and explore VPNs, which combine networking and encryption concepts from this week.
