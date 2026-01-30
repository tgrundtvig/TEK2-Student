# Azure VM Tutorial

This tutorial walks you through creating a Virtual Machine on Microsoft Azure.

**Estimated time: 30-45 minutes**

---

## Prerequisites

- Microsoft account (personal or school)
- Credit card (for account verification, won't be charged if using free tier)
- SSH key ready (see [pre-class exercises](../pre-class/exercises.md))

---

## Step 1: Create an Azure Account

### For Students

1. Go to [Azure for Students](https://azure.microsoft.com/en-us/free/students/)
2. Click "Start free"
3. Sign in with your school email
4. Verify your student status
5. You'll receive $100 credit without a credit card

### For Everyone Else

1. Go to [Azure Free Account](https://azure.microsoft.com/en-us/free/)
2. Click "Start free"
3. Sign in with Microsoft account
4. Add payment method (won't be charged for free tier)
5. You'll receive $200 credit for 30 days

---

## Step 2: Navigate to Virtual Machines

1. Sign in to [Azure Portal](https://portal.azure.com)
2. In the search bar at the top, type "Virtual machines"
3. Click on "Virtual machines" in the results
4. Click "+ Create" → "Azure virtual machine"

---

## Step 3: Configure the Basics

### Project Details

| Setting | Value |
|---------|-------|
| Subscription | Your subscription (Azure for Students or Free Trial) |
| Resource group | Click "Create new" → Name: `tek2-resources` |

### Instance Details

| Setting | Value |
|---------|-------|
| Virtual machine name | `tek2-server` (or your choice) |
| Region | Choose closest to you (e.g., "North Europe" for Denmark) |
| Availability options | No infrastructure redundancy required |
| Security type | Standard |
| Image | Ubuntu Server 22.04 LTS - x64 Gen2 |
| VM architecture | x64 |
| Size | Click "See all sizes" → Select "Standard_B1s" (1 vCPU, 1 GB RAM) |

**Note:** Standard_B1s is free tier eligible and sufficient for this course.

### Administrator Account

| Setting | Value |
|---------|-------|
| Authentication type | SSH public key |
| Username | `azureuser` (or your choice) |
| SSH public key source | Use existing public key |
| SSH public key | Paste your public key from `~/.ssh/id_ed25519.pub` |

### Inbound Port Rules

| Setting | Value |
|---------|-------|
| Public inbound ports | Allow selected ports |
| Select inbound ports | SSH (22) |

Click **"Next: Disks"**

---

## Step 4: Configure Disks

Keep the defaults:

| Setting | Value |
|---------|-------|
| OS disk type | Standard SSD |
| Delete with VM | Checked |

Click **"Next: Networking"**

---

## Step 5: Configure Networking

Keep the defaults, but verify:

| Setting | Value |
|---------|-------|
| Virtual network | (new) tek2-server-vnet |
| Subnet | default |
| Public IP | (new) tek2-server-ip |
| NIC network security group | Basic |
| Public inbound ports | Allow selected ports |
| Select inbound ports | SSH (22) |

**Important:** We'll add HTTP (80) and HTTPS (443) later via the firewall.

Click **"Next: Management"**

---

## Step 6: Configure Management

Keep defaults. You may want to:

| Setting | Value |
|---------|-------|
| Auto-shutdown | Enable and set a time (saves money) |

Click **"Review + create"**

---

## Step 7: Review and Create

1. Review all settings
2. Azure will validate the configuration
3. Click **"Create"**
4. Deployment will take 1-3 minutes

---

## Step 8: Get Your Server's IP Address

1. Once deployment completes, click "Go to resource"
2. On the Overview page, find **"Public IP address"**
3. Copy this IP address - you'll need it!

Example: `20.234.123.45`

---

## Step 9: Connect via SSH

Open your terminal and connect:

```bash
ssh azureuser@YOUR_PUBLIC_IP
```

Example:
```bash
ssh azureuser@20.234.123.45
```

First connection will ask to verify fingerprint. Type `yes`.

You should see:
```
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-91-generic x86_64)
...
azureuser@tek2-server:~$
```

---

## Step 10: Open HTTP/HTTPS Ports

### Option A: Using Azure Portal (Recommended)

1. Go to your VM in Azure Portal
2. In left menu, click "Networking" → "Network settings"
3. Click "Create port rule" → "Inbound port rule"
4. Add rule for HTTP:
   - Destination port ranges: `80`
   - Protocol: TCP
   - Name: `AllowHTTP`
   - Click "Add"
5. Repeat for HTTPS:
   - Destination port ranges: `443`
   - Protocol: TCP
   - Name: `AllowHTTPS`
   - Click "Add"

### Option B: Using Azure CLI

```bash
# Install Azure CLI if needed
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Login
az login

# Add HTTP rule
az network nsg rule create \
  --resource-group tek2-resources \
  --nsg-name tek2-serverNSG \
  --name AllowHTTP \
  --priority 1001 \
  --destination-port-ranges 80 \
  --protocol Tcp \
  --access Allow

# Add HTTPS rule
az network nsg rule create \
  --resource-group tek2-resources \
  --nsg-name tek2-serverNSG \
  --name AllowHTTPS \
  --priority 1002 \
  --destination-port-ranges 443 \
  --protocol Tcp \
  --access Allow
```

---

## Your Server Details

Record these for later:

| Detail | Value |
|--------|-------|
| IP Address | |
| Username | azureuser |
| SSH Command | `ssh azureuser@YOUR_IP` |

---

## Cost Management Tips

1. **Use auto-shutdown**: Configure in VM settings
2. **Stop when not in use**: VM → Overview → Stop (still pays for storage)
3. **Delete when done**: Delete the resource group to remove everything
4. **Monitor spending**: Cost Management + Billing in portal

### Estimated Costs (if not using free credits)

| Resource | Cost |
|----------|------|
| Standard_B1s VM | ~$7-10/month |
| Storage (30GB) | ~$1.50/month |
| Public IP | ~$3/month |
| **Total** | ~$12-15/month |

**With free credits:** $0 until credits run out

---

## Troubleshooting

### "Permission denied (publickey)"

1. Check your SSH key was added correctly during VM creation
2. Verify username is correct (`azureuser` or what you set)
3. Try specifying the key explicitly:
   ```bash
   ssh -i ~/.ssh/id_ed25519 azureuser@YOUR_IP
   ```

### "Connection refused"

1. Check VM is running (not stopped/deallocated)
2. Check Network Security Group allows SSH (port 22)
3. Wait a minute after VM starts for SSH to be ready

### "Connection timed out"

1. Verify the IP address is correct
2. Check VM is running in Azure Portal
3. Check Network Security Group rules

### Can't find VM

1. Make sure you're in the correct subscription
2. Search for "Virtual machines" in the portal
3. Check resource groups

---

## Cleaning Up (After Course)

To avoid charges, delete everything:

1. Go to "Resource groups" in Azure Portal
2. Click on `tek2-resources`
3. Click "Delete resource group"
4. Type the resource group name to confirm
5. Click "Delete"

This removes the VM, storage, network, and IP address.

---

## Next Steps

Return to [pre-class exercises](../pre-class/exercises.md#exercise-4-verify-ssh-access) to verify your SSH access.
