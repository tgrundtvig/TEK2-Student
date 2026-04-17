# Hints for Advanced Tasks

Try to work through each task on your own first! These hints are here if you get stuck.

---

## Task 1: Deploy WireGuard to Your Cloud Server

<details>
<summary>Hint 1: No handshake showing in `wg show`</summary>

The most common causes, in order of how often they happen:

1. **Keys in the wrong places.** Double-check: the server's `[Peer] PublicKey` is the *client's* public key (not the client's private key, not the server's public key). Triple-check this.
2. **Firewall blocking UDP 51820.** Run `sudo ufw status` — you should see `51820/udp ALLOW`. If not: `sudo ufw allow 51820/udp`.
3. **Endpoint wrong on the client.** It must be your server's **public** IP or DNS name, not a private/internal IP.
4. **The `PersistentKeepalive = 25` line is missing** on the client. Most NAT routers drop UDP idle connections; keepalives prevent this.

Quick diagnosis: from your laptop, try `nc -u -v your-server-public-ip 51820`. If it says "connection refused" or "timed out," the problem is network-level (firewall or wrong IP). If it connects but WireGuard still doesn't work, the problem is in the configs.
</details>

<details>
<summary>Hint 2: I'm connected but the internet doesn't work</summary>

If `wg show` shows a handshake but websites don't load, you're missing one of:

1. **IP forwarding.** On the server: `sudo sysctl net.ipv4.ip_forward` should print `1`. If not, add the line to `/etc/sysctl.conf` and run `sudo sysctl -p`.
2. **NAT/masquerade.** The `PostUp`/`PostDown` lines in the server config apply iptables NAT rules. If `eth0` isn't your actual WAN interface, these rules target the wrong interface. Check with `ip -4 route show default` — the `dev` name at the end is your real WAN interface.
3. **DNS.** If `curl example.com` fails but `curl 93.184.216.34` works, your DNS is broken. Add `DNS = 1.1.1.1` to the `[Interface]` section on the client.
</details>

<details>
<summary>Hint 3: I'm worried about locking myself out</summary>

WireGuard doesn't affect your SSH login — SSH uses TCP port 22, WireGuard uses UDP port 51820. They're independent.

That said, before enabling NAT or firewall rules, keep a second SSH session open as a safety net. If the primary session dies, you can use the backup to roll back.

You can also enable WireGuard to start on boot:

```bash
sudo systemctl enable wg-quick@wg0
```

And stop it cleanly:

```bash
sudo wg-quick down wg0
```

Nothing in this task permanently changes your SSH access — if the VPN breaks, SSH still works.
</details>

---

## Task 2: IPv6 Adoption Audit

<details>
<summary>Hint 1: Reading dig output for AAAA records</summary>

A typical successful AAAA query looks like:

```
$ dig AAAA google.com +short
2a00:1450:400f:80d::200e
```

No output means no AAAA record exists — that site is IPv4-only. If you want confirmation, run without `+short`:

```
$ dig AAAA somesite.com

;; ANSWER SECTION:
(empty)
```

An empty ANSWER SECTION confirms it.
</details>

<details>
<summary>Hint 2: My connection doesn't have IPv6</summary>

That's normal. Many ISPs still haven't rolled out IPv6 to consumers (especially DSL and some cable providers). Indicators:

- `curl -6 https://ifconfig.co` fails immediately
- `ip -6 addr` shows only link-local `fe80::...` addresses on your active interface
- No global `2001:...` or `2a00:...` address on your Wi-Fi

You can still complete the task — just note which ISP you're using and that it doesn't provide IPv6. This is itself useful data for Part D.

As a workaround, use your phone's mobile data (most carriers have IPv6) as a hotspot for one test.
</details>

---

## Task 3: Map an OSI Stack for a Real Interaction

<details>
<summary>Hint 1: Where does TLS fit?</summary>

TLS is the classic example of "doesn't map neatly." The OSI purist answer is Layer 6 (Presentation — encryption is presentation-layer). The networking-engineer answer is "somewhere between 4 and 7."

For your diagram, pick one and note in the surprises section that it's a grey area. Both answers are defensible.
</details>

<details>
<summary>Hint 2: What goes at Layer 5?</summary>

