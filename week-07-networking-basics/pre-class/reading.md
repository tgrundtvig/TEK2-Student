# Pre-class Reading: Introduction to Computer Networks

**Estimated reading time: 45-60 minutes**

---

## The Network You've Already Been Using

Think back to what you've done so far in this course:

- Week 1: `docker run -p 8080:80 nginx` — you opened `localhost:8080` in your browser and a web page appeared
- Week 2: In Docker Compose, your app connected to a database using the service name `db`
- Week 4: `ssh student@157.245.32.10` — you connected to a server on the other side of the internet
- Week 4: `ufw allow 80` — you opened a port in the firewall

You've been using computer networks the whole time. But what actually happened when you typed those commands?

- What does `-p 8080:80` really do?
- How did your SSH command reach a server in a data center?
- What is a "port" and why did you open port 80?
- How did the container named `db` know which IP address to use?

By the end of this week, you'll understand every part of these commands.

---

## What is a Network?

A computer network is simply two or more devices connected so they can communicate. That's it.

### An Analogy: The Postal System

Networks work remarkably like postal mail:

| Postal System | Computer Network |
|---------------|------------------|
| Your street address | IP address |
| Your name on the mailbox | Port number |
| The physical road | Network cable or Wi-Fi |
| A letter in an envelope | A packet of data |
| Post offices along the route | Routers |
| Your house's unique lot number | MAC address |

Just like a letter needs an address to reach the right house, and a name to reach the right person at that house, network data needs an **IP address** to reach the right computer and a **port number** to reach the right application on that computer.

### Types of Networks

