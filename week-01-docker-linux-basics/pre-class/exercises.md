# Pre-class Exercises: Docker Installation and First Steps

**Estimated time: 30-60 minutes**

Complete these exercises before class. If you get stuck, note where you had problems - we'll address them at the start of class.

> **Tip:** If something isn't working, don't spend more than 10 minutes on it. Note the error message and move on - we'll troubleshoot together in class.

---

## Pro Tips for Command Line

Before you start, here are some essential shortcuts:

| Shortcut | What it does |
|----------|--------------|
| **Tab** | Autocomplete commands and file names |
| **Up Arrow** | Recall previous command |
| **Ctrl+C** | Cancel current command |
| **Ctrl+L** | Clear the screen |

These will save you a lot of typing!

---

## Exercise 1: Install Docker (~10 minutes)

### Windows

1. Download **Docker Desktop for Windows** from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
2. Run the installer
3. When prompted, ensure **WSL 2** is selected (recommended)
4. Restart your computer when prompted
5. Start Docker Desktop from the Start menu
6. Wait for Docker to finish starting (the whale icon in the system tray should stop animating)

### macOS

1. Download **Docker Desktop for Mac** from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
   - Choose the correct version for your chip (Intel or Apple Silicon)
2. Open the downloaded `.dmg` file
3. Drag Docker to your Applications folder
4. Open Docker from Applications
5. Follow the prompts to complete setup
6. Wait for Docker to finish starting (check the whale icon in the menu bar)

### Linux (Ubuntu/Debian)

Run these commands in your terminal:

```bash
# Update package list
sudo apt update

# Install Docker
sudo apt install docker.io

# Start Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to the docker group (so you don't need sudo)
sudo usermod -aG docker $USER

# Log out and log back in for group changes to take effect
```

> **Note for Windows/Mac users:** Docker Desktop also has a graphical interface. Feel free to explore it, but we'll primarily use the command line in this course because it's more powerful and works the same everywhere.

---

## Exercise 2: Verify Installation (~5 minutes)

Open a terminal (Command Prompt or PowerShell on Windows, Terminal on Mac/Linux).

### Step 2.1: Check Docker version

```bash
docker --version
```

**Expected output** (version numbers may vary):
```
Docker version 24.0.7, build afdd53b
```

### Self-check
- [ ] Command runs without error
- [ ] You see a version number

### Step 2.2: Run the hello-world container

```bash
docker run hello-world
```

**Expected output**:
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
...
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

### Self-check
- [ ] Docker downloaded the image automatically
- [ ] You see "Hello from Docker!"

### What just happened?

1. You asked Docker to run the `hello-world` image
2. Docker didn't find it locally, so it **pulled** (downloaded) it from Docker Hub
3. Docker created a **container** from the image
4. The container ran, printed its message, and exited

---

## Exercise 3: Run an Interactive Container (~10 minutes)

Now let's run a container you can explore inside.

### Step 3.1: Start an Ubuntu container

```bash
docker run -it ubuntu:22.04 bash
```

Let's break down this command:
- `docker run` - create and start a container
- `-it` - interactive mode with a terminal
- `ubuntu:22.04` - the image to use (Ubuntu version 22.04)
- `bash` - the command to run inside (the bash shell)

**Expected output**:
```
root@a1b2c3d4e5f6:/#
```

You are now **inside** a Linux container! The prompt shows:
- `root` - you're the root (administrator) user
- `a1b2c3d4e5f6` - the container's ID (yours will be different)
- `/` - you're in the root directory
- `#` - indicates root user

### Step 3.2: Explore the container

Try these commands inside the container:

```bash
# Where am I?
pwd

# What's in this directory?
ls

# Show more details
ls -la

# What version of Linux is this?
cat /etc/os-release

# What user am I?
whoami
```

### Step 3.3: Exit the container

```bash
exit
```

You're back on your own computer.

### Self-check
- [ ] You saw the `root@...` prompt inside the container
- [ ] `pwd` showed `/`
- [ ] `cat /etc/os-release` showed Ubuntu information
- [ ] `exit` returned you to your normal terminal

---