In modern networking, Layer 5 is mostly absorbed into 4 and 7. For most scenarios, you can leave Layer 5 blank or write "not meaningfully separate in modern stacks."

If you want to find something at Layer 5: SSH channel multiplexing (one SSH connection carries multiple logical channels) is the closest example.
</details>

<details>
<summary>Hint 3: "Evidence I could capture" — what should I write?</summary>

Examples:

- Layer 2: `ip neigh show` (ARP/NDP table) or Wireshark Ethernet frame headers
- Layer 3: `traceroute`, `ip route`, Wireshark IP headers
- Layer 4: `ss -tlnp`, Wireshark TCP/UDP headers
- Layer 7: `curl -v` request/response output, browser dev tools Network tab

The point of this column is to make each layer feel *observable*, not just abstract.
</details>

---

## Task 4: VPN vs Proxy vs Tor

<details>
<summary>Hint 1: The "who sees what" question</summary>

A useful way to think about these tools: *who gets to see which piece of information?*

For each tool, ask:
- Your ISP — sees what?
- The proxy/VPN/Tor nodes — see what?
- The destination site — sees what?

Then fill in the table. You'll notice that Tor's design makes sure no single party sees both your identity AND your destination — that's the whole point.
</details>

<details>
<summary>Hint 2: Latency for Tor</summary>

Tor typically routes through 3 relays, often in different countries. Expect 5-10x latency increase compared to direct connection. For casual browsing it's usable; for video calls or gaming it's impractical.
</details>

---

## Task 5: Configure Split Tunneling

<details>
<summary>Hint 1: How `AllowedIPs` actually works</summary>

`AllowedIPs` on the client side is both:

1. A **filter** — outgoing traffic that matches these ranges goes through the tunnel
2. A **route** — WireGuard installs a route for each CIDR pointing at the tunnel interface

If you set `AllowedIPs = 10.0.0.0/8`, then only traffic destined for `10.x.x.x` enters the tunnel. Everything else takes your normal route (default gateway).
</details>

<details>
<summary>Hint 2: Exclude-mode with wg-quick</summary>

There's a shortcut for exclude-mode (Option 3). `wg-quick` has a `Table = off` option and a `PostUp` hook that lets you write custom routes. For most people, just pasting the precomputed range list is simpler. Search for "WireGuard AllowedIPs calculator" online for tools that generate the range list automatically.
</details>

<details>
<summary>Hint 3: Verify which path traffic is taking</summary>

```bash
traceroute -n 8.8.8.8
```

The first hop tells you a lot:

- If the first hop is `10.100.0.1` (your VPN server's tunnel IP), the traffic is going through the VPN
- If the first hop is your home router (`192.168.x.x`), the traffic is going direct

You can also check your apparent public IP:

```bash
curl ifconfig.me
```

If it's your server's IP, traffic is tunneled. If it's your home IP, it's direct.
</details>

---

## General Troubleshooting

**`wg: command not found`** — Install WireGuard tools. On Ubuntu/Debian: `sudo apt install wireguard-tools`. On macOS: `brew install wireguard-tools` (or just install the WireGuard app from the App Store).

**`Cannot find device "wg0"`** — The interface doesn't exist yet. Make sure you ran `wg-quick up wg0`. On Windows or macOS, use the app instead of `wg-quick`.

**`Operation not permitted`** — Run the command with `sudo`. WireGuard requires root to create network interfaces.

**Docker commands fail with "cannot create container"** — Some hosting environments restrict `--cap-add NET_ADMIN`. On Docker Desktop this usually works, but CI environments might not allow it.

**IPv6 works locally but not over the internet** — Your ISP or router doesn't route IPv6. This is very common — about half of all home connections in Europe still don't have IPv6. The local tests (loopback, link-local, Docker) will still work.

**`dig` returns SERVFAIL** — Try another DNS server: `dig @1.1.1.1 example.com`. Sometimes your local resolver has issues.

---

## Still Stuck?

- [ ] Re-read the relevant section in [reading.md](../pre-class/reading.md)
- [ ] Check the [quick-reference.md](../quick-reference.md) for command syntax
- [ ] Search the error message online — the WireGuard community is active on Reddit and Stack Overflow
- [ ] Ask in class or on the course forum
