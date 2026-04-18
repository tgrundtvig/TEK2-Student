# Pre-Class Reading: OSI Model, IPv6 & VPN

**Estimated reading time: 45-60 minutes**

Read through this before class. You don't need to memorize everything — focus on the big picture. The exercises will pin down the details.

> **Watch first:** [**The OSI Model** (10-minute video)](https://tek2.apps.tobiasgrundtvig.dk/week-11/) — a fast way to get the whole picture before you dig into the text below.

---

## Introduction: Zooming Out

You've now spent weeks looking at individual pieces of networking:

- **Week 5**: MAC addresses, IP addresses, TCP, UDP, ports
- **Week 8**: HTTP, HTTPS, Wireshark packet analysis
- **Week 9**: DNS, TLS certificates, chain of trust
- **Week 10**: Symmetric and asymmetric encryption, how TLS uses both

This week we do three things with that foundation:

1. **Organize it** using the OSI model — a 7-layer framework that groups protocols by what they do
2. **Extend it** with IPv6 — the next-generation addressing scheme that's slowly replacing IPv4
3. **Apply it** with VPNs — combining encryption and networking to create private tunnels across public networks

This is the last "theory" week before the project phase. When you finish, you'll have the vocabulary to discuss networks at any level of detail.

---

## Part 1: The OSI Model — A Map for What You Already Know

### Why Do We Need a Model?

Networks are complicated. When you open a web page, dozens of things happen: cables conduct electricity, radio waves carry Wi-Fi signals, MAC addresses route frames locally, IP packets cross continents, TCP guarantees reliable delivery, TLS encrypts everything, and HTTP asks the server for a page.

If we tried to think about all of this at once, our brains would melt. So engineers decided to divide the problem into **layers** — each layer does one job well and relies on the layer below it for services.

The **OSI (Open Systems Interconnection) model** is the most famous of these layering schemes. It was designed by the International Organization for Standardization in the late 1970s, and while the real internet ended up using a slightly different model, OSI remains the standard vocabulary for talking about networks.

### The Seven Layers

```
┌─────────────────────────────────────────────────────────────┐
│ 7. Application   │ What the user sees                        │
│ 6. Presentation  │ Data format, encoding, encryption         │
│ 5. Session       │ Sessions and conversations                │
│ 4. Transport     │ End-to-end delivery between applications  │
│ 3. Network       │ Routing between networks (global)         │
│ 2. Data Link     │ Frames within one local network           │
│ 1. Physical      │ Cables, radio waves, electrical signals   │
└─────────────────────────────────────────────────────────────┘
```

**Mnemonic:** *All People Seem To Need Data Processing* (top to bottom)
Or the bottom-up version: *Please Do Not Throw Sausage Pizza Away*

Let's walk through each layer with something you've already done.

### Layer 1: Physical

**Job:** Move raw bits from one device to another.

This is the physics layer: electrical voltages on copper cables, light pulses in fiber optic, radio waves in Wi-Fi. A `1` and a `0` are physical phenomena — different voltage levels, different flashes of light.

You haven't directly worked at Layer 1 in this course, but you're using it constantly. Every time you run `ping`, the request travels as physical signals through your Ethernet cable or Wi-Fi antenna.

**You don't program at this layer** — operating systems and hardware handle it. But understanding it exists is important: when your Wi-Fi is flaky or your cable is unplugged, everything above Layer 1 breaks.

### Layer 2: Data Link

**Job:** Move frames between two devices on the same local network.

This is where **MAC addresses** and **Ethernet** live. When your laptop sends data to your router, the Ethernet frame contains:

```
┌────────────┬────────────┬───────────────┬─────────┐
│ Dst MAC    │ Src MAC    │ Type (e.g. IP)│ Payload │
└────────────┴────────────┴───────────────┴─────────┘
```

Switches (the boxes with many Ethernet ports) operate at Layer 2. They learn which MAC addresses live on which port and forward frames accordingly. **Switches don't understand IP** — they just see "send this frame to MAC `aa:bb:cc:...`" and do it.

**From Week 5:** Remember ARP? That's how a device at Layer 3 (which thinks in IP) finds the Layer 2 MAC address to send to.

### Layer 3: Network

**Job:** Route packets across multiple networks (potentially the whole internet).

This is where **IP** lives — both IPv4 and IPv6. While MAC addresses only work locally, IP addresses are globally routable. Every packet carries:

```
┌────────────┬────────────┬─────────────┬──────────┐
│ Version+TTL│ Src IP     │ Dst IP      │ Payload  │
└────────────┴────────────┴─────────────┴──────────┘
```

**Routers** operate at Layer 3. They look at the destination IP address, consult their routing table, and decide which neighbor to forward the packet to.

**From Week 5:** The `ping` command uses ICMP, which is part of Layer 3. When you `ping google.com`, your computer sends ICMP echo requests inside IP packets.

**From Week 9:** DNS is technically Layer 7, but it produces Layer 3 addresses — translating `google.com` into `142.250.74.46`.

### Layer 4: Transport

**Job:** End-to-end data delivery between applications on different machines.

This is where **TCP** and **UDP** live. IP gets data from one machine to another, but doesn't care *which application* on that machine should receive it. Layer 4 solves that with **port numbers** and adds reliability (TCP) or speed (UDP).

TCP adds:
- Port numbers (source and destination)
- Sequence numbers (for ordering)
- Acknowledgements (for reliability)
- Flow control (so the receiver isn't overwhelmed)

**From Week 5:** The three-way handshake (SYN → SYN-ACK → ACK) you saw in Wireshark — that's TCP at Layer 4.

**From Week 8:** Every HTTP request rides on top of a TCP connection. "Port 80 for HTTP" and "port 443 for HTTPS" are Layer 4 concepts.

### Layer 5: Session

**Job:** Manage the dialog — establish, maintain, and terminate sessions between applications.

This layer is a bit of a ghost. In theory it handles things like re-establishing a broken session without requiring the application to start over. In practice, most of its responsibilities got absorbed into Layers 4 and 7.

You can mostly ignore Layer 5 in the modern internet. Some people even describe the stack as "5 or 6 layers" because Session is so weakly defined.

### Layer 6: Presentation

**Job:** Translate data formats — encoding, compression, and **encryption**.

This is where **TLS** (the thing that puts the S in HTTPS) lives in theory. Character encoding (ASCII, UTF-8), data compression (gzip), and serialization formats (JSON, XML) also fit here.

**From Week 10:** When TLS encrypts your HTTP request before sending it down to TCP, that's Layer 6 doing its job. The fact that HTTP is identical whether or not TLS is in the path is exactly what Layer 6 promises — upper layers don't have to change when lower-layer features change.

> **Note:** In real life, TLS is often described as sitting "between Layer 4 and Layer 7," because it doesn't fit perfectly anywhere. The OSI model is a reference, not a rigid classification system.

### Layer 7: Application

**Job:** Speak the protocol that the user actually cares about.

This is where **HTTP, HTTPS, SSH, DNS, SMTP, FTP** all live. When you type a URL into your browser, it's a Layer 7 application sending Layer 7 messages.

**From Week 8:** The entire HTTP protocol — methods, headers, status codes — is Layer 7.

**From Week 4:** SSH is also Layer 7. It speaks a protocol designed for secure remote shell access.

**From Week 9:** DNS is Layer 7. It's an application-level protocol that produces Layer 3 information.

### The TCP/IP Model — What the Internet Actually Uses

Real networking protocols weren't designed from the OSI model — they evolved separately, in a simpler form called the **TCP/IP model** (also called the "DoD model" because the US Department of Defense funded early internet research).

```
┌─────────────────────────────────────────────────────────┐
│ OSI Layers          │         TCP/IP Model              │
├─────────────────────┼───────────────────────────────────┤
│ 7. Application      │                                   │
│ 6. Presentation     │ Application (HTTP, HTTPS, SSH)    │
│ 5. Session          │                                   │
├─────────────────────┼───────────────────────────────────┤
│ 4. Transport        │ Transport (TCP, UDP)              │
├─────────────────────┼───────────────────────────────────┤
│ 3. Network          │ Internet (IP, ICMP)               │
├─────────────────────┼───────────────────────────────────┤
│ 2. Data Link        │ Link (Ethernet, Wi-Fi)            │
│ 1. Physical         │                                   │
└─────────────────────┴───────────────────────────────────┘
```

So why learn the OSI model if the internet uses TCP/IP? Because OSI is the vocabulary. When a network engineer says "this is a Layer 7 problem" or "the issue is at Layer 2," they're speaking OSI. Even vendors of TCP/IP equipment use OSI numbering in their documentation.

### Encapsulation — How Layers Work Together

Each layer wraps the data from the layer above with its own header:

```
Application sends:    GET / HTTP/1.1\r\nHost: example.com
                         │
Presentation:        [TLS: encrypt + add TLS record header]
                         │
Transport (TCP):     [TCP header: port 443, seq 1000, flags] + [TLS data]
                         │
Network (IP):        [IP header: src 192.168.1.42, dst 142.250.74.46] + [TCP segment]
                         │
Data Link:           [Ethernet: dst aa:bb:cc..., src 11:22:33...] + [IP packet]
                         │
Physical:            0101011100... (actual electrical signals on the wire)
```

When the receiver gets the frame, each layer peels off its header in reverse order and hands the payload up. This is called **encapsulation** on the way down and **decapsulation** on the way up.

You've actually seen this in Wireshark. Every packet panel shows layers stacked top-to-bottom — each expandable section is one layer.

### Troubleshooting with OSI

The OSI model is surprisingly useful when debugging. The standard approach is to **work up from Layer 1**:

1. **Layer 1:** Is the cable plugged in? Is Wi-Fi connected?
2. **Layer 2:** Does `arp` show my gateway? Am I getting frames?
3. **Layer 3:** Can I `ping` anything? Is my IP configured?
4. **Layer 4:** Can I reach the port? `nc -zv host port`
5. **Layer 5–7:** Does my HTTP request work? `curl -v URL`

Most problems resolve themselves once you find the lowest broken layer. "Is it plugged in?" is a serious Layer 1 question.

---

## Part 2: IPv6 — Why and How

### The IPv4 Exhaustion Problem

IPv4 has 32-bit addresses, which gives about **4.3 billion** possible addresses. That sounded infinite in 1983. Then smartphones happened. Then IoT happened. Then every lightbulb started needing an IP address.

The regional internet registries ran out of new IPv4 blocks to hand out around 2011-2019 (different regions exhausted at different times). The internet didn't collapse, because of some clever workarounds:

- **NAT** (Network Address Translation): Your home router uses *one* public IP, and dozens of devices behind it share it.
- **CGNAT** (Carrier-Grade NAT): Your ISP puts *thousands* of customers behind one public IP.

These hacks work, but they come at a cost: NAT breaks peer-to-peer connections, complicates security, and the workarounds keep piling up. The real fix is more addresses. That's IPv6.

### IPv6 Addressing

An IPv6 address is **128 bits** — four times as long as IPv4. That gives about **340 undecillion** (3.4 × 10³⁸) possible addresses. Enough to assign billions of addresses to every grain of sand on Earth, with astronomical headroom left over.

An address looks like this:

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

That's 8 groups of 4 hexadecimal digits, separated by colons. Each group is 16 bits (128 bits ÷ 8 groups = 16).

### Shorthand Rules

Full IPv6 addresses are unwieldy. Two compression rules make them readable:

**Rule 1: Drop leading zeros in each group**

```
2001:0db8:0000:0000:0000:0000:0000:0001
    ↓
2001:db8:0:0:0:0:0:1
```

**Rule 2: Replace one run of consecutive zero groups with `::`**

```
2001:db8:0:0:0:0:0:1
    ↓
2001:db8::1
```

You can **only use `::` once** per address. If you used it twice, nobody could tell how many zero groups each `::` represented.

**Examples:**

| Full | Compressed |
|------|-----------|
| `2001:0db8:0000:0000:0000:0000:0000:0001` | `2001:db8::1` |
| `0000:0000:0000:0000:0000:0000:0000:0001` | `::1` (loopback) |
| `0000:0000:0000:0000:0000:0000:0000:0000` | `::` (unspecified) |
| `fe80:0000:0000:0000:0202:b3ff:fe1e:8329` | `fe80::202:b3ff:fe1e:8329` |

### Address Types

IPv6 has distinct address *types*, each with a reserved prefix:

| Prefix | Type | Purpose | IPv4 Analogue |
|--------|------|---------|---------------|
| `::1/128` | Loopback | Your own machine | `127.0.0.1` |
| `fe80::/10` | Link-local | Local segment only — never routed | `169.254.x.x` |
| `fc00::/7` | Unique local | Private networks | `10.x.x.x`, `192.168.x.x` |
| `2000::/3` | Global unicast | Public, routable internet | Public IPv4 addresses |
| `ff00::/8` | Multicast | One-to-many | IPv4 multicast |
| `::/128` | Unspecified | "No address" | `0.0.0.0` |

Notice what's missing: **broadcast**. IPv4 had a broadcast address (`255.255.255.255`) that sent a packet to every device on the network. IPv6 replaced broadcast with multicast — more efficient because only subscribed devices receive it.

**Link-local addresses** (`fe80::...`) are especially interesting. Every IPv6 interface *automatically* gets one, with no configuration needed. You can ping between neighbors immediately, even before a router hands out global addresses.

### IPv4 vs IPv6 Comparison

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Address size | 32 bits | 128 bits |
| Header size | 20-60 bytes (variable) | 40 bytes (fixed) |
| Auto-configuration | Needs DHCP | SLAAC built in (router advertisements) |
| NAT | Required (scarcity) | Avoided (abundance) |
| Broadcast | Yes | No — uses multicast |
| Fragmentation | Routers can fragment | Only sender fragments |
| Checksum in IP header | Yes | No (removed — TCP/UDP handle it) |
| Security | Optional (IPsec) | IPsec part of the spec (but rarely used in practice) |

IPv6 isn't just "more addresses." The designers used the opportunity to simplify the header, remove obsolete features, and make auto-configuration the default.

### Dual-Stack and Transition

In reality, IPv4 is not going away soon. Operating systems, cloud providers, and home routers run **dual-stack** — both IPv4 and IPv6 simultaneously. When you type `www.google.com`, your browser might get both an `A` record (IPv4) and an `AAAA` record (IPv6). Modern browsers prefer IPv6 when available (a strategy called *Happy Eyeballs*).

As of recent measurements, about 40-50% of Google traffic uses IPv6. Countries like India, the US, and Germany have high IPv6 adoption; others lag behind. Many cloud providers (AWS, Azure, Google Cloud) now offer IPv6 by default for new infrastructure.

### IPv6 in URLs

One annoying quirk: IPv6 addresses contain colons, which conflict with the `host:port` syntax in URLs. The solution is square brackets:

```
http://[2001:db8::1]/
http://[2001:db8::1]:8080/
```

If you just type `http://2001:db8::1:8080/`, the parser will be confused — is `8080` a port, or part of the address?

---

## Part 3: VPN — Encryption Meets Networking

### What a VPN Is

A **VPN** (Virtual Private Network) makes a public network like the internet behave like a private one. It does this through an **encrypted tunnel** between your device and a remote VPN server.

The magic is simple. Before a packet leaves your device, the VPN software:

1. Encrypts the whole packet (contents and original headers)
2. Wraps it in a new packet addressed to the VPN server
3. Sends that encrypted wrapper through the public internet

The VPN server:

4. Receives the wrapper, decrypts it, and finds the original packet
5. Forwards that packet to its real destination
6. When a reply comes back, encrypts it and sends it back through the tunnel

```
Without VPN:
  You ─── plain IP packet to Destination ────────► Internet ───► Destination

With VPN:
  You ─── [encrypted: original packet] to VPN server ───► Internet ───► VPN server
                                                                           │
                                                                           ▼
                                                                      Destination
```

Anyone watching the public network sees only encrypted traffic between you and the VPN server. They can't see what sites you visit, what data you send, or what responses you get.

### Why People Use VPNs

1. **Privacy from your ISP.** Your ISP can see every site you visit (even over HTTPS, they can see *which* sites — just not the content). A VPN hides that.
2. **Public Wi-Fi safety.** On coffee shop Wi-Fi, others on the network could potentially snoop. A VPN encrypts everything before it leaves your device.
3. **Remote access to private networks.** Companies use VPNs so employees can access internal servers from home.
4. **Bypassing geographic restrictions.** A VPN in another country makes you appear to be there. (This is why Netflix tries to block VPNs.)
5. **Hiding your IP from destination sites.** Sites see the VPN server's IP, not yours.

### What a VPN Is Not

A VPN is **not** total anonymity. Common misconceptions:

- **The VPN provider can still see everything you do.** You're trading trust in your ISP for trust in the VPN operator.
- **Cookies and logins still identify you.** Logging into Gmail with a VPN doesn't make you anonymous — Gmail still knows who you are.
- **Browser fingerprinting still works.** Sites can identify your device through other signals.
- **HTTPS already protects the content** of your traffic. A VPN mostly adds privacy about *which sites* you visit and your IP address.
- **Your employer's VPN often logs everything.** Corporate VPNs typically monitor traffic by design.

A useful mental model: a VPN is like moving your ISP connection to a different location. Everything that would have been visible to your old ISP is now visible to the VPN operator (or not visible to either, depending on the protocol).

### How a VPN Uses Everything You've Learned

A VPN is a beautiful example of stacking networking and encryption concepts:

```
Layer 7 (Application):   Your HTTP request
                              │
Layer 6 (Presentation):  Still encrypted by HTTPS (Week 10)
                              │
Layer 4 (Transport):     Still wrapped in TCP to the destination
                              │
Layer 3 (Network):       Still addressed from you to destination
                              │
      ── WHOLE PACKET encrypted by VPN (Week 10) ──
                              │
New Layer 3 (Network):   NEW IP header: from you to VPN server
                              │
Layer 4 (Transport):     VPN uses UDP (WireGuard) or TCP/UDP (OpenVPN)
                              │
Layer 2/1:               Normal Ethernet/Wi-Fi
```

The clever part: the VPN wraps *entire Layer 3 packets* (including their headers) inside new Layer 3 packets. From the public internet's perspective, the encrypted outer packet looks like ordinary UDP traffic to some server.

### VPN Protocols

There are several VPN protocols, each with trade-offs:

**WireGuard** — The modern favorite.
- Small codebase (~4,000 lines of kernel code)
- Fast — consistently outperforms OpenVPN
- Fixed set of modern cryptographic primitives (Curve25519, ChaCha20, Poly1305, BLAKE2s)
- Simple configuration — often under 10 lines
- Built into the Linux kernel since 5.6

**OpenVPN** — The battle-tested classic.
- Uses TLS for the tunnel (same crypto as HTTPS)
- Flexible — runs over TCP or UDP, supports certificates or passwords
- More moving parts, more configuration required
- Still widely used, especially by commercial VPN providers

**IPsec / IKEv2** — The standards-body favorite.
- Built into most operating systems and phones
- Complex to configure but transparent to users
- Common for corporate VPNs and mobile roaming

**SSH tunnel** — Not a real VPN, but useful.
- Encrypts TCP connections over an SSH session
- Can forward specific ports or act as a SOCKS proxy
- Doesn't tunnel arbitrary IP traffic (only TCP)

In class, we'll use WireGuard because it's the easiest to understand and set up, and because it's what you'd actually deploy today.

### A WireGuard Configuration

A minimal WireGuard config file is about 10 lines:

```ini
# On the client
[Interface]
PrivateKey = oK56DE9Ue9zK76rAc8pBl6opph+1v36lm0cWV3I9Pnc=
Address = 10.0.0.2/24

[Peer]
PublicKey = GtL7fZc2Xm1uS3Hdn7qqRlVhYyIkMvO9PpU4W6aBc2A=
Endpoint = vpn.example.com:51820
AllowedIPs = 0.0.0.0/0
```

That's it. `0.0.0.0/0` for `AllowedIPs` means "route everything through the VPN."

Compare that to OpenVPN, where configuring a simple client typically involves a certificate file, a CA certificate, a private key, and 20+ config directives.

---

## Putting It All Together

After this week, you can tell the full story of visiting `https://example.com`:

```
Layer 7 (App):     Browser sends HTTP request
Layer 6 (Pres):    TLS encrypts it (from Week 10)
Layer 4 (Trans):   TCP wraps it, port 443
Layer 3 (Net):     IP header — might be IPv4 OR IPv6 (from this week)
Layer 2 (Link):    Ethernet frame to your router
Layer 1 (Phys):    Signals on the wire / over Wi-Fi

[optionally:] Before leaving the machine, a VPN encrypts all of this
              and wraps it in a new Layer 3 packet to the VPN server.
```

And if you run `ssh user@my-server` on a Wednesday at 3pm, you're using:
- Layer 1 (Physical — Wi-Fi or Ethernet)
- Layer 2 (Ethernet to your router)
- Layer 3 (IPv4 or IPv6 to the server)
- Layer 4 (TCP, port 22)
- Layer 6 (SSH does its own encryption, similar spirit to TLS)
- Layer 7 (SSH protocol: authentication, channels, shell)
- And that SSH connection contains a digital signature (Week 10) that proved your identity

Every layer. Every week.

---

## Self-Check Questions

Test your understanding before moving on.

<details>
<summary>1. In the OSI model, which layer is responsible for port numbers?</summary>

**Layer 4 (Transport).** TCP and UDP both use port numbers to identify which application on a machine should receive the data. Layer 3 (IP) only identifies the machine itself.
</details>

<details>
<summary>2. Compress the IPv6 address `2001:0db8:0000:0000:0000:0000:0abc:0001` using both shorthand rules.</summary>

Step 1 (drop leading zeros): `2001:db8:0:0:0:0:abc:1`
Step 2 (collapse zero run with `::`): `2001:db8::abc:1`

The final compressed form is `2001:db8::abc:1`.
</details>

<details>
<summary>3. Why can't you use `::` more than once in an IPv6 address?</summary>

Because each `::` represents "one or more runs of zero groups," and the total address must be 128 bits. If you had two `::` in the same address, there'd be no way to tell how many zero groups belong to each. The single-`::` rule keeps the address unambiguous.
</details>

<details>
<summary>4. What does a VPN NOT protect against?</summary>

A VPN doesn't protect against:
- The VPN provider itself (you're trusting them now instead of your ISP)
- Cookies and logins (if you log into Gmail, Gmail still knows who you are)
- Browser fingerprinting
- Malware on your device
- Tracking by the destination website across sessions

A VPN mainly hides your IP from destinations and hides traffic contents + destinations from anyone between you and the VPN server.
</details>

<details>
<summary>5. Which OSI layer does HTTPS encryption primarily sit at?</summary>

**Layer 6 (Presentation).** TLS handles encoding and encryption, which is Layer 6's job. In practice, TLS is sometimes described as sitting "between Layer 4 and Layer 7" because it doesn't fit perfectly into OSI's strict model — another reminder that OSI is a reference, not an exact classification.
</details>

<details>
<summary>6. You SSH to a server using its domain name: `ssh me@my-server.com`. Walk through the layers involved.</summary>

- **Layer 7 (Application):** Your SSH client starts the SSH protocol.
- **Layer 7 (Application):** First it asks DNS to resolve `my-server.com` to an IP address.
- **Layer 4 (Transport):** A TCP connection is established to the server on port 22 (three-way handshake).
- **Layer 3 (Network):** Each packet has IP headers with your source and the server's destination.
- **Layer 2 (Data Link):** Frames with MAC addresses hop across your local network to the router.
- **Layer 1 (Physical):** Signals travel over the wire or through the air.
- **SSH** then performs its own key exchange and encryption (conceptually Layer 6).
- **Digital signature (Week 10)** proves your identity using your private key.
</details>

---

## Further Reading (Optional)

- [Cloudflare: What is the OSI Model?](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/) — short, clear explanation
- [APNIC Blog: Why IPv6?](https://blog.apnic.net/2020/11/12/ipv6-only-and-happy-eyeballs/) — the transition story from a regional registry
- [WireGuard: Formal Paper](https://www.wireguard.com/papers/wireguard.pdf) — the official WireGuard paper (readable!)
- [How to Configure WireGuard](https://www.wireguard.com/quickstart/) — the official quickstart

---

**Next step:** Continue to [exercises.md](exercises.md) to apply these ideas hands-on before class.
