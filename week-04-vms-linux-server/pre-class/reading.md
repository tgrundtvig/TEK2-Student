# Pre-class Reading: VMs, SSH, and Cloud Servers

**Estimated reading time: 45-60 minutes**

This week you'll deploy your application to a real server in the cloud. By the end, your code will automatically deploy when you push to GitHub.

---

## From Containers to Servers

In Week 2, you built Docker images. In Week 3, you pushed them to GitHub Container Registry. But where do these containers actually run in production?

**Locally** (what you've done):
```
Your laptop
    └── Docker
        └── Your container
```

**In production** (what you'll do this week):
```
Cloud Provider (Azure/DigitalOcean/Hetzner/Google Cloud/Oracle Cloud)
    └── Virtual Machine (Linux server)
        └── Docker
            └── Your container
```

The container is the same - but now it runs on a server that's accessible from the internet.

---

## What is a Virtual Machine?

A **Virtual Machine (VM)** is a software-based computer that runs on physical hardware. Cloud providers have massive data centers with thousands of physical servers. When you create a VM, you get a slice of one of these servers.

```
Physical Server in Data Center
├── VM 1 (Your Linux server)
├── VM 2 (Someone else's Windows server)
├── VM 3 (Another Linux server)
└── ... (many more VMs)
```

### VMs vs Containers

| Aspect | Virtual Machine | Container |
|--------|-----------------|-----------|
| **What it is** | Full operating system | Isolated process |
| **Boot time** | Minutes | Seconds |
| **Size** | Gigabytes | Megabytes |
| **Isolation** | Complete (separate kernel) | Process-level |
| **Use case** | Running multiple services | Running single applications |

**Key insight:** You run Docker (and containers) *inside* a VM. The VM provides the Linux server; Docker runs your applications.

```
Cloud VM (Ubuntu)
├── SSH server (for remote access)
├── Docker
│   ├── Container A (your web app)
│   └── Container B (your database)
└── Other services
```

---

## Cloud Service Models (XaaS)

Cloud computing offers different levels of abstraction. The "as a Service" models describe what the provider manages vs. what you manage.

### The Cloud Service Pyramid

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   MORE ABSTRACTION (less control, less responsibility)             │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                         SaaS                                 │  │
│   │              Software as a Service                           │  │
│   │         Gmail, Slack, Salesforce, GitHub                     │  │
│   │      You use the software; provider handles everything       │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                         PaaS                                 │  │
│   │              Platform as a Service                           │  │
│   │         Heroku, Railway, Coolify, Azure App Service          │  │
│   │      You deploy code; provider handles infrastructure        │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                         IaaS                                 │  │
│   │           Infrastructure as a Service                        │  │
│   │  Azure VMs, Digital Ocean, Hetzner, Google Cloud, Oracle Cloud│  │
│   │      You get VMs; provider handles physical hardware         │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│   ┌─────────────────────────────────────────────────────────────┐  │
│   │                     On-Premises                              │  │
│   │              Your own data center                            │  │
│   │      You manage everything: hardware, power, cooling         │  │
│   └─────────────────────────────────────────────────────────────┘  │
│                                                                     │
│   LESS ABSTRACTION (more control, more responsibility)             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### What Each Model Means

| Model | You Manage | Provider Manages | Examples |
|-------|------------|------------------|----------|
| **On-Premises** | Everything | Nothing | Your own server room |
| **IaaS** | OS, runtime, apps, data | Hardware, network, VMs | Azure VMs, EC2, Hetzner, Google Cloud, Oracle Cloud |
| **PaaS** | Apps and data | Everything else | Heroku, Railway, Coolify |
| **SaaS** | Just your data | Everything | Gmail, Slack, GitHub |

### Infrastructure as a Service (IaaS)

**What you get:** A virtual machine - essentially a blank Linux (or Windows) server.

**You're responsible for:**
- Operating system configuration
- Installing software (Docker, databases, etc.)
- Security updates and patches
- Firewall configuration
- Monitoring and backups
- Scaling (adding more servers)

**Examples:** Azure Virtual Machines, Digital Ocean Droplets, Hetzner Cloud, Google Cloud Compute Engine, Oracle Cloud Compute, AWS EC2

**Pros:**
- Full control over the environment
- Learn valuable Linux administration skills
- Often more cost-effective at scale
- Can run any software

**Cons:**
- More responsibility
- Steeper learning curve
- Security is on you
- More time spent on operations

### Platform as a Service (PaaS)

**What you get:** A platform that runs your code or containers automatically.

**You're responsible for:**
- Your application code
- Your data
- Application configuration

**Provider handles:**
- Server management
- Operating system updates
- Scaling
- Load balancing
- SSL certificates
- Monitoring infrastructure

**Examples:** Heroku, Railway, Render, Coolify, Azure App Service, Google App Engine

**Pros:**
- Much simpler deployment
- Focus on code, not infrastructure
- Automatic scaling
- Built-in monitoring
- Faster time to production

**Cons:**
- Less control
- Can be more expensive
- Vendor lock-in risk
- Limited customization

### Software as a Service (SaaS)

**What you get:** Ready-to-use software accessible via browser or API.

**You're responsible for:**
- Your data
- How you use the software

**Provider handles:**
- Everything else

**Examples:** Gmail, Slack, Salesforce, GitHub, Jira, Microsoft 365

**Pros:**
- No installation or maintenance
- Accessible anywhere
- Always up to date

**Cons:**
- Least control
- Data lives on someone else's servers
- Subscription costs add up

### Other "as a Service" Models

You may also encounter:

| Model | What It Means | Examples |
|-------|---------------|----------|
| **FaaS** | Functions as a Service (serverless) | AWS Lambda, Azure Functions |
| **CaaS** | Containers as a Service | AWS ECS, Azure Container Apps |
| **DBaaS** | Database as a Service | AWS RDS, MongoDB Atlas |
| **BaaS** | Backend as a Service | Firebase, Supabase |

### What We'll Learn

This course focuses on **IaaS** (VMs) because:

1. **Foundational skills** - Understanding servers helps you understand all other models
2. **Industry expectation** - DevOps and backend roles require this knowledge
3. **Flexibility** - You can run anything on a VM
4. **Cost-effective** - Often cheaper for consistent workloads
5. **Debugging ability** - When PaaS fails, you need to understand what's underneath

**Coolify** is available as a **PaaS backup** option if you have trouble with VM setup. It's a valid deployment path, but you'll learn fewer infrastructure skills.

### Making the Right Choice

In real projects, the choice depends on:

| Factor | Favors IaaS | Favors PaaS |
|--------|-------------|-------------|
| Team expertise | Have ops/DevOps skills | Developers only |
| Time to market | Can invest in setup | Need to ship fast |
| Customization | Need specific configs | Standard setups work |
| Scale | Predictable, large scale | Variable, smaller scale |
| Budget | Have ops time | Have money, not time |
| Compliance | Need full control | Standard compliance OK |

**Rule of thumb:** Start with PaaS for prototypes and MVPs. Move to IaaS when you need more control or scale.

---

## SSH: Secure Shell

**SSH** is how you connect to remote servers securely. Instead of physically sitting at the server, you connect over the internet.

```
┌────────────────┐         SSH Tunnel        ┌────────────────┐
│  Your Laptop   │  ────────────────────────▶│  Cloud Server  │
│                │   Encrypted connection    │                │
│  Terminal      │                           │  Linux         │
│  $ ssh user@ip │                           │  Command line  │
└────────────────┘                           └────────────────┘
```

### Why SSH Keys Instead of Passwords?

Passwords are vulnerable:
- Can be guessed or brute-forced
- Can be phished
- Users pick weak passwords

SSH keys are much more secure:
- Mathematically generated key pair
- Private key never leaves your computer
- Public key goes on the server
- Practically impossible to crack

### How SSH Keys Work

```
Your Computer                     Server
┌─────────────────────────┐      ┌─────────────────────────┐
│ Private Key             │      │ authorized_keys file    │
│ (id_ed25519)            │      │                         │
│ KEEP THIS SECRET!       │      │ Contains your           │
│                         │      │ Public Key              │
│ Public Key              │ ──▶  │ (id_ed25519.pub)        │
│ (id_ed25519.pub)        │      │                         │
│ Safe to share           │      │ Server trusts anyone    │
│                         │      │ with matching private   │
└─────────────────────────┘      └─────────────────────────┘

Connection:
1. You: "I want to connect"
2. Server: "Prove you have the private key for this public key"
3. You: (cryptographic proof using private key)
4. Server: "Verified! Welcome."
```

### SSH Key Types

- **RSA** - Older, widely compatible, use 4096 bits
- **Ed25519** - Newer, faster, more secure, recommended

We'll use **Ed25519** in this course.

---

## Linux Server Basics

When you connect to your server, you'll use the command line. Here are the essential concepts:

### The Shell

The shell interprets your commands. Most servers use **bash** (Bourne Again Shell).

```bash
user@server:~$
│    │      │└─ Prompt ($ for user, # for root)
│    │      └── Current directory (~ = home)
│    └────────── Server hostname
└───────────────── Username
```

### Root and sudo

- **Root** is the superuser with full system access
- You typically log in as a regular user
- **sudo** lets you run commands as root

```bash
# This fails - regular users can't install packages
apt install docker

# This works - sudo runs as root
sudo apt install docker
```

### Package Management with apt

Ubuntu uses `apt` (Advanced Package Tool) to install software:

```bash
# Update list of available packages
sudo apt update

# Install a package
sudo apt install nginx

# Upgrade all installed packages
sudo apt upgrade

# Remove a package
sudo apt remove nginx
```

### Service Management with systemctl

Linux uses **systemd** to manage services (programs that run in the background):

```bash
# Check if a service is running
sudo systemctl status docker

# Start a service
sudo systemctl start docker

# Stop a service
sudo systemctl stop docker

# Enable service to start at boot
sudo systemctl enable docker
```

### The Firewall with ufw

**ufw** (Uncomplicated Firewall) controls network access:

```bash
# Check firewall status
sudo ufw status

# Allow SSH (always do this BEFORE enabling!)
sudo ufw allow 22

# Allow HTTP and HTTPS
sudo ufw allow 80
sudo ufw allow 443

# Enable the firewall
sudo ufw enable
```

**Warning:** If you enable the firewall without allowing SSH (port 22), you'll lock yourself out!

---

## Cloud Providers Overview

### Azure (Microsoft)

- **Free tier:** $200 credit for 30 days, 12 months of free services
- **Good for:** Enterprise environments, Windows integration
- **Interface:** Complex but powerful Azure Portal
- **Student benefit:** May have academic credits available

### Digital Ocean

- **Pricing:** Starts at $4/month for basic droplets
- **Good for:** Simplicity, excellent documentation
- **Interface:** Clean, user-friendly
- **Student benefit:** GitHub Student Pack includes credits

### Hetzner

- **Pricing:** Starts at ~€4/month
- **Good for:** European servers, best price/performance
- **Interface:** Straightforward
- **Note:** Servers in Germany and Finland

### Google Cloud (GCP)

- **Free tier:** $300 credit for 90 days, plus Always Free e2-micro VM
- **Good for:** Enterprise environments, AI/ML services
- **Interface:** Feature-rich Google Cloud Console
- **Student benefit:** Education credits available through participating institutions
- **Note:** Always Free VM limited to US regions only

### Oracle Cloud (OCI)

- **Free tier:** $300 credit for 30 days, plus **very generous Always Free** instances
- **Good for:** Best permanent free tier (up to 4 ARM CPUs + 24 GB RAM — forever free)
- **Interface:** Clean Oracle Cloud Console
- **Student benefit:** Oracle Academy members get $300 credits for 1 year, no credit card required
- **Note:** ARM instances may have limited availability; choose Home Region carefully (permanent)

### Coolify (Self-Hosted PaaS)

- **What it is:** Open-source PaaS platform
- **Access:** Available on request from instructor
- **Good for:** Students with billing issues or who want simpler deployment
- **Trade-off:** Less Linux administration experience

---

## What is Coolify?

Coolify is an open-source alternative to platforms like Heroku. Instead of managing a VM directly, you deploy through a web interface.

```
Traditional VM Deployment          Coolify Deployment
┌─────────────────────────┐       ┌─────────────────────────┐
│ You SSH into server     │       │ You push to Git         │
│ You install Docker      │       │ Coolify detects push    │
│ You pull images         │       │ Coolify builds/deploys  │
│ You manage containers   │       │ Coolify manages server  │
│ You configure firewall  │       │ Automatic HTTPS         │
└─────────────────────────┘       └─────────────────────────┘
```

### When to Use Coolify

Use Coolify if you:
- Have trouble setting up cloud provider billing
- Want to focus on deployment concepts, not server admin
- Need a backup plan if VM setup fails
- Prefer a simpler deployment path

**Note:** Coolify is a backup option. The VM path teaches valuable skills.

---

## The Big Picture

Here's what your complete setup will look like:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Complete CI/CD Pipeline                          │
│                                                                         │
│  LOCAL                    GITHUB                    CLOUD SERVER        │
│  ┌──────────────┐        ┌──────────────┐         ┌──────────────┐     │
│  │              │  push  │              │  deploy │              │     │
│  │  Your Code   │───────▶│  Repository  │────────▶│  Your App    │     │
│  │              │        │              │         │              │     │
│  └──────────────┘        └──────────────┘         └──────────────┘     │
│                                 │                        │              │
│                                 ▼                        │              │
│                          ┌──────────────┐               │              │
│                          │   Actions    │               │              │
│                          │   ┌──────┐   │               │              │
│                          │   │Build │   │               │              │
│                          │   │Test  │   │               │              │
│                          │   │Push  │   │               │              │
│                          │   └──────┘   │               │              │
│                          └──────────────┘               │              │
│                                 │                        │              │
│                                 ▼                        │              │
│                          ┌──────────────┐               │              │
│                          │    GHCR      │──────────────▶│              │
│                          │ Docker Images│  pull         │              │
│                          └──────────────┘               │              │
│                                                          │              │
│                                                          ▼              │
│                                                    Users access          │
│                                                    your app via          │
│                                                    http://server-ip      │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## What You'll Set Up This Week

**Before class (pre-class exercises):**
1. Create an SSH key (if you don't have one)
2. Choose a cloud provider
3. Create a VM following the provider tutorial
4. Verify you can SSH into your server

**During class:**
1. Update and secure your server
2. Configure the firewall
3. Install Docker
4. Deploy your application manually
5. Set up GitHub Actions for automatic deployment

**After class (advanced tasks):**
1. Configure a custom domain
2. Set up HTTPS with Let's Encrypt
3. Add monitoring

---

## Security Considerations

Your server will be on the public internet. Security basics:

1. **Use SSH keys** - Never password authentication
2. **Keep software updated** - Run `apt update && apt upgrade` regularly
3. **Use a firewall** - Only open necessary ports
4. **Don't run as root** - Use sudo when needed
5. **Monitor access** - Check logs for suspicious activity

We'll cover basic server hardening in class.

---

## Self-Check Questions

Before moving to the exercises, make sure you can answer:

1. What's the difference between a VM and a container?

2. Why do we use SSH keys instead of passwords?

3. What command updates the package list on Ubuntu?

4. What command checks if a service is running?

5. What must you do BEFORE enabling the firewall?

<details>
<summary>Click to reveal answers</summary>

1. A **VM** is a full operating system with its own kernel, while a **container** is an isolated process sharing the host kernel. VMs take minutes to start and are gigabytes in size; containers start in seconds and are megabytes.

2. SSH keys are **cryptographically secure** and can't be guessed, phished, or brute-forced like passwords. The private key never leaves your computer.

3. `sudo apt update` - This refreshes the list of available packages from repositories.

4. `sudo systemctl status servicename` - Shows whether the service is active, its recent logs, and resource usage.

5. **Allow SSH (port 22)** with `sudo ufw allow 22`. If you enable the firewall without allowing SSH, you'll lock yourself out of the server!

</details>

---

## Further Reading (Optional)

- [SSH Essentials](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys) - Digital Ocean guide
- [Linux Command Line Basics](https://ubuntu.com/tutorials/command-line-for-beginners) - Ubuntu tutorial
- [UFW Essentials](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands) - Firewall guide

---

**Next step**: Continue to [exercises.md](exercises.md) to create your cloud server.
