# Pre-class Exercises: Maven and GitHub Actions

**Estimated time: 30-60 minutes**

Complete these exercises before class. By the end, you'll have a Java project that automatically builds and tests itself on GitHub every time you push code.

> **Tip:** If something isn't working, don't spend more than 10 minutes on it. Note the error message and move on - we'll troubleshoot together in class.

---

## Exercise 1: Verify Prerequisites (~5 minutes)

Before we start, let's make sure you have everything from Weeks 1-2.

### Step 1.1: Check Docker is running

```bash
docker --version
docker ps
```

If you see an error about connecting to the Docker daemon, start Docker Desktop (Windows/Mac) or the Docker service (Linux).

### Step 1.2: Check Git and GitHub SSH

```bash
git --version
ssh -T git@github.com
```

You should see: `Hi username! You've successfully authenticated...`

If not, revisit Week 1 Exercise 6 to set up your SSH keys.

### Step 1.3: Create a working directory

```bash
mkdir -p ~/week3-maven-ci
cd ~/week3-maven-ci
```

### Self-check
- [ ] `docker --version` shows a version number
- [ ] `docker ps` runs without error
- [ ] `ssh -T git@github.com` shows your GitHub username
- [ ] You're in the `week3-maven-ci` directory

---

## Exercise 2: Build a Java Project with Maven (~15 minutes)

### Goal

Build a Java project using Maven running inside Docker - no Java or Maven installation needed on your computer!

### Step 2.1: Create the project structure

```bash
mkdir -p hello-maven/src/main/java/com/example
mkdir -p hello-maven/src/test/java/com/example
cd hello-maven
```

Your structure should look like:
```
hello-maven/
├── src/
│   ├── main/
│   │   └── java/
│   │       └── com/
│   │           └── example/
│   └── test/
│       └── java/
│           └── com/
│               └── example/
```

### Step 2.2: Create the pom.xml file

Create a file called `pom.xml` in the `hello-maven` directory with this content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>hello-maven</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.example.App</mainClass>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### Step 2.3: Create the Java application

Create `src/main/java/com/example/App.java`:

```java
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Hello from Maven!");
        System.out.println("2 + 3 = " + add(2, 3));
    }

    public static int add(int a, int b) {
        return a + b;
    }

    public static int multiply(int a, int b) {
        return a * b;
    }
}
```

### Step 2.4: Build with Maven using Docker

Now the magic - build without installing Maven:

```bash
docker run --rm -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn clean package -DskipTests
```

> **Windows PowerShell users:** Replace `$(pwd)` with `${PWD}`

**Expected output:**
```
[INFO] Scanning for projects...
[INFO]
[INFO] -----------------------< com.example:hello-maven >-----------------------
[INFO] Building hello-maven 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
...
[INFO] Building jar: /app/target/hello-maven-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

### Step 2.5: Verify the build

```bash
ls -la target/
```

You should see `hello-maven-1.0-SNAPSHOT.jar`.

### Step 2.6: Run the application

```bash
docker run --rm -v "$(pwd)":/app -w /app eclipse-temurin:17 java -jar target/hello-maven-1.0-SNAPSHOT.jar
```

**Expected output:**
```
Hello from Maven!
2 + 3 = 5
```

### What just happened?

1. Docker downloaded the `maven:3.9-eclipse-temurin-17` image (Maven + Java 17)
2. Your project files were mounted into the container via `-v`
3. Maven compiled your Java code
4. Maven packaged it into a JAR file
5. We ran the JAR using a Java runtime container

**You built a Java application without installing Java or Maven!**

### Self-check
- [ ] `mvn clean package` completed with "BUILD SUCCESS"
- [ ] `target/hello-maven-1.0-SNAPSHOT.jar` exists
- [ ] Running the JAR prints "Hello from Maven!"
- [ ] You understand the Docker command structure

---

## Exercise 3: Add Tests (~10 minutes)

### Goal

Add JUnit tests and see Maven run them automatically.

### Step 3.1: Create a test file

Create `src/test/java/com/example/AppTest.java`:

```java
package com.example;

