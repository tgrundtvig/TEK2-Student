# Week 11 Quick Reference Card

Print this page or keep it open while working!

---

## The OSI Model (7 Layers)

```
┌─────────────────────────────────────────────────────────────────────┐
│ 7. Application     │ HTTP, HTTPS, SSH, DNS, SMTP    │ What YOU see  │
│ 6. Presentation    │ TLS, compression, encoding     │ Format data   │
│ 5. Session         │ Session establishment          │ Dialog control│
│ 4. Transport       │ TCP, UDP                       │ Port to port  │
│ 3. Network         │ IP (IPv4, IPv6), ICMP, routers │ Host to host  │
│ 2. Data Link       │ Ethernet, MAC, switches, ARP   │ Local frame   │
│ 1. Physical        │ Cables, Wi-Fi, fiber, voltages │ Raw bits      │
└─────────────────────────────────────────────────────────────────────┘
```

**Mnemonic (top→bottom):** *All People Seem To Need Data Processing*
**Mnemonic (bottom→top):** *Please Do Not Throw Sausage Pizza Away*

### OSI vs TCP/IP Model

| OSI Layers | TCP/IP Model | Example Protocols |
|------------|--------------|-------------------|
| 7, 6, 5 | **Application** | HTTP, HTTPS, SSH, DNS |
| 4 | **Transport** | TCP, UDP |
| 3 | **Internet** | IP, ICMP |
| 1, 2 | **Link** | Ethernet, Wi-Fi |

OSI = theoretical reference model. TCP/IP = what the internet actually uses.

---

## Mapping Your Tools to OSI Layers

| Tool / Command | Primary Layer | Why |
|----------------|---------------|-----|
| `ping` | 3 (Network) | Uses ICMP on top of IP |
| `traceroute` | 3 (Network) | Maps router hops |
| `arp` | 2 (Data Link) | Resolves IP to MAC |
| `netcat` | 4 (Transport) | Opens TCP/UDP sockets |
| `ss -tlnp` | 4 (Transport) | Lists TCP/UDP ports |
| `curl` | 7 (Application) | Speaks HTTP |
| `dig` | 7 (Application) | Speaks DNS |
| `openssl s_client` | 6 (Presentation) | Does TLS |
| Wireshark | All | Captures every layer |

---

## IPv6 Address Basics

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
│    │    │    │    │    │    │    │
└──── 8 groups of 4 hex digits (128 bits total) ────┘
```

### Compression Rules

1. **Leading zeros** in a group can be dropped:
   `2001:0db8:0000:0000:0000:0000:0000:0001` → `2001:db8:0:0:0:0:0:1`

2. **One consecutive run of all-zero groups** can be replaced with `::`:
   `2001:db8:0:0:0:0:0:1` → `2001:db8::1`

3. You can only use `::` **once** per address (otherwise the length is ambiguous).

### Common Address Types

| Range | Type | Purpose |
|-------|------|---------|
| `::1/128` | Loopback | Like `127.0.0.1` — your own machine |
| `fe80::/10` | Link-local | Local network only, auto-configured |
| `fc00::/7` | Unique local | Private networks (like `10.0.0.0/8`) |
| `2000::/3` | Global unicast | Public internet addresses |
| `ff00::/8` | Multicast | One-to-many (replaces broadcast) |
| `::/128` | Unspecified | "No address assigned yet" |

### IPv4 vs IPv6

| | IPv4 | IPv6 |
|--|------|------|
| **Address size** | 32 bits | 128 bits |
| **Total addresses** | 4.3 billion | 340 undecillion |
| **Notation** | Dotted decimal (192.168.1.1) | Colon hex (2001:db8::1) |
| **Private addresses** | 10.x, 172.16-31.x, 192.168.x | fc00::/7 (unique local) |
| **Auto-config** | DHCP required | SLAAC built-in |
| **NAT needed?** | Almost always | Rarely (enough addresses) |
| **Broadcast?** | Yes | No — uses multicast |
| **Header** | Variable length | Fixed 40 bytes |

---

## IPv6 Commands

| Command | What It Does |
|---------|--------------|
| `ip -6 addr` | Show IPv6 addresses (Linux) |
| `ifconfig` | Show all addresses, IPv4 + IPv6 (macOS/BSD) |
| `ipconfig` | Show all addresses (Windows) |
| `ping6 ::1` or `ping -6 ::1` | Ping an IPv6 address |
| `ping ipv6.google.com` | Most pings auto-use IPv6 if available |
| `dig AAAA google.com` | Query an IPv6 DNS record |
| `curl -6 https://google.com` | Force IPv6 |
| `curl -4 https://google.com` | Force IPv4 |
| `ss -6 -tlnp` | IPv6 listening sockets (Linux) |
| `traceroute6 google.com` | IPv6 traceroute |

