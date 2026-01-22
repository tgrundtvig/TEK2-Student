# Week 3: Maven + GitHub Actions

## Overview

This week we complete the DevOps foundation by adding **automation**. You'll learn how to:

1. Build Java projects with **Maven** (the standard Java build tool)
2. Automate builds and tests with **GitHub Actions** (CI/CD)

By the end of this week, every push to your repository will automatically trigger a build and test cycle - catching errors before they reach production.

---

## Learning Objectives

After this week, you should be able to:

- [ ] Explain what CI/CD means and why it's valuable
- [ ] Create a Maven project with proper directory structure
- [ ] Write a `pom.xml` file with dependencies
- [ ] Build Java projects using Maven in Docker
- [ ] Write JUnit tests for Java code
- [ ] Create GitHub Actions workflows to automate builds
- [ ] Read workflow run logs and understand build results

---

## Prerequisites

From Weeks 1-2, you should have:

- [ ] Docker installed and running
- [ ] Git installed
- [ ] GitHub account with SSH keys configured (`ssh -T git@github.com` works)
- [ ] Basic understanding of Docker volumes (`-v` flag)

---

## Materials

| Resource | Description | Time |
|----------|-------------|------|
| [Quick Reference](quick-reference.md) | Commands and templates cheat sheet | Reference |
| [Pre-class Reading](pre-class/reading.md) | CI/CD concepts, Maven basics, GitHub Actions | 45-60 min |
| [Pre-class Exercises](pre-class/exercises.md) | Build Java project, set up CI | 30-60 min |
| [Class Exercises](class/exercises.md) | Maven caching, Docker builds, Docker Hub push | 3.5 hours |
| [Post-class Tasks](post-class/advanced-tasks.md) | Code quality, environments, releases | 1-2 hours |
| [Post-class Hints](post-class/hints.md) | Troubleshooting and detailed hints | Reference |

---

## Time Investment

| Phase | Time | Content |
|-------|------|---------|
| **Pre-class** | 1.5-2 hours | Reading + hands-on exercises |
| **Class** | 3.5 hours | Extend CI, Docker builds, deployment |
| **Post-class** | 1-2 hours | Advanced CI/CD tasks |

---

## Week 3 in Context

```
Week 1: Docker + Linux       →  Run containers
Week 2: Dockerfiles + Compose →  Build and orchestrate containers
Week 3: Maven + GitHub Actions →  Automate building and testing  ← YOU ARE HERE
Week 4: Cloud + Linux Server  →  Deploy to production
```

After this week, you'll have all the pieces needed to:
1. Write code
2. Build it into a Docker image
3. Automatically test it on every push
4. (Week 4) Deploy it to the cloud

---

## What You'll Build

By the end of pre-class exercises, you'll have:

1. **A Java project** with Maven build configuration
2. **Unit tests** that verify your code works
3. **A GitHub repository** with your project
4. **A CI pipeline** that builds and tests automatically on every push

---

## Getting Help

If you run into problems:

1. Check the troubleshooting section in [exercises.md](pre-class/exercises.md)
2. Note the exact error message
3. Bring questions to class - we'll troubleshoot together

---

## Quick Links

- [Maven Documentation](https://maven.apache.org/guides/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [JUnit 4 Documentation](https://junit.org/junit4/)
