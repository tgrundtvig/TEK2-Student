# Class Exercises: OSI Model, IPv6 & VPN

**Work through these exercises during class. Ask for help if you get stuck!**

---

## Warm-Up (~5 minutes)

Quick check — can you answer these from the pre-class reading?

1. Name the seven OSI layers from top to bottom (or bottom to top — whichever you find easier).
2. What's the difference between an `A` record and an `AAAA` record?
3. What does a VPN *not* protect you against?

Compare your answers with a classmate. If you disagree, discuss!

---

## Exercise 1: OSI Scavenger Hunt in Wireshark (~40 minutes)

**Goal:** Capture a single HTTPS request and identify every OSI layer present in one packet. This turns the OSI model from an abstract diagram into something you can point to and name.

### Part A: Set up a capture

1. Open Wireshark.
2. Select your main network interface (Wi-Fi or Ethernet — the same one you used in Weeks 5 and 8).
3. Click the blue shark-fin **Start** button.

### Part B: Generate traffic

In a terminal, run a single HTTPS request so your capture has something clear to look at:

```bash
curl -s -o /dev/null https://example.com
```

Then stop the Wireshark capture (the red stop button).

### Part C: Filter down

Type this in the Wireshark filter bar and press Enter:

```
tls.handshake.type == 1 and ip.addr == 93.184.216.34
```

