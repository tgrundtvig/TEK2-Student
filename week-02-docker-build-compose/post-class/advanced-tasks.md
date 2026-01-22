# Post-class Advanced Tasks

**Estimated time: 1-2 hours**

These tasks will deepen your understanding of Docker images and Compose. Try to complete them before next week's class.

If you get stuck, check the [hints.md](hints.md) file - but try on your own first!

---

## Task 1: Build a Node.js Application (~30 minutes)

### Objective

Build a complete Node.js REST API with Docker, demonstrating proper Dockerfile structure and layer caching.

### Instructions

1. Create a project folder:

```bash
mkdir ~/docker-node-app && cd ~/docker-node-app
```

2. Create `package.json`:

```json
{
  "name": "docker-node-app",
  "version": "1.0.0",
  "description": "Node.js app for Docker practice",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

3. Create `server.js`:

```javascript
const express = require('express');
const os = require('os');

const app = express();
const port = process.env.PORT || 3000;

// JSON middleware
app.use(express.json());

// Root endpoint
app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Node.js in Docker!',
    hostname: os.hostname(),
    platform: os.platform(),
    uptime: process.uptime(),
    timestamp: new Date().toISOString()
  });
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', uptime: process.uptime() });
});

// Environment info endpoint
app.get('/env', (req, res) => {
  res.json({
    nodeVersion: process.version,
    environment: process.env.NODE_ENV || 'development'
  });
});

app.listen(port, () => {
  console.log(`Server running on port ${port}`);
  console.log(`Environment: ${process.env.NODE_ENV || 'development'}`);
});
```

4. Create `Dockerfile`:

```dockerfile
# Use official Node.js Alpine image (smaller)
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files first (for layer caching)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Document the port
EXPOSE 3000

# Set default environment
ENV NODE_ENV=production

# Start the application
CMD ["npm", "start"]
```

5. Create `.dockerignore`:

```
node_modules
npm-debug.log
.git
.gitignore
*.md
.env
.DS_Store
```

6. Build and test:

```bash
# Build the image
docker build -t node-app:1.0 .

# Run the container
docker run -d -p 3000:3000 --name myapp node-app:1.0

# Test the endpoints
curl http://localhost:3000
curl http://localhost:3000/health
curl http://localhost:3000/env
```

### Verify Success

- [ ] Image builds without errors
- [ ] Container starts and stays running
- [ ] All three endpoints return JSON responses
- [ ] `/` shows the container hostname (different from your computer)
- [ ] `/env` shows `production` environment

### Challenge Extension

- Add a `POST /message` endpoint that stores messages in memory
- Add a `GET /messages` endpoint to retrieve stored messages
- How would you persist these messages using a volume?

---

## Task 2: Multi-Stage Build (~20 minutes)

### Objective

Create a smaller, more secure production image using multi-stage builds.

### Background

The Node.js image from Task 1 includes npm, development tools, and other things not needed in production. Multi-stage builds let you:
1. Use a full image for building
2. Copy only what you need to a minimal runtime image

### Instructions

1. In your `docker-node-app` folder, create `Dockerfile.multistage`:

```dockerfile
# ====================
# Stage 1: Build
# ====================
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ALL dependencies (including devDependencies)
RUN npm ci

# Copy source code
COPY . .

# If you had a build step (TypeScript, webpack), it would go here:
# RUN npm run build

# ====================
# Stage 2: Production
# ====================
FROM node:18-alpine AS production

WORKDIR /app

# Copy only production dependencies from builder
COPY --from=builder /app/node_modules ./node_modules

# Copy application code
COPY server.js .
COPY package.json .

# Use non-root user for security
USER node

# Document port
EXPOSE 3000

# Set production environment
ENV NODE_ENV=production

# Start application
CMD ["node", "server.js"]
```

2. Build both versions and compare:

```bash
# Build the regular version
docker build -t node-app:regular -f Dockerfile .

# Build the multi-stage version
docker build -t node-app:multistage -f Dockerfile.multistage .

