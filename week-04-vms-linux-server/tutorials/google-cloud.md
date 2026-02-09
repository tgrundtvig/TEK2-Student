# Google Cloud VM Tutorial

This tutorial walks you through creating a Compute Engine VM instance on Google Cloud Platform (GCP).

**Estimated time: 30-45 minutes**

---

## Prerequisites

- Google account (personal or school)
- Credit card (for account verification, won't be charged if using free tier)
- SSH key ready (see [pre-class exercises](../pre-class/exercises.md))

---

## Step 1: Create a Google Cloud Account

### For Students

Google Cloud offers education credits through your institution:

1. Ask your instructor if your school participates in [Google Cloud for Education](https://cloud.google.com/edu/students)
2. If available, your instructor can distribute credits for coursework
3. Credits are valid for one year after redemption

### For Everyone (Free Trial)

1. Go to [Google Cloud Free Tier](https://cloud.google.com/free)
2. Click "Get started for free"
3. Sign in with your Google account
4. Add a payment method (you won't be charged unless you manually upgrade)
5. You'll receive **$300 in free credits for 90 days**

### Always Free Tier

After the trial ends, Google Cloud offers an **Always Free** tier that includes:

| Resource | Free Allowance |
|----------|----------------|
| VM instance | 1x e2-micro (shared 2 vCPU, 1 GB RAM) |
| Boot disk | 30 GB standard persistent disk |
| Outbound transfer | 1 GB/month from North America |
| Regions | US only: Oregon, Iowa, or South Carolina |

**Note:** The Always Free VM is limited to US regions. If you're in Europe, this means higher latency. Consider using your $300 free trial credits to create a VM in a European region instead.

---

## Step 2: Navigate to Compute Engine

1. Sign in to [Google Cloud Console](https://console.cloud.google.com)
2. If prompted, select or create a project (e.g., `tek2-project`)
3. In the left navigation menu, click **"Compute Engine"** → **"VM instances"**
4. If this is your first time, click **"Enable"** to enable the Compute Engine API (takes about a minute)
5. Click **"Create Instance"**

---

## Step 3: Configure the VM

### Name and Region

| Setting | Value |
|---------|-------|
| Name | `tek2-server` (or your choice) |
| Region | `us-central1` (Iowa) for free tier, or closest to you if using trial credits |
| Zone | Any available (e.g., `us-central1-a`) |

### Machine Configuration

| Setting | Value |
|---------|-------|
| Series | E2 |
| Machine type | `e2-micro` (2 shared vCPU, 1 GB memory) — **free tier eligible** |

**Note:** The `e2-micro` is sufficient for this course. If using trial credits, `e2-small` (2 GB RAM) gives more room.

### Boot Disk

Click **"Change"** under Boot disk:

| Setting | Value |
|---------|-------|
| Operating system | Ubuntu |
| Version | Ubuntu 22.04 LTS |
| Boot disk type | Standard persistent disk |
| Size | 30 GB (max for free tier) |

Click **"Select"**

### Firewall

Check both boxes:

| Setting | Value |
|---------|-------|
| Allow HTTP traffic | Checked |
| Allow HTTPS traffic | Checked |

This creates firewall rules for ports 80 and 443.

---

## Step 4: Add Your SSH Key

1. Click **"Advanced options"** (at the bottom of the page)
2. Expand **"Security"**
3. Expand **"Manage access"**
4. Under **"Add manually generated SSH keys"**, click **"Add item"**
5. Paste your public key from `~/.ssh/id_ed25519.pub`

**Important:** Google Cloud extracts the username from the end of your SSH key. If your key ends with `user@hostname`, that becomes your SSH username.

To check what username will be used:

```bash
cat ~/.ssh/id_ed25519.pub
# ssh-ed25519 AAAAC3Nz... yourname@yourlaptop
#                           ^^^^^^^^ this becomes your username
```

---

## Step 5: Create the VM

1. Review your settings
2. Click **"Create"**
3. Wait for the VM to be created (about 1-2 minutes)
4. A green checkmark appears when it's ready

---

## Step 6: Get Your Server's IP Address

1. In the VM instances list, find your VM
2. Copy the **External IP** address shown in the table

Example: `34.123.45.67`

**Note:** The external IP may change if you stop and start the VM. To get a static IP:
1. Go to **VPC network** → **IP addresses**
2. Click **"Reserve"** next to your VM's IP (free for running VMs, charged if VM is stopped)

---

## Step 7: Connect via SSH

Open your terminal and connect:

```bash
ssh yourname@YOUR_EXTERNAL_IP
```

Example:
```bash
ssh yourname@34.123.45.67
```

**Replace `yourname`** with the username from your SSH key (see Step 4).

First connection will ask to verify fingerprint. Type `yes`.

You should see:
```
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-91-generic x86_64)
...
yourname@tek2-server:~$
```

### Alternative: Browser-based SSH

Google Cloud also offers SSH directly in the browser:
1. In VM instances list, click **"SSH"** button next to your VM
2. A terminal opens in your browser

This is useful for troubleshooting if your local SSH doesn't work.

---

## Step 8: Configure Firewall

The web interface already opened ports 80 and 443 (Step 3). Now set up `ufw` on the server:

```bash
# Allow SSH first!
sudo ufw allow 22

# Allow HTTP and HTTPS
sudo ufw allow 80
sudo ufw allow 443

# Enable firewall
sudo ufw enable

# Verify
sudo ufw status
```

**Note:** Google Cloud has two layers of firewall: the cloud firewall (configured in Console) and the OS-level firewall (`ufw`). Both must allow traffic.

---

## Your Server Details

Record these for later:

| Detail | Value |
|--------|-------|
| IP Address | |
| Username | (from your SSH key) |
| SSH Command | `ssh yourname@YOUR_IP` |
| Project | |

---

## Cost Management Tips

1. **Use the free tier VM**: `e2-micro` in US regions is always free
2. **Stop when not in use**: VM → Stop (storage still incurs cost if over 30 GB)
3. **Set budget alerts**: Billing → Budgets & alerts → Create budget
4. **Delete when done**: Delete the VM and its disk to stop all charges
5. **Monitor spending**: Billing → Reports

### Estimated Costs (if not using free tier)

| Resource | Cost |
|----------|------|
| e2-micro VM (free tier region) | Free |
| e2-micro VM (non-free region) | ~$7-10/month |
| e2-small VM | ~$14-17/month |
| Storage (30 GB) | Free (up to 30 GB) |
| Static IP (while VM runs) | Free |
| Static IP (VM stopped) | ~$3/month |

**With free trial credits:** $0 for 90 days (up to $300 usage)

---

## Troubleshooting

### "Permission denied (publickey)"

1. Verify your SSH key was added in Step 4
2. Check the username matches the one in your SSH key:
   ```bash
   # Check what username is in your key
   cat ~/.ssh/id_ed25519.pub
   ```
3. Try specifying the key explicitly:
   ```bash
   ssh -i ~/.ssh/id_ed25519 yourname@YOUR_IP
   ```

### "Connection refused"

1. Check VM is running (not stopped) in Console
2. Check Compute Engine firewall allows SSH (port 22)
3. Wait a minute after VM starts for SSH to be ready

### "Connection timed out"

1. Verify the External IP address is correct
2. Check VM status in Console
3. Make sure the VM has an external IP assigned

### SSH username confusion

Google Cloud uses the username from your SSH key comment field. If your key looks like:
```
ssh-ed25519 AAAAC3Nz... student@laptop
```
Then your username is `student`. If unsure, use the browser-based SSH (click "SSH" in Console) to check.

### VM IP changed after restart

The external IP can change when you stop/start the VM. Either:
- Reserve a static IP (see Step 6)
- Check the new IP in Console after starting

---

## Cleaning Up (After Course)

To stop all charges:

### Option A: Delete just the VM

1. Go to **Compute Engine** → **VM instances**
2. Check the box next to your VM
3. Click **"Delete"** at the top
4. Confirm deletion

### Option B: Delete the entire project (removes everything)

1. Go to **IAM & Admin** → **Settings**
2. Click **"Shut down"**
3. Enter the project ID to confirm
4. Click **"Shut down"**

**Note:** This permanently deletes all data and resources in the project.

---

## Next Steps

Return to [pre-class exercises](../pre-class/exercises.md#exercise-4-verify-ssh-access) to verify your SSH access.
