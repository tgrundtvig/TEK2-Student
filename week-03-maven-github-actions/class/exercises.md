# Class Exercises: Maven + GitHub Actions

Work through these exercises during class. Ask for help if you get stuck!

---

## Warm-up: Verify Pre-class Work (~10 minutes)

Quick check that everyone's CI pipeline is working.

### Check your GitHub repository

1. Go to your `hello-maven-ci` repository on GitHub
2. Click the **Actions** tab
3. You should see at least one successful workflow run (green checkmark)

### If you don't have the setup

Quick setup for those who need to catch up:

```bash
mkdir -p ~/week3-maven-ci/hello-maven && cd ~/week3-maven-ci/hello-maven

# Create pom.xml
cat > pom.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>hello-maven</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
EOF

# Create App.java
mkdir -p src/main/java/com/example
cat > src/main/java/com/example/App.java << 'EOF'
package com.example;
public class App {
    public static void main(String[] args) {
        System.out.println("Hello from Maven!");
    }
    public static int add(int a, int b) { return a + b; }
}
EOF

# Create AppTest.java
mkdir -p src/test/java/com/example
cat > src/test/java/com/example/AppTest.java << 'EOF'
package com.example;
import org.junit.Test;
import static org.junit.Assert.*;
public class AppTest {
    @Test
    public void testAdd() { assertEquals(5, App.add(2, 3)); }
}
EOF

# Create workflow
mkdir -p .github/workflows
cat > .github/workflows/ci.yml << 'EOF'
name: Java CI
on:
  push:
    branches: [ main ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
    - run: mvn clean package
EOF

# Create .gitignore
echo -e "target/\n.idea/\n*.iml" > .gitignore

# Initialize and push
git init
git add .
git commit -m "Initial commit"
# Create repo on GitHub first, then:
# git remote add origin git@github.com:YOUR_USERNAME/hello-maven-ci.git
# git branch -M main
# git push -u origin main
```

### Self-check
- [ ] Repository exists on GitHub with code
- [ ] At least one successful workflow run in Actions tab
- [ ] You can build locally with `docker run --rm -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn clean package`

---

## Exercise 1: Add Maven Dependency Caching (~15 minutes)

### Goal

Speed up CI builds by caching Maven dependencies.

### Why Caching Matters

Every time your workflow runs, Maven downloads all dependencies from the internet. For a real project with dozens of dependencies, this can take minutes.

GitHub Actions can cache these downloads between runs!

### Part A: Check current build time (~2 minutes)

1. Go to your latest workflow run in GitHub Actions
2. Click on the "build" job
3. Expand "Build with Maven"
4. Note how long it took: _______________

You'll see output like:
```
Downloading from central: https://repo.maven.apache.org/maven2/...
```

### Part B: Add caching to the workflow (~5 minutes)

Edit `.github/workflows/ci.yml`:

```yaml
name: Java CI with Maven

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven    # <-- Add this line!

    - name: Build with Maven
      run: mvn clean package

    - name: Run tests
      run: mvn test
```

The `cache: maven` line tells the setup-java action to cache the `~/.m2/repository` directory.

### Part C: Push and observe (~5 minutes)

```bash
git add .
git commit -m "Add Maven dependency caching"
git push
```

Watch the workflow run. The first run after adding caching still needs to download everything (to populate the cache).

### Part D: Trigger another build (~3 minutes)

Make a small change (edit a comment or add a blank line) and push again:

```bash
echo "// Updated" >> src/main/java/com/example/App.java
git add .
git commit -m "Test caching"
git push
```

Now check the workflow:
1. Look for "Cache restored successfully" in the Java setup step
2. The Maven build should be faster - no more "Downloading from central" messages

**New build time:** _______________

### Self-check
- [ ] Added `cache: maven` to workflow
- [ ] Saw "Cache restored" in second build
- [ ] Build time improved (or at least no downloads)

---

## Exercise 2: Build Docker Image in CI (~25 minutes)

### Goal

Automatically build a Docker image when code is pushed.

### Part A: Create a Dockerfile (~5 minutes)

In your `hello-maven` project, create a `Dockerfile`:

```dockerfile
# Stage 1: Build with Maven
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Create runtime image
FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

This is a **multi-stage build**:
1. First stage: Uses Maven to build the JAR
2. Second stage: Uses a smaller JRE image to run it

### Part B: Test the Docker build locally (~3 minutes)

```bash
docker build -t hello-maven:local .
docker run --rm hello-maven:local
```

You should see "Hello from Maven!" output.

### Part C: Update the workflow to build Docker (~8 minutes)

Edit `.github/workflows/ci.yml`:

```yaml
name: Java CI with Maven

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Build with Maven
      run: mvn clean package

    - name: Run tests
      run: mvn test

    - name: Build Docker image
      run: docker build -t hello-maven:${{ github.sha }} .

    - name: List Docker images
      run: docker images | grep hello-maven
