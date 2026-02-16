# Tutorial: Fork and Deploy the Todo App on Azure

This tutorial walks you through forking a real 3-service web application and setting up a complete CI/CD pipeline to deploy it on your Azure VM.

**Estimated time: 60-90 minutes**

---

## What You'll Deploy

The **todo-app** is a full-stack web application with three Docker containers:

| Service | Technology | What it does |
|---------|-----------|--------------|
| **frontend** | Nginx | Serves the web interface, proxies API requests to the backend |
| **app** | Spring Boot (Java) | REST API for managing todos |
| **db** | PostgreSQL 17 | Stores the todo data |

The app is currently set up for deployment via Coolify. You'll modify it to deploy automatically to your Azure VM instead.

```
┌─────────────────────────────────────────────────┐
│  Your Azure VM                                   │
│                                                   │
│  ┌───────────┐   ┌───────────┐   ┌───────────┐  │
│  │ frontend  │──▶│    app    │──▶│    db      │  │
│  │  (nginx)  │   │ (Spring)  │   │(PostgreSQL)│  │
│  │  port 80  │   │ port 8080 │   │ port 5432  │  │
│  └───────────┘   └───────────┘   └───────────┘  │
│       ▲                                           │
│       │ port 80                                   │
└───────┼───────────────────────────────────────────┘
        │
    Internet
```

Only the frontend is exposed to the internet. It forwards `/api/` requests to the backend internally.

---

## Prerequisites

Before starting this tutorial, you should have:

