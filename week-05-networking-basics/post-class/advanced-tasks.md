# Post-class Advanced Tasks: Networking Basics

**Estimated time: 1-2 hours**

These tasks will deepen your understanding of networking concepts. Try to complete them before next week's class.

If you get stuck, check the [hints.md](hints.md) file - but try on your own first!

---

## Task 1: TCP vs UDP Deep Comparison (~25 minutes)

### Objective

Experience the real behavioral differences between TCP and UDP by sending data with netcat and observing what happens.

### Instructions

#### Part A: TCP reliability test

Set up two containers on a shared network (or reuse from class):

```bash
docker network create task-net
docker run -dit --name sender --network task-net alpine sh
docker run -dit --name receiver --network task-net alpine sh
docker exec sender apk add --no-cache netcat-openbsd
docker exec receiver apk add --no-cache netcat-openbsd
```

**Terminal 1 — Receiver listens on TCP:**

```bash
docker exec -it receiver nc -l -p 5000
```

**Terminal 2 — Sender connects and sends:**

```bash
docker exec -it sender sh -c 'echo "Message 1" | nc receiver 5000'
```

Observe: the message appears in Terminal 1 reliably.

#### Part B: UDP behavior

**Terminal 1 — Receiver listens on UDP:**

```bash
docker exec -it receiver nc -u -l -p 6000
```

**Terminal 2 — Sender sends via UDP:**

```bash
docker exec -it sender sh -c 'echo "Hello UDP" | nc -u receiver 6000'
```

Notice: UDP messages are sent without a connection being established first. There's no handshake.

#### Part C: Research and reflect

Answer these questions (write them down):

1. Why does DNS use UDP for queries instead of TCP? (Hint: think about speed and message size)
2. Why does video streaming prefer UDP? What happens if a video packet is lost?
3. Why would a database connection always use TCP?
4. SSH uses TCP. What would happen if SSH used UDP instead?

### Verify Success

- [ ] You sent messages over both TCP and UDP
- [ ] You can explain at least two real-world reasons for choosing TCP vs UDP
- [ ] You wrote down answers to the research questions

---

## Task 2: Wireshark TCP Handshake Analysis (~30 minutes)

### Objective

Capture and analyze a complete TCP connection: the three-way handshake, data transfer, and connection teardown.

### Instructions

#### Part A: Capture a clean connection

1. Open Wireshark and start capturing on your active interface
2. In your terminal, make a simple HTTP request:

```bash
curl http://example.com
```

3. Stop the Wireshark capture

#### Part B: Find the three-way handshake

Apply the filter: `tcp.flags.syn==1`

This shows only SYN and SYN-ACK packets — the start of TCP connections.

Find the pair that connects to `example.com`:
- **Packet 1** (`[SYN]`): Your computer → example.com
- **Packet 2** (`[SYN, ACK]`): example.com → Your computer

Now remove the filter and look at the packet right after the SYN-ACK:
- **Packet 3** (`[ACK]`): Your computer → example.com

That's the complete three-way handshake.

#### Part C: Document the connection details

For each of the three handshake packets, write down:

| Packet | Source IP | Dest IP | Source Port | Dest Port | Flags |
|--------|-----------|---------|-------------|-----------|-------|
| 1 (SYN) | | | | | |
| 2 (SYN-ACK) | | | | | |
| 3 (ACK) | | | | | |

Notice:
- Your **source port** is a high number (ephemeral port)
- The **destination port** is 80 (HTTP)
- The roles flip for the SYN-ACK (server → client)

#### Part D: Find the connection teardown

Apply the filter: `tcp.flags.fin==1`

This shows FIN (finish) packets. A TCP connection is closed with a similar handshake:
- **FIN**: "I'm done sending"
- **ACK**: "I got your FIN"
- (Repeated in the other direction)

### Challenge Extension

- Apply the filter `tcp.stream eq 0` (adjust the number) to see the entire conversation for a single TCP connection in order
- Can you identify where the HTTP data (the actual web page) was sent?

### Verify Success

- [ ] You found the SYN → SYN-ACK → ACK handshake
- [ ] You documented source/destination IPs and ports for each packet
- [ ] You found the FIN packets that close the connection
- [ ] You can explain the complete lifecycle of a TCP connection

---

## Task 3: Docker Network Topologies (~30 minutes)

### Objective

Build a realistic three-tier network architecture using Docker — the same pattern used in production web applications.

### Instructions

#### Part A: Create the networks

```bash
docker network create public-net
docker network create private-net
docker network create data-net
```

#### Part B: Deploy the "application"

```bash
# Nginx (web frontend) - only on public network
docker run -d --name nginx --network public-net -p 8080:80 nginx:1.25-alpine

# "API server" - bridges public and private networks
docker run -d --name api --network public-net alpine sleep 3600
docker network connect private-net api

# "Database" - only on private network
docker run -d --name db --network private-net alpine sleep 3600
docker network connect data-net db

# "Backup service" - only on data network
docker run -d --name backup --network data-net alpine sleep 3600
```

The architecture:

```
Internet (via -p 8080:80)
    │
    ▼
┌─ public-net ──────────────────┐
│   ┌───────┐     ┌─────┐      │
│   │ nginx │     │ api │──────────┐
│   └───────┘     └─────┘      │  │
└───────────────────────────────┘  │
                                   │
┌─ private-net ─────────────────┐  │
│   ┌─────┐       ┌────┐       │  │
│   │ api │◄──────│ db │───────────┐
│   └─────┘       └────┘       │  ││
└───────────────────────────────┘  ││
                                   ││
┌─ data-net ────────────────────┐  ││
│   ┌────┐       ┌────────┐    │  ││
│   │ db │◄──────│ backup │    │◄─┘│
│   └────┘       └────────┘    │   │
└───────────────────────────────┘   │
```

#### Part C: Test the isolation

Install ping in the containers that need it:

```bash
docker exec api apk add --no-cache iputils-ping
docker exec db apk add --no-cache iputils-ping
docker exec backup apk add --no-cache iputils-ping
```

Test these connections and record the results:

| From | To | Expected | Actual |
|------|----|----------|--------|
| api | nginx | Works (same public-net) | |
| api | db | Works (same private-net) | |
| db | backup | Works (same data-net) | |
| nginx | db | Fails (different networks) | |
| nginx | backup | Fails (different networks) | |
| backup | api | Fails (different networks) | |

```bash
# Example test commands:
docker exec api ping -c 1 nginx
docker exec api ping -c 1 db
docker exec db ping -c 1 backup
docker exec nginx ping -c 1 db          # Should fail
docker exec backup ping -c 1 api        # Should fail
```

#### Part D: Why this matters

This three-tier pattern is the standard for production applications:
- **Public tier**: Only web servers are exposed to the internet
- **Application tier**: Business logic, accessible from web tier but not from the internet
- **Data tier**: Databases, only accessible from the application tier

An attacker who compromises the nginx server cannot directly access the database.

### Part E: Clean up

```bash
docker stop nginx api db backup
docker rm nginx api db backup
docker network rm public-net private-net data-net
```

### Verify Success

- [ ] Built a three-tier network topology
- [ ] Verified that isolation works (unauthorized connections fail)
- [ ] You can explain why this pattern improves security

---

## Task 4: Investigate Your Cloud Server (~20 minutes)

### Objective

Connect the networking concepts from this week to your real cloud server from Week 4.

> **Note:** If you no longer have your cloud server running, skip this task or do it with a classmate who does.

### Instructions

#### Part A: SSH into your server

```bash
ssh student@YOUR_SERVER_IP
```

#### Part B: Check listening ports

```bash
ss -tulpn
```

Answer these questions:
- Which ports are listening?
- Which ones do you recognize? (Hint: 22 = SSH, 80/443 = web)
- Are there any ports you don't recognize? Can you figure out what they are?

#### Part C: Check the server's IP configuration

```bash
ip addr
```

- What is the server's IP address?
- Is it the same IP you SSH'd to?
- Does the server have a private IP, a public IP, or both?

#### Part D: Check the firewall

```bash
sudo ufw status
```

- Which ports are allowed through the firewall?
- How do the firewall rules relate to the listening ports?
- What would happen if you removed the rule for port 22? (Don't actually do this!)

#### Part E: Check Docker's port mapping on the server

If you have Docker containers running:

```bash
docker ps
```

- Which ports are mapped?
- How does Docker's port mapping on the server relate to the firewall rules?

### Verify Success

- [ ] You can identify all listening ports on your server
- [ ] You understand the relationship between firewall rules and listening ports
- [ ] You can explain how traffic reaches your Docker container through the firewall and port mapping
- [ ] You can map the full path: Internet → Firewall → Docker port mapping → Container

---

## Task 5: Document Your Networking Knowledge (~15 minutes)

### Objective

Create a personal reference document that captures what you've learned and identifies gaps in your understanding.

### Instructions

Create a file called `networking-notes.md` with the following sections:

#### 1. Data Flow Diagram

Draw (in ASCII or describe) how data flows from a user's browser to your Docker container on a cloud server. Include:
- The user's computer and IP
- The internet (routers)
- Your server's firewall
- Docker port mapping
- The container

#### 2. TCP vs UDP in My Own Words

Explain the difference between TCP and UDP as if you're explaining to a friend who doesn't know programming. Use your own analogies.

#### 3. Key Connections to Earlier Weeks

List at least three things from Weeks 1-4 that now make more sense with your networking knowledge. For example:
- "Now I understand why `-p 8080:80` means..."
- "The reason `ufw allow 80` works is because..."
- "Docker Compose services can find each other because..."

#### 4. Things I Still Don't Fully Understand

List questions you still have. These might be answered in the coming weeks:
- Week 8: HTTP/HTTPS
- Week 9: DNS and certificates
- Week 10: Encryption
- Week 11: OSI model, IPv6, VPN

### Verify Success

- [ ] You created a networking-notes.md file
- [ ] Your data flow diagram covers the full path from browser to container
- [ ] You made connections to at least three things from earlier weeks
- [ ] You identified questions for future weeks

---

## Clean Up

Make sure all containers and networks from these tasks are removed:

```bash
docker stop sender receiver 2>/dev/null
docker rm sender receiver 2>/dev/null
docker network rm task-net 2>/dev/null
```