import org.junit.Test;
import static org.junit.Assert.*;

public class AppTest {

    @Test
    public void testAdd() {
        assertEquals(5, App.add(2, 3));
    }

    @Test
    public void testAddNegative() {
        assertEquals(-1, App.add(2, -3));
    }

    @Test
    public void testMultiply() {
        assertEquals(6, App.multiply(2, 3));
    }
}
```

### Step 3.2: Run the tests

```bash
docker run --rm -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn test
```

**Expected output:**
```
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.example.AppTest
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.05 s
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 3, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

### Step 3.3: See what a failing test looks like

Temporarily edit `AppTest.java` and add this test:

```java
@Test
public void testWillFail() {
    assertEquals(100, App.add(2, 3));  // This expects 100, but 2+3=5!
}
```

Run tests again:

```bash
docker run --rm -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn test
```

**Expected output:**
```
[INFO] Tests run: 4, Failures: 1, Errors: 0, Skipped: 0
...
[ERROR] testWillFail  Time elapsed: 0.003 s  <<< FAILURE!
java.lang.AssertionError: expected:<100> but was:<5>
...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
```

**Important:** The build FAILS when tests fail. This is exactly what CI will catch!

### Step 3.4: Remove the failing test

Delete the `testWillFail` method from `AppTest.java` so all tests pass again.

### Step 3.5: Full build with tests

```bash
docker run --rm -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn clean package
```

This time, tests run as part of `package` (we removed `-DskipTests`).

### Self-check
- [ ] You saw "Tests run: 3, Failures: 0"
- [ ] You saw what a failing test looks like
- [ ] You understand that Maven stops the build when tests fail
- [ ] All tests pass now

---

## Exercise 4: Push to GitHub (~10 minutes)

### Goal

Create a GitHub repository and push your Maven project.

### Step 4.1: Initialize Git

Make sure you're in the `hello-maven` directory:

```bash
pwd  # Should show .../hello-maven
git init
```

### Step 4.2: Create .gitignore

Create a file called `.gitignore`:

```
# Maven build output
target/

# IDE files
.idea/
*.iml
.vscode/
.project
.classpath
.settings/

# OS files
.DS_Store
Thumbs.db
```

### Step 4.3: Make your first commit

```bash
git add .
git commit -m "Initial Maven project with tests"
```

### Step 4.4: Create a GitHub repository

