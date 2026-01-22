# Pre-class Reading: Introduction to Docker and Linux

**Estimated reading time: 45-60 minutes**

---

## The Problem Docker Solves

Have you ever heard (or said) any of these?

> "It works on my machine!"

> "I spent two days setting up the development environment."

> "The app works in development but crashes in production."

> "I can't run the old project anymore - something about Python versions."

These are real problems that software developers face every day. Let's understand why they happen.

### The Dependency Problem

Every application depends on other software:
- A Java application needs a specific Java version
- A Python script needs Python plus various packages
- A web application might need Node.js, a database, and more

When you install these on your computer, they become part of your system. Problems arise when:

1. **Different projects need different versions** - Project A needs Java 11, Project B needs Java 17
2. **Your computer differs from the server** - You develop on Windows, the server runs Linux
3. **Team members have different setups** - It works for you but not your colleague
4. **Time passes** - A project that worked last year doesn't work now because dependencies changed

### The Traditional Solution: Virtual Machines

Before Docker, the solution was **Virtual Machines (VMs)**:

```
┌─────────────────────────────────────────┐
│           Your Computer (Host)           │
│  ┌─────────────────────────────────────┐ │
│  │        Virtual Machine               │ │
│  │  ┌─────────────────────────────────┐ │ │
│  │  │     Guest Operating System      │ │ │
│  │  │  (Full Windows/Linux install)   │ │ │
│  │  ├─────────────────────────────────┤ │ │
│  │  │     Your Application            │ │ │
│  │  └─────────────────────────────────┘ │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

VMs work, but they have drawbacks:
- **Heavy**: Each VM runs a complete operating system (several GB)
- **Slow**: Takes minutes to start
- **Resource-hungry**: Uses lots of RAM and CPU
- **Hard to share**: Sending a VM to a colleague means sending huge files

### The Docker Solution: Containers

Docker takes a different approach with **containers**:

```
┌─────────────────────────────────────────┐
│           Your Computer (Host)           │
│           Host Operating System          │
│              Docker Engine               │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│  │Container1│ │Container2│ │Container3│ │
│  │   App A  │ │   App B  │ │   App C  │ │
│  └──────────┘ └──────────┘ └──────────┘ │
└─────────────────────────────────────────┘
```

Containers share the host's operating system kernel, making them:
- **Lightweight**: Megabytes instead of gigabytes
- **Fast**: Start in seconds, not minutes
- **Efficient**: Can run many containers on one machine
- **Portable**: Easy to share and run anywhere

---

## What is Docker?

Docker is a platform for developing, shipping, and running applications in containers.

### Key Components

**Docker Engine**: The software that runs on your computer and manages containers.

**Docker Desktop**: A user-friendly application for Windows and Mac that includes Docker Engine and other tools.

**Docker Hub**: A public registry (like GitHub, but for Docker images) where you can find and share container images.

---

## Images vs Containers

This distinction is crucial. Many beginners confuse these terms.

### Image

An **image** is a read-only template containing:
- A base operating system (usually Linux)
- Your application code
- All dependencies needed to run the application

Think of an image as a **recipe** or **blueprint**. You can't eat a recipe, but you can use it to make a meal.

**Key properties:**
- Immutable (cannot be changed once created)
- Can be stored and shared
- Has a name and tag (e.g., `ubuntu:22.04`, `nginx:latest`)

### Container

A **container** is a running instance of an image.

Think of a container as the **actual meal** made from the recipe. You can make multiple meals from the same recipe.

**Key properties:**
- Created from an image
- Has its own isolated filesystem, network, and processes
- Can be started, stopped, and deleted
- Changes inside a container don't affect the original image

### Analogy: Classes and Objects

If you know object-oriented programming:
- **Image** = Class (the definition)
- **Container** = Object (an instance of the class)

You can create many containers from one image, just like you can create many objects from one class.

---

## Docker Architecture

Here's how Docker works:

```
┌─────────────────────────────────────────────────────────────┐
│                        Your Computer                         │
│                                                              │
│  ┌──────────────┐        ┌───────────────────────────────┐  │
│  │ Docker CLI   │───────▶│       Docker Daemon           │  │
│  │ (commands)   │        │   (background service)        │  │
│  └──────────────┘        │                               │  │
│                          │  ┌─────────┐  ┌─────────┐     │  │
│                          │  │Container│  │Container│     │  │
│                          │  └─────────┘  └─────────┘     │  │
│                          └───────────────────────────────┘  │
│                                      │                       │
└──────────────────────────────────────│───────────────────────┘
                                       │
                                       ▼
                          ┌───────────────────────┐
                          │     Docker Hub        │
                          │  (image registry)     │
                          │                       │
                          │  nginx  mysql  ubuntu │
                          │  python redis  node   │
                          └───────────────────────┘
