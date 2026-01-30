# Pre-class Exercises: Create Your Cloud Server

**Estimated time: 1-2 hours**

In these exercises, you'll create a cloud server and verify SSH access. Complete these **before class** so we can focus on server administration and deployment during class time.

---

## Exercise 1: Prepare Your SSH Key (~10 minutes)

### Check for Existing Key

First, check if you already have an SSH key:

```bash
ls -la ~/.ssh/
```

If you see `id_ed25519` and `id_ed25519.pub` (or `id_rsa` and `id_rsa.pub`), you already have a key. Skip to "View Your Public Key."

### Generate a New Key (if needed)

If you don't have a key, create one:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

When prompted:
- **File location**: Press Enter to accept default (`~/.ssh/id_ed25519`)
- **Passphrase**: Enter a passphrase (recommended) or press Enter for none

### View Your Public Key

Display your public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

You'll see something like:
```
ssh-ed25519 AAAAC3NzaC1lZDI1... your_email@example.com
```

**Copy this entire line** - you'll need it when creating your server.

### Verify SSH Agent

Make sure your key is loaded:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### Self-check
- [ ] I have an SSH key pair (`id_ed25519` and `id_ed25519.pub`)
- [ ] I can view my public key with `cat ~/.ssh/id_ed25519.pub`
- [ ] I have copied my public key for later use

---

## Exercise 2: Choose Your Provider (~5 minutes)

Select **one** provider based on your situation:

| If you... | Choose | Tutorial |
|-----------|--------|----------|
| Have Azure student credits | Azure | [tutorials/azure.md](../tutorials/azure.md) |
| Want simplicity with good docs | Digital Ocean | [tutorials/digitalocean.md](../tutorials/digitalocean.md) |
| Want the cheapest option | Hetzner | [tutorials/hetzner.md](../tutorials/hetzner.md) |
| Have billing issues or want simpler path | Coolify | [tutorials/coolify.md](../tutorials/coolify.md) |

### Provider Comparison

**Azure:**
- Pros: Free credits, enterprise standard
- Cons: Complex interface
- Cost: Free with credits, otherwise ~$5-15/month

**Digital Ocean:**
- Pros: Simple interface, excellent tutorials
- Cons: No free tier (but GitHub Student Pack has credits)
- Cost: $4-6/month

**Hetzner:**
- Pros: Cheapest, good performance
- Cons: European servers only, less documentation
- Cost: ~€4/month

**Coolify (backup option):**
- Pros: Free, simple web interface
- Cons: Less Linux experience
- Access: Request from instructor

### Self-check
- [ ] I have chosen a provider
- [ ] I understand the approximate cost
- [ ] I have the tutorial ready to follow

---

## Exercise 3: Create Your Server (~30-60 minutes)

Follow your chosen provider's tutorial:

- [Azure Tutorial](../tutorials/azure.md)
- [Digital Ocean Tutorial](../tutorials/digitalocean.md)
- [Hetzner Tutorial](../tutorials/hetzner.md)
- [Coolify Tutorial](../tutorials/coolify.md)

### Server Specifications

When creating your server, use these settings:

| Setting | Value |
|---------|-------|
| **Operating System** | Ubuntu 22.04 LTS or 24.04 LTS |
| **Size** | Smallest/cheapest option (1 vCPU, 1GB RAM is fine) |
| **Region** | Closest to you (for lower latency) |
| **Authentication** | SSH Key (paste your public key) |

### What to Note Down

After creating your server, record these details:

```
Server IP Address: _______________
Username: _______________  (usually "root" or your chosen username)
Provider: _______________
```

### Self-check
- [ ] Server is created and running
- [ ] I have noted the IP address
- [ ] I have noted the username
- [ ] SSH key was added during creation

---

## Exercise 4: Verify SSH Access (~10 minutes)

### Connect to Your Server

Open a terminal and connect:

```bash
ssh username@your-server-ip
```

For example:
```bash
ssh root@64.23.145.101
```

### First Connection Warning

On first connection, you'll see:
```
The authenticity of host '64.23.145.101' can't be established.
ED25519 key fingerprint is SHA256:xxxxx...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes` and press Enter. This adds the server to your known hosts.

### Successful Connection

You should see something like:
```
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

root@myserver:~#
```

### Verify Basic Information

Run these commands on the server:

```bash
# Check hostname
hostname

# Check Ubuntu version
lsb_release -a

# Check disk space
df -h

# Check memory
free -h

# Check who you are
whoami
```

### Exit the Session

When done exploring:

```bash
exit
```

You're back on your local machine.

### Self-check
- [ ] I can connect via SSH without entering a password
- [ ] I see the Ubuntu welcome message
- [ ] I can run basic commands on the server
- [ ] I can exit and reconnect

---

## Exercise 5: Create a Non-Root User (Recommended) (~10 minutes)

If your provider gave you root access, it's best practice to create a regular user:

### Connect as root

```bash
ssh root@your-server-ip
```

### Create a new user

```bash
adduser yourusername
```

Enter a password when prompted. You can skip the other fields (press Enter).

### Grant sudo privileges

```bash
usermod -aG sudo yourusername
```

### Copy SSH key to new user

```bash
# Create .ssh directory
mkdir -p /home/yourusername/.ssh

# Copy authorized_keys
cp /root/.ssh/authorized_keys /home/yourusername/.ssh/

# Set correct ownership
chown -R yourusername:yourusername /home/yourusername/.ssh

# Set correct permissions
chmod 700 /home/yourusername/.ssh
chmod 600 /home/yourusername/.ssh/authorized_keys
```

### Test the new user

Exit and reconnect as the new user:

```bash
exit
ssh yourusername@your-server-ip
```

Verify sudo works:
```bash
sudo whoami
```

Should output: `root`

### Self-check
- [ ] New user created
- [ ] User can use sudo
- [ ] Can SSH as new user (without password)

---

## Troubleshooting

### "Permission denied (publickey)"

Your SSH key isn't recognized. Check:

1. **Key is correct**: Verify your public key was added to the server
   ```bash
   # On server, check authorized_keys
   cat ~/.ssh/authorized_keys
   ```

2. **Key permissions**: Fix permissions on your local machine
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/id_ed25519
   chmod 644 ~/.ssh/id_ed25519.pub
   ```

3. **Right key is used**: Specify the key explicitly
   ```bash
   ssh -i ~/.ssh/id_ed25519 user@server-ip
   ```

### "Connection refused"

The server isn't accepting SSH connections:

1. **Server is running**: Check in your provider's dashboard
2. **SSH service is running**: (This would require console access from provider)
3. **Firewall blocking**: Some providers have a separate firewall setting

### "Connection timed out"

Can't reach the server at all:

1. **IP address correct**: Double-check the IP
2. **Server is running**: Check provider dashboard
3. **Network issue**: Try from a different network

### "Host key verification failed"

The server's identity changed (or you're connecting to a different server):

```bash
# Remove old key from known_hosts
ssh-keygen -R your-server-ip

# Try connecting again
ssh user@your-server-ip
```

---

## Before Class Checklist

Ensure you have:

- [ ] SSH key generated and available
- [ ] Cloud server created and running
- [ ] Server IP address noted
- [ ] Username noted
- [ ] Successfully connected via SSH
- [ ] (Optional) Non-root user created with sudo access

If you have issues, note the exact error message and bring it to class.

---

## What's Next?

In class, we'll:

1. Verify everyone's SSH access
2. Update and secure the server
3. Configure the firewall
4. Install Docker
5. Deploy your application
6. Set up automated deployment

Come to class ready to work on your server!
