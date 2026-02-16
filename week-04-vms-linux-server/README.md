# Week 4: VMs and Linux Server Administration

## Overview

This week you'll deploy your Docker containers to a **real cloud server**. You'll learn how to:

1. Create and manage a **Virtual Machine** (VM) in the cloud
2. Connect securely using **SSH**
3. Administer a **Linux server** (packages, services, firewall)
4. Install **Docker** on your server
5. Deploy containers and set up **automated deployment** with CI/CD

By the end of this week, pushing code to GitHub will automatically deploy your application to a live server.

---

## Learning Objectives

After this week, you should be able to:

- [ ] Explain the difference between VMs, containers, and PaaS platforms
- [ ] Create and configure SSH keys for secure server access
- [ ] Connect to a remote server via SSH
- [ ] Manage Linux services with `systemctl`
- [ ] Configure a firewall with `ufw`
- [ ] Install and manage packages with `apt`
- [ ] Install Docker on a Linux server
- [ ] Deploy containers to a remote server
- [ ] Set up GitHub Actions to auto-deploy on push

---

## Prerequisites

From Weeks 1-3, you should have:

- [ ] Docker experience (running, building, compose)
- [ ] GitHub account with SSH keys configured
- [ ] Basic GitHub Actions knowledge (workflows, triggers)
- [ ] A Docker image pushed to GitHub Container Registry (GHCR)

---

## Materials

| Resource | Description | Time |
|----------|-------------|------|
| [Quick Reference](quick-reference.md) | SSH, systemctl, ufw, apt commands cheat sheet | Reference |
| [Pre-class Reading](pre-class/reading.md) | VMs vs containers, SSH, cloud providers | 45-60 min |
| [Pre-class Exercises](pre-class/exercises.md) | Create VM, configure SSH, verify access | 1-2 hours |
| [Class Exercises](class/exercises.md) | Server setup, Docker, deploy, CI/CD | 3.5 hours |
| [Post-class Tasks](post-class/advanced-tasks.md) | Custom domain, HTTPS, monitoring | 1-2 hours |
| [Post-class Hints](post-class/hints.md) | Troubleshooting and detailed hints | Reference |

### Provider Tutorials

Choose **one** provider and follow its tutorial:

| Tutorial | Description |
|----------|-------------|
| [Azure](tutorials/azure.md) | Microsoft Azure Virtual Machines |
| [Digital Ocean](tutorials/digitalocean.md) | Digital Ocean Droplets |
| [Hetzner](tutorials/hetzner.md) | Hetzner Cloud Servers |
| [Google Cloud](tutorials/google-cloud.md) | Google Cloud Compute Engine |
| [Oracle Cloud](tutorials/oracle-cloud.md) | Oracle Cloud Infrastructure |
| [Coolify](tutorials/coolify.md) | Self-hosted PaaS (backup option) |

### Full-Stack Project Tutorial

| Tutorial | Description |
|----------|-------------|
| [Todo App on Azure](tutorials/todo-app-azure.md) | Fork and deploy a 3-service app (Nginx + Spring Boot + PostgreSQL) with CI/CD |

---

## Time Investment

| Phase | Time | Content |
|-------|------|---------|
| **Pre-class** | 2-3 hours | Reading + VM setup |
| **Class** | 3.5 hours | Server admin, Docker, deployment |
| **Post-class** | 1-2 hours | Domain, HTTPS, monitoring |

---

## Week 4 in Context

```
Week 1: Docker + Linux       →  Run containers locally
Week 2: Dockerfiles + Compose →  Build and orchestrate containers
Week 3: Maven + GitHub Actions →  Automate building and testing
Week 4: VMs + Linux Server    →  Deploy to production  ← YOU ARE HERE
```

After this week, you'll have a complete CI/CD pipeline:
1. Write code locally
2. Push to GitHub
3. GitHub Actions builds and tests
4. Automatically deploys to your cloud server
5. Users can access your application

---

## What You'll Build

By the end of pre-class exercises, you'll have:

1. **A cloud server** running Linux (Ubuntu)
2. **SSH access** configured with your key
3. **Verified connectivity** from your computer

By the end of class, you'll have:

1. **A hardened server** with firewall configured
2. **Docker installed** and running
3. **Your application deployed** via Docker
4. **Automated deployment** triggered by git push

---

## Choosing Your Provider

You'll need a cloud server for this week. Here are your options:

| Provider | Cost | Pros | Cons |
|----------|------|------|------|
| **Azure** | Free credits available | Enterprise standard, good docs | Complex interface |
| **Digital Ocean** | ~$6/month | Simple, great tutorials | No free tier |
| **Hetzner** | ~$4/month | Cheapest, European servers | Less documentation |
| **Google Cloud** | Always Free VM available | Enterprise standard, browser SSH | Free VM limited to US regions |
| **Oracle Cloud** | Always Free (generous) | Best free tier (4 CPU, 24 GB RAM forever) | ARM architecture, capacity limits |
| **Coolify** | Free (provided) | Simplest, no billing | Less Linux experience |

**Recommendation:**
- If you have student credits or want enterprise experience → **Azure**
- If you want simplicity and great docs → **Digital Ocean**
- If you want the cheapest option → **Hetzner**
- If you want a powerful free-forever server → **Oracle Cloud**
- If you want a free VM and don't mind US regions → **Google Cloud**
- If you have billing issues or want a simpler path → **Coolify** (ask instructor)

---

## Getting Help

If you run into problems:

1. Check the [quick-reference.md](quick-reference.md) for common commands
2. Check the [hints.md](post-class/hints.md) for troubleshooting
3. Check your provider's documentation
4. Note the exact error message
5. Bring questions to class - we'll troubleshoot together

---

## Quick Links

- [SSH Documentation](https://www.openssh.com/manual.html)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Docker on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [GitHub Actions Deployment](https://docs.github.com/en/actions/deployment)
