# Hints for Advanced Tasks

Use these hints if you get stuck. Try to solve problems on your own first!

---

## Task 1: Node.js Application

<details>
<summary>Hint 1: npm install fails with "file not found"</summary>

Make sure you have both `package.json` AND `server.js` created before building.

Check the files exist:
```bash
ls -la
# Should show: package.json, server.js, Dockerfile, .dockerignore
```

Check the content is correct:
```bash
cat package.json
# Should be valid JSON
```

</details>

<details>
<summary>Hint 2: "Cannot find module 'express'"</summary>

This means npm install didn't run, or the files are in the wrong order.

Check your Dockerfile order:
```dockerfile
COPY package*.json ./   # 1. Copy package files
RUN npm install         # 2. Install dependencies
COPY . .                # 3. Copy code AFTER install
```

If COPY . . comes before npm install, the node_modules from your host (if any) might overwrite the installed ones.

</details>

<details>
<summary>Hint 3: Container exits immediately</summary>

Check the logs:
```bash
docker logs myapp
```

Common issues:
- Syntax error in server.js
- Missing dependency in package.json
- Port conflict (another container using port 3000)

Try running without `-d` to see output directly:
```bash
docker run -p 3000:3000 node-app:1.0
```

</details>

<details>
<summary>Hint 4: Endpoints return connection refused</summary>

Make sure:
1. Container is running: `docker ps`
2. Port mapping is correct: `-p 3000:3000`
3. Server is listening on 0.0.0.0, not 127.0.0.1

In server.js, `app.listen(port)` binds to all interfaces by default, which is correct.

</details>

---

## Task 2: Multi-Stage Build

<details>
<summary>Hint 1: "COPY --from=builder" fails</summary>

Make sure the first stage has the `AS builder` name:
```dockerfile
FROM node:18-alpine AS builder
```

The name must match exactly:
```dockerfile
COPY --from=builder /app/node_modules ./node_modules
```

</details>

<details>
<summary>Hint 2: Understanding the stages</summary>

Think of it like this:

```
Stage 1 (builder):
┌─────────────────────────┐
│ Full Node.js image      │
│ - npm included          │
│ - All build tools       │
│ - Install dependencies  │
│ - Run build commands    │
└─────────────────────────┘
           │
           │ COPY --from=builder
           │ (only what we need)
           ▼
Stage 2 (production):
┌─────────────────────────┐
│ Minimal Node.js image   │
│ - Just node runtime     │
│ - Only node_modules     │
│ - Only production code  │
│ - Much smaller!         │
└─────────────────────────┘
```

</details>

<details>
<summary>Hint 3: Why use npm ci instead of npm install?</summary>

`npm ci` (clean install):
- Deletes node_modules first
- Installs exactly what's in package-lock.json
- Faster and more reliable for CI/CD
- Fails if package-lock.json is missing or out of sync

`npm install`:
- Updates package-lock.json
- May install slightly different versions
- More flexible but less reproducible

For production Docker images, `npm ci` is preferred.

</details>

---

## Task 3: Full Stack Application

<details>
<summary>Hint 1: API can't connect to database</summary>

Make sure:

1. `depends_on` is set correctly (though this only waits for container start, not app ready)
2. Database hostname is `db` (the service name in compose.yaml)
3. Database credentials match between services

Connection string format:
```
postgres://username:password@hostname:port/database
postgres://postgres:secret@db:5432/appdb
```

The `db` in the URL is the **service name** from compose.yaml, not localhost!

</details>

<details>
<summary>Hint 2: Database isn't ready when API starts</summary>

The API has retry logic built in:
```javascript
setTimeout(initDb, 2000);
```

If you see database connection errors in the logs, wait a few seconds and check again:
```bash
docker compose logs api
```

For production, consider using a proper health check or a tool like `wait-for-it`.

</details>

<details>
<summary>Hint 3: Frontend can't reach API</summary>

Important: The frontend runs in your **browser**, not in Docker!

The browser needs to reach the API through your host machine, so use:
- `http://localhost:4000` in the frontend JavaScript

NOT:
- `http://api:4000` (this only works between containers)

The `api:4000` hostname only works for container-to-container communication. Your browser doesn't know about Docker's internal DNS.

</details>

<details>
<summary>Hint 4: CORS errors in browser console</summary>

If you see "CORS policy" errors, the API needs to allow cross-origin requests.

Quick fix - add to `api/server.js`:
```javascript
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Headers', 'Content-Type');
  next();
});
```

Add this BEFORE your routes.

</details>

<details>
<summary>Hint 5: Messages don't persist after restart</summary>

Check if you're using `docker compose down -v` (with `-v`).

The `-v` flag removes volumes, deleting your data!

Use `docker compose down` (without `-v`) to preserve volumes.

Verify your volume exists:
```bash
docker volume ls | grep pgdata
```

</details>

---

## Task 4: Permission Issues

<details>
<summary>Hint 1: Finding your user and group ID</summary>

