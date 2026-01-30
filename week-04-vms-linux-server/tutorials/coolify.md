# Coolify Deployment Tutorial

This tutorial walks you through deploying your application using Coolify, a self-hosted PaaS platform.

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

## Prerequisites

- Instructor-provided Coolify access (URL and login)
- GitHub repository with your application
- GitHub account

---

## Step 1: Request Coolify Access

Contact your instructor to get:

1. Coolify URL (e.g., `https://coolify.tek2.example.com`)
2. Login credentials or invitation

**Note:** Coolify access is provided on request for students who need it.

---

## Step 2: Log In to Coolify

1. Go to the Coolify URL provided by your instructor
2. Click "Login" or "Sign Up" (depending on setup)
3. Enter your credentials

---

## Step 3: Connect GitHub

### Add GitHub Integration

1. In Coolify, go to "Sources" (or "Git Sources")
2. Click "Add New Source" → "GitHub App"
3. Follow the OAuth flow to authorize Coolify
4. Grant access to your repositories

### Alternative: Manual Repository

If GitHub integration isn't available:

1. Go to your project
2. Click "New Resource" → "Public Repository"
3. Enter your repository URL manually

---

## Step 4: Create a New Project

1. Click "Projects" in the sidebar
2. Click "New Project"
3. Name: `tek2-deployment`
4. Click "Create"

---

## Step 5: Deploy Your Application

### Option A: Deploy from Docker Image (Recommended)

If you have a Docker image in GHCR from Week 3:

1. In your project, click "New Resource"
2. Select "Docker Image"
3. Configure:
   - **Image**: `ghcr.io/YOUR_USERNAME/hello-maven:latest`
   - **Port**: 8080 (or your app's port)
4. If GHCR is private, add credentials:
   - Registry URL: `ghcr.io`
   - Username: Your GitHub username
   - Password: Your Personal Access Token (with `read:packages` scope)
5. Click "Deploy"

### Option B: Deploy from GitHub Repository

If deploying directly from source:

1. In your project, click "New Resource"
2. Select your GitHub source
3. Choose your repository
4. Configure build:
   - For Dockerfile: Select "Dockerfile" build type
   - For buildpacks: Let Coolify auto-detect
5. Set the exposed port
6. Click "Deploy"

---

## Step 6: Configure Environment Variables

If your application needs environment variables:

1. Go to your deployment
2. Click "Environment Variables"
3. Add variables:
   - Key: `DATABASE_URL`
   - Value: `your-value`
   - Check "Build" if needed during build
   - Check "Preview" for preview deployments
4. Save and redeploy

---

## Step 7: Access Your Application

After deployment completes:

1. Coolify assigns a domain automatically (or uses custom domain)
2. Find the URL in your deployment dashboard
3. Click the URL to open your application

Example URL: `https://hello-maven-abc123.coolify.example.com`

---

## Step 8: Set Up Automatic Deployments

### Enable Auto-Deploy from GitHub

1. Go to your deployment settings
2. Find "Automatic Deployments" or "Webhooks"
3. Enable "Deploy on push"
4. Select which branch triggers deployment (usually `main`)

Now, pushing to GitHub automatically deploys your new version!

### Manual Deployment

To deploy manually:

1. Go to your deployment
2. Click "Redeploy" or "Deploy"

---

## Viewing Logs

1. Go to your deployment
2. Click "Logs" or "Application Logs"
3. View real-time logs from your application

For build logs:
1. Click on a deployment in history
2. View "Build Logs"

---

## Monitoring

Coolify provides basic monitoring:

1. Go to your deployment
2. Click "Monitoring" or "Metrics"
3. View CPU, memory, and network usage

---

## Custom Domain (Optional)

If you have a custom domain:

1. Go to your deployment settings
2. Find "Domains" section
3. Add your domain: `myapp.example.com`
4. Create DNS record pointing to Coolify's server
5. Coolify automatically provisions SSL certificate

---

## Comparing Coolify to VM Deployment

| Aspect | VM Approach | Coolify Approach |
|--------|-------------|------------------|
| SSH access | Yes, you manage server | No, Coolify manages |
| Firewall config | You do it | Automatic |
| Docker install | You do it | Pre-installed |
| HTTPS setup | You do it (Certbot) | Automatic |
| Deployment | CI/CD with SSH | Push to Git or click Deploy |
| Server updates | You do it | Platform managed |
| Cost | Pay for VM | Provided by instructor |
| Learning | More Linux skills | More platform skills |

---

## Troubleshooting

### Build Fails

1. Check build logs for error messages
2. Common issues:
   - Missing Dockerfile
   - Wrong port configuration
   - Missing dependencies

### Application Not Accessible

1. Check deployment status is "Running"
2. Verify port configuration matches your app
3. Check application logs for errors

### GitHub Connection Issues

1. Re-authorize the GitHub App
2. Check repository permissions
3. Try manual repository URL

### Image Pull Fails

For private GHCR images:
1. Verify credentials are correct
2. Check PAT has `read:packages` scope
3. Verify image name and tag

---

## Class Exercises Adaptation

If using Coolify instead of a VM, adapt the class exercises:

### Exercise 2: Firewall
- **Skip**: Coolify handles this automatically

### Exercise 3: Install Docker
- **Skip**: Docker is pre-installed

### Exercise 4: Deploy Application
- **Replace with**: Deploy via Coolify web interface (Step 5 above)

### Exercise 5: Automatic Deployment
- **Replace with**: Configure auto-deploy in Coolify (Step 8 above)
- No need for SSH keys or deployment workflow
- Coolify handles the CI/CD

---

## Limitations

Be aware of Coolify limitations compared to VMs:

1. **No SSH access**: Can't log into the server directly
2. **Less control**: Platform makes some decisions for you
3. **Dependency on platform**: If Coolify is down, can't deploy
4. **Less learning**: Fewer Linux administration skills gained

---

## When to Use a VM Instead

Consider switching to a VM if you:

- Want to learn Linux server administration
- Need SSH access for debugging
- Want full control over the environment
- Are preparing for DevOps/SRE roles

The VM approach is more valuable for learning, but Coolify is a valid option to get your application deployed.

---

## Next Steps

After deploying with Coolify:

1. Verify your application is accessible
2. Test automatic deployments by pushing a change
3. Explore Coolify's monitoring and logging features

For post-class tasks:
- **Custom domain**: Configure in Coolify (HTTPS is automatic)
- **Monitoring**: Already built into Coolify
- **Multi-container**: Use Coolify's Docker Compose support
