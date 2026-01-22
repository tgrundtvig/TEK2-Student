# Pre-class Exercises: Build Your First Docker Image

**Estimated time: 30-60 minutes**

Complete these exercises before class. If you get stuck, note where you had problems - we'll address them at the start of class.

> **Tip:** If something isn't working, don't spend more than 10 minutes on it. Note the error message and move on - we'll troubleshoot together in class.

---

## Exercise 1: Verify Setup (~5 minutes)

Before we start building images, let's make sure Docker is working.

### Step 1.1: Check Docker is running

```bash
docker --version
docker ps
```

If you see an error about connecting to the Docker daemon, start Docker Desktop (Windows/Mac) or the Docker service (Linux).

### Step 1.2: Create a working directory

```bash
mkdir -p ~/week2-docker
cd ~/week2-docker
```

### Self-check
- [ ] `docker --version` shows a version number
- [ ] `docker ps` runs without error
- [ ] You're in the `week2-docker` directory

---

## Exercise 2: Your First Dockerfile (~15 minutes)

### Goal

Build a custom nginx image that serves your own HTML page.

### Step 2.1: Create project folder

```bash
mkdir hello-docker
cd hello-docker
```

### Step 2.2: Create your HTML file

Create a file called `index.html` with this content:

```html
<!DOCTYPE html>
<html>
<head>
    <title>My First Docker Image</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        h1 {
            color: #0066cc;
        }
        .info {
            background: white;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body>
    <h1>Hello from my custom Docker image!</h1>
    <div class="info">
        <p><strong>Built by:</strong> [Your Name]</p>
        <p><strong>Week:</strong> 2 - Docker Build & Compose</p>
        <p>This page is served by nginx running in a container built from my Dockerfile.</p>
    </div>
</body>
</html>
```

> **Note:** Replace `[Your Name]` with your actual name.

### Step 2.3: Create the Dockerfile

Create a file called `Dockerfile` (no extension) with this content:

```dockerfile
# Start from the official nginx image
FROM nginx:1.25

# Copy our HTML file to nginx's web directory
COPY index.html /usr/share/nginx/html/
```

Your folder should now look like:
```
hello-docker/
├── Dockerfile
└── index.html
```

### Step 2.4: Build the image

```bash
docker build -t hello-docker:1.0 .
```

Breaking down this command:
- `docker build` - build an image from a Dockerfile
- `-t hello-docker:1.0` - tag (name) the image as "hello-docker" version "1.0"
- `.` - use the current directory as the build context

**Expected output:**
```
[+] Building 2.1s (7/7) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [internal] load metadata for docker.io/library/nginx:1.25
 => [1/2] FROM docker.io/library/nginx:1.25
 => [2/2] COPY index.html /usr/share/nginx/html/
 => exporting to image
 => => naming to docker.io/library/hello-docker:1.0
```

### Step 2.5: Verify the image was created

```bash
docker images | grep hello-docker
```

**Expected output:**
```
hello-docker   1.0       abc123def456   10 seconds ago   187MB
```

### Step 2.6: Run your image

```bash
docker run -d -p 8080:80 --name my-web hello-docker:1.0
```

### Step 2.7: Test it

Open your browser and go to: **http://localhost:8080**

You should see your custom HTML page!

### Step 2.8: Clean up (for now)

```bash
docker stop my-web
docker rm my-web
```

### Self-check
- [ ] The `docker build` command completed without errors
- [ ] `docker images` shows your `hello-docker:1.0` image
- [ ] You saw your custom page in the browser
- [ ] You understand what each line in the Dockerfile does

---

## Exercise 3: Add Build-Time Commands (~10 minutes)

### Goal

Use the `RUN` instruction to execute commands during the build.

### Step 3.1: Update your Dockerfile

Edit `Dockerfile` to add a build timestamp:

```dockerfile
# Start from the official nginx image
FROM nginx:1.25

# Create a file with build information
RUN echo "Image built on: $(date)" > /usr/share/nginx/html/build-info.txt
RUN echo "Built from nginx:1.25 base image" >> /usr/share/nginx/html/build-info.txt

# Copy our HTML file to nginx's web directory
COPY index.html /usr/share/nginx/html/
```

### Step 3.2: Build a new version

```bash
docker build -t hello-docker:1.1 .
```

### Step 3.3: Run and verify

```bash
docker run -d -p 8080:80 --name my-web hello-docker:1.1
```

Open your browser to: **http://localhost:8080/build-info.txt**

You should see the build timestamp!

### Step 3.4: Understand the difference

- `RUN` commands execute **during build** - the results are saved in the image
- The timestamp was recorded when you ran `docker build`, not when you ran `docker run`
- Every container from this image will show the **same** build time

### Step 3.5: Clean up

```bash
docker stop my-web
docker rm my-web
```

### Self-check
- [ ] You see the build timestamp at `/build-info.txt`
- [ ] You understand that `RUN` executes during build, not during run
- [ ] You understand the timestamp is "baked into" the image

---

## Exercise 4: Explore Image Layers and Caching (~10 minutes)