### IPv6 in URLs

IPv6 addresses contain colons, which conflict with the `host:port` separator. Wrap the address in square brackets:

```
http://[2001:db8::1]:8080/
```

---

## VPN Quick Overview

```
Without VPN:
  You ──[plain]──► ISP ──► Internet ──► Destination
                   (sees everything)

With VPN:
  You ──[encrypted tunnel]──► VPN Server ──► Internet ──► Destination
        (ISP sees only noise going to VPN server)
```

### What a VPN Protects Against

| Threat | Protected? |
|--------|-----------|
| Your ISP snooping traffic contents | Yes |
| Your ISP seeing which sites you visit | Yes (but they see VPN server) |
| Public Wi-Fi attackers | Yes |
| The destination site logging your IP | Yes (they see VPN's IP) |
| The destination site tracking cookies | No |
| The VPN provider itself logging you | No |
| Browser fingerprinting | No |

### VPN Protocol Comparison

| Protocol | Speed | Config | Notes |
|----------|-------|--------|-------|
| **WireGuard** | Very fast | Minimal (~10 lines) | Modern, uses Curve25519 + ChaCha20. Recommended. |
| **OpenVPN** | Moderate | Complex certificates | Battle-tested, very flexible, runs over TCP or UDP. |
| **IPsec / IKEv2** | Fast | Complex | Built into most operating systems and phones. |
| **SSH tunnel** | Slow | Just SSH | Not a real VPN, but the concept is similar. |

---

## SSH Tunneling (VPN-Lite)

### Local Port Forward (`-L`)

Forward a local port to a port on a remote server through SSH:

```bash
ssh -L 8080:localhost:80 user@server.com
```

Now `http://localhost:8080` on your machine routes through the SSH tunnel to `localhost:80` on the server.

### Dynamic SOCKS Proxy (`-D`)

Turn SSH into a general-purpose proxy:

```bash
ssh -D 1080 user@server.com
```

Then configure your browser to use `SOCKS5 proxy localhost:1080`. All traffic exits through the server.

### Remote Port Forward (`-R`)

Expose a local service to a remote server:

```bash
ssh -R 9000:localhost:8080 user@server.com
```

Now someone on the server can reach your local port 8080 at `server.com:9000`.

---

## WireGuard Essentials

### Key Generation

```bash
# Server or client — same commands
wg genkey | tee privatekey | wg pubkey > publickey
cat privatekey publickey
```

### Minimal Config File (`/etc/wireguard/wg0.conf`)

```ini
[Interface]
PrivateKey = <your-private-key>
Address = 10.0.0.1/24
ListenPort = 51820

[Peer]
PublicKey = <peer-public-key>
AllowedIPs = 10.0.0.2/32
```

### Bring It Up

| Command | What It Does |
|---------|--------------|
| `wg-quick up wg0` | Start the WireGuard interface |
| `wg-quick down wg0` | Stop it |
| `wg show` | Show current status, handshakes, transfer bytes |
| `wg` | Same as `wg show` |

---

## Wireshark Filters for This Week

| Filter | What It Shows |
|--------|---------------|
| `ipv6` | All IPv6 traffic |
| `icmpv6` | IPv6 ping, neighbor discovery |
| `ipv6.addr == 2001:db8::1` | Traffic to/from a specific IPv6 address |
| `udp.port == 51820` | WireGuard traffic (default port) |
| `tcp.port == 22` | SSH traffic (includes tunnel data) |
| `eth.addr == aa:bb:cc:dd:ee:ff` | Traffic for a specific MAC (Layer 2) |

---

## Encapsulation — Layers in Action

When your browser sends an HTTPS request:

```
Layer 7 (Application):   GET / HTTP/1.1
                             │
Layer 6 (Presentation):  [TLS encrypt]
                             │
Layer 4 (Transport):     [TCP header | port 443 | seq# | flags] + data
                             │
Layer 3 (Network):       [IP header | src IP | dst IP] + TCP segment
                             │
Layer 2 (Data Link):     [Ethernet header | src MAC | dst MAC] + IP packet
                             │
Layer 1 (Physical):      Bits on the wire / radio waves
```

Each layer **wraps** the one above it with its own header. The receiver unwraps in reverse.

---

## Remember

1. **OSI is a reference model** — real protocols don't map one-to-one, but it's great for teaching and troubleshooting
2. **Layer 2 = local (MAC addresses), Layer 3 = global (IP addresses)**
3. **IPv6 is not optional anymore** — many ISPs and cloud providers already use it
4. **`::` replaces one run of zero groups** in IPv6, and only one run
5. **A VPN is just an encrypted tunnel** — everything still goes over the public internet
6. **WireGuard > OpenVPN** for new deployments — simpler, faster, and secure by default
7. **SSH tunnels** are a practical mini-VPN you can use right now on your Week 4 server
