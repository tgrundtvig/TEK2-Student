# Class Exercises: Docker Build & Compose

Work through these exercises during class. Ask for help if you get stuck!

---

## Warm-up: Verify Pre-class Work (~5 minutes)

Quick check that everyone's pre-class setup is working.

### Check your image exists

```bash
docker images | grep hello-docker
```

You should see your `hello-docker` image(s) from the pre-class exercises.

### If you don't have the image

Quick build:

```bash
mkdir -p ~/week2-docker/hello-docker && cd ~/week2-docker/hello-docker
echo '<!DOCTYPE html><html><body><h1>Hello Docker!</h1></body></html>' > index.html
echo 'FROM nginx:1.25' > Dockerfile
echo 'COPY index.html /usr/share/nginx/html/' >> Dockerfile
docker build -t hello-docker:1.0 .
```

### Self-check
- [ ] `docker images` shows at least one hello-docker image
- [ ] You understand what a Dockerfile is

Once verified, move on to the exercises.

---

## Exercise 1: Fix the MySQL Data Loss Problem (~15 minutes)

### Goal

Use volumes to persist database data across container removal.

### Part A: Experience the problem again (~3 minutes)

Let's recreate the Week 1 frustration to understand what we're solving.

```bash
# Run MySQL without a volume
docker run -d --name mysql-no-volume \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0
```

Wait for MySQL to start (check with `docker logs mysql-no-volume` - look for "ready for connections").

```bash
# Create a database
docker exec -it mysql-no-volume mysql -uroot -psecret \
  -e "CREATE DATABASE important_data;"

# Verify it exists
docker exec -it mysql-no-volume mysql -uroot -psecret \
  -e "SHOW DATABASES;"
```

You should see `important_data` in the list.

Now remove the container:

```bash
docker rm -f mysql-no-volume
```

Start a new container with the same name:

```bash
docker run -d --name mysql-no-volume \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0
```

Wait for it to start, then check:

```bash
docker exec -it mysql-no-volume mysql -uroot -psecret \
  -e "SHOW DATABASES;"
```

**Where is `important_data`?** _______________

Clean up:

```bash
docker rm -f mysql-no-volume
```

### Part B: Solve it with volumes (~10 minutes)

Now let's use a named volume to persist the data.

```bash
# Create a named volume
docker volume create mysql-data

# Run MySQL WITH the volume attached
docker run -d --name mysql-persistent \
  -e MYSQL_ROOT_PASSWORD=secret \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

The `-v mysql-data:/var/lib/mysql` means:
- `mysql-data` = the volume name (Docker manages where it's stored)
- `/var/lib/mysql` = the path inside the container where MySQL stores data

Wait for MySQL to start, then create data:

```bash
docker exec -it mysql-persistent mysql -uroot -psecret \
  -e "CREATE DATABASE important_data;"

docker exec -it mysql-persistent mysql -uroot -psecret \
  -e "SHOW DATABASES;"
```

Now the test - remove the container:

```bash
docker rm -f mysql-persistent
```

**The volume still exists!** Check:

```bash
docker volume ls
```

Start a new container attached to the same volume:

```bash
docker run -d --name mysql-persistent \
  -e MYSQL_ROOT_PASSWORD=secret \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

Check if our data survived:

```bash
docker exec -it mysql-persistent mysql -uroot -psecret \
  -e "SHOW DATABASES;"
```

**Is `important_data` there?** _______________

### Part C: Explore volume commands (~2 minutes)

```bash
# List all volumes
docker volume ls

# See volume details
docker volume inspect mysql-data

# DON'T run this yet - it would delete your data:
# docker volume rm mysql-data
```

### Self-check
- [ ] `important_data` database survived container removal
- [ ] I understand `-v mysql-data:/var/lib/mysql` connects the volume
- [ ] I know volumes persist until explicitly deleted with `docker volume rm`

### Keep mysql-persistent running - we might use it later!

---

## Exercise 2: Build Custom nginx with Your HTML (~20 minutes)

### Goal

Create a Dockerfile that builds a custom nginx image with your content.

### Part A: Setup (~2 minutes)

```bash
mkdir -p ~/week2-docker/custom-nginx && cd ~/week2-docker/custom-nginx
```

### Part B: Create content (~5 minutes)

Create `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Week 2 - Custom nginx</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
        }
        h1 { color: #0066cc; }
        .card {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 8px;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <h1>Custom nginx Image</h1>
    <div class="card">
        <h2>Built in Class</h2>
        <p><strong>Name:</strong> [Your Name]</p>
        <p><strong>Date:</strong> [Today's Date]</p>
    </div>
    <div class="card">
        <h2>What I Learned</h2>
        <ul>
            <li>Volumes persist data outside containers</li>
            <li>Dockerfiles create custom images</li>
            <li>Layer caching speeds up builds</li>
        </ul>
    </div>
</body>
</html>
```