```
┌──────────────────────────────────────────────────────────────────┐
│                          The Internet                             │
│   ┌──────────────┐       ┌──────────────┐       ┌────────────┐  │
│   │   Your LAN   │       │  School LAN  │       │ Data Center│  │
│   │ ┌──┐ ┌──┐    │       │ ┌──┐ ┌──┐    │       │ ┌────────┐ │  │
│   │ │PC│ │📱│    │       │ │PC│ │PC│    │       │ │ Server │ │  │
│   │ └──┘ └──┘    │       │ └──┘ └──┘    │       │ └────────┘ │  │
│   │    Router     │───────│    Router     │───────│   Router   │  │
│   └──────────────┘       └──────────────┘       └────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

- **LAN** (Local Area Network): Your home network, school network, office network. Devices connected to the same router.
- **WAN** (Wide Area Network): Connects LANs together across distances.
- **The Internet**: The global network connecting millions of LANs together.

When you SSH'd into your cloud server in Week 4, your data traveled from your LAN, through several routers on the internet, to the data center's LAN where your server lives.

---

## MAC Addresses — Physical Identity

Every network device (your laptop's Wi-Fi card, a server's Ethernet port, a Docker container's virtual interface) has a **MAC address**.

### What is a MAC Address?

MAC stands for **Media Access Control**. It's a hardware address assigned when the device is manufactured.

**Format:** Six pairs of hexadecimal characters, separated by colons:

```
02:42:ac:11:00:02
```

Think of it like the **serial number on your front door** — it uniquely identifies the physical device. Every network interface in the world has a different MAC address.

### Where MAC Addresses Are Used

MAC addresses are used for communication on the **local network only**. When your laptop sends data to your router, the data includes both your laptop's MAC address and the router's MAC address.

But MAC addresses don't travel beyond your local network. Once a router forwards your data to the internet, the original MAC address is replaced.

### ARP: Connecting MAC and IP

How does your computer know the MAC address of the router? It uses **ARP** (Address Resolution Protocol):

1. Your computer knows the router's IP address (e.g., `192.168.1.1`)
2. Your computer broadcasts: "Who has IP 192.168.1.1? Tell me your MAC address!"
3. The router replies: "That's me! My MAC is `aa:bb:cc:dd:ee:ff`"
4. Your computer caches this mapping for future use

You don't need to memorize how ARP works — just know that it's the bridge between IP addresses and MAC addresses on a local network.

---

## IP Addresses — Logical Identity

While MAC addresses identify the physical hardware, **IP addresses** identify devices logically on a network. IP addresses can change (unlike MAC addresses), and they are what makes routing across the internet possible.

### IPv4 Format

An IPv4 address is four numbers (0-255) separated by dots:

```
192.168.1.42
```

Each number is one **byte** (8 bits), so an IPv4 address is 32 bits total. This gives us about 4.3 billion possible addresses.

### Private vs Public IP Addresses

Not all IP addresses are visible on the internet. Some ranges are reserved for **private networks**:

| Range | Common Use |
|-------|------------|
| `10.0.0.0` — `10.255.255.255` | Large organizations, cloud providers |
| `172.16.0.0` — `172.31.255.255` | Docker uses this range! |
| `192.168.0.0` — `192.168.255.255` | Home and small office networks |

**Your home computer** probably has an IP like `192.168.1.42` — a private address. Your router has a **public address** (like `87.54.123.200`) that the internet can see. When you access a website, your router translates between your private address and its public address. This is called **NAT** (Network Address Translation).

**Docker containers** get addresses in the `172.17.0.x` range by default — also a private range. Now you know why!

### Special IP Addresses

| Address | Meaning |
|---------|---------|
| `127.0.0.1` | **localhost** — your own computer. When you open `localhost:8080`, you're connecting to yourself |
| `0.0.0.0` | "All interfaces" — when a server listens on `0.0.0.0`, it accepts connections from anywhere |

Remember when you typed `localhost:8080` in your browser in Week 1? Now you know: `localhost` = `127.0.0.1` = your own machine.

### IPv6: The Future (Brief Preview)

With 4.3 billion IPv4 addresses and billions of devices online, we're running out of addresses. **IPv6** solves this with 128-bit addresses:

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

IPv6 provides approximately 340 undecillion addresses (3.4 × 10³⁸) — enough for every grain of sand on Earth to have its own address. We'll dive deeper into IPv6 in Week 11.

---

## TCP vs UDP — Two Ways to Send Data

When data travels across a network, it needs a transport protocol. The two main options are **TCP** and **UDP**. They solve different problems.

### TCP: Reliable Delivery

**TCP** (Transmission Control Protocol) guarantees that data arrives correctly and in order.

**Analogy: Registered mail**
- You send a letter via registered mail
- The post office confirms delivery
- If the letter is lost, it's re-sent
- Letters arrive in the order they were sent

**How TCP works:**
1. **Connection first**: Before sending data, the two computers establish a connection (the "handshake")
2. **Ordered delivery**: Data packets are numbered, so they can be reassembled in order
3. **Error checking**: Each packet is verified; if one is lost or corrupted, it's re-sent
4. **Connection closed**: When done, both sides formally close the connection

**TCP is used for:**
- Web browsing (HTTP/HTTPS)
- SSH (your terminal connection)
- Email
- File transfers
- Database connections

Basically, anything where you need **every byte to arrive correctly**.

### UDP: Fast Delivery

**UDP** (User Datagram Protocol) sends data without guarantees.

**Analogy: Shouting across a room**
- You shout your message
- You don't know if anyone heard it
- No confirmation, no re-sending
- Fast and simple

**How UDP works:**
1. **No connection**: Just send the data
2. **No ordering**: Packets may arrive out of order
3. **No error recovery**: Lost packets are gone
4. **No overhead**: Much faster than TCP

**UDP is used for:**
- DNS lookups (translating domain names to IP addresses)
- Video streaming (a dropped frame is better than a delayed stream)
- Online gaming (fast updates matter more than perfect accuracy)
- Voice calls (slight data loss is better than lag)

### TCP vs UDP Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Required (handshake first) | None (just send) |
| Reliability | Guaranteed delivery | Best-effort |
| Ordering | Packets arrive in order | Packets may arrive out of order |
| Speed | Slower (overhead for reliability) | Faster (no overhead) |
| Use case | Web, SSH, email, databases | DNS, streaming, gaming, voice |

### The TCP Three-Way Handshake

Before TCP sends any data, the two computers perform a **three-way handshake** to establish the connection:

```
  Your Computer                    Server
       │                              │
       │── SYN ──────────────────────▶│  "I want to connect"
       │                              │
       │◀──────────────── SYN-ACK ────│  "OK, I'm ready too"
       │                              │
       │── ACK ──────────────────────▶│  "Great, let's go!"
       │                              │
       │◀──── Data flows both ways ──▶│
       │                              │
