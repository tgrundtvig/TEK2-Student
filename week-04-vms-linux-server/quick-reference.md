# Quick Reference: VMs and Linux Server Administration

**Print this page for easy reference during class**

---

## SSH Commands

```bash
# Connect to server
ssh username@server-ip

# Connect with specific key
ssh -i ~/.ssh/mykey username@server-ip

# Copy file to server
scp localfile.txt username@server-ip:/path/on/server/

# Copy file from server
scp username@server-ip:/path/on/server/file.txt ./local/

# Copy directory to server
scp -r localdir/ username@server-ip:/path/on/server/

# Test SSH connection
ssh -T git@github.com
```

---

## SSH Key Management

```bash
# Generate new SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# View public key (to copy to server)
cat ~/.ssh/id_ed25519.pub

# Start SSH agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# List added keys
ssh-add -l
```

---

## Package Management (apt)

```bash
# Update package list
sudo apt update

# Upgrade installed packages
sudo apt upgrade -y

# Install a package
sudo apt install package-name

# Remove a package
sudo apt remove package-name

# Search for packages
apt search keyword

# Show package info
apt show package-name

# Clean up unused packages
sudo apt autoremove
```

---

## Service Management (systemctl)

```bash
# Check service status
sudo systemctl status servicename

# Start a service
sudo systemctl start servicename

# Stop a service
sudo systemctl stop servicename

# Restart a service
sudo systemctl restart servicename

# Enable service at boot
sudo systemctl enable servicename

# Disable service at boot
sudo systemctl disable servicename

# View service logs
sudo journalctl -u servicename

# View recent logs (follow mode)
sudo journalctl -u servicename -f
```

---

## Firewall (ufw)

```bash
# Check firewall status
sudo ufw status

# Enable firewall
sudo ufw enable

# Disable firewall
sudo ufw disable

# Allow a port
sudo ufw allow 22        # SSH
sudo ufw allow 80        # HTTP
sudo ufw allow 443       # HTTPS

# Allow specific port/protocol
sudo ufw allow 8080/tcp

# Deny a port
sudo ufw deny 3000

# Delete a rule
sudo ufw delete allow 8080

# Reset all rules
sudo ufw reset
```

---

## Docker Commands (on server)

```bash
# Check Docker is running
sudo systemctl status docker

# Run a container
docker run -d -p 80:8080 imagename

# List running containers
docker ps

# List all containers
docker ps -a

# View container logs
docker logs container-id

# Stop a container
docker stop container-id

# Remove a container
docker rm container-id

# Pull latest image
docker pull ghcr.io/username/imagename:latest

# Remove old images
docker image prune
```

---

## File System Navigation

```bash
# Current directory
pwd

# List files
ls -la

# Change directory
cd /path/to/dir

# Create directory
mkdir dirname

# Remove file
rm filename

# Remove directory
rm -rf dirname

# Edit file
nano filename

# View file
cat filename

# View file with paging
less filename
```

---

## User Management

```bash
# Current user
whoami

# Switch to root
sudo -i

# Exit root/session
exit

# Add user to group
sudo usermod -aG groupname username

# Add user to docker group (no sudo needed for docker)
sudo usermod -aG docker $USER
```

---

## System Information

```bash
# System info
uname -a

# Disk usage
df -h

# Memory usage
free -h

# Running processes
htop    # or top

# Network interfaces
ip addr

# Open ports
sudo ss -tulpn
```

---

## Common Server Paths

| Path | Purpose |
|------|---------|
| `/home/username/` | User home directory |
| `/var/log/` | System logs |
| `/etc/` | Configuration files |
| `/opt/` | Optional/third-party software |
| `~/.ssh/` | SSH keys and config |
| `~/.ssh/authorized_keys` | Allowed public keys |

---

## GitHub Actions Deployment

```yaml
# Deploy to server via SSH
- name: Deploy to server
  uses: appleboy/ssh-action@v1.0.0
  with:
    host: ${{ secrets.SERVER_IP }}
    username: ${{ secrets.SERVER_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      docker pull ghcr.io/${{ github.repository_owner }}/myapp:latest
      docker stop myapp || true
      docker rm myapp || true
      docker run -d --name myapp -p 80:8080 ghcr.io/${{ github.repository_owner }}/myapp:latest
```

---

## Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| "Permission denied (publickey)" | Check SSH key is added, correct permissions |
| "Connection refused" | Check server is running, firewall allows port 22 |
| "Connection timed out" | Check IP address, server is running, firewall |
| "docker: command not found" | Docker not installed, or need `sudo` |
| "Permission denied" with Docker | Add user to docker group, log out/in |
| "Port already in use" | Another process using port, stop it first |

---

## SSH Key Permissions

```bash
# Fix key permissions (if permission denied)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
```

---

## Deployment Workflow

```
1. Push code to GitHub
        │
        ▼
2. GitHub Actions triggered
        │
        ▼
3. Build and test
        │
        ▼
4. Build Docker image
        │
        ▼
5. Push to GHCR
        │
        ▼
6. SSH to server
        │
        ▼
7. Pull new image
        │
        ▼
8. Stop old container
        │
        ▼
9. Start new container
        │
        ▼
10. Application updated!
```

---

## Remember

- **SSH** = Secure Shell (encrypted remote access)
- **apt** = Package manager for Ubuntu/Debian
- **systemctl** = Service manager for systemd
- **ufw** = Uncomplicated Firewall
- Always use `sudo` for administrative commands
- Keep your server updated: `sudo apt update && sudo apt upgrade`