Create `Dockerfile`:

```dockerfile
FROM nginx:1.25

# Add metadata labels
LABEL maintainer="your.email@example.com"
LABEL description="Custom nginx for Week 2 class"
LABEL version="1.0"

# Copy our HTML to nginx's default directory
COPY index.html /usr/share/nginx/html/

# nginx already exposes port 80, but document it
EXPOSE 80

# CMD is inherited from nginx image - no need to specify
```

Create `.dockerignore`:

```
*.md
*.log
.git
.DS_Store
```

### Part C: Build the image (~3 minutes)

```bash
# Build with a tag
docker build -t custom-nginx:1.0 .

# Verify it was created
docker images | grep custom-nginx
```

**What's the image size?** _______________

### Part D: Run and test (~3 minutes)

```bash
# Run the container
docker run -d -p 8080:80 --name my-nginx custom-nginx:1.0

# Test with curl
curl http://localhost:8080

# Or open browser to http://localhost:8080
```

### Part E: Make a change and observe caching (~5 minutes)

Edit `index.html` - change something visible (the title, your name, etc.).

Rebuild:

```bash
docker build -t custom-nginx:1.1 .
```

**Watch the output** - notice which layers say "CACHED" and which rebuild.

Update the running container:

```bash
docker rm -f my-nginx
docker run -d -p 8080:80 --name my-nginx custom-nginx:1.1
```

Verify your change appears at http://localhost:8080

### Part F: View labels (~2 minutes)

```bash
docker inspect custom-nginx:1.1 --format='{{json .Config.Labels}}' | python3 -m json.tool
```

You should see the labels you added in the Dockerfile.

### Self-check
- [ ] Image builds without errors
- [ ] Custom page shows in browser at localhost:8080
- [ ] I observed "CACHED" messages when rebuilding
- [ ] I understand each line in the Dockerfile

---

## Exercise 3: Web + Database with Docker Compose (~25 minutes)

### Goal

Define and run a multi-container application with Docker Compose.

### Part A: Setup (~2 minutes)

```bash
mkdir -p ~/week2-docker/compose-demo && cd ~/week2-docker/compose-demo
```

### Part B: Create the application files (~8 minutes)

Create `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Docker Compose Demo</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
        }
        .container {
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 10px 40px rgba(0,0,0,0.2);
        }
        h1 { color: #667eea; margin-top: 0; }
        .service {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 5px;
            margin: 15px 0;
            border-left: 4px solid #667eea;
        }
        .service h3 { margin-top: 0; color: #333; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Docker Compose Demo</h1>
        <p>This application runs multiple containers defined in a single compose.yaml file.</p>

        <div class="service">
            <h3>Web Service (nginx)</h3>
            <p>Serves this HTML page on port 8080.</p>
        </div>

        <div class="service">
            <h3>Database Service (PostgreSQL)</h3>
            <p>Stores data in a persistent volume.</p>
        </div>

        <div class="service">
            <h3>Adminer Service</h3>
            <p>Database management UI on port 8081.</p>
        </div>
    </div>
</body>
</html>
```

Create `compose.yaml`:

```yaml
services:
  web:
    image: nginx:1.25
    ports:
      - "8080:80"
    volumes:
      - ./index.html:/usr/share/nginx/html/index.html:ro
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - pgdata:/var/lib/postgresql/data

  adminer:
    image: adminer:4
    ports:
      - "8081:8080"
    depends_on:
      - db

volumes:
  pgdata:
```

Let's understand this compose.yaml:

| Section | Purpose |
|---------|---------|
| `services:` | Defines three containers: web, db, adminer |
| `image:` | Use existing images from Docker Hub |
| `ports:` | Map host:container ports |
| `volumes:` | Mount files/volumes |
| `:ro` | Read-only mount |
| `environment:` | Set environment variables |
| `depends_on:` | Start order (db starts before web and adminer) |
| `volumes:` (bottom) | Declare named volumes |

### Part C: Start the application (~3 minutes)

```bash
# Start all services in background
docker compose up -d

# Check status
docker compose ps
```

**Expected output:**
```
NAME                    SERVICE   STATUS    PORTS
compose-demo-adminer-1  adminer   running   0.0.0.0:8081->8080/tcp
compose-demo-db-1       db        running   5432/tcp
compose-demo-web-1      web       running   0.0.0.0:8080->80/tcp
```