# Compare sizes
docker images | grep node-app
```

### Verify Success

- [ ] Multi-stage image builds successfully
- [ ] Multi-stage image is smaller than regular image
- [ ] Application still works: `docker run -d -p 3001:3000 node-app:multistage`
- [ ] You understand the purpose of each stage

### Questions to Consider

1. Why is the multi-stage image smaller?
2. What does `USER node` do and why is it important?
3. When would you NOT use multi-stage builds?

---

## Task 3: Full Stack Application with Compose (~30 minutes)

### Objective

Build a complete application with frontend, API, and database using Docker Compose.

### Instructions

1. Create project structure:

```bash
mkdir -p ~/fullstack-app/{frontend,api}
cd ~/fullstack-app
```

2. Create `api/package.json`:

```json
{
  "name": "fullstack-api",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "pg": "^8.11.3"
  }
}
```

3. Create `api/server.js`:

```javascript
const express = require('express');
const { Pool } = require('pg');

const app = express();
app.use(express.json());

// Database connection
const pool = new Pool({
  connectionString: process.env.DATABASE_URL
});

// Initialize database
async function initDb() {
  try {
    await pool.query(`
      CREATE TABLE IF NOT EXISTS messages (
        id SERIAL PRIMARY KEY,
        content TEXT NOT NULL,
        created_at TIMESTAMP DEFAULT NOW()
      )
    `);
    console.log('Database initialized');
  } catch (err) {
    console.error('Database init error:', err.message);
    // Retry after 2 seconds (database might not be ready)
    setTimeout(initDb, 2000);
  }
}

