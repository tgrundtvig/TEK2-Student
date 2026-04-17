# Pre-Class Exercises: OSI Model, IPv6 & VPN

**Estimated time: 45-60 minutes**

These exercises connect the reading to things you can see and run on your own machine. If any exercise takes more than 10 minutes, note where you got stuck and move on — we'll cover it in class.

---

## Exercise 1: Map Tools to OSI Layers

**Goal:** Connect the commands and concepts you've already used to OSI layers. This anchors the model in real experience.

### 1.1 Fill in the table

Copy this table to a note or whiteboard. For each tool/concept, write the primary OSI layer (1-7). Some may span multiple — write the most specific layer.

| Tool / Concept | Layer | Why |
|----------------|-------|-----|
| Ethernet cable | | |
| MAC address | | |
| `ping google.com` | | |
| IPv4 address `192.168.1.1` | | |
| `ss -tlnp` | | |
| TCP three-way handshake | | |
| `curl https://example.com` | | |
| HTTP `GET /` | | |
| DNS lookup (`dig example.com`) | | |
| TLS handshake (`ClientHello`) | | |
| Docker `-p 8080:80` | | |
| Wi-Fi signal | | |
| ARP request | | |
| A Docker bridge network | | |

### 1.2 Check your answers

<details>
<summary>Expected answers</summary>

| Tool / Concept | Layer | Why |
|----------------|-------|-----|
| Ethernet cable | **1** | Physical signals |
| MAC address | **2** | Data link, local addressing |
| `ping google.com` | **3** | ICMP rides on IP |
| IPv4 address `192.168.1.1` | **3** | Network layer addressing |
| `ss -tlnp` | **4** | Lists TCP/UDP ports (transport) |
| TCP three-way handshake | **4** | TCP is transport |
| `curl https://example.com` | **7** | HTTP client (though it uses TLS at Layer 6) |
| HTTP `GET /` | **7** | Application protocol |
| DNS lookup (`dig example.com`) | **7** | DNS is an application-layer protocol |
| TLS handshake (`ClientHello`) | **6** | Encryption and encoding |
| Docker `-p 8080:80` | **4** | Port forwarding is transport layer |
| Wi-Fi signal | **1** | Physical radio waves |
| ARP request | **2** | Translates IP to MAC (data link) |
| A Docker bridge network | **2 + 3** | Bridge = Layer 2 switch, but Docker assigns IPs (3) |