```

1. **Docker CLI**: The command-line tool you use to interact with Docker
2. **Docker Daemon**: A background service that manages images and containers
3. **Docker Hub**: Remote registry where images are stored

When you run `docker run nginx`:
1. Docker checks if the `nginx` image exists locally
2. If not, it downloads (pulls) the image from Docker Hub
3. Docker creates a container from the image
4. The container starts running

---

## Introduction to Linux

Most Docker images are based on Linux. Even if you use Windows or Mac, when you're inside a container, you're in a Linux environment.

### Why Linux?

- **Servers run Linux**: Over 90% of servers on the internet run Linux
- **Containers are Linux-native**: Docker uses Linux kernel features
- **It's free and open-source**: No licensing costs

### The Shell

The **shell** is a command-line interface where you type commands. When you're "inside" a container, you're using a shell.

The most common shell is **bash** (Bourne Again Shell).

```
user@container:~$ _
```

This is a **prompt** - it's waiting for you to type a command.

### Basic Linux Commands

Here are the essential commands you'll learn this week:

| Command | Description | Example |
|---------|-------------|---------|
| `pwd` | Print working directory (where am I?) | `pwd` |
| `ls` | List files and directories | `ls`, `ls -la` |
| `cd` | Change directory | `cd /home`, `cd ..` |
| `cat` | Display file contents | `cat file.txt` |
| `mkdir` | Create a directory | `mkdir mydir` |
| `rm` | Remove files | `rm file.txt` |
| `cp` | Copy files | `cp file.txt copy.txt` |
| `mv` | Move or rename files | `mv old.txt new.txt` |
| `echo` | Print text | `echo "Hello"` |

### The Linux Filesystem

Linux organizes everything in a tree structure starting from `/` (root):

```
/
├── bin/        # Essential commands (ls, cat, etc.)
├── etc/        # Configuration files
├── home/       # User home directories
│   └── user/   # Your files go here
├── tmp/        # Temporary files
├── usr/        # User programs
│   └── bin/    # More commands
└── var/        # Variable data (logs, databases)
```

**Key differences from Windows:**
- Uses `/` instead of `\` for paths
- No drive letters (C:, D:) - everything under `/`
- Case-sensitive: `File.txt` and `file.txt` are different files

---

## Common Docker Commands

Here's a preview of commands you'll use:

| Command | Description |
|---------|-------------|
| `docker run <image>` | Create and start a container |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker stop <container>` | Stop a running container |
| `docker rm <container>` | Remove a container |
| `docker images` | List downloaded images |
| `docker pull <image>` | Download an image from Docker Hub |
| `docker exec -it <container> bash` | Open a shell inside a running container |

---

## What's Next?

In the exercises, you will:
1. Install Docker on your computer
2. Run your first container
3. Explore a container's Linux environment
4. Practice basic commands

Don't worry if everything doesn't click immediately. Understanding comes with practice, and we'll reinforce these concepts in class.

---

## Self-Check Questions

Before moving to the exercises, make sure you can answer these:

1. What problem does Docker solve?
2. What is the difference between a Docker image and a container?
3. Where are Docker images stored and shared?
4. Why do most Docker containers run Linux?
5. What command shows your current directory in Linux?

<details>
<summary>Click to reveal answers</summary>

1. Docker solves the "it works on my machine" problem by packaging applications with their dependencies in containers that run the same everywhere.

2. An **image** is a read-only template (like a recipe). A **container** is a running instance of an image (like a meal made from the recipe).

3. Docker images are stored and shared on **Docker Hub** (or other registries).

4. Most containers run Linux because: servers run Linux, containers use Linux kernel features, and Linux is free.

5. `pwd` (print working directory)

</details>

---

## Further Reading (Optional)

If you want to learn more:
- [Docker Overview](https://docs.docker.com/get-started/overview/) - Official Docker documentation
- [What is a Container?](https://www.docker.com/resources/what-container/) - Docker's explanation

---

**Next step**: Continue to [exercises.md](exercises.md) to install Docker and run your first container.
