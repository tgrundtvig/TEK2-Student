# Tutorial: Deploy Your Own Project with CI/CD on Azure

This tutorial walks you through setting up a complete CI/CD pipeline for **your own project** — from writing Dockerfiles to fully automated deployment on your Azure VM.

By the end, every push to `main` will automatically build, test, and deploy your application.

**Estimated time: 90-120 minutes**

---

## What You'll Build

A fully automated pipeline that deploys your project — a Java/Spring Boot application (serving both the backend API and the HTML/CSS/JS frontend) with a MySQL database — to your Azure VM.

```
┌─────────────────────────────────────────────────┐
│  Your Azure VM                                   │
│                                                   │
│  ┌───────────────────┐        ┌───────────┐      │
│  │       app         │───────▶│    db      │      │
│  │    (Spring Boot)  │        │  (MySQL)   │      │
│  │   serves HTML +   │        │ port 3306  │      │
│  │   API on port 80  │        └───────────┘      │
│  └───────────────────┘                            │
│          ▲                                        │
│          │ port 80                                 │
└──────────┼────────────────────────────────────────┘
           │
       Internet
```

Spring Boot serves everything: your static files (HTML, CSS, JS) and your API endpoints. Only the app container is exposed to the internet.

---

## Example Project

There's a complete, working example of what you'll build in this tutorial:

**[github.com/tgrundtvig/tek2-week04-example](https://github.com/tgrundtvig/tek2-week04-example)**

It's a minimal Spring Boot app with a single API endpoint, a static HTML frontend, and all the Docker and CI/CD files already set up. Use it as a reference if you get stuck, or clone it to see the full setup running before you start on your own project.

---

## Prerequisites

Before starting, you should have:

- [ ] **Docker** installed on your local machine — this is the only tool you need. You do **not** need Java, Maven, or anything else installed locally.
- [ ] A GitHub repository containing your project code
- [ ] A Java/Spring Boot backend (with a `pom.xml`)
- [ ] An HTML/CSS/JS frontend (in `src/main/resources/static/`)
- [ ] An Azure VM with SSH access and Docker installed — if you don't have one yet, see [Appendix A](#appendix-a-azure-vm-setup) at the bottom of this tutorial

---

## Project Structure

This tutorial assumes your project is organized like this (the standard Spring Boot layout):

```
your-project/
├── pom.xml                    ← Spring Boot project at root
├── Dockerfile                 ← Production image (created in Step 1)
├── Dockerfile.build           ← Dev build environment (created below)
├── src/
│   └── main/
│       ├── java/...           ← Your Java code
│       └── resources/
│           ├── application.properties
│           └── static/
│               ├── index.html ← Your HTML/CSS/JS frontend
│               ├── style.css
│               └── script.js
└── README.md
```

Spring Boot automatically serves files from `src/main/resources/static/` at the root URL. So `static/index.html` becomes available at `http://localhost:8080/`.

> **Frontend in a different folder?** If your HTML files are currently in a separate `frontend/` folder, move them into `src/main/resources/static/`. Spring Boot won't serve them otherwise.

---

## Build Environment: No Java Installation Needed

A common source of frustration is getting Java, Maven, and `JAVA_HOME` set up correctly on your machine. Different versions, different paths, different operating systems — it's a mess.

The good news: **you don't need any of that.** Docker can be your build environment. Instead of installing JDK 21 and Maven locally, you'll use a Docker container that has everything pre-configured. The build runs inside the container, and the result (your JAR file) appears on your machine.

```
┌──────────────────────────────────────────┐
│  Dev build container                      │
│                                           │
│  ✓ JDK 21 — already installed            │
│  ✓ Maven 3.9.12 — already installed      │
│  ✓ JAVA_HOME — already configured        │
│                                           │
│  Your project files are mounted into /app │
│  Build output goes to target/ on your     │
│  machine                                  │
└──────────────────────────────────────────┘
```

Create a file called `Dockerfile.build` in your project root:

```dockerfile
FROM eclipse-temurin:21-jdk

# Install Maven 3.9.12
ENV MAVEN_VERSION=3.9.12
ENV MAVEN_HOME=/opt/maven
ENV PATH="${MAVEN_HOME}/bin:${PATH}"

RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    curl -fsSL https://dlcdn.apache.org/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz \
      | tar xz -C /opt && \
    mv /opt/apache-maven-${MAVEN_VERSION} ${MAVEN_HOME} && \
    apt-get remove -y curl && apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
```

That's it — JDK 21, Maven 3.9.12, and nothing else. You'll wire this into Docker Compose in Step 2. Once it's ready, you'll build your project like this:

```bash
docker compose run --rm build mvn clean package
```

No Java installation, no `JAVA_HOME` issues, and everyone on the team gets the exact same build environment.

> **Can I still use my local Java?** Of course. If you have Java and Maven working on your machine, you can keep using `mvn clean package` directly. The Docker build environment is there for when things go wrong or when you want a guaranteed consistent setup.

---

## Step 1: Dockerize Your Application (~15 minutes)

Your Spring Boot application needs a `Dockerfile` that builds the Java code and creates a runnable container image.

### 1.1: Check your pom.xml

Before creating the Dockerfile, make sure your `pom.xml` includes the Spring Boot Maven plugin. This is what creates the executable JAR file. Open your `pom.xml` and look for this in the `<build>` section:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

If it's missing, add it. Without this plugin, Maven creates a regular JAR that can't be run on its own.

### 1.2: Check your MySQL dependency

Your `pom.xml` also needs the MySQL connector so Spring Boot can talk to the database. Check that you have this in the `<dependencies>` section:

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

If it's missing, add it.

### 1.3: Configure database connection

Your `src/main/resources/application.properties` needs to be configured so Spring Boot can connect to the MySQL database. Add or update these lines:

```properties
spring.datasource.url=${DATABASE_URL:jdbc:mysql://localhost:3306/mydb}
spring.datasource.username=${DATABASE_USERNAME:root}
spring.datasource.password=${DATABASE_PASSWORD:root}
spring.jpa.hibernate.ddl-auto=update
```

> **What does this mean?** The `${DATABASE_URL:jdbc:mysql://...}` syntax means "use the `DATABASE_URL` environment variable if it exists, otherwise fall back to the default value." This way, the same code works both locally and in Docker.

Replace `mydb` with whatever you want to call your database.

### 1.4: Create the Dockerfile

Create a file called `Dockerfile` in your project root (next to `pom.xml`):

```dockerfile
# Stage 1: Build the application
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Run the application
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**What each line does:**

| Line | Purpose |
|------|---------|
| `FROM maven:3.9-eclipse-temurin-21 AS build` | Start with a Maven + JDK image for building |
| `COPY pom.xml .` then `COPY src ./src` | Copy your source code into the build container |
| `RUN mvn clean package -DskipTests` | Compile your code and create the JAR file |
| `FROM eclipse-temurin:21-jre-alpine` | Start a fresh, small image with only the JRE (no build tools) |
| `COPY --from=build /app/target/*.jar app.jar` | Copy the built JAR from the build stage |
| `ENTRYPOINT ["java", "-jar", "app.jar"]` | Run the application when the container starts |

> **Why two stages?** The build stage contains Maven, the full JDK, and all your source code — about 800 MB. The runtime stage only has the JRE and your JAR — about 200 MB. Your server doesn't need the build tools.

> **What about the frontend files?** Your HTML/CSS/JS files live inside `src/main/resources/static/`, so they get compiled into the JAR file automatically. Spring Boot serves them — no separate web server needed.

### 1.5: Test the Dockerfile locally

```bash
docker build -t my-app .
docker run --rm -p 8080:8080 my-app
```

The application will probably crash because there's no database — that's fine! You should see Spring Boot starting up before the database error. If you see `Started MyApplication in X seconds`, your Dockerfile works. Press `Ctrl+C` to stop.

> **If the build fails:** Check that your `pom.xml` is valid. You can verify with the build service from Step 2: `docker compose run --rm build mvn clean package` — this runs Maven inside Docker, so Java version and JAVA_HOME issues can't be the cause.

---

## Step 2: Create Docker Compose Files (~10 minutes)

You'll create two compose files:
- **`docker-compose.yml`** — for local development (builds from your source code)
- **`docker-compose.prod.yml`** — for production (pulls pre-built images from GitHub)

Both compose files read the database password from a `.env` file, so the password never ends up in your Git history.

### 2.1: Create a `.env` file

Create a file called `.env` in your project root:

```
DB_PASSWORD=root
```

> **You can change `root` to any password you like.** This is just the default for local development.

Now **add `.env` to your `.gitignore`** so it never gets committed:

```bash
echo ".env" >> .gitignore
```

> **Why a `.env` file?** Docker Compose automatically reads variables from `.env` in the same directory. By keeping passwords in `.env` and adding it to `.gitignore`, your `docker-compose.yml` can be safely pushed to GitHub without leaking secrets.

### 2.2: Development compose file

Create `docker-compose.yml` in your project root:

```yaml
services:
  build:
    build:
      context: .
      dockerfile: Dockerfile.build
    working_dir: /app
    volumes:
      - .:/app
      - maven-cache:/root/.m2
    profiles: ["tools"]

  db:
    image: mysql:8
    environment:
      MYSQL_DATABASE: mydb
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
    volumes:
      - mysqldata:/var/lib/mysql
    ports:
      - "3306:3306"

  app:
    build: .
    environment:
      DATABASE_URL: jdbc:mysql://db:3306/mydb
      DATABASE_USERNAME: root
      DATABASE_PASSWORD: ${DB_PASSWORD}
    depends_on:
      - db
    ports:
      - "8080:8080"

volumes:
  mysqldata:
  maven-cache:
```

> **Replace `mydb`** with the database name you used in `application.properties`.

**The `build` service** uses your `Dockerfile.build` to create a container with JDK 21 and Maven. It mounts your project directory into the container and caches Maven dependencies so they're only downloaded once. The `profiles: ["tools"]` line means it won't start when you run `docker compose up` — it's only used when you explicitly call it:

```bash
# Build your project (same as mvn clean package, but inside Docker)
docker compose run --rm build mvn clean package

# Run tests
docker compose run --rm build mvn test

# Any Maven command works
docker compose run --rm build mvn dependency:tree
```

The `--rm` flag removes the container after it finishes, keeping things tidy. The built JAR file appears in your local `target/` folder as usual.

**Test the full stack:**

```bash
docker compose up --build
```

Open `http://localhost:8080` — your full application should be running! Spring Boot serves your HTML frontend and handles API requests, all connected to MySQL.

Press `Ctrl+C` to stop.

### 2.3: Production compose file

Create `docker-compose.prod.yml` in your project root:

```yaml
services:
  db:
    image: mysql:8
    environment:
      MYSQL_DATABASE: mydb
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
    volumes:
      - mysqldata:/var/lib/mysql

  app:
    image: ghcr.io/YOUR_USERNAME/YOUR_PROJECT:latest
    environment:
      DATABASE_URL: jdbc:mysql://db:3306/mydb
      DATABASE_USERNAME: root
      DATABASE_PASSWORD: ${DB_PASSWORD}
    depends_on:
      - db
    ports:
      - "80:8080"

volumes:
  mysqldata:
```

**Replace these placeholders:**

| Placeholder | Replace with | Example |
|-------------|-------------|---------|
| `YOUR_USERNAME` | Your GitHub username (lowercase!) | `johndoe` |
| `YOUR_PROJECT` | Your repository name (lowercase!) | `my-cool-app` |
| `mydb` | Your database name | `mydb` |

> **Why lowercase?** GitHub Container Registry (GHCR) requires image names to be lowercase. If your GitHub username is `JohnDoe`, use `johndoe` in the image names.

**What's different from the dev file?**

| | Development | Production |
|---|---|---|
| App | `build: .` (builds from source) | `image: ghcr.io/...` (pulls pre-built image) |
| App port | `8080:8080` | `80:8080` (accessible on port 80) |
| DB port | Exposed (`3306:3306`) | Not exposed (internal only) |

---

## Step 3: Create the GitHub Actions Workflow (~15 minutes)

This is the heart of the CI/CD pipeline. You'll create a workflow file that GitHub runs automatically every time you push to `main`.

### 3.1: Create the workflow file

Create the file `.github/workflows/ci.yml` (you'll need to create the `.github/workflows/` directories):

```bash
mkdir -p .github/workflows
```

Then create `.github/workflows/ci.yml` with this content:

```yaml
name: CI/CD

on:
  push:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'

      - name: Build and test with Maven
        run: mvn clean package

  build-and-push:
    needs: build-and-test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/YOUR_USERNAME/YOUR_PROJECT:latest
            ghcr.io/YOUR_USERNAME/YOUR_PROJECT:${{ github.sha }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Azure VM
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_IP }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd ~/my-app

            # Pull the latest images
            docker compose -f docker-compose.prod.yml pull

            # Restart with the new images
            docker compose -f docker-compose.prod.yml up -d

            # Clean up old images
            docker image prune -f
```

**Replace `YOUR_USERNAME` and `YOUR_PROJECT`** with your lowercase GitHub username and repository name — in both image tag lines.

> **Important:** Also change `cd ~/my-app` in the deploy script to match the directory name you'll create on your server in Step 6.

### 3.2: How the pipeline works

Here's what happens when you push to `main`:

```
    build-and-test
   (compile + tests)
         │
         ▼
    build-and-push
 (Docker image → GHCR)
         │
         ▼
       deploy
(SSH → pull → restart)
```

| Job | What it does |
|-----|-------------|
| **build-and-test** | Checks out your code, installs JDK 21, runs `mvn clean package` (compiles + runs tests) |
| **build-and-push** | Builds a Docker image from your `Dockerfile`, pushes it to GHCR |
| **deploy** | SSHes into your Azure VM, pulls the new image, restarts the containers |

### 3.3: Verify your image names match

The image names must be **exactly the same** in three places:

1. `.github/workflows/ci.yml` (the `tags:` lines)
2. `docker-compose.prod.yml` (the `image:` line)
3. What you log in to pull on the server

Quick check:

```bash
# This should show only YOUR username/project, not the placeholders
grep -r "ghcr.io/" .github/ docker-compose.prod.yml
```

---

## Step 4: Set Up SSH Keys for Deployment (~15 minutes)

The deploy job needs to SSH into your Azure VM. You'll create a **dedicated deploy key** — a separate SSH key used only by GitHub Actions.

### 4.1: Why a dedicated key?

You probably already have an SSH key (`~/.ssh/id_ed25519`) that you use to connect to your VM. So why create a new one?

- **Separation of concerns:** If the deploy key is compromised, you can revoke it without losing your personal SSH access
- **Principle of least privilege:** The deploy key only needs to exist on GitHub and on the server — nowhere else
- **Easy to rotate:** You can regenerate a deploy key without disrupting your day-to-day SSH access

### 4.2: Understanding SSH key pairs

An SSH key pair consists of two files:

```
┌─────────────────┐           ┌──────────────────┐
│   PRIVATE KEY   │           │   PUBLIC KEY      │
│                 │           │                   │
│  deploy_key     │           │  deploy_key.pub   │
│                 │           │                   │
│  Stays secret!  │           │  Safe to share    │
│  Like a         │           │  Like a           │
│  password       │           │  lock             │
└─────────────────┘           └──────────────────┘
     Goes to:                      Goes to:
  GitHub Secrets               Your Azure VM
  (SSH_PRIVATE_KEY)            (~/.ssh/authorized_keys)
```

The private key proves your identity. The public key is the lock that only the private key can open.

### 4.3: Generate the deploy key

On your **local machine**, run:

```bash
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/deploy_key -N ""
```

| Flag | Meaning |
|------|---------|
| `-t ed25519` | Use the Ed25519 algorithm (modern, secure, fast) |
| `-C "github-actions-deploy"` | A comment to identify this key's purpose |
| `-f ~/.ssh/deploy_key` | Save to this file (not the default `id_ed25519`) |
| `-N ""` | No passphrase (GitHub Actions can't type a password) |

This creates two files:
- `~/.ssh/deploy_key` — the private key
- `~/.ssh/deploy_key.pub` — the public key

### 4.4: Add the public key to your Azure VM

You need to add the public key to your server so it accepts connections from GitHub Actions.

First, display the public key:

```bash
cat ~/.ssh/deploy_key.pub
```

It looks something like:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... github-actions-deploy
```

Now SSH into your server (using your normal key) and add the deploy key:

```bash
ssh azureuser@YOUR_SERVER_IP
```

Once on the server:

```bash
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI..." >> ~/.ssh/authorized_keys
```

(Paste the actual public key you copied above.)

> **`>>` not `>`!** Using `>>` appends to the file. Using `>` would overwrite the file, deleting your personal key and locking you out!

Verify it was added:

```bash
cat ~/.ssh/authorized_keys
```

You should see at least two keys: your personal key and the new deploy key.

Then exit the server:

```bash
exit
```

### 4.5: Test the deploy key

Before configuring GitHub, verify the key works from your local machine:

```bash
ssh -i ~/.ssh/deploy_key azureuser@YOUR_SERVER_IP echo "Deploy key works!"
```

You should see: `Deploy key works!`

**If it fails, check these common issues:**

| Problem | Fix |
|---------|-----|
| `Permission denied (publickey)` | The public key wasn't added correctly to `authorized_keys`. SSH in with your normal key and check the file. |
| `WARNING: UNPROTECTED PRIVATE KEY FILE!` | Run `chmod 600 ~/.ssh/deploy_key` — the private key file must not be readable by others. |
| `Connection refused` | Is the VM running? Check Azure Portal. |
| `Connection timed out` | Is the IP correct? Is port 22 open in Azure firewall? |

---

## Step 5: Configure GitHub Secrets (~5 minutes)

GitHub Secrets are encrypted variables that only GitHub Actions can read. You'll store your server connection details here.

### 5.1: Navigate to secrets

1. Go to your repository on GitHub
2. Click **Settings** (top menu, far right)
3. In the left sidebar, click **Secrets and variables** → **Actions**
4. Click **"New repository secret"**

### 5.2: Add three secrets

Add each of these one at a time:

**Secret 1: `SERVER_IP`**

| Field | Value |
|-------|-------|
| Name | `SERVER_IP` |
| Secret | Your Azure VM's public IP address (e.g., `20.234.123.45`) |

Click **"Add secret"**.

**Secret 2: `SERVER_USER`**

| Field | Value |
|-------|-------|
| Name | `SERVER_USER` |
| Secret | `azureuser` (or whatever username you set when creating the VM) |

Click **"Add secret"**.

**Secret 3: `SSH_PRIVATE_KEY`**

| Field | Value |
|-------|-------|
| Name | `SSH_PRIVATE_KEY` |
| Secret | The **entire** contents of `~/.ssh/deploy_key` |

To get the private key:

```bash
cat ~/.ssh/deploy_key
```

Copy **everything** — including the `-----BEGIN OPENSSH PRIVATE KEY-----` and `-----END OPENSSH PRIVATE KEY-----` lines. Make sure there's no extra whitespace or missing newlines.

Click **"Add secret"**.

### 5.3: Verify

After adding all three, you should see them listed under "Repository secrets":

```
SERVER_IP         Updated just now
SERVER_USER       Updated just now
SSH_PRIVATE_KEY   Updated just now
```

You can't view the values after saving (that's the point of secrets), but you can update them if needed.

---

## Step 6: Prepare Your Azure VM (~15 minutes)

Your server needs Docker (you should already have this) and a few things set up before the first deployment.

### 6.1: Verify Docker is installed

SSH into your server:

```bash
ssh azureuser@YOUR_SERVER_IP
```

Check Docker:

```bash
docker --version
sudo systemctl status docker
```

If Docker isn't installed, follow the [Docker installation exercise](../class/exercises.md#exercise-3-install-docker-20-minutes) from the class materials.

### 6.2: Authenticate with GitHub Container Registry

Your server needs permission to pull your Docker images from GHCR. You'll create a Personal Access Token (PAT) for this.

1. Go to GitHub → click your avatar → **Settings**
2. Scroll down in the left sidebar → **Developer settings**
3. **Personal access tokens** → **Tokens (classic)**
4. Click **"Generate new token (classic)"**
5. Configure:

| Setting | Value |
|---------|-------|
| Note | `server-docker-access` |
| Expiration | 90 days (or longer) |
| Scopes | Check **`read:packages`** only |

6. Click **"Generate token"**
7. **Copy the token immediately** — you won't see it again!

Now log in on your server:

```bash
echo "YOUR_TOKEN" | docker login ghcr.io -u YOUR_USERNAME --password-stdin
```

Replace `YOUR_TOKEN` with the token you just copied and `YOUR_USERNAME` with your GitHub username.

You should see: `Login Succeeded`

### 6.3: Create the app directory and compose file

The deploy job expects to find a production compose file on the server. Create the directory:

```bash
mkdir -p ~/my-app
```

> **Match the directory name** to what you used in the `cd ~/my-app` line in your workflow file (Step 3).

Create the compose file:

```bash
nano ~/my-app/docker-compose.prod.yml
```

Paste the contents of your `docker-compose.prod.yml` (from Step 2.3) — with your actual username and project name already filled in.

Save with `Ctrl+O`, Enter, `Ctrl+X`.

### 6.4: Create a `.env` file on the server

The compose file reads the database password from a `.env` file. Create one:

```bash
echo "DB_PASSWORD=$(openssl rand -base64 24)" > ~/my-app/.env
cat ~/my-app/.env   # See what password was generated
```

This generates a strong random password for your production database. Docker Compose automatically reads this file and injects `DB_PASSWORD` into the containers.

### 6.5: Check the firewall

Make sure ports 80 and 443 are open. Check both levels:

**On the server (UFW):**

```bash
sudo ufw status
```

If port 80 isn't listed, add it:

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

**On Azure Portal:**

Go to your VM → **Networking** → **Network settings** → Check that inbound rules allow ports 80 and 443. If not, see the [Azure tutorial](azure.md#step-10-open-httphttps-ports).

Now exit the server:

```bash
exit
```

---

## Step 7: Push and Deploy (~10 minutes)

Everything is in place. Time to push your changes and watch the magic happen!

### 7.1: Commit and push

From your project directory on your local machine:

```bash
git add -A
git commit -m "Add Docker and CI/CD pipeline for Azure deployment"
git push
```

### 7.2: Watch the pipeline

1. Go to your repository on GitHub
2. Click the **"Actions"** tab
3. You should see a workflow running called "CI/CD"

Click on it to see the live logs for each job:

```
build-and-test          ✅ (compile + tests)
       │
       ▼
build-and-push          ⏳ (building Docker image...)
       │
       ▼
deploy                  ⏳ (waiting...)
```

The full pipeline usually takes 3-5 minutes.

### 7.3: Verify it works

Once the pipeline shows all green checkmarks:

**Check the containers on your server:**

```bash
ssh azureuser@YOUR_SERVER_IP
docker ps
```

You should see two containers running:

```
CONTAINER ID   IMAGE                                              STATUS
abc123...      ghcr.io/yourname/your-project:latest               Up 30 seconds
def456...      mysql:8                                             Up 35 seconds
```

**Check in your browser:**

Open `http://YOUR_SERVER_IP` — you should see your application!

---

## Step 8: Test Automatic Deployment (~5 minutes)

The whole point of CI/CD is that changes deploy automatically. Let's verify:

1. Open your `src/main/resources/static/index.html` locally
2. Make a visible change (e.g., change a heading or add some text)
3. Commit and push:
   ```bash
   git add src/main/resources/static/index.html
   git commit -m "Update frontend heading"
   git push
   ```
4. Go to the **Actions** tab on GitHub — watch the pipeline run
5. After it completes (~3-5 minutes), refresh `http://YOUR_SERVER_IP`

You should see your change live on the server. No manual SSH, no manual deployment. Push to `main` = automatic deployment.

---

## How It All Fits Together

### The full cycle

Here's the complete flow from editing code to seeing your change live:

```
You edit code locally
        │
        ▼
docker compose run --rm build mvn test     ← verify it compiles and tests pass
        │
        ▼
docker compose up --build                   ← run the full stack locally
        │                                     (app + database)
        ▼
Open http://localhost:8080                  ← check it works
        │
        ▼
git add, git commit, git push               ← triggers the pipeline
        │
        ▼
┌──────────────────────────────────────────────┐
│          GitHub Actions Pipeline              │
│                                               │
│  1. Checkout code                             │
│  2. Build Java app with Maven                 │
│  3. Run tests                                 │
│  4. Build Docker image                        │
│  5. Push image to GHCR                        │
│  6. SSH into your Azure VM                    │
│  7. Pull the new image                        │
│  8. Restart all containers                    │
└──────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────┐
│            Your Azure VM                      │
│                                               │
│  app (Spring Boot, port 80)                   │
│    ├── serves HTML/CSS/JS                     │
│    ├── handles /api/ requests                 │
│    └── connects to ──▶ db (MySQL)             │
└──────────────────────────────────────────────┘
        │
        ▼
  Users visit http://YOUR_SERVER_IP             ← live in ~3-5 minutes
```

### Where secrets live

Passwords and keys are spread across three places — none of them end up in Git:

| Secret | Where it lives | Purpose |
|--------|---------------|---------|
| `DB_PASSWORD` | `.env` in your project root (git-ignored) | MySQL password for local development |
| `DB_PASSWORD` | `~/my-app/.env` on Azure VM | MySQL password for production |
| `SERVER_IP` | GitHub Secrets | Your VM's public IP address |
| `SERVER_USER` | GitHub Secrets | SSH username (`azureuser`) |
| `SSH_PRIVATE_KEY` | GitHub Secrets | Deploy key for automated SSH access |
| GitHub PAT | Docker login on Azure VM | Allows the server to pull images from GHCR |

---

## Troubleshooting

### Pipeline fails at "Build and test"

The Java build or tests failed. Click on the failed job to see Maven output.

**Common causes:**
- Compilation error in your code
- A test is failing
- Missing dependency in `pom.xml`
- Wrong Java version (workflow uses 21 — does your project use a different version?)

**Quick fix:** If you don't have tests yet, your build might still fail if Maven can't compile. Fix the compilation errors first.

### Pipeline fails at "Build and push image"

**Common causes:**
- `Dockerfile` syntax error — check that the file is in the right location (project root, next to `pom.xml`)
- GHCR permissions — the workflow needs `packages: write` permission (already set in the workflow file)
- Image name contains uppercase — use lowercase everywhere

### Pipeline fails at "Deploy to Azure VM"

The SSH connection failed. Check:

1. **SERVER_IP** — is the IP still correct? Azure may have changed it if you stopped/restarted the VM. Check Azure Portal → VM → Overview → Public IP
2. **SERVER_USER** — is it `azureuser`?
3. **SSH_PRIVATE_KEY** — did you copy the entire key including the BEGIN/END lines?
4. **VM is running** — check Azure Portal
5. **Public key on server** — is the deploy key's public key in `~/.ssh/authorized_keys` on the VM?

**Test SSH manually:**

```bash
ssh -i ~/.ssh/deploy_key azureuser@YOUR_SERVER_IP echo "Connection works"
```

### App shows a blank page or "Whitelabel Error Page"

Spring Boot can't find your static files.

1. Check that your HTML files are in `src/main/resources/static/` — not in a separate `frontend/` folder
2. Make sure there's an `index.html` in the `static/` directory
3. Rebuild: `mvn clean package` and check that the JAR contains your files: `jar tf target/*.jar | grep static`

### "Connection refused" in browser

1. Is port 80 open? Check both Azure Portal firewall AND `sudo ufw status` on the server
2. Is the app container running? `docker ps`
3. Are you using `http://` not `https://`?

### Backend can't connect to database

Check that the database name in `application.properties`, `docker-compose.prod.yml`, and the environment variables all match.

```bash
# On the server, check the running compose config
cd ~/my-app
docker compose -f docker-compose.prod.yml config
```

### App starts but crashes after a few seconds

The database might not be ready yet. MySQL can take 10-20 seconds to initialize on the first start. Check the app logs:

```bash
docker logs my-app-app-1
```

If you see connection errors, wait a moment and restart the app container:

```bash
docker compose -f docker-compose.prod.yml restart app
```

> **For a more robust setup**, you can add a health check to the db service and use `depends_on` with `condition: service_healthy`. But for a student project, a simple restart usually does the job.

### "Image not found" or "Manifest unknown"

1. Did the build-and-push job succeed in GitHub Actions?
2. Go to your GitHub profile → **Packages** — do you see your image?
3. Is the image name in `docker-compose.prod.yml` on the server **exactly** the same as in `ci.yml`?
4. Did you run `docker login ghcr.io` on the server? (Step 6.2)

### Images are private and server can't pull them

By default, GHCR packages may be private. To make them public:

1. Go to your GitHub profile → **Packages**
2. Click on the package (e.g., `your-project`)
3. **Package settings** → **Danger Zone** → **Change visibility** → **Public**

Alternatively, keep them private — as long as you ran `docker login ghcr.io` on the server, it can pull private packages.

### Database data is lost after redeploy

Data should survive redeployments — the `mysqldata` Docker volume persists across container restarts.

If data is disappearing, make sure the deploy script uses `docker compose up -d` (which only recreates containers with new images) — NOT `docker compose down` first (which can remove volumes).

### Port 80 is already in use on the server

Something else is using port 80. Check what:

```bash
sudo lsof -i :80
```

If it's Apache or another web server, stop it:

```bash
sudo systemctl stop apache2
sudo systemctl disable apache2
```

---

## What You've Learned

| Skill | What you did |
|-------|-------------|
| **Dockerfiles** | Created a multi-stage build for your Spring Boot application |
| **Spring Boot static files** | Served your HTML/CSS/JS frontend directly from Spring Boot |
| **Docker Compose** | Orchestrated a 2-service application (app + database) |
| **GitHub Actions** | Built a CI/CD pipeline from scratch |
| **GitHub Container Registry** | Pushed and pulled Docker images |
| **SSH key management** | Created and deployed a dedicated key pair |
| **Cloud deployment** | Deployed your own project on Azure |

---

## Appendix A: Azure VM Setup

If you don't have an Azure VM yet, here's a quick-start guide. For more detail, see the [full Azure tutorial](azure.md).

### A.1: Create an Azure account

**Students:** Go to [Azure for Students](https://azure.microsoft.com/en-us/free/students/) — you get $100 free credit with your school email, no credit card needed.

**Everyone else:** Go to [Azure Free Account](https://azure.microsoft.com/en-us/free/) — $200 free credit for 30 days.

### A.2: Create a Virtual Machine

1. Sign in to [Azure Portal](https://portal.azure.com)
2. Search for **"Virtual machines"** → Click **"+ Create"** → **"Azure virtual machine"**

Configure:

| Setting | Value |
|---------|-------|
| Resource group | Create new: `tek2-resources` |
| VM name | `tek2-server` |
| Region | Choose one where your student credit works ([how to check](azure.md#instance-details)) |
| Image | Ubuntu Server 22.04 LTS |
| Size | Standard_B1s (1 vCPU, 1 GB RAM — free tier eligible) |
| Authentication | SSH public key |
| Username | `azureuser` |
| SSH public key | Paste from `~/.ssh/id_ed25519.pub` |
| Inbound ports | SSH (22) |

3. Click **"Review + create"** → **"Create"**
4. Wait 1-3 minutes for deployment

### A.3: Get your IP address

Go to your VM in Azure Portal → **Overview** → copy the **Public IP address**.

### A.4: Connect via SSH

```bash
ssh azureuser@YOUR_PUBLIC_IP
```

Type `yes` when asked about the fingerprint.

### A.5: Install Docker

Run these commands on your server:

```bash
# Update packages
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install -y docker.io docker-compose-v2

# Add your user to the docker group (so you don't need sudo)
sudo usermod -aG docker $USER

# Log out and back in for the group change to take effect
exit
```

SSH back in:

```bash
ssh azureuser@YOUR_PUBLIC_IP
```

Verify:

```bash
docker --version
docker run hello-world
```

### A.6: Open ports 80 and 443

**On the server:**

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

Type `y` when asked.

**On Azure Portal:**

1. Go to your VM → **Networking** → **Network settings**
2. Click **"Create port rule"** → **"Inbound port rule"**
3. Add port 80 (name: `AllowHTTP`)
4. Repeat for port 443 (name: `AllowHTTPS`)

You're ready! Go back to [Step 1](#step-1-dockerize-your-application-15-minutes) to continue with the tutorial.

---

## Optional Next Steps

Want to go further? Try these:

1. **Add a custom domain** — Buy a domain and point it to your server's IP
2. **Add HTTPS** — Use Let's Encrypt with certbot for free SSL certificates
3. **Add environment-specific config** — Use different `application.properties` for dev and prod
4. **Add health checks** — Add Spring Boot Actuator and configure Docker health checks in the compose file