1. Go to [github.com/new](https://github.com/new)
2. Repository name: `hello-maven-ci`
3. Description: "My first Maven project with CI"
4. Keep it **Public** (free GitHub Actions minutes)
5. Do **NOT** check "Add a README file"
6. Click **Create repository**

### Step 4.5: Push to GitHub

GitHub will show instructions. Run these commands (replace `YOUR_USERNAME`):

```bash
git remote add origin git@github.com:YOUR_USERNAME/hello-maven-ci.git
git branch -M main
git push -u origin main
```

### Step 4.6: Verify on GitHub

Go to `https://github.com/YOUR_USERNAME/hello-maven-ci` and confirm you see:
- `pom.xml`
- `src/` folder
- `.gitignore`

### Self-check
- [ ] Repository created on GitHub
- [ ] Code pushed successfully
- [ ] You can see your files on GitHub

---

## Exercise 5: Add GitHub Actions CI (~10 minutes)

### Goal

Create a workflow that automatically builds and tests your code on every push.

### Step 5.1: Create the workflow directory

```bash
mkdir -p .github/workflows
```

### Step 5.2: Create the workflow file

Create `.github/workflows/ci.yml`:

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

    - name: Build with Maven
      run: mvn clean package

    - name: Run tests
      run: mvn test
```

### Step 5.3: Commit and push

```bash
git add .
git commit -m "Add GitHub Actions CI workflow"
git push
```

### Step 5.4: Watch the magic happen!

1. Go to your repository on GitHub
2. Click the **Actions** tab
3. You should see a workflow running (yellow circle)
4. Click on it to see the details
5. Wait for it to complete (green checkmark)

### Step 5.5: Explore the workflow run

Click on the workflow run to see:
- Each step and its output
- The "Build with Maven" step showing compilation
- The "Run tests" step showing test results

### Step 5.6: Trigger another build

Make a small change to verify the automation:

Edit `App.java` and change the greeting:

```java
System.out.println("Hello from Maven with CI!");
```

Commit and push:

```bash
git add .
git commit -m "Update greeting"
git push
```

Go to the Actions tab - a new workflow run should appear!

### What just happened?

1. You pushed code to GitHub
2. GitHub detected the workflow file in `.github/workflows/`
3. GitHub started a virtual machine (runner)
4. The runner executed your workflow steps
5. The result (pass/fail) is shown in the GitHub UI

**From now on, every push triggers an automatic build and test!**

### Self-check
- [ ] `.github/workflows/ci.yml` is created
- [ ] First workflow run completed successfully (green checkmark)
- [ ] Second push triggered a new workflow run
- [ ] You can see build logs in GitHub Actions
- [ ] You understand: push → workflow → build → test → result

---

## Summary

You've learned:

| Concept | Description |
|---------|-------------|
| **Maven** | Java build tool that manages dependencies and standardizes builds |
| **pom.xml** | Project configuration file (dependencies, build settings) |
| **mvn clean package** | Build command: clean + compile + test + package |
| **GitHub Actions** | CI/CD platform built into GitHub |
| **Workflow** | YAML file defining automated build/test process |

Commands you used:

| Command | Purpose |
|---------|---------|
| `docker run ... maven:3.9-eclipse-temurin-17 mvn clean package` | Build with Maven in Docker |
| `docker run ... maven:3.9-eclipse-temurin-17 mvn test` | Run tests |
| `git push` | Push code (triggers CI) |

---

## Troubleshooting

### "Maven command not found"

You're trying to run Maven locally. We use Docker:
```bash
docker run --rm -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn clean package
```

### "Unable to find image" (first time)

This is normal - Docker is downloading the Maven image. It can take a few minutes the first time.

### Build fails with "Source option 17 is not supported"

Make sure you're using the correct Docker image with Java 17:
```bash
docker run --rm -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn clean package
```

### Permission denied on Linux

Maven runs as root in the container, creating files you can't edit. Fix with:
```bash
sudo chown -R $USER:$USER .
```

Or run with your user ID:
```bash
docker run --rm -u $(id -u):$(id -g) -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn clean package
```

### GitHub Actions workflow not running

- Check the file is in `.github/workflows/` (exact path, including the dot!)
- Check YAML syntax - indentation matters (use spaces, not tabs)
- Check branch name: workflow triggers on `main`, not `master`

### "Permission denied (publickey)" when pushing

Your SSH key isn't set up correctly. Revisit Week 1 Exercise 6:
```bash
ssh -T git@github.com
```
Should show your username.

### Tests pass locally but fail in CI

- Check Java version matches (we use 17 everywhere)
- Check for OS-specific code (local might be Windows/Mac, CI runs Linux)

---

## Before Class Checklist

Before coming to class, ensure you can check ALL of these:

**Maven:**
- [ ] You built `hello-maven-1.0-SNAPSHOT.jar` using Docker
- [ ] You understand what `pom.xml` contains
- [ ] You saw tests run with `mvn test`
- [ ] You saw what a failing test looks like

**GitHub Actions:**
- [ ] You have a `hello-maven-ci` repository on GitHub
- [ ] The repository has a `.github/workflows/ci.yml` file
- [ ] You've seen at least one successful workflow run (green checkmark)
- [ ] You saw a second push trigger a new workflow

**Concepts:**
- [ ] You can explain what CI means
- [ ] You understand why automated testing matters
- [ ] You know that GitHub Actions runs on GitHub's servers, not your computer

If any of these aren't working, **note the exact error message** and we'll troubleshoot at the start of class.

---

**You're ready for class!** In class, we'll extend this to build Docker images and push them to Docker Hub automatically.