```

- **SYN**: "Synchronize" — your computer initiates the connection
- **SYN-ACK**: "Synchronize-Acknowledge" — the server agrees
- **ACK**: "Acknowledge" — your computer confirms, connection established

Every time you open a website, SSH into a server, or connect to a database, this handshake happens first. It takes milliseconds, but it's there.

In the class exercises, you'll see this handshake in Wireshark — the SYN, SYN-ACK, and ACK packets will be clearly visible.

---

## Ports — Which Application Gets the Data?

An IP address gets data to the right computer. But a computer runs many applications at once — your web browser, SSH, a database, Docker containers. How does the computer know which application should receive the incoming data?

That's what **ports** are for.

### What is a Port?

A port is a number between **0 and 65535** that identifies a specific application or service on a computer.

**Analogy:** If the IP address is the **building address**, the port is the **apartment number**. Mail arrives at the building (IP), then gets delivered to the right apartment (port).

### Well-Known Ports

Some port numbers are standardized:

| Port | Service | You've Used This In... |
|------|---------|------------------------|
| 22 | SSH | Week 4: `ssh student@server-ip` |
| 80 | HTTP (web) | Week 4: `ufw allow 80` |
| 443 | HTTPS (secure web) | Week 4: `ufw allow 443` |
| 3306 | MySQL | Week 2: Docker Compose with MySQL |
| 5432 | PostgreSQL | Databases |
| 8080 | Alt HTTP | Week 1: `docker run -p 8080:80 nginx` |

### The Docker Port Mapping Revelation

Now the big moment. Remember this command from Week 1?

```bash
docker run -d -p 8080:80 nginx
```

Here's what `-p 8080:80` actually means at the network level:

```
┌──────────────────────────────────────────────────────┐
│                   Your Computer                       │
│                                                       │
│  Browser ──▶ localhost:8080                           │
│                    │                                  │
│                    │ Docker forwards TCP traffic      │
│                    │ from host port 8080              │
│                    ▼ to container port 80             │
│            ┌──────────────┐                           │
│            │  Container   │                           │
│            │  nginx:80    │                           │
│            └──────────────┘                           │
└──────────────────────────────────────────────────────┘
```

- Your browser sends a TCP connection to `127.0.0.1:8080` (localhost, port 8080)
- Docker intercepts this and forwards it to the container's port 80
- nginx inside the container responds on port 80
- Docker sends the response back to your browser

**That's why you can't use the same host port twice!** If two containers both try to claim port 8080, only the first one succeeds — just like two people can't share the same apartment.

### Ephemeral Ports

When your browser connects to a server, it also uses a port on your side — a temporary **ephemeral port** (typically 49152-65535). So a full connection looks like:

```
Your Computer (192.168.1.42:54321) ◄──── TCP ────► Server (157.245.32.10:80)
```

The combination of IP address + port on each side uniquely identifies the connection. This is called a **socket**.

---

## Putting It All Together

Let's trace what actually happens when you visit your deployed website from Week 4:

```
Step 1: You type http://157.245.32.10 in your browser

Step 2: Your computer starts a TCP three-way handshake
        Your IP:54321 ──SYN──► 157.245.32.10:80

Step 3: The server's firewall checks: is port 80 allowed?
        Yes! (you ran ufw allow 80)

Step 4: Docker's port mapping forwards to the container
        Host port 80 → Container port 80

Step 5: nginx receives the request and sends back a web page
        The response travels back the same path

Step 6: Your browser displays the page
```

Every concept from this week was involved:
- **IP addresses**: Your computer's IP and the server's IP
- **TCP**: Reliable connection with three-way handshake
- **Ports**: Port 80 for HTTP, ephemeral port on your side
- **Firewall**: `ufw allow 80` opens the port
- **Docker port mapping**: `-p 80:80` forwards to the container

---

## Self-Check Questions

Before moving to the exercises, make sure you can answer these:

1. What is the difference between a MAC address and an IP address?
2. Why do home computers typically have IP addresses starting with `192.168`?
3. What does TCP's three-way handshake accomplish?
4. Which port does SSH use? Which port does HTTPS use?
5. Explain what `docker run -p 8080:80` does at the network level.

<details>
<summary>Click to reveal answers</summary>

1. A **MAC address** is a physical hardware address burned into the network card (used for local network communication). An **IP address** is a logical address assigned to a device on a network (used for routing across the internet).

2. The `192.168.x.x` range is reserved for **private networks**. Your home router assigns addresses from this range to devices on your LAN. These addresses are not visible on the internet.

3. The three-way handshake (**SYN → SYN-ACK → ACK**) establishes a reliable TCP connection before data is sent. Both computers agree that they're ready to communicate.

4. SSH uses port **22**. HTTPS uses port **443**.

5. `-p 8080:80` tells Docker to forward TCP traffic arriving on the host's port 8080 to the container's port 80. Your browser connects to `localhost:8080`, Docker intercepts it and sends it to nginx listening on port 80 inside the container.

</details>

---

## Further Reading (Optional)

If you want to learn more:
- [How Does the Internet Work?](https://cs.fyi/guide/how-does-internet-work) — A developer-friendly overview
- [TCP vs UDP](https://www.cloudflare.com/learning/ddos/glossary/tcp-ip/) — Cloudflare's explanation

---

**Next step**: Continue to [exercises.md](exercises.md) to install Wireshark and explore your first network.
