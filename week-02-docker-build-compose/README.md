# Week 2: Docker Build & Compose

## Overview

This week solves three frustrations you likely experienced in Week 1:

1. **Data disappeared** when you removed the MySQL container (solved with **volumes**)
2. **Manual setup every time** - installing tools inside containers (solved with **Dockerfiles**)
3. **Too many flags** to remember for `docker run` (solved with **Docker Compose**)

By the end of this week, you'll build custom images and orchestrate multi-container applications.

## Learning Objectives

By the end of this week, you should be able to:

1. **Persist** container data using Docker volumes
2. **Write** Dockerfiles to create custom images
3. **Build** images from Dockerfiles using `docker build`
4. **Push** images to Docker Hub for sharing
5. **Define** multi-container applications using Docker Compose
6. **Manage** application lifecycle with Compose commands
7. **Troubleshoot** common permission issues in containers

## Time Allocation

| Phase | Duration | Focus |
|-------|----------|-------|
| Pre-class | 1-2 hours | Reading + Dockerfile basics + first build |
| Class | 3.5 hours | Volumes + Dockerfiles + Compose hands-on |
| Post-class | 1-2 hours | Real app deployment + optimization |

## Prerequisites

- Completed Week 1 (Docker + Linux Basics)
- Docker Desktop running
- GitHub account with SSH keys configured

## Materials

| File | Description |
|------|-------------|
| [quick-reference.md](quick-reference.md) | Week 2 cheat sheet (print this!) |
| [pre-class/reading.md](pre-class/reading.md) | Why volumes, Dockerfiles, and Compose |
| [pre-class/exercises.md](pre-class/exercises.md) | Build your first image |
| [class/exercises.md](class/exercises.md) | Guided hands-on exercises |
| [post-class/advanced-tasks.md](post-class/advanced-tasks.md) | Real applications + optimization |
| [post-class/hints.md](post-class/hints.md) | Hints for advanced tasks |

## Key Concepts

- **Volume**: Persistent storage that survives container removal
- **Dockerfile**: Recipe (text file) for building a custom image
- **Image Layer**: Cached step in the build process
- **Docker Compose**: Tool for defining multi-container applications
- **compose.yaml**: Configuration file for Compose

## What's Next

In Week 3, we'll learn CI/CD with Maven and GitHub Actions to automate building and deploying containers.
