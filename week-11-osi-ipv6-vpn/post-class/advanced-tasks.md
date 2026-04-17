# Post-Class Advanced Tasks: OSI Model, IPv6 & VPN

**Estimated time: 1-2 hours total**

These tasks go deeper than class time allows. They're independent — do them in any order. If you get stuck, check [hints.md](hints.md) for guidance.

---

## Task 1: Deploy WireGuard to Your Cloud Server (~45 minutes)

**Objective:** Put a real VPN on your Week 4 cloud server and connect to it from your laptop. Once this is set up, you can encrypt all your home traffic through your server from anywhere in the world.

### Part A: Install WireGuard on the server

SSH into your Week 4 cloud server and install WireGuard:

```bash
sudo apt update
sudo apt install -y wireguard wireguard-tools
```

Enable IP forwarding so the server can route traffic for clients:

```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Part B: Generate server keys

```bash
umask 077
cd /etc/wireguard
sudo sh -c "wg genkey | tee server.priv | wg pubkey > server.pub"
sudo cat server.pub
```

Keep the private key private! The `umask 077` and `/etc/wireguard` location ensure the files are root-only.

### Part C: Write the server config

Create `/etc/wireguard/wg0.conf` with `sudo nano /etc/wireguard/wg0.conf`:

```ini
[Interface]
PrivateKey = <paste your server.priv here>
Address = 10.100.0.1/24
ListenPort = 51820