### Goal

Understand how Docker caches build layers.

### Step 4.1: Rebuild the same image

```bash
docker build -t hello-docker:1.1 .
```

**Notice:** The build is almost instant! Look for messages like:
```
=> CACHED [1/3] FROM docker.io/library/nginx:1.25
=> CACHED [2/3] RUN echo "Image built on: $(date)"...
=> CACHED [3/3] COPY index.html /usr/share/nginx/html/
```

Docker reused all the cached layers because nothing changed.

### Step 4.2: Make a change and rebuild

Edit `index.html` - change anything (the title, the text, etc.).

Now rebuild:

```bash
docker build -t hello-docker:1.2 .
```

**Notice:** Only some layers rebuild:
```
=> CACHED [1/3] FROM docker.io/library/nginx:1.25
=> CACHED [2/3] RUN echo "Image built on: $(date)"...
=> [3/3] COPY index.html /usr/share/nginx/html/      # This rebuilds!
```

The COPY layer had to rebuild because the file changed. But the RUN layer was still cached.

### Step 4.3: See the layers

```bash
docker history hello-docker:1.2
```

This shows each layer, its size, and what created it.

### Step 4.4: Why order matters

The `RUN` command that creates build-info.txt is **before** the `COPY` command. So when index.html changes, the RUN doesn't need to rebuild.

If we reversed the order:
```dockerfile
COPY index.html /usr/share/nginx/html/   # If this changes...
RUN echo "Built on: $(date)" > /build.txt  # ...this must rebuild too!
```

**Rule:** Put things that change **least frequently first**.

### Self-check
- [ ] You saw "CACHED" messages when rebuilding without changes
- [ ] You saw partial rebuilds when only index.html changed
- [ ] You understand that layer order affects caching
- [ ] You can view layers with `docker history`

---

## Exercise 5: Create a .dockerignore File (~5 minutes)

### Goal

Exclude files from the build context.

### Step 5.1: See what's sent to Docker

When you run `docker build`, the entire directory (build context) is sent to the Docker daemon. Large directories slow down builds.

### Step 5.2: Create a .dockerignore file

Create a file called `.dockerignore` in your hello-docker folder:

```
# Don't send these to Docker
*.md
*.log
.git
.gitignore
.DS_Store
```

### Step 5.3: Understand the benefit

Now if you have a README.md or log files, they won't:
- Slow down the build
- Be available to COPY into the image
- Invalidate the cache unnecessarily

### Self-check
- [ ] You created a `.dockerignore` file
- [ ] You understand it works like `.gitignore`

---

## Summary

You've learned these concepts and commands:

| Concept | Description |
|---------|-------------|
| Dockerfile | Text file with instructions to build an image |
| `FROM` | Specifies the base image |
| `RUN` | Executes a command during build |
| `COPY` | Copies files from your computer into the image |
| `.dockerignore` | Excludes files from the build context |

| Command | Purpose |
|---------|---------|
| `docker build -t name:tag .` | Build an image from a Dockerfile |
| `docker images` | List images |
| `docker history IMAGE` | Show image layers |
| `docker rmi IMAGE` | Remove an image |

---

## Troubleshooting

### "COPY failed: file not found"

```
COPY failed: file not found in build context
```

**Cause:** The file you're trying to COPY doesn't exist, or you're in the wrong directory.

**Fix:**
1. Make sure you're in the folder with the Dockerfile: `pwd`
2. Check the file exists: `ls`
3. Check the filename matches exactly (case-sensitive!)

### "Cannot connect to the Docker daemon"

**Cause:** Docker isn't running.

**Fix:**
- **Windows/Mac:** Start Docker Desktop from the Start menu or Applications
- **Linux:** Run `sudo systemctl start docker`

### "permission denied while trying to connect to the Docker daemon socket"

**Cause (Linux):** Your user isn't in the docker group.

**Fix:**
```bash
sudo usermod -aG docker $USER
```
Then log out and log back in.

### Build seems to hang

**Cause:** Downloading base image or running slow commands.

**Fix:** Be patient the first time. Subsequent builds will use cached layers and be much faster.

---

## Before Class Checklist

Before coming to class, ensure you can check ALL of these:

**Dockerfile basics:**
- [ ] You built the `hello-docker:1.0` image successfully
- [ ] You understand what `FROM`, `RUN`, and `COPY` do
- [ ] You saw your custom HTML page in the browser
- [ ] You observed layer caching behavior

**Concepts:**
- [ ] You understand `RUN` executes during build (not run)
- [ ] You know why Dockerfile instruction order matters for caching

If any of these aren't working, **note the exact error message** and we'll troubleshoot at the start of class.

---

## Optional: Clean Up Images

If you want to clean up the images we created:

```bash
# Remove specific images
docker rmi hello-docker:1.0 hello-docker:1.1 hello-docker:1.2

# Or remove all unused images (be careful!)
docker image prune
```

---

**You're ready for class!** In class, we'll solve the data persistence problem with volumes and learn Docker Compose.
