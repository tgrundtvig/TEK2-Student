# Class Exercises: Server Setup and Deployment

Work through these exercises during class. Ask for help if you get stuck!

---

## Warm-up: Verify SSH Access (~10 minutes)

Quick check that everyone can connect to their server.

### Connect to your server

```bash
ssh username@your-server-ip
```

### If you don't have a server set up

Talk to the instructor immediately. Options:
1. Quick VM creation (follow provider tutorial)
2. Use Coolify as backup
3. Pair with a classmate

### Verify you're connected

```bash
# Should show your server hostname
hostname

# Should show Ubuntu 22.04 or 24.04
lsb_release -a
```

### Self-check
- [ ] Can connect via SSH
- [ ] See Ubuntu command prompt
- [ ] Know my username (root or created user)

---

## Exercise 1: Update and Secure Your Server (~15 minutes)

### Goal

Get your server up to date and set basic security.

### Part A: Update packages (~5 minutes)

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade -y
```

This may take a few minutes. You'll see packages being downloaded and installed.

### Part B: Set timezone (~2 minutes)

```bash
# Check current timezone
timedatectl

# Set to your timezone (example: Copenhagen)
sudo timedatectl set-timezone Europe/Copenhagen

# Verify
date
```

### Part C: Install essential tools (~3 minutes)

```bash
sudo apt install -y \
    curl \
    wget \
    git \
    htop \
    unzip
```

### Part D: Verify installation (~2 minutes)

```bash
# Check git is installed
git --version

# Check htop (interactive process viewer)
htop
# Press 'q' to exit htop
```

### Self-check
- [ ] Packages are updated
- [ ] Timezone is set correctly
- [ ] Essential tools are installed

---

## Exercise 2: Configure the Firewall (~15 minutes)

### Goal

Secure your server by only allowing necessary traffic.

### Part A: Check firewall status (~2 minutes)

```bash
sudo ufw status
```

You'll likely see "Status: inactive"

### Part B: Configure default policies (~2 minutes)

```bash
# Deny all incoming by default
sudo ufw default deny incoming

# Allow all outgoing by default
sudo ufw default allow outgoing
```

### Part C: Allow essential ports (~5 minutes)

**IMPORTANT: Allow SSH first or you'll lock yourself out!**

```bash
# Allow SSH (port 22) - DO THIS FIRST!
sudo ufw allow 22

# Allow HTTP (port 80) - for web traffic
sudo ufw allow 80

# Allow HTTPS (port 443) - for secure web traffic
sudo ufw allow 443
```

### Part D: Enable the firewall (~3 minutes)

```bash
sudo ufw enable
```

Type `y` when prompted.

### Part E: Verify configuration (~3 minutes)

```bash
sudo ufw status verbose
```

Expected output:
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22                         ALLOW IN    Anywhere
80                         ALLOW IN    Anywhere
443                        ALLOW IN    Anywhere
22 (v6)                    ALLOW IN    Anywhere (v6)
80 (v6)                    ALLOW IN    Anywhere (v6)
443 (v6)                   ALLOW IN    Anywhere (v6)
```

### Self-check
- [ ] SSH was allowed BEFORE enabling firewall
- [ ] HTTP (80) and HTTPS (443) are allowed
- [ ] Firewall is active
- [ ] Can still SSH in (test by opening a new terminal!)

---

## Exercise 3: Install Docker (~20 minutes)

### Goal

Install Docker on your server to run containers.

### Part A: Remove old versions (~2 minutes)

```bash
# Remove any old Docker packages
sudo apt remove docker docker-engine docker.io containerd runc 2>/dev/null || true
```

### Part B: Set up Docker repository (~5 minutes)

```bash
# Install prerequisites
sudo apt install -y ca-certificates curl gnupg

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the Docker repository
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Part C: Install Docker (~5 minutes)

```bash
# Update package list with Docker repo
sudo apt update

# Install Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Part D: Verify installation (~3 minutes)

```bash
# Check Docker version
docker --version

# Check Docker is running
sudo systemctl status docker
```

You should see "active (running)" in the output.