(`93.184.216.34` is example.com's IP at the time of writing. If the filter shows nothing, use `dig example.com` to find the current IP and update the filter. Or just filter on `tls.handshake.type == 1` to see all TLS ClientHello packets and pick one.)

You should see a single packet — the TLS `ClientHello`. Click it.

### Part D: Expand every layer

In the middle pane (Packet Details), you'll see sections like:

```
► Frame N: ...
► Ethernet II, Src: ..., Dst: ...
► Internet Protocol Version 4 (or 6), Src: ..., Dst: ...
► Transmission Control Protocol, Src Port: ..., Dst Port: 443
► Transport Layer Security
   ► TLSv1.3 Record Layer: Handshake Protocol: Client Hello
```

Click the triangle on each line to expand it. Write down:

| OSI Layer | Wireshark Section | One specific field you see |
|-----------|------------------|---------------------------|
| 1 — Physical | (Frame info at top) | Length in bytes: _____ |
| 2 — Data Link | Ethernet II | Destination MAC: _____ |
| 3 — Network | Internet Protocol | Destination IP: _____ |
| 4 — Transport | Transmission Control Protocol | Destination Port: _____ |
| 6 — Presentation | Transport Layer Security | Handshake Type: _____ |
| 7 — Application | (inside TLS: the servername extension) | Server Name: _____ |

### Part E: See encapsulation

Notice how each section is **nested** inside the one above it:

```
Frame (Layer 1)
  └── Ethernet (Layer 2)
        └── IP (Layer 3)
              └── TCP (Layer 4)
                    └── TLS (Layer 6)
                          └── Application data
```

That's encapsulation — each layer wraps the one above with its own header. Wireshark shows it stacked so you can see it.

### Self-check

- [ ] You captured a single HTTPS request
- [ ] You filtered down to one `ClientHello` packet
- [ ] You identified the MAC address, IP address, port, and TLS handshake type
- [ ] You can explain what "encapsulation" means using this packet as an example

---

## Exercise 2: IPv6 in Docker (~35 minutes)

**Goal:** See IPv6 working between containers, and observe link-local addresses in action.

### Part A: Start two Alpine containers

Open two terminal windows. In the first, start a container named `host-a`:

```bash
docker run --rm -it --name host-a alpine sh
```

In the second, start `host-b`:

```bash
docker run --rm -it --name host-b alpine sh
```

### Part B: Install networking tools in both

Inside **each** container, run:

```bash
apk add --no-cache iputils iproute2
```

### Part C: Look at the IPv6 addresses

In `host-a`:

```bash
ip -6 addr
```

You should see something like:

```
1: lo: ...
    inet6 ::1/128 scope host
    ...
2: eth0@if..: ...
    inet6 fe80::42:acff:fe11:2/64 scope link
```

Note the `fe80::...` address — that's a **link-local** address. Every IPv6 interface auto-generates one based on its MAC address. Write it down.

Do the same in `host-b` and write down its link-local address.

### Part D: Ping between containers on IPv6

Link-local addresses need a **zone identifier** — the interface name — because the same `fe80::1` could exist on many interfaces. The syntax is `address%interface`.

In `host-a`, ping `host-b`'s link-local address:

```bash
ping6 -c 3 <host-b-linklocal>%eth0
```

Replace `<host-b-linklocal>` with the address you wrote down. You should see replies.

### Part E: Check what happened

Look at the ARP/neighbor table:

```bash
ip -6 neigh
```

You'll see `host-b`'s address with its MAC — IPv6's equivalent of the ARP table. (In IPv6, this is called **Neighbor Discovery Protocol**, or NDP.)

### Part F: Global IPv6 in Docker (bonus)

By default, Docker doesn't give containers global IPv6 addresses. You can enable it by creating a custom network with an IPv6 subnet. Try this from your host (not inside a container):

```bash
docker network create --ipv6 --subnet="fd00:42::/64" my-ipv6-net
docker run --rm -it --network my-ipv6-net alpine sh -c "apk add --no-cache iputils iproute2 && ip -6 addr && sh"
```

Now your container has a **unique local** address (`fd00:42::...`) — the IPv6 equivalent of `10.x.x.x`.

Don't forget to clean up when done:

```bash
# Exit the containers first, then:
docker network rm my-ipv6-net
```

### Self-check

- [ ] You saw the automatic `fe80::...` link-local address on every container
- [ ] You pinged between containers using IPv6
- [ ] You understand why link-local needs the `%eth0` zone identifier
- [ ] You saw Neighbor Discovery populate the `ip -6 neigh` table

---

## Exercise 3: Build an SSH Tunnel "VPN" (~35 minutes)

**Goal:** Use SSH to tunnel a TCP connection through another container, then observe how the traffic looks from the outside.

We'll simulate "you" and "your server" with two Docker containers on the same network. The goal: send HTTP traffic through an SSH connection so anyone watching the wire between them sees only SSH.

### Part A: Set up a Docker network

```bash
docker network create ssh-lab
```

### Part B: Start an SSH server

```bash
docker run -d --rm --name ssh-server --network ssh-lab \
  -e USER_NAME=student \
  -e USER_PASSWORD=classpass \
  -e PASSWORD_ACCESS=true \
  linuxserver/openssh-server
```

Wait 10 seconds for it to finish starting, then verify:

```bash
docker exec ssh-server ss -tlnp | grep :2222
```

You should see something listening on port 2222 (the image's default SSH port).

### Part C: Start an HTTP server on the same network

```bash
docker run -d --rm --name web-target --network ssh-lab \
  kennethreitz/httpbin
```

This is the same `httpbin` image you used in Week 8.

### Part D: Start a client container

In a **new terminal**, start a container to act as "you":

```bash
docker run --rm -it --name ssh-client --network ssh-lab alpine sh
```

Inside, install the tools we need:

```bash
apk add --no-cache openssh-client curl tcpdump
```

### Part E: Open an SSH tunnel

From inside the client container, open an SSH port-forward:

```bash
ssh -f -N -L 9999:web-target:80 student@ssh-server -p 2222
```

- `-f` puts SSH in background after authentication
- `-N` means "no command — just forward"
- `-L 9999:web-target:80` forwards client's local port 9999 to `web-target:80` through the SSH server

When prompted, enter the password: `classpass`.

Also type `yes` when asked about host key fingerprints.

### Part F: Test the tunnel

Still inside the client container:

```bash
curl http://localhost:9999/get
```

You should see httpbin's JSON response. Your HTTP request just traveled through the SSH tunnel, popped out at the SSH server, and was forwarded to `web-target`.

### Part G: Watch the encryption happen

This is the fun part. While the tunnel is up, from a **third terminal**, run tcpdump *on the Docker bridge* to see what's on the wire between client and ssh-server:

```bash
docker exec ssh-client tcpdump -i any -n -X -c 20 host ssh-server and port 2222
```

Now trigger traffic (from the client container, as in Part F):

```bash
curl http://localhost:9999/get
```

Look at the tcpdump output. You'll see:
- Traffic to port 2222 (SSH)
- **Encrypted** payload — no `GET /get` readable in the hex dump

Compare with traffic on port 9999 (the tunnel input inside the client):

```bash
docker exec ssh-client tcpdump -i any -n -X -c 20 port 9999
# run the curl again while this captures
```

You'll see readable `GET /get HTTP/1.1` text — because at that point, the data hasn't entered the tunnel yet.

### Part H: Clean up

Exit the client container shell. Then:

```bash
docker stop ssh-server web-target
docker network rm ssh-lab
```

(If you get "container is using network", stop the containers first.)

### Self-check

- [ ] You opened an SSH tunnel with `-L`
- [ ] An HTTP request through the tunnel succeeded
- [ ] You observed that traffic to port 2222 was encrypted (SSH payload)
- [ ] You observed that traffic to port 9999 (local side of tunnel) was readable HTTP
- [ ] You understand this is a "VPN-lite" — the principle is the same, but it only tunnels one TCP port

---

## Exercise 4: Set Up a WireGuard VPN in Docker (~45 minutes)

**Goal:** Run a real WireGuard VPN between two containers, verify traffic is encrypted, and observe the handshake.

### Part A: Create a network and generate keys

Create a dedicated network:

```bash
docker network create wg-lab
```

Generate two keypairs in a helper container:

```bash
docker run --rm -v wg-keys:/keys alpine sh -c "
  apk add --no-cache wireguard-tools &&
  mkdir -p /keys &&
  wg genkey | tee /keys/server.priv | wg pubkey > /keys/server.pub &&
  wg genkey | tee /keys/client.priv | wg pubkey > /keys/client.pub &&
  echo 'Server priv:' && cat /keys/server.priv &&
  echo 'Server pub:'  && cat /keys/server.pub &&
  echo 'Client priv:' && cat /keys/client.priv &&
  echo 'Client pub:'  && cat /keys/client.pub
"
```

Copy the four keys to a text file for reference. You'll paste them into configs shortly.

### Part B: Write the server config

On your host, create a file named `server.conf`:

```ini
[Interface]
PrivateKey = <paste-server-priv-here>
Address = 10.13.13.1/24
ListenPort = 51820

[Peer]
PublicKey = <paste-client-pub-here>
AllowedIPs = 10.13.13.2/32
```

Replace the two `<paste-...>` placeholders with the actual keys.

### Part C: Write the client config

Create `client.conf` on your host:

```ini
[Interface]
PrivateKey = <paste-client-priv-here>
Address = 10.13.13.2/24

[Peer]
PublicKey = <paste-server-pub-here>
Endpoint = wg-server:51820
AllowedIPs = 10.13.13.0/24
PersistentKeepalive = 25
```

Again, replace the placeholders.

### Part D: Start the server container

```bash
docker run -d --rm --name wg-server \
  --network wg-lab \
  --cap-add NET_ADMIN --cap-add SYS_MODULE \
  -v "$(pwd)/server.conf:/etc/wireguard/wg0.conf" \
  -p 51820:51820/udp \
  alpine sh -c "
    apk add --no-cache wireguard-tools iptables iproute2 &&
    wg-quick up wg0 &&
    tail -f /dev/null
  "
```

Wait a few seconds, then verify it's running:

```bash
docker exec wg-server wg show
```

You should see interface `wg0` and one peer (without a handshake yet, because the client isn't up).

### Part E: Start the client container

```bash
docker run -d --rm --name wg-client \
  --network wg-lab \
  --cap-add NET_ADMIN --cap-add SYS_MODULE \
  -v "$(pwd)/client.conf:/etc/wireguard/wg0.conf" \
  alpine sh -c "
    apk add --no-cache wireguard-tools iptables iproute2 iputils &&
    wg-quick up wg0 &&
    tail -f /dev/null
  "
```

### Part F: Test the tunnel

From the client, ping the server over its VPN address:

```bash
docker exec wg-client ping -c 3 10.13.13.1
```

You should see replies. Your ping just traveled through an encrypted WireGuard tunnel.

Check the handshake was successful:

```bash
docker exec wg-server wg show
docker exec wg-client wg show
```

You should see a `latest handshake` timestamp and non-zero transfer stats.

### Part G: Watch the encrypted traffic

From a new terminal, start tcpdump inside the client container to watch UDP port 51820 (WireGuard's default):

```bash
docker exec wg-client sh -c "apk add --no-cache tcpdump && tcpdump -i any -n -X -c 10 udp port 51820"
```

In another terminal, generate some tunnel traffic:

```bash
docker exec wg-client ping -c 3 10.13.13.1
```

In the tcpdump output you'll see UDP packets to port 51820 with **fully encrypted payloads** — just random-looking bytes. None of the ICMP headers or ping data are readable.

### Part H: Clean up

```bash
docker stop wg-server wg-client
docker network rm wg-lab
docker volume rm wg-keys
rm -f server.conf client.conf
```

### Self-check

- [ ] You generated two WireGuard keypairs
- [ ] You wrote matching client and server configs (with keys in the right places!)
- [ ] Both containers came up and `wg show` reported a handshake
- [ ] Ping worked over the VPN address
- [ ] In tcpdump, the UDP payload was pure encrypted noise

### Troubleshooting

<details>
<summary>"Unable to access kernel interface"</summary>

Some Docker environments (particularly Docker Desktop on Mac/Windows) may lack the WireGuard kernel module. If so, swap `wireguard-tools` for `wireguard-go` (userspace implementation). Ask the instructor for the adjusted command.
</details>

<details>
<summary>No handshake showing in `wg show`</summary>

Common causes:
- Mismatched keys — the server's peer public key must be the client's actual public key, and vice versa
- Wrong `Endpoint` on the client — it should be `wg-server:51820`, matching the Docker name and server `ListenPort`
- `Address` subnets don't match — both must use `10.13.13.x/24`

Restart both containers after fixing: `docker restart wg-client wg-server`.
</details>

---

## Exercise 5: What Does Your VPN Actually Hide? (~25 minutes)

**Goal:** Reason carefully about what a VPN hides and what it doesn't. Pair up for this one.

### Part A: Fill in the threat table

For each scenario, decide whether a VPN helps (**Yes** / **No** / **Partially**) and justify in one sentence.

| Scenario | VPN helps? | Reason |
|----------|-----------|--------|
| Coffee shop Wi-Fi attacker reading your emails (webmail over HTTPS) | | |
| Coffee shop Wi-Fi attacker seeing which sites you visited | | |
| Your ISP selling your browsing history | | |
| Your ISP throttling Netflix | | |
| Facebook tracking your activity across the web | | |
| Your employer's network seeing what you do at work | | |
| A foreign government blocking a specific website | | |
| A website recognizing you after you log in | | |
| The VPN provider logging your activity | | |
| Someone on the LAN doing ARP spoofing | | |

Compare your answers with your partner. Where do you disagree? Why?

### Part B: Draw the encryption boundary

Draw a diagram showing:
- Your device
- The VPN server
- A destination website (e.g., a bank)
- Which segments are encrypted by HTTPS
- Which segments are encrypted by the VPN
- Which segments are encrypted by *both*

Share with your partner. Did you draw it the same way?

### Part C: When would you NOT want a VPN?

Discuss: name three situations where a VPN is unnecessary or actively harmful.

<details>
<summary>Example answers</summary>

1. **Trusted local network.** At home on a trusted router, the only party watching your traffic is your ISP — and many ISPs are fine. The VPN adds latency and a new party (the VPN provider) to trust.
2. **Services that block VPNs.** Banks and streaming services often block connections from known VPN IPs. Using a VPN could lock you out.
3. **Bad VPN providers.** A free VPN may sell your data, inject ads, or worse. Trading trusted ISP for sketchy VPN is a net loss.
4. **Latency-sensitive activities.** Gaming or video calls might be worse through a distant VPN server.

</details>

### Self-check

- [ ] You filled in the threat table with reasoning, not just yes/no
- [ ] You can draw the encryption boundaries in a VPN + HTTPS scenario
- [ ] You can name situations where a VPN is unnecessary

---

## Summary

| Exercise | What You Did | Key Takeaway |
|----------|-------------|--------------|
| 1 — OSI Scavenger | Identified every layer in one packet | Encapsulation is real and you can see it |
| 2 — IPv6 in Docker | Pinged between containers using `fe80::...` | Every IPv6 interface has a link-local address automatically |
| 3 — SSH Tunnel | Built a mini-VPN using SSH port forwarding | An encrypted tunnel is just... an encrypted tunnel |
| 4 — WireGuard VPN | Ran a real VPN between two containers | Modern VPN setup is tiny and fast |
| 5 — What VPNs Hide | Reasoned through a threat model | A VPN is a trust shift, not a trust eliminator |

**Key takeaway:** These three topics aren't separate — they're the *same* story. OSI tells you where things happen, IPv6 tells you how addresses work at Layer 3, and VPNs combine Layer 3 packets with Layer 10 encryption (that's a joke — there's no Layer 10) to create privacy across public networks.

Now go back to your Week 8 Wireshark captures if you still have them. You'll see them differently this time.