```

The `${{ github.sha }}` is a GitHub Actions variable - it's the commit SHA, giving each build a unique tag.

### Part D: Push and verify (~5 minutes)

```bash
git add .
git commit -m "Add Dockerfile and Docker build to CI"
git push
```

Watch the Actions tab. You should see:
1. Maven build succeeds
2. Docker build succeeds
3. `docker images` shows your tagged image

### Part E: Understand what happened (~4 minutes)

The workflow now:
1. Checks out code
2. Sets up Java with caching
3. Builds and tests with Maven
4. Builds a Docker image

**Question:** The Docker image is built but not saved anywhere after the workflow finishes. How could we persist it?

**Answer:** Push it to a registry like Docker Hub or GitHub Container Registry. We'll do this next!

### Self-check
- [ ] Dockerfile created with multi-stage build
- [ ] Local Docker build works
- [ ] CI workflow builds Docker image successfully
- [ ] You understand multi-stage builds

---

## Exercise 3: Push Docker Image to Docker Hub (~20 minutes)

### Goal

Automatically push Docker images to Docker Hub when tests pass.

### Part A: Create Docker Hub access token (~3 minutes)

1. Go to [hub.docker.com](https://hub.docker.com) and sign in
2. Click your username → **Account Settings**
3. Click **Security** → **New Access Token**
4. Description: "GitHub Actions"
5. Access permissions: "Read, Write, Delete"
6. Click **Generate**
7. **Copy the token** - you won't see it again!

### Part B: Add secrets to GitHub (~3 minutes)

1. Go to your `hello-maven-ci` repository on GitHub
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add two secrets:
   - Name: `DOCKERHUB_USERNAME`, Value: your Docker Hub username
   - Name: `DOCKERHUB_TOKEN`, Value: the token you just created

### Part C: Update the workflow (~8 minutes)

Edit `.github/workflows/ci.yml`:

```yaml
name: Java CI with Maven

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Build with Maven
      run: mvn clean package

    - name: Run tests
      run: mvn test

    - name: Log in to Docker Hub
      if: github.event_name == 'push' && github.ref == 'refs/heads/main'
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build and push Docker image
      if: github.event_name == 'push' && github.ref == 'refs/heads/main'
      run: |
        docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/hello-maven:latest .
        docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/hello-maven:${{ github.sha }} .
        docker push ${{ secrets.DOCKERHUB_USERNAME }}/hello-maven:latest
        docker push ${{ secrets.DOCKERHUB_USERNAME }}/hello-maven:${{ github.sha }}
