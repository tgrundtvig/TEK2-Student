# Hints for Advanced Tasks

Try to solve the tasks on your own first! Use these hints only when you're truly stuck.

---

## Task 1: TCP vs UDP Deep Comparison

<details>
<summary>Hint 1: Netcat isn't sending my message</summary>

Make sure you have the right flags:
- **TCP listen:** `nc -l -p PORT`
- **UDP listen:** `nc -u -l -p PORT`
- **TCP connect:** `nc HOSTNAME PORT`
- **UDP connect:** `nc -u HOSTNAME PORT`

The `-u` flag switches between TCP and UDP. Without it, netcat defaults to TCP.

If the receiver isn't getting messages, make sure both containers are on the same Docker network.

</details>

<details>
<summary>Hint 2: "Connection refused" error with TCP</summary>

This means nothing is listening on the port you're trying to connect to. Make sure:
1. The receiver is started **before** the sender
2. The receiver is using the same port number
3. Both containers are on the same network

"Connection refused" is actually a TCP feature — the server actively tells you "no one is listening here." With UDP, you wouldn't get this error — your message would just disappear silently.

</details>

<details>
<summary>Hint 3: Research questions guidance</summary>

**Why DNS uses UDP:**
- DNS queries are small (usually fits in one packet)
- Speed matters — your browser makes many DNS queries
- UDP has less overhead (no handshake needed)
- If the response is lost, the client simply asks again

**Why video streaming uses UDP:**
- Occasional lost frames are invisible to the viewer
- Re-sending old frames would cause delays (lag)
- Real-time delivery is more important than perfect accuracy

**Why databases use TCP:**
- Every query and every row of data must arrive correctly
- Lost or corrupted data could corrupt the database
- Transactions require reliable, ordered delivery

**What if SSH used UDP:**
- Your keystrokes might get lost or arrive out of order
- Commands could execute partially or incorrectly
- The encrypted tunnel requires reliable in-order delivery

</details>

---

## Task 2: Wireshark TCP Handshake Analysis

<details>
<summary>Hint 1: I can't find the SYN packets</summary>

Use the display filter: `tcp.flags.syn==1`

This shows only packets with the SYN flag set (the first two packets of every TCP handshake).

If you see too many results, narrow it down. First, find the IP address of example.com:

```bash
ping -c 1 example.com
```

Then filter for that IP in Wireshark:
```
tcp.flags.syn==1 && ip.addr==93.184.216.34
```

Replace the IP with whatever `ping` showed.

</details>

<details>
<summary>Hint 2: How to read the packet details</summary>

Click on a packet in the top pane. The middle pane shows the layers:

- **Ethernet II**: Click the `>` to expand. You'll see source and destination MAC addresses.
- **Internet Protocol Version 4**: Expand to see source and destination IP addresses.
- **Transmission Control Protocol**: Expand to see:
  - **Source Port**: The port on the sender's side
  - **Destination Port**: The port on the receiver's side (80 for HTTP)
  - **Flags**: Shows which flags are set (SYN, ACK, FIN, etc.)
  - **Sequence number**: Used to order packets

</details>

<details>
<summary>Hint 3: Finding the FIN packets</summary>

Use the display filter: `tcp.flags.fin==1`

FIN packets signal the end of a TCP connection. You should see:
1. One side sends FIN ("I'm done sending")
2. The other side sends ACK ("I got your FIN")
3. The other side sends its own FIN ("I'm done too")
4. First side sends ACK ("Got it, connection closed")

Sometimes steps 2 and 3 are combined into a single FIN-ACK packet.

</details>

<details>
<summary>Hint 4: Seeing the complete TCP stream</summary>

To see all packets from a single TCP connection in order:

1. Right-click on any packet in the connection
2. Select **Follow** → **TCP Stream**
3. This shows the entire conversation in a readable format

Or use the display filter: `tcp.stream eq N` where N is the stream number (shown in the packet details under TCP).

</details>

---

## Task 3: Docker Network Topologies

<details>
<summary>Hint 1: How to connect a container to multiple networks</summary>

When you create a container with `docker run`, you can only specify one network with `--network`. To add it to a second network, use:

```bash
docker network connect NETWORK_NAME CONTAINER_NAME
```

For example:
```bash
docker run -d --name api --network public-net alpine sleep 3600
docker network connect private-net api
```

Now `api` is on both `public-net` and `private-net`.

</details>

<details>
<summary>Hint 2: Ping isn't installed in Alpine</summary>

Alpine containers don't have `ping` by default. Install it:

```bash
docker exec CONTAINER_NAME apk add --no-cache iputils-ping
```

Then you can ping:

```bash
docker exec CONTAINER_NAME ping -c 1 TARGET_NAME
```

</details>

<details>
<summary>Hint 3: Cross-network ping times out instead of failing immediately</summary>

When you ping across networks (which should fail), the behavior depends on the situation:

- **"bad address"**: DNS can't resolve the name (the target container doesn't exist on this network)
- **Timeout (no response)**: The packets are sent but never arrive
- **"Network unreachable"**: No route to the destination network

All of these mean the same thing: the containers are on different networks and can't communicate. This is Docker network isolation working correctly.

</details>

<details>
<summary>Hint 4: Verifying which networks a container is on</summary>

Use `docker inspect` to see a container's networks:

```bash
docker inspect api --format '{{json .NetworkSettings.Networks}}' | python3 -m json.tool
```