## Exercise 4: Run a Web Server (~10 minutes)

Let's run something more practical - a web server.

### Step 4.1: Start nginx web server

```bash
docker run -d -p 8080:80 nginx:1.25
```

Breaking down the new flags:
- `-d` - **detached** mode (runs in background)
- `-p 8080:80` - **publish** port: connect your computer's port 8080 to the container's port 80
- `nginx:1.25` - nginx version 1.25 (always use specific versions!)

**Expected output**:
```
Unable to find image 'nginx:1.25' locally
1.25: Pulling from library/nginx
...
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6
```

The long string is the container ID.

### Step 4.2: Verify it's running

Open your web browser and go to: **http://localhost:8080**

You should see the nginx welcome page: "Welcome to nginx!"

### Step 4.3: Check running containers

```bash
docker ps
```

**Expected output**:
```
CONTAINER ID   IMAGE        COMMAND                  CREATED          STATUS          PORTS                  NAMES
a1b2c3d4e5f6   nginx:1.25   "/docker-entrypoint.…"   30 seconds ago   Up 29 seconds   0.0.0.0:8080->80/tcp   relaxed_einstein
```

This shows your nginx container is running.

### Step 4.4: Stop the container

You can use either the container ID or the container name:

```bash
# Using container ID (first few characters are enough)
docker stop a1b2

# OR using container name
docker stop relaxed_einstein
```

### Step 4.5: Verify it stopped

Refresh http://localhost:8080 - the page should no longer load.

```bash
docker ps
```

