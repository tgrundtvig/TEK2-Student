# Week 1: Docker + Linux Basics

## Overview

This week introduces containerization with Docker and basic Linux command-line skills. Docker containers provide an ideal environment to learn Linux commands safely - if you make a mistake, simply delete the container and start fresh.

By the end of this week, you will understand why Docker exists, how to run containers, and how to navigate a Linux environment.

## Learning Objectives

By the end of this week, you should be able to:

1. **Explain** why containerization exists and what problems it solves
2. **Distinguish** between Docker images and containers
3. **Run** containers from existing images on Docker Hub
4. **Execute** basic Linux commands for navigation and file operations
5. **Interact** with running containers using `docker exec`
6. **Manage** container lifecycle (start, stop, remove)

## Time Allocation

| Phase | Duration | Focus |
|-------|----------|-------|
| Pre-class | 1-2 hours | Reading + Docker installation + first exercises |
| Class | 3.5 hours | Hands-on exploration + Linux commands + guided exercises |
| Post-class | 1-2 hours | Advanced exploration + documentation |

## Prerequisites

- A computer with Windows 10/11, macOS, or Linux
- Administrator/sudo access to install software
- Basic understanding of what a terminal/command line is

## Videos (pick any — each is self-contained)

All videos live at **[tek2.apps.tobiasgrundtvig.dk/week-01](https://tek2.apps.tobiasgrundtvig.dk/#week-01)**.

| 🎬 Video | Duration | What it covers |
|---|---|---|
| [Why Docker exists](https://tek2.apps.tobiasgrundtvig.dk/week-01/why-docker-exists/) | 5 min | The "works on my machine" problem in three scenarios |
| [VMs vs Containers](https://tek2.apps.tobiasgrundtvig.dk/week-01/vms-vs-containers/) | 5 min | What's actually shared — kernel, userspace, overhead |
| [Image vs Container](https://tek2.apps.tobiasgrundtvig.dk/week-01/image-vs-container/) | 5 min | The recipe vs the meal — the single most confused concept |
| [`docker run` anatomy](https://tek2.apps.tobiasgrundtvig.dk/week-01/docker-run-anatomy/) | 5 min | Every flag explained: image, tag, -it, -d, --name, -p |
| [`run` vs `exec`](https://tek2.apps.tobiasgrundtvig.dk/week-01/run-vs-exec/) | 3 min | When to make a new container, when to enter an existing one |
| [The Linux filesystem](https://tek2.apps.tobiasgrundtvig.dk/week-01/linux-filesystem/) | 5 min | The single tree under `/`, pwd/ls/cd, important directories |
| [Container lifecycle](https://tek2.apps.tobiasgrundtvig.dk/week-01/container-lifecycle/) | 4 min | States and transitions: created → running → exited → removed |

## Materials

| File | Description |
|------|-------------|
| [quick-reference.md](quick-reference.md) | One-page cheat sheet (print this!) |
| [pre-class/reading.md](pre-class/reading.md) | Conceptual introduction to Docker |
| [pre-class/exercises.md](pre-class/exercises.md) | Installation and first steps |
| [class/exercises.md](class/exercises.md) | Guided hands-on exercises |
| [post-class/advanced-tasks.md](post-class/advanced-tasks.md) | Challenges to deepen understanding |
| [post-class/hints.md](post-class/hints.md) | Hints for advanced tasks |

## Key Concepts

- **Container**: A lightweight, isolated environment that runs an application
- **Image**: A read-only template used to create containers (like a recipe)
- **Docker Hub**: Public registry where images are stored and shared
- **Shell**: Command-line interface to interact with Linux

## What's Next

In Week 2, we'll build on this foundation to create our own Docker images using Dockerfiles and orchestrate multiple containers with Docker Compose.
