# Oracle Cloud VM Tutorial

This tutorial walks you through creating a Compute Instance on Oracle Cloud Infrastructure (OCI).

**Estimated time: 30-45 minutes**

Oracle Cloud has one of the most generous free tiers of any cloud provider, including **always-free** VM instances that don't expire.

---

## Prerequisites

- Email address
- Credit card or debit card (for account verification — you won't be charged)
- SSH key ready (see [pre-class exercises](../pre-class/exercises.md))

---

## Step 1: Create an Oracle Cloud Account

### For Students (Oracle Academy)

If your school participates in [Oracle Academy](https://academy.oracle.com/en/solutions-cloud-program.html):

1. Ask your instructor if your school is an Oracle Academy member
2. Students get **$300 in free credits for one year**
3. **No credit card required** when signing up through Oracle Academy
4. Students must be the age of legal majority in their country

### For Everyone

1. Go to [Oracle Cloud Free Tier](https://www.oracle.com/cloud/free/)
2. Click "Start for free"
3. Fill in your details (name, email, country)
4. Verify your email
5. Add a payment method (credit/debit card for verification only)
6. Choose your **Home Region** (pick the closest to you — this cannot be changed later!)
7. You'll receive **$300 in free credits for 30 days**

**Important:** Your **Home Region** is permanent and determines where your Always Free resources will run. Choose carefully — pick the region closest to you.

---

## Step 2: Understand the Always Free Tier

Oracle's Always Free tier is unusually generous and **never expires**:

### Always Free Compute Instances

| Instance Type | Specs | Quantity |
|---------------|-------|----------|
| **AMD Micro** (VM.Standard.E2.1.Micro) | 1/8 OCPU, 1 GB RAM | Up to 2 instances |
| **ARM Ampere A1** (VM.Standard.A1.Flex) | Up to 4 OCPUs, 24 GB RAM total | Flexible (1-4 instances) |

### Always Free Storage

| Resource | Free Allowance |
|----------|----------------|
| Boot volumes + Block volumes | 200 GB total |
| Object Storage | 20 GB |
| Volume backups | 5 backups |

### ARM vs AMD — Which to Choose?

For this course, we recommend the **ARM Ampere A1** instance:

| | AMD Micro | ARM Ampere A1 |
|---|-----------|---------------|
| **CPU** | 1/8 OCPU (very limited) | 1-4 OCPUs (flexible) |
| **RAM** | 1 GB | Up to 24 GB (flexible) |
| **Architecture** | x86_64 | aarch64 (ARM) |
| **Docker support** | Yes | Yes (ARM images) |
| **Performance** | Basic (fine for learning) | Very good |

**Recommendation:** Create **1 ARM Ampere A1 instance with 1 OCPU and 6 GB RAM**. This is more than enough for this course and leaves room for more free instances later.

**Note on ARM architecture:** ARM instances run ARM-based Docker images. Most popular images (Ubuntu, Nginx, PostgreSQL) have ARM versions. Your Java/Spring Boot app will also work on ARM. If you encounter image compatibility issues, use the AMD Micro instance instead.

---

## Step 3: Navigate to Compute Instances

1. Sign in to [Oracle Cloud Console](https://cloud.oracle.com)
2. Click the **hamburger menu** (☰) in the top left
3. Go to **Compute** → **Instances**
4. Click **"Create instance"**

---

## Step 4: Configure the Instance

### Name and Compartment

| Setting | Value |
|---------|-------|
| Name | `tek2-server` (or your choice) |
| Compartment | Your root compartment (default) |

### Placement

| Setting | Value |
|---------|-------|
| Availability domain | Any available |

### Image and Shape

Click **"Edit"** in the Image and shape section.

#### Select Image

1. Click **"Change image"**
2. Select **"Ubuntu"**
3. Choose **Ubuntu 22.04** (Canonical Ubuntu)
4. Check **"I have reviewed and accept..."**
5. Click **"Select image"**

#### Select Shape

1. Click **"Change shape"**
2. Select **"Ampere"** (for ARM) under Shape series
3. Select **VM.Standard.A1.Flex**
4. Set the resources:

| Setting | Value |
|---------|-------|
| Number of OCPUs | 1 |
| Amount of memory (GB) | 6 |

5. Click **"Select shape"**

**Alternative (AMD):** If you prefer x86, select "AMD" → VM.Standard.E2.1.Micro (1/8 OCPU, 1 GB RAM).

### Networking

Keep the defaults or configure:

| Setting | Value |
|---------|-------|
| Virtual cloud network | Create new VCN |
| VCN name | `tek2-vcn` |
| Subnet | Create new public subnet |
| Public IPv4 address | Assign a public IPv4 address |

**Important:** Make sure "Assign a public IPv4 address" is selected. You need this to connect via SSH.

### Add SSH Key

| Setting | Value |
|---------|-------|
| SSH keys | Paste public keys |

Paste your public key from `~/.ssh/id_ed25519.pub`

### Boot Volume

Keep the defaults:

| Setting | Value |
|---------|-------|
| Boot volume size | 47 GB (default, minimum for Always Free) |
| Encryption | Encrypt in transit (default) |

---

## Step 5: Create the Instance

1. Review your settings
2. Click **"Create"**
3. Wait for the instance to be provisioned (2-5 minutes)
4. Status changes from "Provisioning" to **"Running"**

**Note:** ARM Ampere A1 instances are popular and sometimes unavailable in certain regions. If you get a "capacity" error, try:
- A different availability domain
- The AMD Micro instance instead
- Trying again later (capacity changes frequently)

---

## Step 6: Get Your Server's IP Address

1. On the Instance details page, find **"Public IP address"** under Primary VNIC
2. Copy this IP address

Example: `129.213.45.67`

---

## Step 7: Connect via SSH

Open your terminal and connect:

```bash
ssh ubuntu@YOUR_PUBLIC_IP
```

Example:
```bash
ssh ubuntu@129.213.45.67
```

**Default username is `ubuntu`** for Ubuntu images on Oracle Cloud.

First connection will ask to verify fingerprint. Type `yes`.

You should see:
```
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-91-generic aarch64)
...
ubuntu@tek2-server:~$
```

**Note:** If you chose the AMD Micro instance, it will show `x86_64` instead of `aarch64`.

---

## Step 8: Open Firewall Ports

Oracle Cloud has **two firewalls** you need to configure: the cloud Security List and the OS-level firewall (`iptables`/`ufw`).

### Part A: Open Ports in Security List (Cloud Firewall)

1. Go to your instance details page
2. Under **Primary VNIC**, click the **Subnet** link
3. Click on your **Security List** (e.g., "Default Security List for tek2-vcn")
4. Click **"Add Ingress Rules"**
5. Add rule for HTTP:

| Setting | Value |
|---------|-------|
| Source CIDR | `0.0.0.0/0` |
| Destination Port Range | `80` |
| Description | HTTP |

6. Click **"Add Ingress Rules"**
7. Repeat for HTTPS:

| Setting | Value |
|---------|-------|
| Source CIDR | `0.0.0.0/0` |
| Destination Port Range | `443` |
| Description | HTTPS |

### Part B: Open Ports on the Server (OS Firewall)

Oracle Cloud Ubuntu images use `iptables` by default. You can either configure `iptables` directly or switch to `ufw`:

```bash
# Install ufw (if not present)
sudo apt update
sudo apt install -y ufw

# Allow SSH first!
sudo ufw allow 22

# Allow HTTP and HTTPS
sudo ufw allow 80
sudo ufw allow 443

# Enable firewall
sudo ufw enable

# Disable the default iptables rules that Oracle sets
sudo iptables -F
sudo netfilter-persistent save

# Verify
sudo ufw status
```

**Important:** You must open ports in **both** the Security List (cloud) and the OS firewall. Traffic must pass through both layers.

---

## Your Server Details

Record these for later:

| Detail | Value |
|--------|-------|
| IP Address | |
| Username | ubuntu |
| SSH Command | `ssh ubuntu@YOUR_IP` |
| Instance type | |

---

## Cost Information

| Resource | Cost |
|----------|------|
| ARM A1 (within Always Free limits) | Free forever |
| AMD Micro (within Always Free limits) | Free forever |
| Boot volume (up to 200 GB total) | Free forever |
| Public IP (while instance runs) | Free |

**Always Free means always free** — these resources don't expire, even after the 30-day trial ends. As long as your instance stays within the Always Free limits, you won't be charged.

### What happens after the 30-day trial?

- Your $300 trial credits expire
- Any **paid** resources are terminated
- Your **Always Free** resources continue running at no cost
- Your account converts to a "Pay As You Go" or stays as "Always Free" depending on your choice

---

## Troubleshooting

### "Permission denied (publickey)"

1. Verify your SSH key was added during instance creation
2. Check you're using the correct username (`ubuntu` for Ubuntu images)
3. Try specifying the key:
   ```bash
   ssh -i ~/.ssh/id_ed25519 ubuntu@YOUR_IP
   ```

### "Connection refused"

1. Check instance is running in Console
2. Wait 2-3 minutes after instance starts
3. Verify Security List allows SSH (port 22) — it's allowed by default

### "Connection timed out"

1. Verify the public IP address is correct
2. Check instance status in Console
3. Verify the instance has a public IP assigned
4. Check Security List rules include port 22

### "Out of capacity" when creating ARM instance

ARM Ampere A1 instances are popular and may be temporarily unavailable:

1. Try a different **Availability Domain** (AD-1, AD-2, AD-3)
2. Try creating the instance at a different time (early morning tends to have more capacity)
3. Use the **AMD Micro** instance instead (always available, but less powerful)
4. Reduce the OCPU/memory request (try 1 OCPU, 6 GB)

### Port not accessible from browser

Remember to open ports in **both** places:
1. **Security List** (cloud firewall) — see Step 8, Part A
2. **OS firewall** (ufw/iptables) — see Step 8, Part B

### Docker images not working (ARM)

If a Docker image doesn't support ARM:
1. Check Docker Hub for the image — look for `arm64` or `aarch64` tags
2. Try adding `--platform linux/arm64` to your Docker commands
3. Most official images (Ubuntu, Nginx, PostgreSQL, OpenJDK) support ARM
4. If stuck, use the AMD Micro instance instead

---

## Cleaning Up (After Course)

To remove your instance:

1. Go to **Compute** → **Instances**
2. Click on your instance
3. Click **"More actions"** → **"Terminate"**
4. Check **"Permanently delete the attached boot volume"**
5. Click **"Terminate instance"**

**Note:** Since Always Free resources are free, you can also keep the instance running for personal projects after the course.

---

## Next Steps

Return to [pre-class exercises](../pre-class/exercises.md#exercise-4-verify-ssh-access) to verify your SSH access.