**Discussion-worthy:** Docker networks are partially Layer 2 (virtual switches) and partially Layer 3 (Docker's embedded DNS and routing). TLS is often described as "between 4 and 7." These grey areas are why OSI is a *reference* model, not a classification system.

</details>

### Self-check

- [ ] You could make a first pass on every row
- [ ] You understand why `ping` is Layer 3 but `curl` is Layer 7
- [ ] You see the pattern: the lower the layer, the less the user notices it

---

## Exercise 2: Discover Your Own Addresses

**Goal:** Find all your IPv4 and IPv6 addresses, and identify which type each one is.

### 2.1 List your network interfaces

Run the appropriate command for your platform:

**Linux (or Docker container):**
```bash
ip addr
```

**macOS:**
```bash
ifconfig
```

**Windows (PowerShell):**
```powershell
ipconfig /all
```

Look at the output. You should see multiple interfaces — likely `lo` (loopback), your Wi-Fi or Ethernet, and possibly `docker0` or virtual interfaces.

### 2.2 Count your IPv6 addresses

A single interface often has **multiple** IPv6 addresses. Find at least one of each type if possible.

Examples to look for:
- `::1` — loopback (on the `lo` interface)
- `fe80:...` — link-local (every interface has one automatically)
- `2001:...` or `2a00:...` — global unicast (if your ISP supports IPv6)
- `fd00:...` or `fc00:...` — unique local (private networks)

Write down each address and its type.

### 2.3 Ping yourself on IPv6

```bash
ping6 ::1
# or on some systems:
ping -6 ::1
```

You should see replies. This works even without a network — `::1` is the IPv6 loopback, just like `127.0.0.1` in IPv4.

Press Ctrl+C to stop.

### 2.4 Ping an IPv6 site

```bash
ping6 ipv6.google.com
# or:
ping -6 ipv6.google.com
```

If you get replies, your network has IPv6 connectivity. If you get "network unreachable" or similar, your ISP or router doesn't provide IPv6 — that's fine for this exercise, just note it.

### 2.5 Compare dual-stack DNS

Look up both record types for `google.com`:

```bash
dig A google.com +short
dig AAAA google.com +short
```

You should see one or more IPv4 addresses from the first command and one or more IPv6 addresses from the second.

### 2.6 Force curl over each protocol

```bash
curl -4 -s -o /dev/null -w "IPv4: %{remote_ip}\n" https://www.google.com
curl -6 -s -o /dev/null -w "IPv6: %{remote_ip}\n" https://www.google.com
```

This tells curl which protocol to use and prints only the remote IP. If IPv6 fails, you'll get an error — note that too.

### Self-check

- [ ] You found at least 2 different IPv6 addresses on your machine (at minimum `::1` and one `fe80:...`)
- [ ] You identified each by type
- [ ] You pinged `::1` successfully
- [ ] You saw both A and AAAA records for `google.com`
- [ ] You know whether your network has IPv6 internet access

---

## Exercise 3: IPv6 Notation Drills

**Goal:** Practice compressing and expanding IPv6 addresses until the shorthand is second nature.

### 3.1 Compress these addresses

Rewrite each full address in its shortest valid form.

1. `2001:0db8:0000:0000:0000:0000:0000:0001`
2. `fe80:0000:0000:0000:0000:0000:0000:0001`
3. `2001:0db8:0000:0000:1234:0000:0000:0001`
4. `0000:0000:0000:0000:0000:0000:0000:0000`
5. `2001:0db8:abcd:0012:0000:0000:0000:0001`

### 3.2 Expand these compressed addresses

Rewrite each compressed address back to 8 groups of 4 hex digits.

1. `2001:db8::ff00:42:8329`
2. `::1`
3. `fe80::1%eth0` (ignore the `%eth0` — it's a zone identifier, not part of the address)
4. `2a00:1450::200e`

### 3.3 Spot the invalid ones

Which of these are invalid IPv6 addresses, and why?

1. `2001:db8::1::2`
2. `2001:db8:0:0:0:0:0:0:1`
3. `::1`
4. `2001:0db8:85a3::8a2e:0370:7334`
5. `::g::1`

### 3.4 Check your answers

<details>
<summary>Expected answers</summary>

**3.1 Compress:**

1. `2001:db8::1`
2. `fe80::1`
3. `2001:db8::1234:0:0:1` — note: collapse the longer zero run, leave the shorter one
4. `::`
5. `2001:db8:abcd:12::1`

**3.2 Expand:**

1. `2001:0db8:0000:0000:0000:ff00:0042:8329`
2. `0000:0000:0000:0000:0000:0000:0000:0001`
3. `fe80:0000:0000:0000:0000:0000:0000:0001`
4. `2a00:1450:0000:0000:0000:0000:0000:200e`

**3.3 Invalid:**

1. `2001:db8::1::2` — **Invalid.** Two `::` are not allowed.
2. `2001:db8:0:0:0:0:0:0:1` — **Invalid.** That's 9 groups, but an IPv6 address has only 8.
3. `::1` — Valid (loopback).
4. `2001:0db8:85a3::8a2e:0370:7334` — Valid.
5. `::g::1` — **Invalid.** `g` is not a hex character (only 0-9, a-f allowed), and there are also two `::`.

</details>

### Self-check

- [ ] You understand when to use `::` (longest zero run, only once)
- [ ] You can expand a compressed address back to 8 groups of 4 hex digits
- [ ] You can spot common invalid notations

---

## Exercise 4: A Simple VPN-Lite with SSH

**Goal:** Use SSH as a mini-VPN to see the concept of "encrypted tunnel" in action. You need any SSH server — your cloud VM from Week 4 is perfect.

If you no longer have your cloud VM, skip this exercise; you'll do a version of it in class with Docker.

### 4.1 Open a SOCKS proxy through SSH

Run this from your local terminal, leaving the connection open:

```bash
ssh -D 1080 -N user@your-server-ip
```

- `-D 1080` creates a SOCKS proxy on your local port 1080
- `-N` means "don't run a shell, just maintain the tunnel"
- Leave this terminal open; Ctrl+C closes the tunnel

### 4.2 Route curl through the proxy

In a **second** terminal:

```bash
curl --socks5 localhost:1080 https://ifconfig.me
echo
```

You should see your **server's** public IP address, not your local one. Your HTTP request traveled through SSH to the server, then out to `ifconfig.me` from there.

Compare to the same request without the proxy:

```bash
curl https://ifconfig.me
echo
```

This shows *your* public IP.

### 4.3 Use the proxy in your browser (optional)

1. Firefox → Settings → Network Settings → "Manual proxy configuration"
2. Set **SOCKS Host** to `localhost`, **Port** `1080`, check **SOCKS v5**
3. Also check "Proxy DNS when using SOCKS v5" so DNS queries also go through the tunnel
4. Visit [ifconfig.me](https://ifconfig.me) or [whatismyip.com](https://whatismyip.com) — you'll see your server's IP

Remember to turn the proxy off when you're done, or you'll wonder why websites load slowly.

### 4.4 What was encrypted?

The connection between your machine and the server was SSH — encrypted. The connection *from the server* to the destination site was whatever the destination uses (HTTPS, probably). Draw a small diagram of which parts are encrypted and which aren't.

### Self-check

- [ ] You opened an SSH SOCKS tunnel successfully
- [ ] A curl request through the tunnel shows your server's IP, not your home IP
- [ ] You understand what got encrypted and what didn't
- [ ] You closed the tunnel when you were done

---

## Exercise 5: Generate a WireGuard Keypair (No Install Required)

**Goal:** Create a WireGuard keypair in a Docker container so you've seen the tool once before class.

### 5.1 Start an Alpine container with WireGuard tools

```bash
docker run --rm -it alpine sh -c "apk add --no-cache wireguard-tools && sh"
```

This drops you into an Alpine shell with the `wg` command available.

### 5.2 Generate a keypair

Inside the container:

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

Look at them:

```bash
echo "--- Private ---"; cat privatekey
echo
echo "--- Public ---"; cat publickey
```

Both keys are 44 characters of base64. The private key is what you'd keep secret (like `chmod 600` territory from Week 10). The public key is what you'd share with peers.

### 5.3 Look at a sample config

Print a minimal config to the screen — you don't need to apply it, just read it:

```bash
cat <<EOF
[Interface]
PrivateKey = $(cat privatekey)
Address = 10.0.0.1/24
ListenPort = 51820

[Peer]
PublicKey = <insert-peer-public-key-here>
AllowedIPs = 10.0.0.2/32
EOF
```

That's a complete WireGuard server config. Notice how much simpler it is than anything involving certificates.

### 5.4 Exit and clean up

```bash
exit
```

The `--rm` flag we started with ensures Docker deletes the container automatically.

### Self-check

- [ ] You generated a WireGuard keypair
- [ ] You can explain why the private key must stay secret
- [ ] You've seen a complete WireGuard config and noticed how short it is

---

## Before Class Checklist

**Tools Ready:**

- [ ] Wireshark installed and working (from Week 5)
- [ ] Docker installed and running
- [ ] `ping6` or `ping -6` works on your machine
- [ ] `curl` works
- [ ] `dig` works (from Week 9)
- [ ] (Optional) Your Week 4 cloud server is still running, or you have SSH access somewhere

**Knowledge:**

- [ ] You can describe each OSI layer in one sentence
- [ ] You can compress and expand IPv6 addresses
- [ ] You know what `::1`, `fe80::...`, and `2001:...` addresses mean
- [ ] You understand what a VPN does and doesn't protect against

---

## Troubleshooting

**`ping6: command not found`** — On newer Linux distributions, `ping6` is merged into `ping`. Use `ping -6 ::1` instead. Same goes for modern macOS.

**`ip: command not found`** (macOS) — Use `ifconfig` instead. macOS follows the BSD tradition.

**IPv6 ping times out immediately** — Your network doesn't have IPv6 internet access. That's fine — the local-only IPv6 exercises still work, because link-local and loopback addresses don't need internet.

**`dig: command not found`** — On Ubuntu/Debian: `sudo apt install dnsutils`. On macOS: pre-installed. On Windows: use WSL or Git Bash, or use `nslookup` as a substitute.

**Alpine container doesn't find `apk`** — Make sure you started it as `alpine`, not some other image. Try `docker run --rm -it alpine:latest sh`.

**SSH SOCKS proxy: "Unable to connect"** — Your server's firewall may not allow outbound connections to arbitrary ports, or the `-D` option requires `AllowTcpForwarding yes` in `sshd_config` (it's the default on most systems).
