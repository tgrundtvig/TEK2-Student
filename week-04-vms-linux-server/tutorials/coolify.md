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

- Building your application from source or Docker image
- Deploying to servers
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
- **Your app URL will be:** `https://<your-app-name>.apps.tobiasgrundtvig.dk`
- **HTTPS:** Automatic — all HTTP traffic is redirected to HTTPS
- **Wildcard DNS:** Already configured, so any subdomain works immediately

For example, if you deploy an app called `hello-maven`, it will be available at:

```
https://hello-maven.apps.tobiasgrundtvig.dk
```

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
https://coolify.apps.tobiasgrundtvig.dk
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

### Option A: Deploy from Docker Image (Recommended)

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
   https://YOUR-APP-NAME.apps.tobiasgrundtvig.dk
   ```
   Replace `YOUR-APP-NAME` with something unique (e.g., `hello-maven-anna`)
6. Click **"Deploy"**

### Option B: Deploy from Public Git Repository

If deploying directly from source:

1. Inside your project, click **"+ New"** → **"Public Repository"**
2. Select the **localhost** server
3. Enter your GitHub repository URL (e.g., `https://github.com/YOUR_USERNAME/hello-maven`)
4. Configure build:
   - If your repo has a Dockerfile, Coolify detects it automatically
   - Set the exposed port (e.g., 8080)
5. Under **"Domains"**, set:
   ```
   https://YOUR-APP-NAME.apps.tobiasgrundtvig.dk
   ```
6. Click **"Deploy"**

---

## Step 4: Access Your Application

After deployment completes (watch the build logs):

1. The status should show **"Running"**
2. Click the URL shown in your deployment dashboard
3. Your application should be live at `https://YOUR-APP-NAME.apps.tobiasgrundtvig.dk`

> **Important:** Choose a unique app name! If two groups pick the same subdomain, only one will work. A good pattern is `<app>-<group-name>`, e.g., `hello-maven-group1`.

---

## Step 5: Configure Environment Variables (If Needed)

If your application needs environment variables:

1. Go to your resource and click **"Environment Variables"**
2. Add variables as key-value pairs
3. Check **"Build"** if the variable is needed during build
4. Save and redeploy

---

## Step 6: Set Up Automatic Deployments

### Using Webhooks (Git Repository deployments)

If you deployed from a Git repository:

1. Go to your resource settings
2. Find **"Webhooks"** section
3. Enable **"Deploy on push"**
4. Now pushing to your `main` branch triggers a redeployment

### Manual Redeployment

To deploy manually at any time:

1. Go to your resource
2. Click **"Redeploy"**

---

## Viewing Logs

1. Go to your resource
2. Click **"Logs"** to see live application output
3. For build logs, click on a deployment in the history

This is useful for debugging if your app doesn't start correctly.

---

## Comparing Coolify to VM Deployment

| Aspect | VM Approach | Coolify Approach |
|--------|-------------|------------------|
| SSH access | Yes, you manage server | No, Coolify manages |
| Firewall config | You do it | Automatic |
| Docker install | You do it | Pre-installed |
| HTTPS setup | You do it (Certbot) | Automatic |
| DNS config | You do it | Wildcard already set up |
| Deployment | CI/CD with SSH | Push to Git or click Deploy |
| Server updates | You do it | Platform managed |
| Cost | Pay for VM | Provided by instructor |
| Learning | More Linux skills | More platform skills |

---

## Troubleshooting

### Build Fails

1. Check build logs — click on the failed deployment to see details
2. Common issues:
   - Missing Dockerfile in repository root
   - Wrong port configuration (must match what your app listens on)
   - Missing dependencies in Dockerfile

### Application Not Accessible

1. Check deployment status is **"Running"**
2. Verify the domain is set to `https://YOUR-APP-NAME.apps.tobiasgrundtvig.dk`
3. Verify the port matches what your application listens on
4. Check application logs for startup errors

### Image Pull Fails (GHCR)

1. Verify your PAT has the `read:packages` scope
2. Check the image name and tag are correct
3. Make sure the image exists: `docker pull ghcr.io/YOUR_USERNAME/hello-maven:latest`

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
- **Replace with**: Configure webhooks in Coolify (Step 6 above)
- No need for SSH keys or GitHub Actions deployment workflow
- Coolify handles the CI/CD pipeline

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