// Routes
app.get('/api/messages', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM messages ORDER BY created_at DESC');
    res.json(result.rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.post('/api/messages', async (req, res) => {
  try {
    const { content } = req.body;
    const result = await pool.query(
      'INSERT INTO messages (content) VALUES ($1) RETURNING *',
      [content]
    );
    res.status(201).json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.get('/api/health', (req, res) => {
  res.json({ status: 'healthy' });
});

const port = process.env.PORT || 4000;
app.listen(port, () => {
  console.log(`API running on port ${port}`);
  initDb();
});
```

4. Create `api/Dockerfile`:

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 4000
CMD ["npm", "start"]
```

5. Create `frontend/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Full Stack Docker App</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
        }
        h1 { color: #333; }
        .form-group {
            margin: 20px 0;
        }
        input[type="text"] {
            padding: 10px;
            width: 70%;
            font-size: 16px;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            background: #0066cc;
            color: white;
            border: none;
            cursor: pointer;
        }
        button:hover { background: #0052a3; }
        .messages {
            margin-top: 30px;
        }
        .message {
            background: #f5f5f5;
            padding: 15px;
            margin: 10px 0;
            border-radius: 5px;
        }
        .message small {
            color: #666;
        }
        .error {
            color: red;
            padding: 10px;
            background: #fee;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <h1>Full Stack Docker App</h1>
    <p>Frontend (nginx) + API (Node.js) + Database (PostgreSQL)</p>

    <div class="form-group">
        <input type="text" id="messageInput" placeholder="Enter a message...">
        <button onclick="addMessage()">Add Message</button>
    </div>

    <div id="error"></div>
    <div id="messages" class="messages"></div>

    <script>
        const API_URL = 'http://localhost:4000';

        async function loadMessages() {
            try {
                const res = await fetch(`${API_URL}/api/messages`);
                const messages = await res.json();
                document.getElementById('messages').innerHTML = messages
                    .map(m => `
                        <div class="message">
                            <p>${m.content}</p>
                            <small>${new Date(m.created_at).toLocaleString()}</small>
                        </div>
                    `).join('');
                document.getElementById('error').innerHTML = '';
            } catch (err) {
                document.getElementById('error').innerHTML =
                    '<p class="error">Could not connect to API. Is it running?</p>';
            }
        }

        async function addMessage() {
            const input = document.getElementById('messageInput');
            const content = input.value.trim();
            if (!content) return;

            try {
                await fetch(`${API_URL}/api/messages`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ content })
                });
                input.value = '';
                loadMessages();
            } catch (err) {
                document.getElementById('error').innerHTML =
                    '<p class="error">Could not add message</p>';
            }
        }

        // Load messages on page load
        loadMessages();
        // Refresh every 5 seconds
        setInterval(loadMessages, 5000);
    </script>
</body>
</html>
```

6. Create `compose.yaml` in the root folder:

```yaml
services:
  frontend:
    image: nginx:1.25
    ports:
      - "3000:80"
    volumes:
      - ./frontend:/usr/share/nginx/html:ro
    depends_on:
      - api

  api:
    build: ./api
    ports:
      - "4000:4000"
    environment:
      DATABASE_URL: postgres://postgres:secret@db:5432/appdb
      PORT: 4000
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

7. Start and test:

```bash
docker compose up -d
docker compose ps
docker compose logs api
```

8. Open http://localhost:3000 in your browser. Add some messages!

### Verify Success

- [ ] All three services start successfully
- [ ] Frontend loads at http://localhost:3000
- [ ] You can add and view messages
- [ ] Messages persist after `docker compose down` and `up` (without `-v`)

### Challenge Extension

- Add an Adminer service for database management
- Add CORS headers to the API (it's currently relying on same-origin)
- Add a Redis service for caching

---

## Task 4: Troubleshoot Permission Issues (~20 minutes)

### Objective

Understand and fix common permission problems with Docker volumes.

### Scenario 1: Files owned by root

```bash
# Create a test directory
mkdir ~/permission-test && cd ~/permission-test

# Create a file via Docker
docker run --rm -v $(pwd):/data alpine sh -c "echo 'hello' > /data/file.txt"

# Check ownership
ls -la file.txt
```

**What user owns the file?** _______________

**Why?** The container runs as root by default.

### Fix: Run as your user

```bash
# Run as your user ID
docker run --rm -u $(id -u):$(id -g) -v $(pwd):/data alpine sh -c "echo 'hello' > /data/file2.txt"

# Check ownership
ls -la file2.txt
```

**What user owns file2.txt now?** _______________

### Scenario 2: Node.js can't write

Create a Dockerfile that demonstrates the issue:

```dockerfile
FROM node:18-alpine
WORKDIR /app
RUN echo '{"test": true}' > /app/data.json
USER node
CMD ["cat", "/app/data.json"]
```

```bash
docker build -t permission-demo .
docker run --rm permission-demo
```

This works. But try writing:

```bash
docker run --rm permission-demo sh -c "echo 'new' > /app/test.txt"
```

**What error do you get?** _______________

### Fix: Set proper ownership

```dockerfile
FROM node:18-alpine
WORKDIR /app
RUN echo '{"test": true}' > /app/data.json
# Fix ownership BEFORE switching to node user
RUN chown -R node:node /app
USER node
CMD ["cat", "/app/data.json"]
```

### Verify Success

- [ ] You understand why files are owned by root by default
- [ ] You can run containers as your user with `-u $(id -u):$(id -g)`
- [ ] You know to use `chown` in Dockerfile before `USER`

---

## Task 5: Optimize a Dockerfile (~20 minutes)

### Objective

Practice improving a poorly-written Dockerfile.

### The Bad Dockerfile

Create this inefficient Dockerfile:

```dockerfile
FROM node:18
COPY . /app
WORKDIR /app
RUN apt-get update
RUN apt-get install -y curl
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]
```

### Problems to Fix

1. **Large base image** - `node:18` is ~900MB
2. **No .dockerignore** - sends unnecessary files
3. **Bad layer order** - `npm install` runs on every code change
4. **Multiple RUN commands** - creates extra layers
5. **Running as root** - security concern

### Your Task

Create an optimized `Dockerfile.optimized` that:

1. Uses `node:18-alpine` base image
2. Orders commands for optimal caching
3. Combines apt-get commands (if still needed)
4. Adds a non-root user
5. Creates a proper `.dockerignore`

### Verify Success

```bash
# Build both versions
docker build -t app:bad -f Dockerfile .
docker build -t app:optimized -f Dockerfile.optimized .

# Compare sizes
docker images | grep app

# Time a rebuild after changing code
time docker build -t app:bad -f Dockerfile .
time docker build -t app:optimized -f Dockerfile.optimized .
```

- [ ] Optimized image is significantly smaller
- [ ] Optimized image builds faster when only code changes
- [ ] You can explain each optimization

---

## Going Further (Optional)

If you finish early and want more challenges:

1. **Health checks in Compose:**
   ```yaml
   healthcheck:
     test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
     interval: 30s
     timeout: 10s
     retries: 3
   ```

2. **Build arguments:**
   ```dockerfile
   ARG NODE_VERSION=18
   FROM node:${NODE_VERSION}-alpine
   ```
   Build with: `docker build --build-arg NODE_VERSION=20 .`

3. **Compose profiles:**
   ```yaml
   services:
     debug-tools:
       profiles: ["debug"]
   ```
   Start with: `docker compose --profile debug up`

---

## Before Next Class

Come prepared to:

- Share one thing you learned from these tasks
- Ask questions about anything that confused you
- Show your optimized Dockerfile (Task 5)

No formal submission required, but completing these tasks will help you in Week 3 when we add CI/CD!
