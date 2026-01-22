# Pre-class Reading: Docker Build & Compose

**Estimated reading time: 45-60 minutes**

This week we solve three problems you experienced in Week 1. Read through each section to understand *why* we need these tools before learning *how* to use them.

---

## The Data Persistence Problem

Remember the MySQL exercise from Week 1? You created a database, removed the container, and... the database was gone.

> "I spent an hour setting up my database, and then it just disappeared!"

This is one of the most common frustrations when starting with Docker. Let's understand why it happens and how to fix it.

### Why Data Disappears

Every container has its own **writable layer** on top of the read-only image layers. When you write files inside a container, they go into this writable layer.

```
┌─────────────────────────┐
│   Container Writable    │  ← Your data lives here
│        Layer            │
├─────────────────────────┤
│   Image Layer 3         │  (read-only)
├─────────────────────────┤
│   Image Layer 2         │  (read-only)
├─────────────────────────┤
│   Image Layer 1         │  (read-only)
└─────────────────────────┘
```

When you remove the container with `docker rm`, the writable layer is **deleted**. The image layers remain (they're shared), but your data is gone.

### The Solution: Volumes

A **volume** is storage that exists outside the container. It's managed by Docker and persists even when containers are removed.

```
Container                         Volume
┌─────────────────────┐          ┌─────────────────────┐
│                     │          │                     │
│   Writable Layer    │          │    mysql-data       │
│                     │          │                     │
├─────────────────────┤          │   /var/lib/mysql    │
│                     │◀────────▶│                     │
│   Image Layers      │          │   Your data lives   │
│                     │          │   HERE now!         │
└─────────────────────┘          └─────────────────────┘
     Container can be                Volume persists
     removed and recreated           independently
```

With a volume:
1. Data is stored outside the container
2. Remove the container - data stays
3. Create a new container - attach the same volume
4. Your data is back!

### Two Types of Volumes

#### Named Volumes (Docker manages the location)

```bash
docker run -v mysql-data:/var/lib/mysql mysql:8.0
```

- Docker decides where to store the data on your computer
- Best for **database data** and other persistent storage
- Easy to backup and manage with Docker commands

#### Bind Mounts (You specify the location)

```bash
docker run -v $(pwd)/src:/app/src myapp
```

- You specify an exact path on your computer
- Best for **development** - edit files on your computer, see changes in container
- The container sees your files directly

### When to Use Each

| Use Case | Volume Type |
|----------|-------------|
| Database storage | Named volume |
| Uploaded files | Named volume |
| Development code | Bind mount |
| Configuration files | Bind mount |

---

## The Manual Setup Problem

Another frustration from Week 1: every time you started a new Ubuntu container, it was a fresh, minimal system. Want to use `curl`? Install it. Want `vim`? Install it again.

> "I keep installing the same tools over and over. There must be a better way!"

### The Solution: Dockerfiles

A **Dockerfile** is a text file with instructions for building a custom image. Think of it as a recipe.

```
Dockerfile              docker build              Image              docker run             Container
┌────────────┐           ─────────▶           ┌────────────┐         ─────────▶          ┌────────────┐
│ FROM       │                                │            │                             │            │
│ RUN        │                                │   myapp    │                             │  Running   │
│ COPY       │                                │   :1.0     │                             │  instance  │
│ CMD        │                                │            │                             │            │
└────────────┘                                └────────────┘                             └────────────┘
   Recipe                                       Snapshot                                  Live system
```

Instead of:
1. Start container
2. Install curl
3. Install vim
4. Copy your files
5. Hope you remember all the steps next time

You write a Dockerfile once:
```dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y curl vim
COPY myfiles/ /app/
```

Then `docker build` creates an image with everything pre-installed.

### Dockerfile Instructions

Here are the most common instructions:

#### FROM - Where to start

Every Dockerfile starts with `FROM`. This is your base image.

```dockerfile
FROM ubuntu:22.04
FROM node:18-alpine
FROM python:3.11
```

#### RUN - Execute during build

`RUN` executes commands when building the image. The results are saved in the image.

```dockerfile
RUN apt-get update && apt-get install -y curl
RUN npm install
RUN pip install -r requirements.txt
```

#### COPY - Add files from your computer

`COPY` adds files from your computer into the image.

```dockerfile
COPY index.html /usr/share/nginx/html/
COPY src/ /app/src/
COPY package.json .
```

#### WORKDIR - Set the working directory

`WORKDIR` sets the directory for subsequent commands (like `cd`).

```dockerfile
WORKDIR /app
COPY . .        # Copies to /app
RUN npm install # Runs in /app
```

#### CMD - Default command

`CMD` specifies what runs when a container starts. There's only one CMD (the last one wins).

```dockerfile
CMD ["node", "server.js"]
CMD ["python", "app.py"]
CMD ["nginx", "-g", "daemon off;"]
```

#### EXPOSE - Document ports

`EXPOSE` documents which port your application uses. It doesn't actually open the port (you still need `-p` when running).

```dockerfile
EXPOSE 3000
EXPOSE 80
```

#### ENV - Set environment variables

`ENV` sets environment variables available in the container.

```dockerfile
ENV NODE_ENV=production
ENV DATABASE_URL=postgres://localhost:5432/app
```

### A Complete Example

```dockerfile
# Start from Node.js base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy dependency files first (for caching - explained below)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Document the port
EXPOSE 3000

# Start the application
CMD ["node", "server.js"]
```

### Image Layers and Caching

Each instruction in a Dockerfile creates a **layer**. Docker caches these layers to speed up builds.

```
Dockerfile                    Image Layers
┌────────────────────┐       ┌────────────────────┐
│ FROM node:18       │  ──▶  │ Layer 1: base      │ (cached from Docker Hub)
├────────────────────┤       ├────────────────────┤
│ WORKDIR /app       │  ──▶  │ Layer 2: workdir   │ (cached)
├────────────────────┤       ├────────────────────┤
│ COPY package.json  │  ──▶  │ Layer 3: pkg file  │ (cached if unchanged)
├────────────────────┤       ├────────────────────┤
│ RUN npm install    │  ──▶  │ Layer 4: deps      │ (cached if pkg unchanged)
├────────────────────┤       ├────────────────────┤
│ COPY . .           │  ──▶  │ Layer 5: code      │ (rebuilds on code change)
├────────────────────┤       ├────────────────────┤
│ CMD ["node"...]    │  ──▶  │ Layer 6: cmd       │ (rebuilds)
└────────────────────┘       └────────────────────┘
```

**The key insight**: When a layer changes, all layers after it must rebuild.

This is why order matters:

```dockerfile
# BAD - npm install runs every time you change any code
COPY . .
RUN npm install

# GOOD - npm install only runs when package.json changes
COPY package*.json ./
RUN npm install
COPY . .
```

Put things that change **least frequently first**.

---

## The Command Complexity Problem

By now, your `docker run` commands are getting complex:

```bash
docker run -d \
  --name myapp \
  -p 3000:3000 \
  -e DATABASE_URL=postgres://db:5432/app \
  -v $(pwd)/src:/app/src \
  --network mynetwork \
  myapp:1.0
```

And that's just one container. What if you need a database too?

```bash
# First, create a network
docker network create mynetwork

# Start the database
docker run -d \
  --name db \
  --network mynetwork \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16

# Now start the app
docker run -d \
  --name myapp \
  --network mynetwork \
  -p 3000:3000 \
  -e DATABASE_URL=postgres://postgres:secret@db:5432/app \
  myapp:1.0
```

> "I can never remember all these flags. And I have to run them in the right order!"

### The Solution: Docker Compose

**Docker Compose** lets you define multi-container applications in a single YAML file.

The commands above become:

```yaml
services:
  myapp:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://postgres:secret@db:5432/app
    volumes:
      - ./src:/app/src
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

Then one command starts everything:

```bash
docker compose up -d
```

### compose.yaml Structure

```yaml
services:          # Define your containers
  service-name:
    image: ...     # OR
    build: ...     # Use existing image OR build from Dockerfile
    ports: ...
    environment: ...
    volumes: ...
    depends_on: ...

volumes:           # Declare named volumes
  volume-name:

networks:          # Declare custom networks (optional)
  network-name:
```

### Key Compose Concepts

#### Services

Each service becomes a container. The service name becomes the hostname.

```yaml
services:
  web:         # Container named "web", hostname "web"
    image: nginx:1.25

  api:         # Container named "api", hostname "api"
    build: ./api

  db:          # Container named "db", hostname "db"
    image: postgres:16
```

#### Automatic Networking

Compose creates a network for your application automatically. Services can reach each other by name:

```yaml
services:
  web:
    environment:
      API_URL: http://api:3000    # "api" is the service name

  api:
    environment:
      DB_HOST: db                  # "db" is the service name

  db:
    image: postgres:16
```

#### depends_on

Control startup order:

```yaml
services:
  web:
    depends_on:
      - api      # Wait for api to start

  api:
    depends_on:
      - db       # Wait for db to start

  db:
    image: postgres:16
```

Note: `depends_on` waits for the container to *start*, not for the application to be *ready*. A database might need a few seconds after starting before it accepts connections.

#### Volumes in Compose

```yaml
services:
  db:
    volumes:
      - pgdata:/var/lib/postgresql/data    # Named volume
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # Bind mount

volumes:
  pgdata:    # Declare the named volume
```

### Common Compose Commands

| Command | What it does |
|---------|--------------|
| `docker compose up` | Start all services (foreground) |
| `docker compose up -d` | Start all services (background) |
| `docker compose down` | Stop and remove containers |
| `docker compose down -v` | Also remove volumes (careful!) |
| `docker compose ps` | List running services |
| `docker compose logs` | View all logs |
| `docker compose logs web` | View specific service logs |
| `docker compose exec web bash` | Shell into running service |

---

## Docker Hub: Sharing Images

**Docker Hub** is like GitHub, but for Docker images. It's where images like `nginx`, `postgres`, and `node` come from.

You can also push your own images:

```bash
# Log in to Docker Hub
docker login

# Tag your image for Docker Hub
docker tag myapp:1.0 yourusername/myapp:1.0

# Push to Docker Hub
docker push yourusername/myapp:1.0
```

Now anyone can pull your image:

```bash
docker pull yourusername/myapp:1.0
```

### Image Naming Convention

```
registry/username/repository:tag

Examples:
docker.io/library/nginx:1.25      # Official nginx image
docker.io/yourusername/myapp:1.0  # Your image
ghcr.io/company/app:latest        # GitHub Container Registry
```

When you don't specify a registry, Docker assumes `docker.io` (Docker Hub).

---

## Self-Check Questions

Before moving to the exercises, make sure you can answer these:

1. What's the difference between a named volume and a bind mount? When would you use each?

2. In a Dockerfile, what's the difference between `RUN` and `CMD`?

3. Why should you copy `package.json` before copying the rest of your code in a Node.js Dockerfile?

4. What does `depends_on` do in a compose.yaml? What doesn't it do?

5. What command starts all services defined in compose.yaml in the background?

<details>
<summary>Click to reveal answers</summary>

1. **Named volume**: Docker manages the storage location. Use for database data and persistent storage. **Bind mount**: You specify the exact path. Use for development when you want to edit files on your computer and see changes in the container.

2. **RUN** executes during image build - the results are saved in the image (installing packages, copying files). **CMD** specifies what runs when a container starts from the image - it's the default command.

3. For layer caching. If you copy everything first, then `npm install` runs every time any file changes. By copying `package.json` first, `npm install` only runs when dependencies change - saving build time.

4. `depends_on` controls startup order - the dependent service waits for the other to start. But it does NOT wait for the application inside to be ready. A database container might start but need seconds before accepting connections.

5. `docker compose up -d` (the `-d` flag means detached/background mode)

</details>

---

## Summary

| Problem | Solution |
|---------|----------|
| Data disappears when container is removed | **Volumes** - persistent storage outside containers |
| Manual setup every time | **Dockerfiles** - recipes for building custom images |
| Too many flags to remember | **Docker Compose** - define everything in one YAML file |

---

## Further Reading (Optional)

- [Docker Volumes documentation](https://docs.docker.com/storage/volumes/)
- [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)
- [Compose file reference](https://docs.docker.com/compose/compose-file/)
- [Docker Hub](https://hub.docker.com/)

---

**Next step**: Continue to [exercises.md](exercises.md) to build your first custom image.