# These allow VPN clients to reach the internet through this server.
# Replace eth0 with your actual network interface if it differs (check `ip addr`).
PostUp   = iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# This will be filled in once your client is set up
PublicKey = <client public key>
AllowedIPs = 10.100.0.2/32
```

### Part D: Open the firewall

Your `ufw` rules from Week 4 block UDP 51820 by default. Open it:

```bash
sudo ufw allow 51820/udp
```

### Part E: Generate client keys on your laptop

On your laptop:

```bash
wg genkey | tee client.priv | wg pubkey > client.pub
cat client.pub
```

Paste this public key into the server's `wg0.conf` in the `[Peer]` section you left blank. Then on the server:

```bash
sudo wg-quick up wg0
```

Check it's running:

```bash
sudo wg show
```

### Part F: Write the client config

On your laptop, create `client.conf`:

```ini
[Interface]
PrivateKey = <your client.priv>
Address = 10.100.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <your server.pub>
Endpoint = <your-server-public-ip>:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
```

`AllowedIPs = 0.0.0.0/0, ::/0` means "send ALL traffic through the VPN." Change it to specific ranges if you only want partial tunneling.

### Part G: Connect

Import `client.conf` into the [WireGuard client app](https://www.wireguard.com/install/) (available for macOS, Windows, Linux, iOS, Android). Toggle the tunnel on.

Verify:

```bash
curl ifconfig.me
```

You should see your **server's** public IP, not your home IP. You're now tunneling through your server.

### Verify Success

- [ ] WireGuard is installed on your server with a config in `/etc/wireguard/wg0.conf`
- [ ] UDP 51820 is open in the firewall
- [ ] Your client connects and `wg show` reports a recent handshake on both sides
- [ ] `curl ifconfig.me` from your laptop returns your server's IP while the VPN is on

### Security tips

- **Never commit `server.priv` or `client.priv` to git.**
- Use `chmod 600 *.priv` on any private key files you keep locally.
- If your server also runs other services, be aware that VPN clients on `10.100.0.0/24` can reach those services from inside the LAN. Adjust firewall rules if you need stricter isolation.

---

## Task 2: IPv6 Adoption Audit (~25 minutes)

**Objective:** Measure how well different websites and services support IPv6, and reflect on what the transition really looks like in practice.

### Part A: Survey common services

For each domain below, record whether it has an `AAAA` record and what the record contains. Use `dig`:

```bash
dig AAAA google.com +short
dig AAAA facebook.com +short
dig AAAA github.com +short
dig AAAA microsoft.com +short
dig AAAA amazon.com +short
dig AAAA netflix.com +short
dig AAAA wikipedia.org +short
dig AAAA <your-university-or-workplace-domain> +short
dig AAAA <your-favorite-local-news-site> +short
```

Record your findings in a small table.

### Part B: Compare with IPv4

For the same list, run `dig A <domain> +short`. Every domain should have an IPv4 address. Which ones have both? Which ones only IPv4?

### Part C: Measure your own setup

```bash
curl -4 https://ifconfig.co
curl -6 https://ifconfig.co
```

The first shows your IPv4 public address. The second either shows your IPv6 public address or fails with a network error.

### Part D: Write a short reflection

Answer these in 3-5 sentences total:

1. Which of the services in Part A had IPv6, and which didn't?
2. Does your own internet connection have working IPv6?
3. Based on your audit, do you think IPv6 adoption is driven more by **consumer demand** or by **infrastructure/cloud providers**? Why?

### Verify Success

- [ ] You audited at least 8 domains for IPv4/IPv6 coverage
- [ ] You tested your own connectivity for both
- [ ] You wrote a brief reflection on what you observed

---

## Task 3: Map an OSI Stack for a Real Interaction (~20 minutes)

**Objective:** Trace a real network event through every OSI layer, explaining what happens at each. This solidifies the layer model.

Choose one of these scenarios (or invent your own):

- A: You SSH into your cloud server from a coffee shop
- B: You make a `docker pull` from Docker Hub
- C: You open a website via a VPN (from Task 1)

### Part A: Diagram the stack

Draw a table like this, for your chosen scenario:

| Layer | What's happening here | Specific protocol / component | Concrete evidence you could capture |
|-------|----------------------|-------------------------------|-------------------------------------|
| 7 | | | |
| 6 | | | |
| 5 | | | (you can skip 5 for this exercise) |
| 4 | | | |
| 3 | | | |
| 2 | | | |
| 1 | | | |

"Concrete evidence" means: if you wanted to prove this layer was involved, what would you look at (Wireshark filter, command output, log line)?

### Part B: Find surprises

Answer: was there anything at any layer that surprised you? Something that looked like it should be at one layer but actually lives at another? (TLS is a famous example — it doesn't fit neatly.)

### Verify Success

- [ ] You filled in every layer (skip Layer 5 if it doesn't apply)
- [ ] You named specific protocols/components for each
- [ ] You identified at least one "grey area" where the layer model is ambiguous

---

## Task 4: Research — VPN vs Proxy vs Tor (~25 minutes)

**Objective:** Understand how VPNs fit into a broader family of privacy tools, and the trade-offs of each.

### Part A: Read up

Research these three technologies briefly (Wikipedia or any trusted source):

1. **SOCKS / HTTP Proxy** — what you used in pre-class Exercise 4
2. **VPN** — what you set up this week
3. **Tor** — the anonymity network

### Part B: Fill in a comparison table

| Property | SOCKS proxy | VPN | Tor |
|----------|-------------|-----|-----|
| Encrypts traffic to provider? | | | |
| Hides your IP from destination? | | | |
| Number of hops | 1 | 1 | 3+ |
| Who has your real IP? | | | |
| Who can see the destination? | | | |
| Typical latency impact | | | |
| Primary use case | | | |

### Part C: Pick the right tool

For each of these use cases, which tool would you pick, and why? (One or two sentences each.)

1. You want to check your email from an untrusted hotel Wi-Fi.
2. You're a journalist in a country with heavy surveillance, communicating with a source.
3. You're a developer who wants to test how your site looks to users in another country.
4. You work from home and need to access internal servers at your company.
5. You just want to keep your ISP from profiling your browsing habits.

### Verify Success

- [ ] You completed the comparison table
- [ ] You made a reasoned choice for each use case
- [ ] You understand that these tools solve overlapping but different problems

---

## Task 5: Configure Split Tunneling (~30 minutes, requires Task 1 done)

**Objective:** Change your VPN so only *specific* traffic goes through the tunnel. This is useful for real-world scenarios — e.g., "only internal company traffic through VPN, everything else direct."

### Part A: Decide what to tunnel

Pick one of these scenarios for your split-tunnel:

- **Option 1:** Only traffic to a specific server (e.g., your Week 4 server's private services)
- **Option 2:** Only traffic to a specific subnet (e.g., `10.0.0.0/8`)
- **Option 3:** Everything *except* traffic to a specific network (e.g., don't tunnel your home LAN)

### Part B: Update the client config

On your laptop, edit `client.conf`. The key change is the `AllowedIPs` line under `[Peer]`.

**For Option 1** (only tunnel traffic to `203.0.113.10`):

```ini
AllowedIPs = 10.100.0.0/24, 203.0.113.10/32
```

**For Option 2** (only tunnel `10.0.0.0/8`):

```ini
AllowedIPs = 10.0.0.0/8
```

**For Option 3** (everything except your home LAN `192.168.0.0/16`) — this is harder because WireGuard uses route-based matching. You need to list all non-excluded ranges:

```ini
# Tunnel everything EXCEPT 192.168.0.0/16
AllowedIPs = 0.0.0.0/1, 128.0.0.0/2, 192.0.0.0/9, 192.128.0.0/11, 192.160.0.0/13, 192.169.0.0/16, 192.170.0.0/15, 192.172.0.0/14, 192.176.0.0/12, 192.192.0.0/10, 193.0.0.0/8, 194.0.0.0/7, 196.0.0.0/6, 200.0.0.0/5, 208.0.0.0/4, 224.0.0.0/3
```

(There are online calculators that generate this list for you.)

### Part C: Test your split

Apply the new config, restart the tunnel, then test:

```bash
# Should go through VPN (traffic you meant to tunnel)
curl ifconfig.me

# Should NOT go through VPN (direct)
curl https://<something-in-excluded-range>
```

Use `traceroute` to see the path in each case.

### Verify Success

- [ ] You edited `AllowedIPs` to split tunnel specific traffic
- [ ] Traffic you meant to tunnel goes through the VPN
- [ ] Traffic you excluded goes direct
- [ ] You understand when split tunneling makes sense (corporate access, privacy for specific sites, bandwidth optimization)

---

## Optional: Write a Short Reflection

Before moving on from this week, jot down 5-7 sentences answering:

1. Which OSI layer do you now feel most confident about? Which least?
2. How has IPv6 addressing changed since you first saw it in Week 5?
3. Would you actually use a VPN in daily life? Why or why not?
4. Looking at the arc from Week 5 through Week 11 — what's the single biggest shift in how you think about networks?

Save this — you'll thank yourself during exam prep in Week 13.
