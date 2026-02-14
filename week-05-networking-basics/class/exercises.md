# Class Exercises: Networking Basics

Work through these exercises during class. Ask for help if you get stuck!

---

## Warm-up: Verify Pre-class Work (~5 minutes)

Quick check that everyone's tools are ready.

### Check Wireshark

1. Open Wireshark
2. You should see a list of network interfaces
3. Close it for now — we'll use it later

### Check Docker

```bash
docker ps
```

This should run without errors (it's fine if no containers are listed).

### If you don't have Wireshark installed

Quick install:

**Windows:** Download from [wireshark.org/download](https://www.wireshark.org/download.html), run installer (include Npcap).

**macOS:** Download from [wireshark.org/download](https://www.wireshark.org/download.html), drag to Applications.

**Linux:**
```bash
sudo apt update && sudo apt install wireshark -y
sudo usermod -aG wireshark $USER
```
(Log out and back in after running this.)

### Self-check

- [ ] Wireshark is installed and opens
- [ ] Docker is running (`docker ps` works)

---

## Exercise 1: Network Identity Deep Dive (~20 minutes)

### Goal

Understand how Docker assigns network addresses to containers and see the network structure.

### Part A: Your host's network

Find your computer's IP address:

**Linux:**
```bash
ip addr
```

**macOS:**
```bash
ifconfig
```

**Windows (PowerShell):**
```powershell
ipconfig
```

Find your active interface (Wi-Fi or Ethernet) and note your IP address.

### Part B: Container network identities

Start three containers on the default bridge network:

```bash
docker run -d --name net-test-1 alpine sleep 3600
docker run -d --name net-test-2 alpine sleep 3600
docker run -d --name net-test-3 alpine sleep 3600
```

Now find each container's IP address:

```bash
docker inspect net-test-1 | grep IPAddress
docker inspect net-test-2 | grep IPAddress
docker inspect net-test-3 | grep IPAddress
```

**What do you notice?** The IPs should be sequential on the `172.17.0.x` subnet:
- net-test-1: `172.17.0.2`
- net-test-2: `172.17.0.3`
- net-test-3: `172.17.0.4`

### Part C: Look inside a container

Check the network from inside one of the containers:

```bash
docker exec net-test-1 ip addr
```

You'll see:
- `lo` (loopback): `127.0.0.1` — the container's localhost
- `eth0`: The container's IP on the Docker bridge network

### Part D: Different network, different subnet

Create a custom network and start a container on it:

```bash
docker network create test-net
docker run -d --name net-test-4 --network test-net alpine sleep 3600
docker inspect net-test-4 | grep IPAddress
```

**What do you notice?** This container gets an IP from a different subnet (likely `172.18.0.x` or `172.19.0.x`). Docker gives each custom network its own IP range.

### Part E: Clean up

```bash
docker stop net-test-1 net-test-2 net-test-3 net-test-4
docker rm net-test-1 net-test-2 net-test-3 net-test-4
docker network rm test-net
```

### Self-check

- [ ] Found your host IP address
- [ ] Containers on the bridge network get sequential 172.17.0.x addresses
- [ ] Containers on a custom network get a different subnet
- [ ] You understand that Docker manages a separate IP space for containers

---

## Exercise 2: Docker Networking and Isolation (~25 minutes)

### Goal

Build a multi-network Docker setup and understand network isolation — the same concept Docker Compose uses.

### Part A: Create two networks

```bash
docker network create frontend-net
docker network create backend-net
```

### Part B: Start containers on different networks

```bash
# Web server on the frontend network
docker run -d --name web --network frontend-net alpine sleep 3600

# API server on both networks (it bridges frontend and backend)
docker run -d --name api --network frontend-net alpine sleep 3600
docker network connect backend-net api

# Database on the backend network only
docker run -d --name database --network backend-net alpine sleep 3600
```

Let's visualize what we built:

```
┌─ frontend-net ─────────────────┐
│                                │
│   ┌─────┐        ┌─────┐      │
│   │ web │        │ api │──────────┐
│   └─────┘        └─────┘      │  │
│                                │  │
└────────────────────────────────┘  │
                                    │
┌─ backend-net ─────────────────┐   │
│                               │   │
│   ┌──────────┐    ┌─────┐    │   │
│   │ database │    │ api │◄───────┘
│   └──────────┘    └─────┘    │
│                               │
└───────────────────────────────┘
```

### Part C: Test connectivity

**web → api (same network: frontend-net):**
```bash
docker exec web ping -c 2 api
```
This should work.

**api → database (same network: backend-net):**
```bash
docker exec api ping -c 2 database
```
This should work.

**web → database (different networks!):**
```bash
docker exec web ping -c 2 database
```
This should **fail**! The web container is only on frontend-net and cannot reach the backend-net.

### Part D: Why this matters

This is exactly how Docker Compose isolates services. In a typical web application:
- The **web frontend** can talk to the **API** but not directly to the **database**
- The **API** bridges both networks — it can talk to the frontend and the database
- The **database** is protected — only the API can reach it

This is a real security pattern used in production systems.

### Part E: Verify with docker network inspect

```bash
docker network inspect frontend-net
```

In the `"Containers"` section, you should see `web` and `api`. The `database` container is NOT listed here — it's only on `backend-net`.

### Part F: Clean up

```bash
docker stop web api database
docker rm web api database
docker network rm frontend-net backend-net
```

### Self-check

- [ ] Created two separate Docker networks
- [ ] Containers on the same network can ping each other
- [ ] Containers on different networks cannot communicate
- [ ] A container connected to both networks can bridge them
- [ ] You understand how this relates to Docker Compose service isolation

---

## Exercise 3: TCP Conversation with Netcat (~25 minutes)

### Goal

See TCP and UDP in action by sending messages between containers using `netcat` — a simple networking tool.

### Part A: Set up the containers

Create a network and two containers with netcat installed:

```bash
docker network create chat-net

docker run -dit --name alice --network chat-net alpine sh
docker run -dit --name bob --network chat-net alpine sh
```

Install netcat in both containers:

```bash
docker exec alice apk add --no-cache netcat-openbsd
docker exec bob apk add --no-cache netcat-openbsd
```

### Part B: TCP chat

In this part, you'll need **two terminal windows** (or tabs) open side by side.

**Terminal 1 — Alice listens for connections:**

```bash
docker exec -it alice nc -l -p 9999
```

Breaking this down:
- `nc` — netcat, the networking tool
- `-l` — **listen** mode (wait for incoming connections)
- `-p 9999` — listen on **port 9999**

Alice is now waiting for someone to connect.

**Terminal 2 — Bob connects to Alice:**

```bash
docker exec -it bob nc alice 9999
```

Breaking this down:
- `nc alice 9999` — connect to host `alice` on port `9999`
- Docker DNS resolves `alice` to its IP address

**Now type messages!** Type something in Terminal 2 (Bob) and press Enter. It appears in Terminal 1 (Alice). Type in Terminal 1 — it appears in Terminal 2.

You are looking at a **raw TCP connection**. This is what happens underneath HTTP, SSH, and every other TCP-based protocol. They just add structure to the messages.

Press **Ctrl+C** in either terminal to close the connection.

### Part C: UDP chat

Let's compare with UDP. Again, two terminal windows:

**Terminal 1 — Alice listens for UDP:**

```bash
docker exec -it alice nc -u -l -p 8888
```

The `-u` flag means **UDP** instead of TCP.

**Terminal 2 — Bob sends via UDP:**

```bash
docker exec -it bob nc -u alice 8888
```

Type messages back and forth. It looks similar, but the behavior is fundamentally different:

- **TCP (Part B)**: A connection was established first (three-way handshake). Data delivery is guaranteed.
- **UDP (Part C)**: No connection. Messages are just sent. If one gets lost, it's gone.

Press **Ctrl+C** to close.

### Part D: What port is being used?

While a netcat listener is running, open a **third terminal** and check what's happening:

Start Alice listening again:
```bash
docker exec -it alice nc -l -p 9999
```

In the third terminal, check open ports in the Alice container:

```bash
docker exec alice netstat -tlnp
```

You should see port 9999 listed as LISTEN. This is the same concept as when nginx listens on port 80 or SSH listens on port 22.

### Part E: Don't clean up yet!

Keep the `chat-net` network and the `alice` and `bob` containers running — we'll use them in Exercise 5.

### Self-check

- [ ] You sent messages over TCP between two containers
- [ ] You sent messages over UDP between two containers
- [ ] You can see that TCP requires a connection while UDP doesn't
- [ ] You understand this is the foundation underneath HTTP, SSH, and other protocols

---

## Exercise 4: Port Mapping Revealed (~20 minutes)

### Goal

See Docker port mapping at the operating system level — understanding what `-p 8080:80` actually does.

### Part A: Check current ports on your host

**Linux:**
```bash
ss -tulpn
```

**macOS:**
```bash
netstat -an | grep LISTEN
```

**Windows (PowerShell):**
```powershell
netstat -an | findstr LISTENING
```

This shows all ports currently in use on your computer. You might see Docker, your IDE, and other services.

### Part B: Start a web server with port mapping

```bash
docker run -d -p 8080:80 --name web-test nginx:1.25
```

### Part C: See the port appear

Run the same command from Part A again:

**Linux:**
```bash
ss -tulpn
```

**macOS:**
```bash
netstat -an | grep LISTEN
```

**Windows (PowerShell):**
```powershell
netstat -an | findstr LISTENING
```

Look for port **8080** — it's now listed! Docker created a real TCP listener on your host.

### Part D: Verify it works

Open your browser to [http://localhost:8080](http://localhost:8080) — you should see the nginx welcome page.

What just happened:
1. Your browser connected to `127.0.0.1:8080` (TCP)
2. Docker intercepted the connection on port 8080
3. Docker forwarded it to the nginx container's port 80
4. nginx responded, and Docker forwarded the response back

### Part E: Port conflict

Try starting a second container on the same port:

```bash
docker run -d -p 8080:80 --name web-test-2 nginx:1.25
```

**This fails!** You'll see an error like: `port is already allocated`

Only one process can listen on a given port. This is a fundamental networking rule.

### Part F: Use a different port

```bash
docker run -d -p 8081:80 --name web-test-2 nginx:1.25
```

Now open [http://localhost:8081](http://localhost:8081) — a second nginx instance, on a different port.

Both containers run nginx on their internal port 80, but they're mapped to different host ports (8080 and 8081).

> **⏱️ If short on time:** Skip Part E-F and move to Exercise 5.

### Part G: Clean up

```bash
docker stop web-test web-test-2 2>/dev/null
docker rm web-test web-test-2 2>/dev/null
```

### Self-check

- [ ] You can see which ports are in use on your computer
- [ ] Port mapping creates a real port on your host
- [ ] Two containers cannot share the same host port
- [ ] You now fully understand what `-p 8080:80` does

---

## Exercise 5: Your First Wireshark Capture (~25 minutes)

### Goal

Use Wireshark to capture real network traffic and observe the concepts from today's reading: ICMP packets and TCP three-way handshakes.

### Part A: Start Wireshark

1. Open Wireshark
2. You'll see a list of network interfaces
3. Select your **active interface**:

| Platform | Likely Interface |
|----------|-----------------|
| Linux | `eth0` or `wlan0` or `enp0s3` |
| macOS | `en0` (Wi-Fi) or `en1` (Ethernet) |
| Windows | `Wi-Fi` or `Ethernet` |

> **Tip:** Look for the interface with a small activity graph — that's the one carrying traffic.

4. Double-click the interface to start capturing

### Part B: Capture a ping

With Wireshark capturing, open a terminal and ping an external host:

**Linux/macOS:**
```bash
ping -c 5 google.com
```

**Windows:**
```powershell
ping -n 5 google.com
```

### Part C: Filter and examine ICMP packets

1. In Wireshark, click the **Stop** button (red square) to stop capturing
2. In the filter bar at the top, type: `icmp` and press Enter
3. You should see pairs of packets:
   - **Echo (ping) request**: Your computer → Google
   - **Echo (ping) reply**: Google → Your computer

4. Click on one of the packets. In the lower panes, you'll see the **layers**:
   - **Ethernet II**: The MAC addresses (Layer 2)
   - **Internet Protocol**: The source and destination IP addresses (Layer 3)
   - **Internet Control Message Protocol**: The ping data

Every concept from the reading is visible right here!

### Part D: Capture a TCP three-way handshake

1. Click the **green shark fin** button to start a new capture (or go to Capture → Restart)
2. In your terminal, make an HTTP request:

```bash
curl http://example.com
```

Or simply open `http://example.com` in your browser.

3. Stop the capture in Wireshark

### Part E: Find the handshake

1. In the filter bar, type: `tcp` and press Enter
2. Look at the **Info** column. Find three consecutive packets:
   - `[SYN]` — Your computer initiates the connection
   - `[SYN, ACK]` — The server acknowledges and agrees
   - `[ACK]` — Your computer confirms

That's the **three-way handshake** you read about! It happened in milliseconds.

3. Click on the `[SYN]` packet. In the details pane, expand **Transmission Control Protocol**. You can see:
   - **Source Port**: Your ephemeral port (a high number like 54321)
   - **Destination Port**: 80 (HTTP)
   - **Flags**: SYN is set

### Part F: Explore the layers

Click on any TCP packet and look at the detail panes. You'll see the layered structure:

```
Ethernet II (Layer 2)      → MAC addresses
Internet Protocol (Layer 3) → IP addresses
Transmission Control (Layer 4) → Ports, flags, sequence numbers
```

These are the layers you read about, visible in real captured data. In Week 8, we'll add a fourth layer — HTTP — and see actual web page content in these packets.

### Self-check

- [ ] You captured network traffic with Wireshark
- [ ] You filtered for ICMP and found ping request/reply pairs
- [ ] You found a TCP three-way handshake (SYN → SYN-ACK → ACK)
- [ ] You can identify source/destination IPs and ports in a packet
- [ ] You can see the layered structure (Ethernet → IP → TCP)

---

## Bonus: Traceroute (~10 minutes, if time permits)

### Goal

See the path packets take across the internet.

### Step 1: Traceroute from a container

```bash
docker run --rm alpine sh -c "apk add --no-cache traceroute > /dev/null 2>&1 && traceroute -m 15 google.com"
```

This installs traceroute and runs it in one command.

### Step 2: Read the output

Each line represents a **router** (hop) between you and Google:

```
 1  172.17.0.1 (172.17.0.1)  0.043 ms
 2  192.168.1.1 (192.168.1.1)  1.234 ms
 3  87.54.X.X (87.54.X.X)  5.678 ms
 ...
12  142.250.X.X (142.250.X.X)  12.345 ms
```

- Hop 1: Docker's gateway (172.17.0.1)
- Hop 2: Your home router (192.168.x.x)
- Hops 3+: Internet routers belonging to ISPs and backbone providers
- Last hop: Google's server

Your data passes through all these routers every time you access a website!

### Self-check

- [ ] You can trace the route packets take to reach a server
- [ ] You understand that data travels through multiple routers

---

## Final Cleanup

Remove all containers and networks from today's exercises:

```bash
# Stop and remove any remaining containers
docker stop alice bob 2>/dev/null
docker rm alice bob 2>/dev/null

# Remove custom networks
docker network rm chat-net 2>/dev/null
```

---

## Summary

| Skill | What You Practiced | Connection to Earlier Weeks |
|-------|-------------------|---------------------------|
| `ip addr` / `ipconfig` | Find IP and MAC addresses | Understanding container networking |
| Docker networks | Create networks, test isolation | Week 2: Docker Compose service communication |
| Netcat (nc) | Raw TCP and UDP communication | Foundation of HTTP, SSH, and all protocols |
| `ss` / `netstat` | See port mapping at OS level | Week 1: `docker run -p 8080:80` |
| Wireshark | Capture and analyze packets | Visualize TCP handshake, ICMP, layers |
| Traceroute | See the path packets travel | Week 4: How SSH reaches your cloud server |

**Key takeaways:**

1. Docker creates isolated networks with their own IP ranges
2. Containers on the same network can communicate; across networks they can't
3. `-p 8080:80` creates a real TCP listener on your host machine
4. TCP establishes a connection (handshake) before sending data; UDP doesn't
5. Wireshark lets you see all of this happening in real time
6. Every concept from today was already part of your work in Weeks 1-4 — now you understand what was happening underneath
