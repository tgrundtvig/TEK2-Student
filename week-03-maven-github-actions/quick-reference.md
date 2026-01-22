# Quick Reference: Maven + GitHub Actions

**Print this page for easy reference during class**

---

## Maven Commands (via Docker)

Run Maven without installing it locally:

```bash
docker run --rm -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn <command>
```

| Command | What it does |
|---------|--------------|
| `mvn compile` | Compile source code |
| `mvn test` | Compile + run tests |
| `mvn package` | Compile + test + create JAR |
| `mvn clean` | Delete target/ directory |
| `mvn clean package` | Clean start + full build |
| `mvn clean package -DskipTests` | Build without running tests |

---

## Maven Project Structure

```
my-project/
├── pom.xml                    # Project configuration
├── src/
│   ├── main/
│   │   └── java/              # Application code
│   │       └── com/example/
│   │           └── App.java
│   └── test/
│       └── java/              # Test code
│           └── com/example/
│               └── AppTest.java
└── target/                    # Build output (generated)
    └── my-project-1.0.jar
```

---

## Minimal pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>my-project</artifactId>
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
```

---

## GitHub Actions Workflow

File location: `.github/workflows/ci.yml`

```yaml
name: Java CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Build and test
      run: mvn clean package
```

---

## GitHub Actions Key Concepts

| Term | Meaning |
|------|---------|
| **Workflow** | Automated process (YAML file in `.github/workflows/`) |
| **Trigger** | What starts workflow (`push`, `pull_request`, `schedule`) |
| **Job** | Group of steps running on same machine |
| **Step** | Single task (command or action) |
| **Runner** | Virtual machine executing the workflow |
| **Action** | Reusable component (`actions/checkout@v4`) |

---

## Common Triggers

```yaml
on:
  push:
    branches: [ main ]          # On push to main
  pull_request:
    branches: [ main ]          # On PR targeting main
  schedule:
    - cron: '0 0 * * *'         # Daily at midnight
  workflow_dispatch:             # Manual trigger button
```

---

## JUnit Test Template

```java
package com.example;

import org.junit.Test;
import static org.junit.Assert.*;

public class AppTest {

    @Test
    public void testSomething() {
        assertEquals(expected, actual);
        assertTrue(condition);
        assertFalse(condition);
        assertNotNull(object);
    }
}
```

---

## Git Commands

```bash
git init                           # Initialize repo
git add .                          # Stage all files
git commit -m "message"            # Commit
git push                           # Push to remote
git remote add origin git@...      # Add remote
git branch -M main                 # Rename branch to main
```

---

## Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| "Maven not found" | Use Docker: `docker run ... maven:3.9-eclipse-temurin-17 mvn ...` |
| Permission denied (Linux) | `sudo chown -R $USER:$USER .` |
| Workflow not running | Check file path: `.github/workflows/ci.yml` |
| SSH permission denied | Run `ssh -T git@github.com` to verify SSH setup |
| Tests fail in CI only | Check Java version (should be 17) |

---

## CI/CD Pipeline Flow

```
Push Code → GitHub detects workflow → Runner starts
                                          │
                    ┌─────────────────────▼───────────────────┐
                    │  1. Checkout code                        │
                    │  2. Setup Java                           │
                    │  3. mvn clean package                    │
                    │  4. Tests pass? ✓ or ✗                   │
                    └─────────────────────────────────────────┘
                                          │
                              ┌───────────▼───────────┐
                              │  Green ✓ or Red ✗     │
                              │  shown on GitHub      │
                              └───────────────────────┘
```

---

## Remember

- **CI** = Continuous Integration (automatic build/test on every push)
- **pom.xml** = Maven's configuration file
- **mvn clean package** = The most common Maven command
- `.github/workflows/` = Where GitHub Actions workflows live
- Workflows run on **GitHub's servers**, not your computer