Or more simply:

```bash
docker inspect api | grep -A 5 "Networks"
```

You should see both `public-net` and `private-net` listed for the `api` container.

</details>

---

## Task 4: Investigate Your Cloud Server

<details>
<summary>Hint 1: Understanding ss -tulpn output</summary>

The flags mean:
- `-t`: Show TCP sockets
- `-u`: Show UDP sockets
- `-l`: Show only listening sockets
- `-p`: Show the process using the socket
- `-n`: Show port numbers instead of service names

Example output:
```
State    Recv-Q Send-Q Local Address:Port  Peer Address:Port Process
LISTEN   0      128    0.0.0.0:22          0.0.0.0:*         sshd
LISTEN   0      511    0.0.0.0:80          0.0.0.0:*         docker-proxy
```

- `0.0.0.0:22` means SSH is listening on all interfaces, port 22
- `0.0.0.0:80` means Docker is proxying port 80 on all interfaces
- `0.0.0.0:*` under Peer means it accepts connections from anywhere

</details>

<details>
<summary>Hint 2: Private vs public IPs on a cloud server</summary>

Many cloud servers have both:
- A **public IP** (like `157.245.32.10`) — this is what you SSH to
- A **private IP** (like `10.19.0.5`) — used for internal cloud networking

When you run `ip addr`, you might only see the private IP. The public IP is often handled by the cloud provider's network (NAT). This is normal!

Some providers (like Hetzner) show the public IP directly. Others (like DigitalOcean, Azure) use NAT.

</details>

<details>
<summary>Hint 3: Connecting firewall, ports, and Docker</summary>

The full chain of traffic to your server:

```
Internet → Cloud Provider's Network → Server's Public IP
    → UFW Firewall (is this port allowed?)
        → Docker Port Mapping (-p HOST:CONTAINER)
            → Container's Application
```

If any step blocks the traffic, the connection fails:
- UFW doesn't allow the port → traffic is dropped
- Docker doesn't have a port mapping → nothing is listening
- Container isn't running → connection refused

</details>

---

## Task 5: Document Your Networking Knowledge

<details>
<summary>Hint 1: Data flow diagram structure</summary>

Here's a template for your ASCII diagram:

```
User's Browser (192.168.1.42:54321)
        │
        │  TCP connection over the Internet
        ▼
Your Server's Public IP (157.245.32.10:80)
        │
        │  UFW firewall: port 80 allowed
        ▼
Docker Port Mapping (-p 80:80)
        │
        │  Forward to container
        ▼
Container (172.17.0.2:80)
        │
        │  nginx processes the request
        ▼
Response travels back the same path
```

Fill in with your own server's actual IPs and ports.

</details>

<details>
<summary>Hint 2: Good analogies for TCP vs UDP</summary>

Some analogies that work well:
- **TCP**: Phone call — you establish a connection, both sides talk, you hang up when done
- **UDP**: Sending postcards — you write and send, no guarantee they arrive or arrive in order
- **TCP**: Certified mail — delivery confirmed, re-sent if lost
- **UDP**: Throwing paper airplanes — fast, fun, but no guarantee where they land

Use whatever analogy makes sense to you. The best analogy is one you come up with yourself.

</details>

---

## General Troubleshooting

<details>
<summary>Wireshark shows "permission denied" (Linux)</summary>

Run this command and then **log out and back in**:

```bash
sudo usermod -aG wireshark $USER
```

If that doesn't work, you can run Wireshark as root (not recommended for regular use):

```bash
sudo wireshark
```

</details>

<details>
<summary>Wireshark captures nothing / no interfaces shown</summary>

**Windows:** Make sure Npcap was installed with Wireshark. Try reinstalling Wireshark and check the Npcap checkbox.

**macOS:** You may need to grant Wireshark permission. Go to System Preferences → Security & Privacy → Privacy, and check if Wireshark has access.

**Linux:** Make sure you're in the `wireshark` group (see above) and that you selected the right interface.

**All platforms:** Make sure you selected an active interface. If you're on Wi-Fi, select the Wi-Fi interface, not Ethernet (and vice versa).

</details>

<details>
<summary>Netcat not found in Alpine container</summary>

Alpine uses `netcat-openbsd`. Install it:

```bash
apk add --no-cache netcat-openbsd
```

After installation, the command is `nc` (not `netcat`).

</details>

<details>
<summary>Container can't reach the internet</summary>

Try:

```bash
docker run --rm alpine ping -c 1 8.8.8.8
```

If this fails, Docker's DNS or network might be misconfigured. Try restarting Docker:

**Linux:** `sudo systemctl restart docker`
**Windows/Mac:** Restart Docker Desktop

</details>

<details>
<summary>"Port already in use" error</summary>

Another container or application is already using that port. Find out what:

**Linux:** `ss -tulpn | grep PORT_NUMBER`
**macOS:** `lsof -i :PORT_NUMBER`
**Windows:** `netstat -an | findstr PORT_NUMBER`

Either stop the process using that port, or choose a different port number.

</details>

---

## Still Stuck?

1. **Read the error message carefully** — networking errors are usually descriptive
2. **Check that containers are on the same network** — `docker network inspect NETWORK_NAME`
3. **Verify the container is running** — `docker ps`
4. **Try the simplest case first** — if a complex setup doesn't work, try with just two containers on one network
5. **Ask in class** — bring your questions to the next session!