The container should no longer appear (it's stopped, not removed).

### Self-check
- [ ] nginx container started successfully
- [ ] You saw the nginx welcome page in your browser
- [ ] `docker ps` showed the running container
- [ ] You successfully stopped the container

---

## Exercise 5: Clean Up (Optional) (~5 minutes)

> This exercise is optional but good practice. If you're short on time, skip to the Summary.

Let's clean up the containers we created.

### Step 5.1: See all containers (including stopped)

```bash
docker ps -a
```

You should see the stopped nginx container and the exited hello-world and ubuntu containers.

### Step 5.2: Remove a container

```bash
docker rm <container_id>
```

### Step 5.3: Remove all stopped containers

```bash
docker container prune
```

Type `y` to confirm.

### Self-check
- [ ] You can list all containers with `docker ps -a`
- [ ] You successfully removed containers

---

## Exercise 6: GitHub Account and SSH Keys (~15 minutes)

### Why GitHub?

Throughout this course, you'll use GitHub for:
- Storing your code
- CI/CD pipelines (Week 3)
- Collaboration on projects

Setting it up now means you're ready when we need it.

### Tasks

**Part A: Create a GitHub Account** (~3 minutes)

If you already have a GitHub account, skip to Part B.

1. Go to [github.com](https://github.com)
2. Click **Sign up**
3. Follow the steps to create an account
4. Verify your email address

**Part B: Install Git** (~2 minutes)

Check if Git is already installed:

```bash
git --version
```

If you see a version number, you're good. If not:

**Windows:**
- Download from [git-scm.com](https://git-scm.com/download/win)
- Run the installer (default options are fine)

**macOS:**
```bash
xcode-select --install
```

**Linux:**
```bash
sudo apt install git
```

**Part C: Generate an SSH Key** (~5 minutes)

SSH keys let you connect to GitHub securely without typing your password every time.

> **Important:** Do this in your **normal terminal**, NOT inside a Docker container. The keys need to be stored on your computer, not inside a temporary container.

1. Open a terminal (PowerShell/Terminal app, not a Docker container) and generate a key:
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```
   Replace with your actual email (the one you used for GitHub).

2. When prompted for a file location, press **Enter** to accept the default.

3. When prompted for a passphrase, you can either:
   - Press **Enter** twice for no passphrase (easier)
   - Enter a passphrase (more secure)

4. You should see output like:
   ```
   Your identification has been saved in /home/you/.ssh/id_ed25519
   Your public key has been saved in /home/you/.ssh/id_ed25519.pub
   ```

5. Display your public key:
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

6. Copy the entire output (starts with `ssh-ed25519` and ends with your email).

**Part D: Add SSH Key to GitHub** (~3 minutes)

1. Go to [github.com](https://github.com) and sign in
2. Click your profile picture → **Settings**
3. In the left sidebar, click **SSH and GPG keys**
4. Click **New SSH key**
5. Give it a title (e.g., "My Laptop")
6. Paste your public key into the "Key" field
7. Click **Add SSH key**

**Part E: Test the Connection** (~2 minutes)

```bash
ssh -T git@github.com
```

If this is your first time connecting, you'll see:
```
The authenticity of host 'github.com (...)' can't be established.
...
Are you sure you want to continue connecting (yes/no)?
```

Type `yes` and press Enter.

**Expected success message:**
```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

### Self-check
- [ ] I have a GitHub account
- [ ] Git is installed (`git --version` works)
- [ ] I generated an SSH key
- [ ] I added my public key to GitHub
- [ ] `ssh -T git@github.com` shows my username

### What's Next?

We'll use Git and GitHub throughout the course. In Week 3, you'll set up GitHub Actions for automated testing and deployment.

---

## Summary

You've learned these commands:

| Command | Purpose |
|---------|---------|
| `docker --version` | Check Docker is installed |
| `docker run IMAGE` | Create and run a container |
| `docker run -it IMAGE bash` | Run interactively with a shell |
| `docker run -d -p 8080:80 IMAGE` | Run in background with port mapping |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker stop CONTAINER` | Stop a container |
| `docker rm CONTAINER` | Remove a container |

Linux commands (inside the container):

| Command | Purpose |
|---------|---------|
| `pwd` | Print working directory |
| `ls` | List files |
| `cat FILE` | Show file contents |
| `whoami` | Show current user |
| `exit` | Exit the container |

Git and SSH commands:

| Command | Purpose |
|---------|---------|
| `git --version` | Check Git is installed |
| `ssh-keygen -t ed25519 -C "email"` | Generate SSH key pair |
| `cat ~/.ssh/id_ed25519.pub` | Display public key |
| `ssh -T git@github.com` | Test GitHub connection |

---

## Troubleshooting

### "docker: command not found"
- Docker isn't installed or isn't in your PATH
- On Windows/Mac: **Make sure Docker Desktop is running** (check for the whale icon)
- On Linux: Make sure you logged out and back in after installation

### "Cannot connect to the Docker daemon"
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```
- **Windows/Mac:** Docker Desktop is not running. Start it from the Start menu (Windows) or Applications (Mac)
- **Linux:** Start Docker with `sudo systemctl start docker`

### "permission denied" (Linux)
- You need to add yourself to the docker group: `sudo usermod -aG docker $USER`
- Then **log out and log back in** (or restart your computer)

### "port already in use"
```
Error response from daemon: driver failed programming external connectivity: Bind for 0.0.0.0:8080 failed: port is already allocated
```
- Something else is using port 8080
- Try a different port: `docker run -d -p 8081:80 nginx:1.25`
- Then visit http://localhost:8081 instead

### Docker Desktop won't start (Windows)
- Make sure WSL 2 is installed and enabled
- Try restarting your computer
- Check Windows Features: "Windows Subsystem for Linux" and "Virtual Machine Platform" should be enabled

### Image download is very slow
- This is normal for the first download - images can be 50-200 MB
- Subsequent runs will be fast because the image is cached locally

---

## Before Class Checklist

Before coming to class, ensure you can check ALL of these:

**Docker:**
- [ ] Docker is installed and running
- [ ] `docker --version` shows a version number
- [ ] `docker run hello-world` works
- [ ] You successfully ran an interactive Ubuntu container
- [ ] You successfully ran nginx and saw the welcome page in your browser
- [ ] You know how to stop containers

**GitHub:**
- [ ] You have a GitHub account
- [ ] Git is installed (`git --version` works)
- [ ] You generated an SSH key
- [ ] `ssh -T git@github.com` shows your username

If any of these aren't working, **note the exact error message** and we'll troubleshoot at the start of class.

---

**You're ready for class!** See you there.