```bash
# Your user ID
id -u
# Example output: 1000

# Your group ID
id -g
# Example output: 1000

# Use in docker run
docker run -u $(id -u):$(id -g) ...
```

On Linux, your first regular user is usually 1000:1000.

</details>

<details>
<summary>Hint 2: Fixing ownership in Dockerfile</summary>

The `chown` must happen BEFORE you switch to a non-root user:

```dockerfile
# Wrong order - will fail:
USER node
RUN chown -R node:node /app  # Can't chown as non-root!

# Correct order:
RUN chown -R node:node /app  # Do this as root
USER node                     # Then switch user
```

Alternative using COPY with ownership:
```dockerfile
COPY --chown=node:node . .
```

</details>

<details>
<summary>Hint 3: Why does this matter?</summary>

Running containers as root is a security risk:
- If the container is compromised, attacker has root access
- Files created by container are owned by root on host
- Principle of least privilege - only use root when necessary

Best practice: Always use a non-root user in production containers.

</details>

---

## Task 5: Dockerfile Optimization

<details>
<summary>Hint 1: .dockerignore contents</summary>

A good `.dockerignore` for Node.js:

```
node_modules
npm-debug.log
.git
.gitignore
*.md
README.md
.env
.env.*
.DS_Store
Dockerfile*
docker-compose*
.dockerignore
coverage
.nyc_output
```

This prevents sending unnecessary files to the Docker daemon.

</details>

<details>
<summary>Hint 2: Optimal layer order</summary>

Things that change LEAST frequently should be FIRST:

```dockerfile
FROM node:18-alpine           # 1. Base image (almost never changes)
WORKDIR /app                  # 2. Working directory (never changes)
COPY package*.json ./         # 3. Dependencies list (changes rarely)
RUN npm ci                    # 4. Install deps (only when package.json changes)
COPY . .                      # 5. Application code (changes frequently)
CMD ["npm", "start"]          # 6. Start command (rarely changes)
```

</details>

<details>
<summary>Hint 3: Combining RUN commands</summary>

Multiple RUN commands create multiple layers:

```dockerfile
# Bad - 3 layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean

# Good - 1 layer, also cleans up
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

The cleanup in the same layer removes the package cache, making the layer smaller.

</details>

<details>
<summary>Hint 4: Complete optimized Dockerfile</summary>

```dockerfile
FROM node:18-alpine

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy dependency files first
COPY package*.json ./

# Install production dependencies only
RUN npm ci --only=production

# Copy application code
COPY --chown=appuser:appgroup . .

# Switch to non-root user
USER appuser

# Document port
EXPOSE 3000

# Start application
CMD ["node", "server.js"]
```

</details>

---

## General Troubleshooting

<details>
<summary>"COPY failed: file not found in build context"</summary>

The file must be in the build context (same directory as Dockerfile or a subdirectory).

Check:
1. You're in the right directory: `pwd`
2. File exists: `ls -la`
3. File name is spelled correctly (case-sensitive!)
4. File isn't in `.dockerignore`

</details>

<details>
<summary>Compose services can't communicate</summary>

Services communicate using **service names** as hostnames:

```yaml
services:
  web:
    environment:
      API_HOST: api    # Use service name, not localhost!

  api:
    environment:
      DB_HOST: db      # Use service name!

  db:
    image: postgres:16
```

Common mistake: Using `localhost` instead of the service name.

</details>

<details>
<summary>Volume data not persisting</summary>

Check:

1. Volume is **named** (not anonymous):
   - Named: `-v mydata:/path` or `volumes: - mydata:/path`
   - Anonymous: `-v /path` (no name - don't use for persistent data)

2. Container path is correct for the application:
   - PostgreSQL: `/var/lib/postgresql/data`
   - MySQL: `/var/lib/mysql`
   - MongoDB: `/data/db`

3. You're NOT using `docker compose down -v` (which removes volumes)

4. Check volume exists:
   ```bash
   docker volume ls
   docker volume inspect <volume-name>
   ```

</details>

<details>
<summary>Build is slow / not using cache</summary>

1. Check Dockerfile order - put frequently changing things last
2. Use `.dockerignore` to exclude unnecessary files
3. Don't invalidate cache unnecessarily:
   ```dockerfile
   # Bad - invalidates cache every second
   RUN echo $(date) > /build-time.txt
   ```

4. If you need to bust the cache intentionally:
   ```bash
   docker build --no-cache -t myapp .
   ```

</details>

---

## Still Stuck?

1. **Read the error message carefully** - Docker errors are usually descriptive

2. **Check the logs:**
   ```bash
   docker logs <container>
   docker compose logs <service>
   ```

3. **Get into the container:**
   ```bash
   docker exec -it <container> sh
   docker compose exec <service> sh
   ```

4. **Search the error message** - Stack Overflow and Docker docs are helpful

5. **Ask in class** - bring your specific error message and what you've tried
