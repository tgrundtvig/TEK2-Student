# Week 7 Quick Reference Card

Print this page or keep it open while working!

---

## Network Commands

| Command | What It Does | Example |
|---------|-------------|---------|
| `ip addr` | Show IP and MAC addresses (Linux) | `ip addr` |
| `ifconfig` | Show IP and MAC addresses (macOS) | `ifconfig` |
| `ipconfig` | Show IP addresses (Windows) | `ipconfig /all` |
| `ping` | Test if a host is reachable | `ping -c 3 google.com` |
| `traceroute` | Show the route packets take | `traceroute google.com` |
| `ss -tulpn` | Show listening ports (Linux) | `ss -tulpn` |
| `netstat -an` | Show listening ports (macOS/Windows) | `netstat -an \| grep LISTEN` |
| `nc -l -p PORT` | Listen for TCP connections | `nc -l -p 9999` |
| `nc HOST PORT` | Connect to a TCP port | `nc server-a 9999` |
| `nc -u` | Use UDP instead of TCP | `nc -u -l -p 9999` |

---

## Docker Network Commands

| Command | What It Does | Example |
|---------|-------------|---------|
| `docker network ls` | List all Docker networks | `docker network ls` |
| `docker network create NAME` | Create a custom network | `docker network create my-net` |
| `docker network inspect NAME` | Show network details | `docker network inspect bridge` |
| `docker network connect NET CTR` | Add container to a network | `docker network connect my-net api` |
| `docker network rm NAME` | Remove a network | `docker network rm my-net` |
| `docker inspect CTR \| grep IP` | Find a container's IP | `docker inspect web \| grep IPAddress` |

### Starting Containers on a Specific Network

```bash
docker run -d --name web --network my-net alpine sleep 3600
```

---

## Well-Known Ports

| Port | Service | Protocol | You Used This In... |
|------|---------|----------|---------------------|
| 22 | SSH | TCP | Week 4: `ssh user@server` |
| 80 | HTTP | TCP | Week 4: `ufw allow 80` |
| 443 | HTTPS | TCP | Week 4: `ufw allow 443` |
| 3306 | MySQL | TCP | Week 2: Docker Compose |
| 5432 | PostgreSQL | TCP | Databases |
| 8080 | Alt HTTP | TCP | Week 1: `-p 8080:80` |

Remember: Ports 0-1023 are "well-known" (reserved for standard services). Ports 1024-65535 can be used by anyone.

---

## Private IP Address Ranges

| Range | CIDR | Common Use |
|-------|------|------------|
| `10.0.0.0` — `10.255.255.255` | `10.0.0.0/8` | Cloud providers, large organizations |
| `172.16.0.0` — `172.31.255.255` | `172.16.0.0/12` | Docker containers (default: `172.17.0.x`) |
| `192.168.0.0` — `192.168.255.255` | `192.168.0.0/16` | Home and small office networks |

Special addresses:
- `127.0.0.1` = **localhost** (your own machine)
- `0.0.0.0` = "all interfaces" (when a server listens on this, it accepts connections from anywhere)

---

## TCP vs UDP

| | TCP | UDP |
|--|-----|-----|
| **Connection** | Yes (three-way handshake) | No |
| **Reliability** | Guaranteed delivery | Best effort |
| **Ordering** | In order | May arrive out of order |
| **Speed** | Slower (overhead) | Faster (no overhead) |
| **Used for** | HTTP, SSH, email, databases | DNS, streaming, gaming, voice |

### TCP Three-Way Handshake

```
Client ── SYN ──────────► Server     "I want to connect"
Client ◄── SYN-ACK ────── Server     "OK, I'm ready too"
Client ── ACK ──────────► Server     "Let's go!"
```

---

## Docker Port Mapping Explained

```bash
docker run -p 8080:80 nginx
#             ▲     ▲
#             │     └── Container port (nginx listens here)
#             └──────── Host port (you connect here)
```

```
Browser → localhost:8080 → Docker → container:80 → nginx
```

- Only one container can use each host port
- The container's internal port can be the same across containers
- Use `ss -tulpn` to see which host ports are in use

---

## Wireshark Basic Filters

| Filter | What It Shows |
|--------|--------------|
| `icmp` | Ping packets (echo request/reply) |
| `tcp` | All TCP traffic |
| `udp` | All UDP traffic |
| `arp` | ARP requests (IP → MAC resolution) |
| `tcp.port==80` | TCP traffic on port 80 (HTTP) |
| `tcp.port==443` | TCP traffic on port 443 (HTTPS) |
| `tcp.flags.syn==1` | TCP SYN packets (connection starts) |
| `tcp.flags.fin==1` | TCP FIN packets (connection ends) |
| `ip.addr==X.X.X.X` | Traffic to/from a specific IP |
| `tcp.stream eq N` | All packets in a specific TCP connection |

### Wireshark Interface Selection

| Platform | Select This Interface |
|----------|----------------------|
| Linux | `eth0`, `wlan0`, or `enp0s3` |
| macOS | `en0` (Wi-Fi) or `en1` (Ethernet) |
| Windows | `Wi-Fi` or `Ethernet` |

---

## Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| "port already in use" | Another process is using that port. Use `ss -tulpn` to find it. |
| Container can't ping another container | Check they're on the same Docker network. |
| Ping by name fails with "bad address" | Use a custom network (`docker network create`). Default bridge doesn't support DNS. |
| Wireshark shows no interfaces | (Linux) Add user to wireshark group. (Windows) Reinstall with Npcap. |
| Wireshark captures too much traffic | Use display filters to narrow down (e.g., `icmp`, `tcp.port==80`). |
| `nc` command not found in Alpine | Install it: `apk add --no-cache netcat-openbsd` |
| `traceroute` not found in Alpine | Install it: `apk add --no-cache traceroute` |

---

## Remember

1. **IP address** = which computer (like a building address)
2. **Port** = which application (like an apartment number)
3. **MAC address** = physical hardware identity (used only on local network)
4. **TCP** = reliable connection (handshake, guaranteed delivery)
5. **UDP** = fast, no guarantees (just send and hope)
6. **Docker networks** provide isolation — containers on different networks can't communicate
7. **`-p 8080:80`** creates a real TCP port on your host that forwards to the container
8. **Wireshark** lets you see all of this happening in real captured packets