- [ ] An Azure VM running with SSH access ([Azure tutorial](azure.md))
- [ ] Docker installed on your VM ([Class Exercise 3](../class/exercises.md#exercise-3-install-docker-20-minutes))
- [ ] Firewall configured with ports 22, 80, 443 open ([Class Exercise 2](../class/exercises.md#exercise-2-configure-the-firewall-15-minutes))
- [ ] A GitHub account

---

## Step 1: Fork the Repository (~5 minutes)

A **fork** creates your own copy of someone else's repository. You'll need your own copy so you can modify the CI/CD pipeline and push images to your own GitHub Container Registry.

1. Go to [https://github.com/tgrundtvig/todo-app](https://github.com/tgrundtvig/todo-app)
2. Click the **"Fork"** button (top right)
3. Keep the default settings (same name is fine)
4. Click **"Create fork"**

You now have `https://github.com/YOUR_USERNAME/todo-app`.

### Clone your fork locally

```bash
git clone git@github.com:YOUR_USERNAME/todo-app.git
cd todo-app
```

Replace `YOUR_USERNAME` with your actual GitHub username throughout this tutorial.

---

## Step 2: Update the Image Names (~10 minutes)

The original repository pushes Docker images to `ghcr.io/tgrundtvig/...`. You need to change these to use **your** GitHub username so the images end up in your own container registry.

### 2.1: Update the CI/CD workflow

Open `.github/workflows/ci.yml` in your editor. You'll see three things to change:

**Change 1** — Backend image tags (around line 47-49):

```yaml
# Change FROM:
tags: |
  ghcr.io/tgrundtvig/todo-app:latest
  ghcr.io/tgrundtvig/todo-app:${{ github.sha }}

# Change TO:
tags: |
  ghcr.io/YOUR_USERNAME/todo-app:latest
  ghcr.io/YOUR_USERNAME/todo-app:${{ github.sha }}
```

**Change 2** — Frontend image tags (around line 73-75):

```yaml
# Change FROM:
tags: |
  ghcr.io/tgrundtvig/todo-app-frontend:latest
  ghcr.io/tgrundtvig/todo-app-frontend:${{ github.sha }}

# Change TO:
tags: |
  ghcr.io/YOUR_USERNAME/todo-app-frontend:latest
  ghcr.io/YOUR_USERNAME/todo-app-frontend:${{ github.sha }}
```

> **Important:** Your GitHub username must be **lowercase** in image tags. GitHub Container Registry requires lowercase. If your username is `JohnDoe`, use `ghcr.io/johndoe/todo-app`.

**Change 3** — Replace the entire `deploy` job (lines 77-89). Delete the old Coolify deploy job:

```yaml
# DELETE this entire job:
  deploy:
    needs: [build-and-push-image, build-and-push-frontend]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Coolify deploy
        env:
          WEBHOOK_URL: ${{ secrets.COOLIFY_WEBHOOK }}
          COOLIFY_TOKEN: ${{ secrets.COOLIFY_TOKEN }}
        run: |
          curl -f -X GET "$WEBHOOK_URL" \
            -H "Authorization: Bearer $COOLIFY_TOKEN"
```

And replace it with this new deploy job that SSHes into your Azure VM:

```yaml
  deploy:
    needs: [build-and-push-image, build-and-push-frontend]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Azure VM
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_IP }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd ~/todo-app

            # Pull the latest images
            docker compose -f docker-compose.prod.yml pull

            # Restart with the new images
            docker compose -f docker-compose.prod.yml up -d

            # Clean up old images
            docker image prune -f
```

### 2.2: Update the production compose file

Open `docker-compose.prod.yml` and change the image names:

```yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_DB: tododb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}
    volumes:
      - pgdata:/var/lib/postgresql/data

  app:
    image: ghcr.io/YOUR_USERNAME/todo-app:latest
    environment:
      DATABASE_URL: jdbc:postgresql://db:5432/tododb
      DATABASE_USERNAME: postgres
      DATABASE_PASSWORD: ${DB_PASSWORD:-postgres}
    depends_on:
      - db

  frontend:
    image: ghcr.io/YOUR_USERNAME/todo-app-frontend:latest
    ports:
      - "80:80"
    depends_on:
      - app

volumes:
  pgdata:
```

Replace `YOUR_USERNAME` with your lowercase GitHub username in both image lines.

### 2.3: Verify your changes

Double-check that you've replaced `tgrundtvig` with your username in all three files. A quick search:

```bash
# This should return NO results if you replaced everything
grep -r "tgrundtvig" .github/ docker-compose.prod.yml
```

---

## Step 3: Set Up GitHub Secrets (~10 minutes)

Your CI/CD pipeline needs to know how to connect to your Azure VM. You'll store this information as GitHub secrets (encrypted variables that only GitHub Actions can read).

### 3.1: Create a deployment SSH key

On your **local machine**, create a dedicated key for GitHub Actions:

```bash
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/deploy_key -N ""
```

This creates two files:
- `~/.ssh/deploy_key` — the private key (GitHub Actions uses this)
- `~/.ssh/deploy_key.pub` — the public key (your server needs this)

### 3.2: Add the public key to your server

Copy the public key and add it to your Azure VM:

```bash
# View the public key
cat ~/.ssh/deploy_key.pub
```

SSH into your server and add it:

```bash
ssh azureuser@YOUR_SERVER_IP
echo "PASTE_THE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
exit
```

### 3.3: Add secrets to GitHub

Go to your forked repository on GitHub:

1. **Settings** → **Secrets and variables** → **Actions**
2. Click **"New repository secret"** and add each of these:

| Secret Name | Value | Example |
|-------------|-------|---------|
| `SERVER_IP` | Your Azure VM's public IP | `20.234.123.45` |
| `SERVER_USER` | Your SSH username | `azureuser` |
| `SSH_PRIVATE_KEY` | The entire contents of `~/.ssh/deploy_key` | Starts with `-----BEGIN OPENSSH PRIVATE KEY-----` |

To copy the private key:

```bash
cat ~/.ssh/deploy_key
```

Copy **everything**, including the `-----BEGIN` and `-----END` lines.

---

## Step 4: Prepare Your Azure VM (~15 minutes)

Your server needs Docker (you should already have this from the class exercises) and a few things set up for the todo-app.

### 4.1: Verify Docker is running

```bash
ssh azureuser@YOUR_SERVER_IP
docker --version
sudo systemctl status docker
```

If Docker isn't installed, follow [Class Exercise 3](../class/exercises.md#exercise-3-install-docker-20-minutes).

### 4.2: Authenticate with GitHub Container Registry

Your server needs to pull your Docker images from GHCR. Create a Personal Access Token (PAT) if you don't already have one:

1. Go to GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. Click **"Generate new token (classic)"**
3. Name: `server-docker-access`
4. Expiration: 90 days
5. Scopes: check **`read:packages`**
6. Click **"Generate token"**
7. **Copy the token immediately!**

Then log in on your server:

```bash
echo "YOUR_TOKEN" | docker login ghcr.io -u YOUR_USERNAME --password-stdin
```

You should see: `Login Succeeded`

### 4.3: Create the app directory and compose file

The deploy job expects the production compose file to be on the server at `~/todo-app/docker-compose.prod.yml`:

```bash
mkdir -p ~/todo-app
```

Create the compose file on the server. You can do this with `nano`:

```bash
nano ~/todo-app/docker-compose.prod.yml
```

Paste the following (replace `YOUR_USERNAME` with your lowercase GitHub username):

```yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_DB: tododb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}
    volumes:
      - pgdata:/var/lib/postgresql/data

  app:
    image: ghcr.io/YOUR_USERNAME/todo-app:latest
    environment:
      DATABASE_URL: jdbc:postgresql://db:5432/tododb
      DATABASE_USERNAME: postgres
      DATABASE_PASSWORD: ${DB_PASSWORD:-postgres}
    depends_on:
      - db

  frontend:
    image: ghcr.io/YOUR_USERNAME/todo-app-frontend:latest
    ports:
      - "80:80"
    depends_on:
      - app

volumes:
  pgdata:
```

Save with `Ctrl+O`, Enter, `Ctrl+X`.

> **Optional but recommended:** Set a real database password instead of the default. Create a `.env` file:
> ```bash
> echo "DB_PASSWORD=$(openssl rand -base64 24)" > ~/todo-app/.env
> ```
> The `${DB_PASSWORD:-postgres}` syntax means it uses `DB_PASSWORD` from the `.env` file if it exists, otherwise falls back to `postgres`.

---

## Step 5: Push and Deploy (~10 minutes)

Everything is in place. Time to push your changes and watch the pipeline run!

### 5.1: Commit and push

```bash
git add -A
git commit -m "Replace Coolify deploy with Azure VM deploy"
git push
```

### 5.2: Watch the pipeline

1. Go to your repository on GitHub
2. Click the **"Actions"** tab
3. You should see a workflow running called "CI/CD"

The pipeline has four jobs that run in sequence:

```
build-and-test          (compile and run tests)
       │
       ├──▶ build-and-push-image      (backend Docker image → GHCR)
       │
       └──▶ build-and-push-frontend   (frontend Docker image → GHCR)
                    │
                    ▼
                 deploy               (SSH into Azure VM, pull images, restart)
```

Click on the workflow run to see live logs for each job.

### 5.3: Verify it works

Once the pipeline completes (usually 3-5 minutes):

**From your server:**

```bash
ssh azureuser@YOUR_SERVER_IP
docker ps
```

You should see three containers running:

```
CONTAINER ID   IMAGE                                          STATUS
abc123...      ghcr.io/your_username/todo-app-frontend:latest Up 30 seconds
def456...      ghcr.io/your_username/todo-app:latest          Up 35 seconds
ghi789...      postgres:17                                     Up 40 seconds
```

**From your browser:**

Open `http://YOUR_SERVER_IP` — you should see the Todo application!

Try adding a few todos to verify the full stack works (frontend → backend → database).

---

## Step 6: Test Automatic Deployment (~5 minutes)

The whole point of CI/CD is that changes deploy automatically. Let's test it:

1. Open `frontend/index.html` in your editor
2. Change the heading (e.g., change `<h1>` text to something like "My Todo List")
3. Commit and push:
   ```bash
   git add frontend/index.html
   git commit -m "Update heading"
   git push
   ```
4. Watch the Actions tab — the pipeline should trigger automatically
5. After it completes, refresh your browser — you should see the new heading

That's it — you now have fully automated deployment!

---

## How It All Fits Together

Here's the complete flow from code change to live application:

```
You push code to GitHub
        │
        ▼
┌─────────────────────────────────────────┐
│         GitHub Actions Pipeline          │
│                                          │
│  1. Checkout code                        │
│  2. Build Java app with Maven            │
│  3. Run tests                            │
│  4. Build backend Docker image           │
│  5. Build frontend Docker image          │
│  6. Push both images to GHCR             │
│  7. SSH into your Azure VM               │
│  8. Pull the new images                  │
│  9. Restart all containers               │
└─────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────┐
│           Your Azure VM                  │
│                                          │
│  frontend (nginx:80) ──▶ app (8080)     │
│                           │              │
│                           ▼              │
│                     db (PostgreSQL)       │
└─────────────────────────────────────────┘
        │
        ▼
  Users visit http://YOUR_SERVER_IP
```

---

## Troubleshooting

### Pipeline fails at "Build and test"

The Java build or tests failed. Click on the failed job to see the Maven output. Common causes:
- Compilation errors in your code
- A test is failing

### Pipeline fails at "Build and push image"

Docker image build failed. Check:
- Is the `Dockerfile` intact? Don't modify it unless you know what you're doing
- GHCR permissions — the workflow needs `packages: write` permission (already set in the workflow)

### Pipeline fails at "Deploy to Azure VM"

The SSH connection to your server failed. Check:
1. **SERVER_IP** secret — is it correct? (Check Azure Portal for your VM's IP)
2. **SERVER_USER** secret — is it `azureuser` (or whatever you set during VM creation)?
3. **SSH_PRIVATE_KEY** secret — did you copy the entire private key including the `BEGIN` and `END` lines?
4. Is your VM running? (Check Azure Portal)
5. Is the public key in `~/.ssh/authorized_keys` on the server?

**Test the SSH key manually:**
```bash
ssh -i ~/.ssh/deploy_key azureuser@YOUR_SERVER_IP echo "Connection works"
```

### App is deployed but shows "502 Bad Gateway"

The frontend can't reach the backend. This usually means the backend hasn't finished starting yet.

1. Check backend logs: `docker logs todo-app-app-1`
2. Wait 15-20 seconds for Spring Boot to start (it's not instant)
3. Refresh the page

### App is deployed but "Connection refused" in browser

1. Is port 80 open in your Azure firewall? Check:
   - **Azure Portal:** VM → Networking → Inbound port rules → port 80 should be allowed
   - **Server firewall:** `sudo ufw status` should show port 80 allowed
2. Is the frontend container running? `docker ps`

### Database data is lost after redeploy

The `pgdata` Docker volume persists across container restarts. Data should survive redeployments. If it doesn't:
1. Make sure you're using `docker compose up -d` (not `docker compose down` first — that removes volumes with `-v`)
2. The deploy script in the workflow uses `up -d` which only recreates containers with new images, keeping volumes intact

### "Image not found" or "Manifest unknown"

Your images haven't been pushed to GHCR yet, or the image name is wrong:
1. Check that the build-and-push jobs succeeded in GitHub Actions
2. Go to your GitHub profile → **Packages** — you should see `todo-app` and `todo-app-frontend`
3. Make sure the image names in `docker-compose.prod.yml` on the server match exactly (lowercase username!)

### Images are private and server can't pull them

By default, GHCR packages from forked repos may be private. To make them public:
1. Go to your GitHub profile → **Packages**
2. Click on the package (e.g., `todo-app`)
3. **Package settings** → **Danger Zone** → **Change visibility** → **Public**
4. Repeat for `todo-app-frontend`

Alternatively, keep them private and make sure you ran `docker login ghcr.io` on the server (Step 4.2).

---

## What You've Learned

By completing this tutorial, you've practiced:

| Skill | What you did |
|-------|-------------|
| **Forking** | Created your own copy of a repository |
| **Docker Compose** | Orchestrating a 3-service application |
| **CI/CD pipeline** | Automated build, test, push, and deploy |
| **GitHub Container Registry** | Storing and pulling Docker images |
| **SSH deployment** | Using GitHub Actions to deploy via SSH |
| **Cloud deployment** | Running a production app on Azure |

This is a real-world deployment pattern used by many teams: push to main → automated pipeline → live on server.

---

## Optional Next Steps

Want to go further? Try these:

1. **Set a custom database password** — Create a `.env` file on the server (see Step 4.3)
2. **Add a custom domain** — Point a domain to your server IP and update the nginx config
3. **Add HTTPS** — Use Let's Encrypt with certbot (see [post-class tasks](../post-class/advanced-tasks.md))
4. **Make a code change** — Add a feature to the todo app and watch it deploy automatically