```

**Key additions:**
- `if: github.event_name == 'push' && github.ref == 'refs/heads/main'` - Only push images on push to main (not on PRs)
- `docker/login-action@v3` - Official action to log in to Docker Hub
- `${{ secrets.DOCKERHUB_USERNAME }}` - Reference to your secret
- Two tags: `latest` and the commit SHA

### Part D: Push and verify (~6 minutes)

```bash
git add .
git commit -m "Add Docker Hub push to CI"
git push
```

Watch the workflow. After it completes:

1. Go to [hub.docker.com](https://hub.docker.com)
2. Find your `hello-maven` repository
3. Check the Tags tab - you should see `latest` and a SHA tag

### Part E: Test pulling your image

Anyone can now run your application:

```bash
docker pull YOUR_USERNAME/hello-maven:latest
docker run --rm YOUR_USERNAME/hello-maven:latest
```

Replace `YOUR_USERNAME` with your Docker Hub username.

### Self-check
- [ ] Docker Hub access token created
- [ ] GitHub secrets configured
- [ ] Workflow pushes image on successful build
- [ ] Image visible on Docker Hub
- [ ] You understand the `if:` condition for conditional steps

---

## Exercise 4: Add Build Status Badge (~10 minutes)

### Goal

Add a badge to your README showing the build status.

### Part A: Get the badge markdown (~2 minutes)

1. Go to your repository's Actions tab
2. Click on your workflow name ("Java CI with Maven")
3. Click the **...** menu (three dots) on the right
4. Click **Create status badge**
5. Copy the Markdown

It looks like:
```markdown
![Java CI with Maven](https://github.com/YOUR_USERNAME/hello-maven-ci/actions/workflows/ci.yml/badge.svg)
```

### Part B: Create/Update README.md (~3 minutes)

Create or update `README.md`:

```markdown
# Hello Maven CI

![Java CI with Maven](https://github.com/YOUR_USERNAME/hello-maven-ci/actions/workflows/ci.yml/badge.svg)

A simple Java project demonstrating CI/CD with Maven and GitHub Actions.

## Features

- Maven build with JUnit tests
- Automatic builds on every push
- Docker image automatically built and pushed to Docker Hub

## Run Locally

```bash
docker run --rm YOUR_USERNAME/hello-maven:latest
```

## Build Locally

```bash
docker run --rm -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn clean package
```
```

### Part C: Push and verify (~3 minutes)

```bash
git add .
git commit -m "Add build status badge to README"
git push
```

Go to your repository on GitHub. You should see the badge showing the build status!

### Part D: Break the build (optional) (~2 minutes)

To see the badge change:

1. Edit `AppTest.java` to add a failing test
2. Push the change
3. Watch the workflow fail
4. See the badge turn red
5. Fix the test and push again

### Self-check
- [ ] Badge added to README
- [ ] Badge shows current build status
- [ ] README renders nicely on GitHub

---

## Exercise 5: Test Matrix - Multiple Java Versions (Bonus) (~15 minutes)

> **Note:** This is a bonus exercise. Complete it if you have time.

### Goal

Test your code against multiple Java versions automatically.

### Part A: Update the workflow

Edit `.github/workflows/ci.yml` to use a matrix strategy:

```yaml
name: Java CI with Maven

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java-version: ['17', '21']

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK ${{ matrix.java-version }}
      uses: actions/setup-java@v4
      with:
        java-version: ${{ matrix.java-version }}
        distribution: 'temurin'
        cache: maven

    - name: Build and test with Maven
      run: mvn clean package

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Build with Maven
      run: mvn clean package -DskipTests

    - name: Log in to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build and push Docker image
      run: |
        docker build -t ${{ secrets.DOCKERHUB_USERNAME }}/hello-maven:latest .
        docker push ${{ secrets.DOCKERHUB_USERNAME }}/hello-maven:latest
```

**Key concepts:**
- `strategy.matrix` - Run the job multiple times with different values
- `${{ matrix.java-version }}` - Reference the current matrix value
- `needs: test` - The build-and-push job waits for all test jobs to pass
- Two separate jobs: `test` (runs on matrix) and `build-and-push` (runs once)

### Part B: Push and observe

```bash
git add .
git commit -m "Add Java version matrix testing"
git push
```

In the Actions tab, you'll see the test job run twice - once for Java 17 and once for Java 21!

### Self-check
- [ ] Workflow runs tests on multiple Java versions
- [ ] Docker image only builds after all tests pass
- [ ] You understand matrix strategies

---

## Final Cleanup

Keep your repository - you'll use it in Week 4!

To clean up local Docker images:

```bash
docker rmi hello-maven:local 2>/dev/null
docker image prune
```

---

## Summary

Today you practiced:

| Skill | What you learned |
|-------|------------------|
| **Dependency caching** | `cache: maven` speeds up builds |
| **Multi-stage Docker builds** | Smaller images, build tools not in final image |
| **CI secrets** | Secure storage for credentials |
| **Conditional steps** | `if:` to control when steps run |
| **Docker Hub integration** | Automatic image publishing |
| **Status badges** | Visual build status in README |
| **Matrix testing** | Test against multiple versions |

**The Complete CI/CD Pipeline:**

```
Push code
    │
    ▼
┌─────────────────────────────────┐
│  GitHub Actions                 │
│                                 │
│  1. Checkout code               │
│  2. Setup Java (cached deps)    │
│  3. mvn clean package           │
│  4. mvn test                    │
│  5. docker build                │
│  6. docker push (if main)       │
└─────────────────────────────────┘
    │
    ▼
Docker Hub
(Image available for deployment)
```

**Key takeaways:**

1. **Caching saves time** - Don't download dependencies every build
2. **Multi-stage builds** - Keep final images small
3. **Secrets are secure** - Never commit credentials to code
4. **Conditional logic** - Only deploy from main branch
5. **Badges communicate status** - Visual feedback for the team
6. **Matrix testing** - Ensure compatibility across versions

---

## What's Next?

In **Week 4**, you'll:
- Create a cloud server
- Deploy your Docker images
- Complete the full DevOps pipeline: Code → Build → Test → Deploy
