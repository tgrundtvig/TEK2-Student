# Week 2 Quick Reference Card

Print this page or keep it open while working!

---

## Volume Commands

| Command | What it does | Example |
|---------|--------------|---------|
| `-v name:/path` | Named volume (Docker manages) | `-v mysql-data:/var/lib/mysql` |
| `-v /host:/container` | Bind mount (you manage) | `-v $(pwd):/app` |
| `docker volume ls` | List all volumes | |
| `docker volume create NAME` | Create a volume | `docker volume create mydata` |
| `docker volume rm NAME` | Remove a volume | `docker volume rm mydata` |
| `docker volume prune` | Remove unused volumes | |
| `docker volume inspect NAME` | Show volume details | `docker volume inspect mydata` |

### Volume Types

```
Named Volume (recommended for data):
  -v mysql-data:/var/lib/mysql
       │              │
       │              └── Path inside container
       └── Volume name (Docker manages where it's stored)

Bind Mount (for development):
  -v $(pwd)/src:/app/src
       │           │
       │           └── Path inside container
       └── Path on your computer
```

---

## Dockerfile Instructions

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image to start from | `FROM node:18-alpine` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `COPY` | Copy files from host to image | `COPY package.json .` |
| `RUN` | Execute command during build | `RUN npm install` |
| `ENV` | Set environment variable | `ENV NODE_ENV=production` |
| `EXPOSE` | Document port (informational only) | `EXPOSE 3000` |
| `CMD` | Default command when container starts | `CMD ["node", "server.js"]` |

### Dockerfile Template

```dockerfile
FROM image:tag
WORKDIR /app
COPY . .
RUN build-commands
EXPOSE port
CMD ["executable", "arg"]
```

### The Most Important Rule: Layer Order!

```dockerfile
# BAD - npm install runs every time code changes
COPY . .
RUN npm install

# GOOD - npm install only runs when package.json changes
COPY package*.json ./
RUN npm install
COPY . .
```

Put things that change LEAST frequently FIRST.

---

## Build Commands

| Command | What it does | Example |
|---------|--------------|---------|
| `docker build -t name .` | Build image from Dockerfile | `docker build -t myapp .` |
| `docker build -t name:tag .` | Build with specific tag | `docker build -t myapp:1.0 .` |
| `docker images` | List all images | |
| `docker rmi IMAGE` | Remove an image | `docker rmi myapp:1.0` |
| `docker image prune` | Remove unused images | |
| `docker history IMAGE` | Show image layers | `docker history myapp:1.0` |

---

## Docker Hub Commands

| Command | What it does | Example |
|---------|--------------|---------|
| `docker login` | Log in to Docker Hub | |
| `docker tag IMAGE USER/REPO:TAG` | Tag image for pushing | `docker tag myapp user/myapp:1.0` |
| `docker push USER/REPO:TAG` | Push to Docker Hub | `docker push user/myapp:1.0` |
| `docker pull USER/REPO:TAG` | Pull from Docker Hub | `docker pull user/myapp:1.0` |

---

## Docker Compose Commands

| Command | What it does |
|---------|--------------|
| `docker compose up` | Start all services (foreground) |
| `docker compose up -d` | Start all services (background) |
| `docker compose down` | Stop and remove containers |
| `docker compose down -v` | Stop, remove containers AND volumes |
| `docker compose ps` | List running services |
| `docker compose logs` | View all logs |
| `docker compose logs SERVICE` | View specific service logs |
| `docker compose exec SERVICE bash` | Shell into running service |
| `docker compose build` | Rebuild images |
| `docker compose restart` | Restart services |

---

## compose.yaml Template

```yaml
services:
  web:
    build: .                          # Build from Dockerfile
    ports:
      - "3000:3000"                   # host:container
    environment:
      - DATABASE_URL=postgres://db:5432/app
    depends_on:
      - db                            # Start db first
    volumes:
      - ./src:/app/src                # Bind mount for dev

  db:
    image: postgres:16                # Use existing image
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:                            # Declare named volume
```

---

## Common Patterns

### Database with persistent data

```bash
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

### Development with live reload

```bash
docker run -d \
  -v $(pwd):/app \
  -p 3000:3000 \
  myapp
```

### Build and run custom image

```bash
docker build -t myapp:1.0 .
docker run -d -p 8080:80 myapp:1.0
```

---

## .dockerignore Template

```
node_modules
.git
.gitignore
*.md
.env
.DS_Store
npm-debug.log
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "COPY failed: file not found" | Check file path, must be in build context (same folder as Dockerfile) |
| Permission denied in container | Check file ownership, may need `chown` or run as different user |
| Volume data not appearing | Check paths match exactly, verify volume exists with `docker volume ls` |
| Build not using cache | Order Dockerfile commands so least-changing lines come first |
| Compose can't find yaml | File must be named `compose.yaml` or use `-f filename.yaml` flag |
| "port already in use" | Stop other container using that port, or use different port mapping |
| Changes not showing | Rebuild image (`docker build`) or restart container |

---

## Remember

1. **Volumes persist data** - even when containers are removed
2. **Named volumes for data** (databases), **bind mounts for code** (development)
3. **Dockerfile order matters** - put things that change less frequently first
4. **Compose simplifies everything** - one file, one command, many containers
5. **Always use specific tags** - both in Dockerfiles (`FROM node:18`) and compose.yaml
6. **`docker compose down -v`** removes volumes too - be careful with data!