### Part E: Run test container (~3 minutes)

```bash
sudo docker run hello-world
```

You should see "Hello from Docker!" message.

### Part F: Allow non-root Docker access (optional) (~2 minutes)

If you want to run Docker without `sudo`:

```bash
# Add your user to docker group
sudo usermod -aG docker $USER

# Log out and back in (or run)
newgrp docker

# Test without sudo
docker run hello-world
```

### Self-check
- [ ] Docker is installed
- [ ] Docker service is running
- [ ] hello-world container runs successfully

---

## Exercise 4: Deploy Your Application (~30 minutes)

### Goal

Pull and run your Docker image from GitHub Container Registry.

### Part A: Authenticate with GHCR (~5 minutes)

You need a GitHub Personal Access Token (PAT) with `read:packages` scope.

**Create a PAT:**
1. Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Name: "Server Docker Access"
4. Expiration: 90 days (or your preference)
5. Scopes: Check `read:packages`
6. Click "Generate token"
7. **Copy the token immediately** (you won't see it again!)

**Login on server:**

```bash
# Login to GHCR
echo "YOUR_TOKEN_HERE" | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

Replace `YOUR_TOKEN_HERE` with your token and `YOUR_GITHUB_USERNAME` with your username.

Expected output: "Login Succeeded"

### Part B: Pull your image (~5 minutes)

If you have an image from Week 3:

```bash
docker pull ghcr.io/YOUR_GITHUB_USERNAME/hello-maven:latest
```

If you don't have an image, use a public one for testing:

```bash
docker pull nginx:alpine
```

### Part C: Run the container (~10 minutes)

**For your Java application:**

```bash
docker run -d \
  --name myapp \
  -p 80:8080 \
  ghcr.io/YOUR_GITHUB_USERNAME/hello-maven:latest
```

**For nginx test:**

```bash
docker run -d \
  --name webserver \
  -p 80:80 \
  nginx:alpine
```

**Flags explained:**
- `-d` - Run in background (detached)
- `--name myapp` - Give the container a name
- `-p 80:8080` - Map port 80 on server to port 8080 in container

### Part D: Verify it's running (~5 minutes)

```bash
# Check container status
docker ps

# Check container logs
docker logs myapp  # or webserver

# Test locally on server
curl http://localhost
```

### Part E: Access from browser (~5 minutes)

Open your browser and navigate to:

```
http://YOUR_SERVER_IP
```

You should see your application (or nginx welcome page)!

### Self-check
- [ ] Authenticated with GHCR
- [ ] Image pulled successfully
- [ ] Container is running (`docker ps` shows it)
- [ ] Can access application in browser

---

## Exercise 5: Set Up Automatic Deployment (~45 minutes)

### Goal

Configure GitHub Actions to automatically deploy when you push code.

### Part A: Create deployment SSH key (~10 minutes)

On your **local machine** (not the server), create a dedicated deployment key:

```bash
# Generate a new key specifically for deployment
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/deploy_key -N ""

# View the public key
cat ~/.ssh/deploy_key.pub

# View the private key (you'll need this for GitHub)
cat ~/.ssh/deploy_key
```

**On your server**, add the public key:

```bash
# Add to authorized_keys
echo "YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
```

### Part B: Add secrets to GitHub (~5 minutes)

In your GitHub repository:

1. Go to **Settings** → **Secrets and variables** → **Actions**
2. Add these secrets:

| Secret Name | Value |
|-------------|-------|
| `SERVER_IP` | Your server's IP address |
| `SERVER_USER` | Username (e.g., `root` or your username) |
| `SSH_PRIVATE_KEY` | Contents of `~/.ssh/deploy_key` (the private key) |

**For SSH_PRIVATE_KEY**: Include everything from `-----BEGIN OPENSSH PRIVATE KEY-----` to `-----END OPENSSH PRIVATE KEY-----`

### Part C: Create deployment workflow (~15 minutes)

In your repository, create `.github/workflows/deploy.yml`:

```yaml
name: Build and Deploy

on:
  push:
    branches: [ main ]

permissions:
  contents: read
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Build with Maven
      run: mvn clean package

    - name: Log in to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push Docker image
      run: |
        docker build -t ghcr.io/${{ github.repository_owner }}/hello-maven:latest .
        docker build -t ghcr.io/${{ github.repository_owner }}/hello-maven:${{ github.sha }} .
        docker push ghcr.io/${{ github.repository_owner }}/hello-maven:latest
        docker push ghcr.io/${{ github.repository_owner }}/hello-maven:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
    - name: Deploy to server
      uses: appleboy/ssh-action@v1.0.0
      with:
        host: ${{ secrets.SERVER_IP }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          # Login to GHCR
          echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin

          # Pull the latest image
          docker pull ghcr.io/${{ github.repository_owner }}/hello-maven:latest

          # Stop and remove old container (if exists)
          docker stop myapp 2>/dev/null || true
          docker rm myapp 2>/dev/null || true

          # Start new container
          docker run -d \
            --name myapp \
            -p 80:8080 \
            --restart unless-stopped \
            ghcr.io/${{ github.repository_owner }}/hello-maven:latest

          # Clean up old images
          docker image prune -f
```

### Part D: Update server to allow GHCR pull (~5 minutes)

The deploy script needs to authenticate with GHCR. You have two options:

**Option 1: Use a PAT stored as a secret (Recommended)**

Add another secret `GHCR_TOKEN` with your Personal Access Token, then update the login line:
```yaml
echo ${{ secrets.GHCR_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
```

**Option 2: Pre-authenticate on server**

If you logged in during Exercise 4, Docker credentials are saved at `~/.docker/config.json`. Remove the login line from the script.

### Part E: Test the deployment (~10 minutes)

1. Commit and push your changes:
   ```bash
   git add .
   git commit -m "Add deployment workflow"
   git push
   ```

2. Go to GitHub Actions tab and watch the workflow

3. After it completes, verify:
   - Check your server: `docker ps`
   - Visit `http://YOUR_SERVER_IP` in browser

### Part F: Test automatic deployment

Make a small change to your code, push it, and watch it automatically deploy!

### Self-check
- [ ] Deployment SSH key created and added to server
- [ ] GitHub secrets configured
- [ ] Workflow file created
- [ ] Push triggers successful deployment
- [ ] New version is live after push

---

## Alternative: Coolify Deployment (~15 minutes)

If you're using Coolify instead of a VM, follow the [Coolify tutorial](../tutorials/coolify.md) for connecting your GitHub repository.

---

## Summary

Today you learned:

| Skill | What you practiced |
|-------|-------------------|
| **Server updates** | `apt update && apt upgrade` |
| **Firewall config** | `ufw allow`, `ufw enable` |
| **Docker installation** | Official Docker repository setup |
| **Container deployment** | `docker run`, `docker ps`, `docker logs` |
| **CI/CD deployment** | GitHub Actions with SSH |

**Your deployment pipeline:**

```
Push code
    │
    ▼
┌─────────────────────────────────┐
│  GitHub Actions                 │
│                                 │
│  1. Build with Maven            │
│  2. Run tests                   │
│  3. Build Docker image          │
│  4. Push to GHCR                │
│  5. SSH to server               │
│  6. Pull new image              │
│  7. Restart container           │
└─────────────────────────────────┘
    │
    ▼
Your app is live at http://YOUR_SERVER_IP
```

---

## Common Issues

| Issue | Solution |
|-------|----------|
| "Permission denied" on SSH | Check SSH key, check secrets, check `authorized_keys` |
| Firewall blocks access | Ensure port 80/443 is allowed (`sudo ufw status`) |
| Container won't start | Check logs: `docker logs myapp` |
| Old container still running | Stop it: `docker stop myapp && docker rm myapp` |
| GHCR authentication fails | Check PAT has `read:packages` scope |
| Port already in use | Stop existing container or use different port |

---

## What's Next?

**Post-class tasks:**
- Configure a custom domain
- Set up HTTPS with Let's Encrypt
- Add monitoring and alerts

Your application is now deployed! The post-class tasks will make it production-ready.