### Part D: Test the services (~3 minutes)

**Web page:** Open http://localhost:8080

**Adminer (database UI):** Open http://localhost:8081

In Adminer, log in with:
- System: PostgreSQL
- Server: db
- Username: demo
- Password: secret
- Database: appdb

### Part E: Interact with the database (~5 minutes)

Connect via command line:

```bash
docker compose exec db psql -U demo -d appdb
```

In the PostgreSQL prompt:

```sql
-- Create a table
CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    content TEXT,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Insert data
INSERT INTO messages (content) VALUES ('Hello from Docker Compose!');
INSERT INTO messages (content) VALUES ('Data persists in volumes!');

-- Query data
SELECT * FROM messages;

-- Exit
\q
```

### Part F: Test persistence (~4 minutes)

Stop and remove all containers:

```bash
docker compose down
```

**Note:** We did NOT use `-v`, so the volume is preserved.

Start again:

```bash
docker compose up -d
```

Check if data persisted:

```bash
docker compose exec db psql -U demo -d appdb -c "SELECT * FROM messages;"
```

**Is your data still there?** _______________

### Part G: View logs

```bash
# All services
docker compose logs

# Specific service
docker compose logs db

# Follow logs (like tail -f)
docker compose logs -f web
```

Press `Ctrl+C` to stop following logs.

### Self-check
- [ ] All three services start with `docker compose up -d`
- [ ] Web page accessible at localhost:8080
- [ ] Adminer accessible at localhost:8081
- [ ] Database data persists after `docker compose down` and `up`
- [ ] I understand each section of compose.yaml

---

## Exercise 4: Push to Docker Hub (Bonus) (~10 minutes)

> **Note:** This exercise requires a Docker Hub account. If you don't have one, create a free account at [hub.docker.com](https://hub.docker.com) or skip this exercise.

### Goal

Share your custom image on Docker Hub.

### Part A: Log in to Docker Hub

```bash
docker login
```

Enter your Docker Hub username and password.

### Part B: Tag your image for Docker Hub

Docker Hub images follow the format: `username/repository:tag`

```bash
# Replace YOUR_USERNAME with your Docker Hub username
docker tag custom-nginx:1.0 YOUR_USERNAME/custom-nginx:1.0
```

### Part C: Push to Docker Hub

```bash
docker push YOUR_USERNAME/custom-nginx:1.0
```

This uploads your image to Docker Hub. It might take a minute.

### Part D: Verify on Docker Hub

Go to https://hub.docker.com and sign in. You should see your `custom-nginx` repository.

### Part E: Test pulling (optional)

To prove it works, you could:

1. Remove the local image: `docker rmi YOUR_USERNAME/custom-nginx:1.0`
2. Pull it back: `docker pull YOUR_USERNAME/custom-nginx:1.0`
3. Run it: `docker run -d -p 8082:80 YOUR_USERNAME/custom-nginx:1.0`

### Self-check
- [ ] Successfully logged in to Docker Hub
- [ ] Image pushed without errors
- [ ] Image visible on Docker Hub website

---

## Final Cleanup

Clean up containers and optionally volumes:

```bash
# Stop the compose application
cd ~/week2-docker/compose-demo
docker compose down

# Stop and remove other containers
docker rm -f mysql-persistent my-nginx 2>/dev/null

# Remove compose volumes if you want (DELETES DATA)
# docker compose down -v

# List remaining containers
docker ps -a

# List remaining volumes
docker volume ls
```

To remove all stopped containers:

```bash
docker container prune
```

To remove unused volumes (be careful - this deletes data!):

```bash
docker volume prune
```

---

## Summary

Today you practiced:

| Skill | Commands/Concepts |
|-------|-------------------|
| Volumes for persistence | `docker volume create`, `-v name:/path` |
| Building images | `docker build -t name:tag .` |
| Dockerfile instructions | `FROM`, `COPY`, `LABEL`, `EXPOSE` |
| Docker Compose | `docker compose up/down/ps/logs/exec` |
| compose.yaml structure | services, ports, volumes, environment, depends_on |
| Docker Hub | `docker login`, `docker tag`, `docker push` |

**Key takeaways:**

1. **Volumes solve data persistence** - use `-v volume-name:/path` for databases
2. **Dockerfiles create repeatable images** - no more manual setup
3. **Compose simplifies multi-container apps** - one file, one command
4. **Service names become hostnames** - containers can reach each other by name
5. **`docker compose down -v` deletes volumes** - be careful with production data!
