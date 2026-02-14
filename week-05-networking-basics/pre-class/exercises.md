# Pre-class Exercises: Wireshark Installation and Network Exploration

**Estimated time: 30-60 minutes**

Complete these exercises before class. If you get stuck, note where you had problems - we'll address them at the start of class.

> **Tip:** If something isn't working, don't spend more than 10 minutes on it. Note the error message and move on - we'll troubleshoot together in class.

---

## Exercise 1: Install Wireshark (~15 minutes)

Wireshark is a network analysis tool that lets you see individual packets traveling across your network — like a microscope for network traffic. We'll use it in class to observe the concepts from the reading in real time.

### Windows

1. Download Wireshark from [wireshark.org/download](https://www.wireshark.org/download.html)
2. Run the installer
3. When prompted, also install **Npcap** (required for packet capture on Windows)
4. Accept the default settings throughout
5. Open Wireshark from the Start menu

### macOS

1. Download Wireshark from [wireshark.org/download](https://www.wireshark.org/download.html)
   - Choose the correct version for your chip (Intel or Apple Silicon/Arm)
2. Open the downloaded `.dmg` file
3. Drag Wireshark to your Applications folder
4. Open Wireshark from Applications
5. If prompted about permissions, follow the on-screen instructions to allow packet capture

### Linux (Ubuntu/Debian)

```bash
# Install Wireshark
sudo apt update
sudo apt install wireshark

# When asked "Should non-superusers be able to capture packets?", select YES

# Add your user to the wireshark group
sudo usermod -aG wireshark $USER

# Log out and log back in for group changes to take effect
```

### Verify Installation

Open Wireshark. You should see a screen listing your network interfaces (Wi-Fi, Ethernet, etc.). Some may show small activity graphs next to them.

**Don't start a capture yet** — we'll do that in class. For now, just verify it opens.

### Self-check

- [ ] Wireshark is installed
- [ ] Wireshark opens and shows a list of network interfaces
- [ ] You can see at least one interface (Wi-Fi, Ethernet, or similar)

---

## Exercise 2: Find Your Network Identity (~10 minutes)

### Goal

Find your computer's IP address and MAC address, then compare them to a Docker container's addresses.

### Step 2.1: Find your host IP and MAC address

**Linux:**

```bash
ip addr
```

**macOS:**

```bash
ifconfig
```

**Windows (PowerShell or CMD):**

```cmd
ipconfig /all
```

Look for your active network interface (Wi-Fi or Ethernet). You should see:

- **IP address**: Something like `192.168.1.42` or `10.0.0.15`
- **MAC address** (may be called "physical address" or "ether"): Something like `a4:83:e7:2b:c1:d5`

> **Note:** You may see several interfaces listed. Look for the one that has an IP address in the `192.168.x.x` or `10.0.x.x` range — that's your active network connection.

Write these down — you'll need them for comparison:
- My IP address: _______________
- My MAC address: _______________

### Step 2.2: Compare with a Docker container

Run an Alpine Linux container and check its network identity:

```bash
docker run --rm alpine ip addr
```

**Expected output** (your numbers will be different):

```
1: lo: <LOOPBACK,UP,LOWER_UP> ...
    inet 127.0.0.1/8 scope host lo
2: eth0@if15: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
```

Notice:
- The container's IP address is `172.17.0.2` — in Docker's private range (remember from the reading?)
- The container's MAC address is `02:42:ac:11:00:02` — a virtual MAC address assigned by Docker
- These are **different** from your host machine's addresses

### Self-check

- [ ] You found your host IP address
- [ ] You found your host MAC address
- [ ] You see the container has a different IP (172.17.x.x range)
- [ ] You understand why the container has a different address

---

## Exercise 3: Explore Docker Networks (~15 minutes)

### Goal

Discover Docker's built-in networking and understand how containers connect.

### Step 3.1: List Docker networks

```bash
docker network ls
```

**Expected output:**

```
NETWORK ID     NAME      DRIVER    SCOPE
a1b2c3d4e5f6   bridge    bridge    local
d7e8f9a0b1c2   host      host      local
f3g4h5i6j7k8   none      null      local
```

Docker creates three networks by default:
- **bridge**: The default network for containers (this is where `172.17.0.x` addresses come from)
- **host**: Shares the host's network directly (no isolation)
- **none**: No networking at all

### Step 3.2: Inspect the bridge network

```bash
docker network inspect bridge
```

This shows detailed information. Look for the `"Subnet"` field:

```json
"Config": [
    {
        "Subnet": "172.17.0.0/16",
        "Gateway": "172.17.0.1"
    }
]
```

This means Docker assigns container IPs from the `172.17.x.x` range, and the gateway (the "door" to the host network) is `172.17.0.1`.

### Step 3.3: See a container appear on the network

Start a container in the background:

```bash
docker run -d --name network-test alpine sleep 3600
```

Now inspect the bridge network again:

```bash
docker network inspect bridge
```

Scroll to the `"Containers"` section — you should see your `network-test` container listed with its IP address.

### Step 3.4: Find a container's IP using docker inspect

```bash
docker inspect network-test | grep IPAddress
```

**Expected output:**

```
"IPAddress": "172.17.0.2",
```

This is another way to find a container's IP address.

### Step 3.5: Clean up

```bash
docker stop network-test && docker rm network-test
```

### Self-check

- [ ] You can list Docker networks
- [ ] You found the bridge network's subnet (172.17.0.0/16)
- [ ] You saw a container appear on the bridge network
- [ ] You can find a container's IP address with `docker inspect`

---

## Exercise 4: Ping Between Containers (~15 minutes)

### Goal

Create a custom Docker network and see how containers communicate on it.

### Step 4.1: Create a custom network

```bash
docker network create week7-net
```

Verify it was created:

```bash
docker network ls
```

You should see `week7-net` in the list.

### Step 4.2: Start two containers on the same network

```bash
docker run -d --name server-a --network week7-net alpine sleep 3600
docker run -d --name server-b --network week7-net alpine sleep 3600
```

### Step 4.3: Ping by container name

Open a shell in server-a and ping server-b by name:

```bash
docker exec server-a ping -c 3 server-b
```

**Expected output:**

```
PING server-b (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.123 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.089 ms
64 bytes from 172.18.0.3: seq=2 ttl=64 time=0.094 ms
```

It works! Docker's built-in DNS resolved the name `server-b` to its IP address.

> **This is why Docker Compose services can find each other by name!** In Week 2, when your app connected to `db`, Docker resolved that name to the database container's IP address — exactly like what you just saw.

### Step 4.4: Try pinging across networks (it fails)

Start a container on the default bridge network (NOT week7-net):

```bash
docker run -d --name outsider alpine sleep 3600
```

Try to ping from outsider to server-a:

```bash
docker exec outsider ping -c 2 server-a
```

**Expected:** This fails! The `outsider` container is on the default bridge network, not on `week7-net`. Docker networks provide **isolation** — containers can only communicate with other containers on the same network.

### Step 4.5: Clean up

```bash
docker stop server-a server-b outsider
docker rm server-a server-b outsider
docker network rm week7-net
```

### Self-check

- [ ] You created a custom Docker network
- [ ] Containers on the same network can ping each other by name
- [ ] Containers on different networks cannot communicate
- [ ] You understand why Docker Compose services can use service names

---

## Troubleshooting

### "Wireshark: no interfaces found" (Windows)

- Make sure Npcap was installed during Wireshark installation
- Try running Wireshark as administrator (right-click → Run as administrator)
- Reinstall Wireshark and ensure the Npcap checkbox is selected

### "wireshark: permission denied" (Linux)

- Make sure you added your user to the wireshark group: `sudo usermod -aG wireshark $USER`
- Log out and log back in for the group change to take effect
- Alternatively, run with sudo: `sudo wireshark` (not recommended for regular use)

### "docker network create" fails

- Make sure Docker is running: `docker ps` should work without errors
- On Linux, make sure you're in the docker group or use `sudo`

### Container ping fails with "bad address"

- Make sure both containers are on the same custom network (not the default bridge)
- The default bridge network does NOT support DNS name resolution between containers
- Custom networks (created with `docker network create`) DO support name resolution

---

## Before Class Checklist

**Tools:**

- [ ] Wireshark is installed and opens
- [ ] Docker is running

**Knowledge:**

- [ ] You know your host IP address and MAC address
- [ ] You've explored Docker networks (bridge, host, none)
- [ ] You can find a container's IP address
- [ ] You've seen containers ping each other on a custom network
- [ ] You've seen that containers on different networks are isolated

**If you're stuck on anything, don't worry!** Note down the specific error or step, and we'll troubleshoot together at the start of class.

---

**Next step**: Bring your laptop to class with Docker running and Wireshark installed. We'll use both extensively!
