# Coolify Deployment Tutorial

This tutorial walks you through deploying your application using Coolify, a self-hosted PaaS platform running on the instructor's server.

**Estimated time: 20-30 minutes**

---

## When to Use Coolify

Coolify is provided as a **backup option** for students who:

- Have trouble setting up cloud provider billing/accounts
- Want a simpler deployment path
- Need a fallback if VM setup fails
- Prefer a PaaS experience over server management

**Trade-off:** You'll get less Linux server administration experience, but you'll still learn deployment concepts.

---

## What is Coolify?

Coolify is an open-source alternative to platforms like Heroku, Vercel, or Netlify. It handles:

- Pulling and running your Docker images
- Routing traffic to your containers (via Traefik reverse proxy)
- Automatic HTTPS certificates
- Environment variables
- Monitoring and logs

**Key difference from VMs:**
- VM: You manage everything (OS, Docker, firewall, etc.)
- Coolify: Platform manages infrastructure; you focus on your app

---

## Our Setup

The instructor runs a Coolify instance with the following configuration:

- **Base domain:** `apps.tobiasgrundtvig.dk`
- **Your app URL will be:** `http://YOUR-APP-NAME.apps.tobiasgrundtvig.dk`
- **HTTPS:** Handled automatically by Coolify's Traefik proxy
- **Wildcard DNS:** Already configured, so any subdomain works immediately
- **Server:** Pre-configured by the instructor — you don't need to set up any servers

For example, if you deploy an app called `hello-maven`, it will be available at:

```
http://hello-maven.apps.tobiasgrundtvig.dk
```

> **Important:** When setting the domain in Coolify, always use `http://` (not `https://`). Coolify's Traefik proxy handles HTTPS automatically. Using `https://` in the domain field causes redirect loops (`ERR_TOO_MANY_REDIRECTS`).

---

## Prerequisites

- An invitation link from your instructor (one per student)
- GitHub repository with your application
- GitHub account

---

## Step 1: Create Your Account

Your instructor will give you a personal invitation link. This link connects you to your group's team in Coolify.

1. Open the invitation link in your browser
2. Create a password and complete registration
3. You're now logged in and part of your group's team

After this, you can always log in at:

```
https://coolify.tobiasgrundtvig.dk
```

> **Note:** Each group has its own team. You can only see and manage your own group's deployments — you can't accidentally break another group's work.

---

## Step 2: Create a New Project

1. Click **"Projects"** in the sidebar
2. Click **"+ Add"**
3. Name your project (e.g., `tek2-group1` or your group name)
4. Click **"Continue"**

---

## Step 3: Deploy Your Application

Choose the option that matches your setup:

- **Option A** — You have a Docker image in GHCR (single container app)
- **Option B** — You want to deploy directly from a public Git repository
- **Option C** — You have a multi-container app (e.g., app + database) using Docker Compose

### Option A: Deploy from Docker Image (Single Container)

If you have a Docker image in GHCR from Week 3:

1. Inside your project, click **"+ New"** → **"Docker Image"**
2. Select the **localhost** server
3. Configure:
   - **Image**: `ghcr.io/YOUR_USERNAME/hello-maven:latest`
   - **Port**: 8080 (or your app's port)
4. If your GHCR image is private, add registry credentials:
   - Registry URL: `ghcr.io`
   - Username: Your GitHub username
   - Password: Your Personal Access Token (with `read:packages` scope)
5. Under **"Domains"**, set your app URL:
   ```
   http://YOUR-APP-NAME.apps.tobiasgrundtvig.dk
   ```
   Replace `YOUR-APP-NAME` with something unique (e.g., `hello-maven-anna`)
6. Click **"Deploy"**

### Option B: Deploy from Public Git Repository (Single Container)

If deploying directly from source:

1. Inside your project, click **"+ New"** → **"Public Repository"**
2. Select the **localhost** server
3. Enter your GitHub repository URL (e.g., `https://github.com/YOUR_USERNAME/hello-maven`)
4. Configure build:
   - If your repo has a Dockerfile, Coolify detects it automatically
   - Set the exposed port (e.g., 8080)
5. Under **"Domains"**, set:
   ```
   http://YOUR-APP-NAME.apps.tobiasgrundtvig.dk
   ```
6. Click **"Deploy"**

### Option C: Deploy a Docker Compose Service (Multi-Container)

For applications with multiple containers (e.g., a web app + database + frontend), you deploy as a **Service** using Docker Compose.

**How it works:** You write a Docker Compose configuration that defines all your services, and Coolify runs them together. Coolify's Traefik proxy routes traffic to whichever service should be publicly accessible. The containers communicate with each other over an internal Docker network — no need to expose ports between them.

**Step-by-step:**

1. Inside your project, click **"+ New"** → **"Service"**
2. Select the **localhost** server
3. Coolify opens a compose editor. Paste your Docker Compose content directly into this editor. Here's an example for a Spring Boot app with a PostgreSQL database:
   ```yaml
   services:
     db:
       image: postgres:17
       environment:
         POSTGRES_DB: mydb
         POSTGRES_USER: postgres
         POSTGRES_PASSWORD: postgres
       volumes:
         - pgdata:/var/lib/postgresql/data

     app:
       image: ghcr.io/YOUR_USERNAME/YOUR_APP:latest
       pull_policy: always
       environment:
         DATABASE_URL: jdbc:postgresql://db:5432/mydb
         DATABASE_USERNAME: postgres
         DATABASE_PASSWORD: postgres
       depends_on:
         - db

   volumes:
     pgdata:
   ```
4. Click **"Save"** — Coolify will parse the compose file and show each service listed separately
5. Click **"Settings"** on the service that should be publicly accessible (e.g., your app)
6. Set the domain:
   ```
   http://YOUR-APP-NAME.apps.tobiasgrundtvig.dk
   ```
7. Make sure the port matches what your app listens on (e.g., 8080)
8. Click **"Deploy"**

**Three important rules for Docker Compose on Coolify:**

1. **No port bindings** — Do **not** add `ports: "80:80"` or similar host port mappings. Coolify's Traefik proxy handles all external routing. Port bindings will conflict with Traefik and other apps on the server.

2. **Always use `pull_policy: always`** — Add this to every service that uses a GHCR image (not needed for standard images like `postgres:17`). Without it, Coolify may reuse a stale cached image instead of pulling the latest version when you redeploy.

3. **Domain uses `http://`** — Always set domains with `http://`, never `https://`. Traefik handles HTTPS automatically.

---

## Step 4: Access Your Application

After deployment completes (watch the build logs):

1. The status should show **"Running"**
2. Click the URL shown in your deployment dashboard
3. Your application should be live at `http://YOUR-APP-NAME.apps.tobiasgrundtvig.dk`

> **Important:** Choose a unique app name! If two groups pick the same subdomain, only one will work. A good pattern is `<app>-<group-name>`, e.g., `hello-maven-group1`.

---

## Step 5: Configure Environment Variables (If Needed)

If your application needs environment variables:

1. Go to your resource and click **"Environment Variables"**
2. Add variables as key-value pairs
3. Check **"Build"** if the variable is needed during build
4. Save and redeploy

For Docker Compose services, you can also define environment variables directly in the compose file (as shown in the examples above).

---

## Step 6: Redeploy After Changes

### Manual Redeployment

When you push a new Docker image to GHCR:

1. Go to your resource in Coolify
2. Click **"Redeploy"**
3. Coolify pulls the latest image (if you have `pull_policy: always`) and restarts your containers

### Automatic Redeployment via CI/CD Webhook

You can trigger Coolify to redeploy automatically from your GitHub Actions pipeline. This way, every push to `main` builds your image, pushes it to GHCR, and then tells Coolify to redeploy — fully automatic.

**1. Get your webhook URL from Coolify:**

- Go to your resource in Coolify
- Look for the **"Webhooks"** section
- Copy the deploy webhook URL — it looks something like:
  ```
  https://coolify.tobiasgrundtvig.dk/api/v1/deploy?uuid=YOUR_UUID&force=true
  ```
- Make sure the URL ends with `&force=true` (change `false` to `true` if needed) — this ensures Coolify actually restarts the containers

**2. Create an API token in Coolify:**

- Switch to the **root team** (your instructor may need to do this for you)
- Go to **"Keys & Tokens"** in the sidebar → **"API Tokens"**
- Enter a description (e.g., `github-deploy`) and set permissions to **deploy**
- Click **"Create"**
- **Copy the token immediately** — it's only shown once!

**3. Add secrets to your GitHub repository:**

Go to your repository on GitHub → **Settings** → **Secrets and variables** → **Actions**, and add two secrets:

- `COOLIFY_WEBHOOK` — paste the webhook URL
- `COOLIFY_TOKEN` — paste the API token

**4. Add a deploy step to your GitHub Actions workflow:**

Add this job to your `.github/workflows/ci.yml`, after your image build job:

```yaml
  deploy:
    needs: build-and-push-image
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

> **Note:** The secrets are passed via `env:` variables, not directly in the `curl` command. This avoids issues with special characters (`&`, `|`) in the URL and token being misinterpreted by the shell.

Now every push to `main` will: build → test → push image to GHCR → trigger Coolify to pull the new image and restart.

---

## Viewing Logs

1. Go to your resource
2. Click **"Logs"** to see live application output
3. For build logs, click on a deployment in the history

This is useful for debugging if your app doesn't start correctly.

---

## How Multiple Apps Share One Server

You might wonder: if everyone deploys to the same server, how do all the apps work on port 80?

Coolify uses **Traefik**, a reverse proxy that sits in front of all your containers. When a request comes in for `hello-maven-group1.apps.tobiasgrundtvig.dk`, Traefik looks at the domain name and routes it to the right container. This means:

- Every team can deploy multiple apps to the same server
- Each app gets its own unique subdomain
- No port conflicts — Traefik handles all the routing
- HTTPS is handled automatically for all apps

This is the same pattern used by cloud platforms like Heroku and Vercel.

---

## Comparing Coolify to VM Deployment

| Aspect | VM Approach | Coolify Approach |
|--------|-------------|------------------|
| SSH access | Yes, you manage server | No, Coolify manages |
| Firewall config | You do it | Automatic |
| Docker install | You do it | Pre-installed |
| HTTPS setup | You do it (Certbot) | Automatic |
| DNS config | You do it | Wildcard already set up |
| Deployment | CI/CD with SSH | Click Deploy or webhook |
| Server updates | You do it | Platform managed |
| Cost | Pay for VM | Provided by instructor |
| Learning | More Linux skills | More platform skills |

---

## Troubleshooting

### "Too Many Redirects" Error

If your browser shows `ERR_TOO_MANY_REDIRECTS`:

1. Check the domain in your service settings — make sure it starts with `http://` not `https://`
2. Coolify's Traefik proxy handles HTTPS automatically. Setting `https://` in the domain causes a redirect loop

### Port Conflict on Deploy

If you see `Bind for 0.0.0.0:80 failed: port is already allocated`:

1. Remove any `ports` mappings from your Docker Compose file
2. Coolify's Traefik proxy handles external routing — you don't need to bind host ports
3. Save the compose file and redeploy

### Redeployed but Still Showing Old Version

If you redeploy but the app doesn't update:

1. Make sure your Docker Compose services have `pull_policy: always` — without this, Docker reuses the cached image instead of pulling the latest from GHCR
2. Edit the compose file in Coolify, add `pull_policy: always` to your GHCR image services, save, and redeploy

### CI/CD Deploy Step Fails with "Malformed URL"

If the deploy step in GitHub Actions fails with `curl: (3) URL rejected: Malformed input to a URL function`:

1. Make sure the secrets are passed via `env:` variables, not inline in the `curl` command
2. The webhook URL contains `&` and the token contains `|` — these characters break when expanded directly in the shell

### Build Fails

1. Check build logs — click on the failed deployment to see details
2. Common issues:
   - Missing Dockerfile in repository root
   - Wrong port configuration (must match what your app listens on)
   - Missing dependencies in Dockerfile

### Application Not Accessible

1. Check deployment status is **"Running"**
2. Verify the domain is set to `http://YOUR-APP-NAME.apps.tobiasgrundtvig.dk`
3. Verify the port matches what your application listens on
4. Check application logs for startup errors

### Image Pull Fails (GHCR)

1. Verify your PAT has the `read:packages` scope
2. Check the image name and tag are correct
3. Make sure the image exists: `docker pull ghcr.io/YOUR_USERNAME/hello-maven:latest`

### Container Starts but App Not Reachable

1. Make sure the port in Coolify's settings matches your app's listening port
2. For Docker Compose services: make sure you set the domain on the correct service (the one that should be publicly accessible)
3. Check your app's logs — it might be crashing on startup (e.g., can't connect to database)

---

## Class Exercises Adaptation

If using Coolify instead of a VM, adapt the class exercises:

### Exercise 2: Firewall
- **Skip**: Coolify handles this automatically

### Exercise 3: Install Docker
- **Skip**: Docker is pre-installed on the server

### Exercise 4: Deploy Application
- **Replace with**: Deploy via Coolify web interface (Step 3 above)

### Exercise 5: Automatic Deployment
- **Replace with**: Set up webhook-based deployment (Step 6 above)

---

## Limitations

Be aware of Coolify limitations compared to VMs:

1. **No SSH access** — you can't log into the server directly
2. **Less control** — the platform makes infrastructure decisions for you
3. **Shared server** — your group shares the server with other groups (but each group's deployments are isolated)
4. **Less learning** — fewer Linux administration skills gained

---

## When to Use a VM Instead

Consider switching to a VM if you:

- Want to learn Linux server administration (recommended!)
- Need SSH access for debugging
- Want full control over the environment
- Are preparing for DevOps/SRE roles

The VM approach is more valuable for learning, but Coolify is a valid option to get your application deployed quickly.
