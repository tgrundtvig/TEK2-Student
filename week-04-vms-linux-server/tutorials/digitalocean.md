# Digital Ocean Droplet Tutorial

This tutorial walks you through creating a Droplet (VM) on Digital Ocean.

**Estimated time: 20-30 minutes**

---

## Prerequisites

- Email address
- Credit card or PayPal (for account verification)
- SSH key ready (see [pre-class exercises](../pre-class/exercises.md))

---

## Step 1: Create a Digital Ocean Account

1. Go to [Digital Ocean](https://www.digitalocean.com/)
2. Click "Sign up"
3. Sign up with email, Google, or GitHub
4. Verify your email
5. Add a payment method

### GitHub Student Pack Bonus

If you have GitHub Student Developer Pack:
1. Go to [GitHub Student Developer Pack](https://education.github.com/pack)
2. Find Digital Ocean
3. Claim $200 in credits

---

## Step 2: Add Your SSH Key

Before creating a Droplet, add your SSH key:

1. Click on your avatar (top right) → "Settings"
2. In left menu, click "Security"
3. Click "Add SSH Key"
4. Paste your public key from `~/.ssh/id_ed25519.pub`
5. Name it something memorable (e.g., "My Laptop")
6. Click "Add SSH Key"

---

## Step 3: Create a Droplet

1. Click the green **"Create"** button (top right)
2. Select **"Droplets"**

### Choose Region

| Setting | Recommendation |
|---------|----------------|
| Region | Closest to you (e.g., "Amsterdam" or "Frankfurt" for Europe) |
| Datacenter | Any available |

### Choose an Image

| Setting | Value |
|---------|-------|
| OS | Ubuntu |
| Version | 22.04 (LTS) x64 |

### Choose Size

| Setting | Value |
|---------|-------|
| Droplet Type | Basic |
| CPU options | Regular (Disk type: SSD) |
| Size | $4/mo (1 GB / 1 CPU / 25 GB SSD / 1000 GB transfer) |

This is the smallest and cheapest option, sufficient for this course.

### Choose Authentication Method

| Setting | Value |
|---------|-------|
| Authentication | SSH Key |
| Select SSH keys | Check your key |

### Finalize Details

| Setting | Value |
|---------|-------|
| Hostname | `tek2-server` (or your choice) |
| Tags | Optional |
| Project | Default |
| Backups | Leave unchecked (costs extra) |

Click **"Create Droplet"**

---

## Step 4: Get Your Server's IP Address

1. Wait for Droplet to be created (about 1 minute)
2. You'll see your Droplet in the list
3. Copy the **IP address** shown

Example: `64.23.145.101`

---

## Step 5: Connect via SSH

Open your terminal and connect:

```bash
ssh root@YOUR_DROPLET_IP
```

Example:
```bash
ssh root@64.23.145.101
```

First connection will ask to verify fingerprint. Type `yes`.

You should see:
```
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-91-generic x86_64)
...
root@tek2-server:~#
```

---

## Step 6: Configure Firewall (on Droplet)

Digital Ocean Droplets don't have ufw enabled by default. Set it up:

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
| IP Address | |
| Username | root |
| SSH Command | `ssh root@YOUR_IP` |

---

## Optional: Create a Non-Root User

For better security, create a regular user:

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
ssh yourusername@YOUR_DROPLET_IP
```

---

## Cost Information

| Size | Cost |
|------|------|
| Basic $4/mo | $4/month (~$0.006/hour) |
| Basic $6/mo | $6/month (~$0.009/hour) |

**Billing:** Charged hourly while Droplet exists (even when powered off).

To stop billing: **Destroy** the Droplet (not just power off).

---

## Digital Ocean Firewall (Alternative)

Instead of ufw, you can use Digital Ocean's cloud firewall:

1. Go to **Networking** → **Firewalls**
2. Click **Create Firewall**
3. Name: `tek2-firewall`
4. Inbound Rules:
   - SSH: Port 22, All IPv4/IPv6
   - HTTP: Port 80, All IPv4/IPv6
   - HTTPS: Port 443, All IPv4/IPv6
5. Outbound Rules: Keep defaults (allow all)
6. Apply to Droplets: Select your Droplet
7. Click **Create Firewall**

This firewall is managed outside the Droplet, so you can't lock yourself out.

---

## Troubleshooting

### "Permission denied (publickey)"

1. Verify SSH key was added to Digital Ocean before creating Droplet
2. Check you're using the correct username (`root`)
3. Try specifying the key:
   ```bash
   ssh -i ~/.ssh/id_ed25519 root@YOUR_IP
   ```

### "Connection refused"

1. Check Droplet is running (not powered off)
2. Wait a minute after creation for SSH to start
3. Check Digital Ocean firewall (if using) allows port 22

### "Connection timed out"

1. Verify the IP address is correct
2. Check Droplet status in dashboard
3. Try from different network

### Forgot to Add SSH Key

If you created a Droplet without SSH key:

1. Go to Droplet → Access → Launch Recovery Console
2. Log in with password (emailed to you)
3. Add your SSH key:
   ```bash
   mkdir -p ~/.ssh
   echo "YOUR_PUBLIC_KEY" >> ~/.ssh/authorized_keys
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

---

## Destroying Your Droplet (After Course)

To stop all charges:

1. Go to your Droplet
2. Click **Destroy** in left menu
3. Click **Destroy this Droplet**
4. Confirm by checking the box
5. Click **Destroy**

**Note:** This permanently deletes all data!

---

## Next Steps

Return to [pre-class exercises](../pre-class/exercises.md#exercise-4-verify-ssh-access) to verify your SSH access.
