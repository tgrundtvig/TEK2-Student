# Hetzner Cloud Server Tutorial

This tutorial walks you through creating a Cloud Server on Hetzner.

**Estimated time: 20-30 minutes**

---

## Prerequisites

- Email address
- Credit card, PayPal, or bank transfer (for account verification)
- SSH key ready (see [pre-class exercises](../pre-class/exercises.md))

---

## Step 1: Create a Hetzner Account

1. Go to [Hetzner Cloud](https://www.hetzner.com/cloud)
2. Click "Sign up"
3. Fill in your details
4. Verify your email
5. Add a payment method
6. Your account may need manual verification (usually same day)

---

## Step 2: Access Cloud Console

1. Go to [Hetzner Cloud Console](https://console.hetzner.cloud/)
2. Sign in with your credentials
3. You'll be in the Cloud Console dashboard

---

## Step 3: Create a Project

Projects organize your resources:

1. If no project exists, click "New Project"
2. Name: `tek2-course`
3. Click "Add project"
4. Click on your project to enter it

---

## Step 4: Add Your SSH Key

Before creating a server, add your SSH key:

1. In your project, click "Security" in left menu
2. Click "SSH Keys" tab
3. Click "Add SSH Key"
4. Paste your public key from `~/.ssh/id_ed25519.pub`
5. Name: "My Laptop" (or descriptive name)
6. Click "Add SSH key"

---

## Step 5: Create a Server

1. Click "Servers" in left menu
2. Click "Add Server"

### Location

| Setting | Recommendation |
|---------|----------------|
| Location | Closest to you (Falkenstein, Nuremberg, or Helsinki for Europe) |

### Image

| Setting | Value |
|---------|-------|
| Image | Ubuntu |
| Version | 22.04 |

### Type

| Setting | Value |
|---------|-------|
| Type | Standard |
| Server | CX11 (or cheapest available, usually €3.79/mo) |

**Note:** CX11 has 1 vCPU, 2 GB RAM - slightly more than needed but cheapest option.

### Networking

| Setting | Value |
|---------|-------|
| IPv4 | Enable |
| IPv6 | Enable (optional, included free) |

### SSH Keys

| Setting | Value |
|---------|-------|
| SSH Key | Select your key |

### Volumes

Skip this section (no extra storage needed).

### Firewalls

Skip for now (we'll use ufw on the server).

### Additional Features

| Setting | Value |
|---------|-------|
| Backups | Optional (costs extra) |
| Placement groups | Skip |

### Name

| Setting | Value |
|---------|-------|
| Name | `tek2-server` |

Click **"Create & Buy now"**

---

## Step 6: Get Your Server's IP Address

1. Wait for server to be created (about 30 seconds)
2. Click on your server name
3. Copy the **IPv4** address shown

Example: `116.203.255.12`

---

## Step 7: Connect via SSH

Open your terminal and connect:

```bash
ssh root@YOUR_SERVER_IP
```

Example:
```bash
ssh root@116.203.255.12
```

First connection will ask to verify fingerprint. Type `yes`.

You should see:
```
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-91-generic x86_64)
...
root@tek2-server:~#
```

---

## Step 8: Configure Firewall (on Server)

```bash
# Allow SSH first!
ufw allow 22

# Allow HTTP and HTTPS
ufw allow 80
ufw allow 443

# Enable firewall
ufw enable

# Verify
ufw status
```

---

## Your Server Details

Record these for later:

| Detail | Value |
|--------|-------|
| IP Address (IPv4) | |
| Username | root |
| SSH Command | `ssh root@YOUR_IP` |

---

## Optional: Create a Non-Root User

```bash
# Create user
adduser yourusername

# Add to sudo group
usermod -aG sudo yourusername

# Copy SSH key to new user
mkdir -p /home/yourusername/.ssh
cp /root/.ssh/authorized_keys /home/yourusername/.ssh/
chown -R yourusername:yourusername /home/yourusername/.ssh
chmod 700 /home/yourusername/.ssh
chmod 600 /home/yourusername/.ssh/authorized_keys
```

Test login as new user:
```bash
ssh yourusername@YOUR_SERVER_IP
```

---

## Cost Information

| Server Type | vCPU | RAM | Cost |
|-------------|------|-----|------|
| CX11 | 1 | 2 GB | €3.79/mo |
| CX21 | 2 | 4 GB | €5.77/mo |
| CX31 | 2 | 8 GB | €9.86/mo |

**Billing:** Charged hourly while server exists. Check exact pricing in console.

**To stop billing:** Delete the server (not just power off).

---

## Hetzner Cloud Firewall (Alternative)

Instead of ufw, you can use Hetzner's cloud firewall:

1. Go to "Firewalls" in left menu
2. Click "Create Firewall"
3. Name: `tek2-firewall`
4. Add Inbound Rules:
   - Protocol: TCP, Port: 22, Source: Any IPv4/IPv6 (SSH)
   - Protocol: TCP, Port: 80, Source: Any IPv4/IPv6 (HTTP)
   - Protocol: TCP, Port: 443, Source: Any IPv4/IPv6 (HTTPS)
5. Apply to servers: Select your server
6. Click "Create Firewall"

---

## Troubleshooting

### "Permission denied (publickey)"

1. Verify SSH key was added before creating server
2. Check you're using username `root`
3. Try specifying the key:
   ```bash
   ssh -i ~/.ssh/id_ed25519 root@YOUR_IP
   ```

### Account Verification Pending

Hetzner manually verifies new accounts. This usually takes a few hours on business days.

If urgent:
- Check email for verification requests
- Contact Hetzner support
- Consider using a different provider temporarily

### "Connection refused"

1. Check server is running (not off)
2. Wait a minute after creation
3. Check firewall allows port 22

### "Connection timed out"

1. Verify IP address is correct
2. Check server status in console
3. Try from different network

### Server Shows Wrong IP

Hetzner assigns both IPv4 and IPv6. Make sure you're using the IPv4 address.

---

## Deleting Your Server (After Course)

To stop all charges:

1. Go to your server in Hetzner Console
2. Click "Delete" in the server actions
3. Confirm deletion

**Note:** This permanently deletes all data!

Also delete:
- SSH keys (if you want)
- Firewalls (if created)
- Project (if empty)

---

## Hetzner Tips

1. **Snapshots**: You can create snapshots of your server (small cost) before making changes
2. **Rescue Mode**: If you break your server, use rescue mode to fix it
3. **Console**: Access server console from web interface if SSH fails

---

## Next Steps

Return to [pre-class exercises](../pre-class/exercises.md#exercise-4-verify-ssh-access) to verify your SSH access.
