# Post-class Advanced Tasks

**Estimated time: 1-2 hours**

These tasks will deepen your understanding of CI/CD and prepare you for Week 4's cloud deployment. Try to complete them before next week's class.

If you get stuck, check the [hints.md](hints.md) file - but try on your own first!

---

## Task 1: Add Code Quality Checks (~25 minutes)

### Objective

Add automated code quality checks to your CI pipeline.

### Background

Professional CI pipelines don't just compile and test - they also check code quality:
- Code formatting consistency
- Static analysis (find potential bugs)
- Code coverage (how much code is tested)

### Instructions

1. Update your `pom.xml` to add quality plugins. Add inside the `<build><plugins>` section:

```xml
<!-- Checkstyle - Code style checker -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.3.1</version>
    <configuration>
        <configLocation>google_checks.xml</configLocation>
        <consoleOutput>true</consoleOutput>
        <failsOnError>false</failsOnError>
    </configuration>
</plugin>

<!-- JaCoCo - Code coverage -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.11</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

2. Update your workflow to run quality checks. Edit `.github/workflows/ci.yml`:

```yaml
name: Java CI with Quality Checks

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
        cache: maven

    - name: Build and test
      run: mvn clean package

    - name: Run Checkstyle
      run: mvn checkstyle:check
      continue-on-error: true

    - name: Generate coverage report
      run: mvn jacoco:report

    - name: Upload coverage report
      uses: actions/upload-artifact@v4
      with:
        name: coverage-report
        path: target/site/jacoco/
```

3. Push and check the Actions tab for the quality reports.

### Verify Success

- [ ] Checkstyle runs (even if there are warnings)
- [ ] JaCoCo coverage report is generated
- [ ] Coverage report is available as a downloadable artifact

### Challenge Extension

- Make the build fail if code coverage is below 50%
- Add SpotBugs for additional static analysis
- Create a custom checkstyle configuration

---

## Task 2: Pull Request Workflow (~20 minutes)

### Objective

Create a workflow that runs only on pull requests and adds helpful comments.

### Background

Good CI provides feedback directly on pull requests. This helps code reviewers see if tests pass before reviewing code.

### Instructions

1. Create a new workflow file `.github/workflows/pr-check.yml`:

```yaml
name: PR Check

on:
  pull_request:
    branches: [ main ]

jobs:
  check:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Build and test
      run: mvn clean package

    - name: Check for test failures
      if: failure()
      run: |
        echo "## ❌ Tests Failed" >> $GITHUB_STEP_SUMMARY
        echo "Please check the test output above." >> $GITHUB_STEP_SUMMARY

    - name: Report success
      if: success()
      run: |
        echo "## ✅ All Tests Passed" >> $GITHUB_STEP_SUMMARY
        echo "Ready for review!" >> $GITHUB_STEP_SUMMARY
```

2. Test it by creating a pull request:

```bash
# Create a new branch
git checkout -b test-pr-workflow

# Make a change
echo "// test change" >> src/main/java/com/example/App.java

# Commit and push
git add .
git commit -m "Test PR workflow"
git push -u origin test-pr-workflow
```

3. Go to GitHub and create a pull request from `test-pr-workflow` to `main`.

4. Watch the PR checks run and see the summary.

### Verify Success

- [ ] PR workflow triggers on pull request creation
- [ ] Job summary appears in the workflow run
- [ ] PR shows check status (green checkmark or red X)

### Challenge Extension

- Add a comment on the PR with test results
- Block merging if tests fail (branch protection rules)
- Add different checks for different file types

---

## Task 3: Environment-Specific Deployments (~30 minutes)

### Objective

Create workflows that deploy to different environments (staging, production).

### Background

Real projects have multiple environments:
- **Development** - for developers to test
- **Staging** - for QA and pre-release testing
- **Production** - for real users

Each might have different Docker image tags or deployment steps.

### Instructions

1. Update your workflow to tag images by environment:

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - main
      - develop

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
        cache: maven

    - name: Build and test
      run: mvn clean package

    - name: Determine environment
      id: env
      run: |
        if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
          echo "environment=production" >> $GITHUB_OUTPUT
          echo "tag=latest" >> $GITHUB_OUTPUT
        else
          echo "environment=staging" >> $GITHUB_OUTPUT
          echo "tag=staging" >> $GITHUB_OUTPUT
        fi

    - name: Log in to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push Docker image
      run: |
        docker build -t ghcr.io/${{ github.repository_owner }}/hello-maven:${{ steps.env.outputs.tag }} .
        docker build -t ghcr.io/${{ github.repository_owner }}/hello-maven:${{ github.sha }} .
        docker push ghcr.io/${{ github.repository_owner }}/hello-maven:${{ steps.env.outputs.tag }}
        docker push ghcr.io/${{ github.repository_owner }}/hello-maven:${{ github.sha }}

    - name: Summary
      run: |
        echo "## Deployment Summary" >> $GITHUB_STEP_SUMMARY
        echo "- **Environment:** ${{ steps.env.outputs.environment }}" >> $GITHUB_STEP_SUMMARY
        echo "- **Image tag:** ${{ steps.env.outputs.tag }}" >> $GITHUB_STEP_SUMMARY
        echo "- **Commit:** ${{ github.sha }}" >> $GITHUB_STEP_SUMMARY
```

> **Note:** Add `permissions: packages: write` at the workflow or job level for GHCR access.

2. Test by pushing to different branches:

```bash
# Create develop branch
git checkout -b develop
git push -u origin develop

# Make a change and push
echo "// staging change" >> src/main/java/com/example/App.java
git add . && git commit -m "Test staging deploy"
git push
```

3. Check GitHub Packages for different tags.

### Verify Success

- [ ] Push to `main` creates `latest` tag
- [ ] Push to `develop` creates `staging` tag
- [ ] Both create a SHA-tagged image
- [ ] Job summary shows environment info

---

## Task 4: Scheduled Workflows (~15 minutes)

### Objective

Create a workflow that runs on a schedule (like a daily build).

### Background

Scheduled builds help catch issues even when no one is pushing code:
- Dependencies might get security updates
- External services might change
- "Bit rot" - code that breaks over time

### Instructions

1. Create `.github/workflows/nightly.yml`:

```yaml
name: Nightly Build

on:
  schedule:
    # Run at 2:00 AM UTC every day
    - cron: '0 2 * * *'
  # Also allow manual trigger
  workflow_dispatch:

jobs:
  nightly:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Update dependencies
      run: mvn versions:display-dependency-updates

    - name: Build and test
      run: mvn clean package

    - name: Check for outdated dependencies
      run: |
        echo "## Dependency Check" >> $GITHUB_STEP_SUMMARY
        mvn versions:display-dependency-updates | grep -E "^\[INFO\].*->" >> $GITHUB_STEP_SUMMARY || echo "All dependencies up to date!" >> $GITHUB_STEP_SUMMARY
```

2. Test it manually:
   - Go to Actions tab
   - Click "Nightly Build" workflow
   - Click "Run workflow" button

### Cron Syntax

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-6, Sunday=0)
│ │ │ │ │
* * * * *
```

Examples:
- `0 2 * * *` - 2:00 AM every day
- `0 0 * * 0` - Midnight every Sunday
- `*/15 * * * *` - Every 15 minutes

### Verify Success

- [ ] Workflow can be triggered manually
- [ ] Dependency updates are reported
- [ ] Build completes successfully

---

## Task 5: Optimize Docker Build in CI (~25 minutes)

### Objective

Speed up Docker builds in CI using layer caching and buildx.

### Background

Building Docker images in CI can be slow because:
- No layer cache between runs
- Images are built from scratch every time

Docker buildx with GitHub Actions cache can dramatically speed this up.

### Instructions

1. Update your workflow to use Docker Buildx with caching:

```yaml
name: Optimized Docker Build

on:
  push:
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
        cache: maven

    - name: Build with Maven
      run: mvn clean package -DskipTests

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Log in to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push with cache
      uses: docker/build-push-action@v5
      with:
        context: .
        push: true
        tags: |
          ghcr.io/${{ github.repository_owner }}/hello-maven:latest
          ghcr.io/${{ github.repository_owner }}/hello-maven:${{ github.sha }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

> **Note:** Add `permissions: packages: write` at the workflow or job level for GHCR access.

2. Push twice and compare build times:

```bash
# First push - will be slower (populates cache)
git add . && git commit -m "Add buildx caching"
git push

# Make a small code change
echo "// cache test" >> src/main/java/com/example/App.java
git add . && git commit -m "Test cache"
git push
```

3. Compare the "Build and push with cache" step times between runs.

### Verify Success

- [ ] First build completes (populates cache)
- [ ] Second build is faster (uses cache)
- [ ] You see "importing cache manifest" in the second run
- [ ] Images are pushed to GitHub Container Registry

### Challenge Extension

- Add multi-platform builds (linux/amd64, linux/arm64)
- Use registry cache instead of GitHub Actions cache
- Build only when Dockerfile or source changes

---

## Task 6: Create a Release Workflow (~20 minutes)

### Objective

Automatically create GitHub releases with Docker images.

### Background

When you're ready to release a version, you want to:
- Tag the code with a version number
- Build and push a Docker image with that version
- Create a GitHub release with release notes

### Instructions

1. Create `.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'  # Trigger on version tags like v1.0.0

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
        cache: maven

    - name: Build and test
      run: mvn clean package

    - name: Get version from tag
      id: version
      run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

    - name: Log in to GitHub Container Registry
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push release image
      run: |
        docker build -t ghcr.io/${{ github.repository_owner }}/hello-maven:${{ steps.version.outputs.version }} .
        docker build -t ghcr.io/${{ github.repository_owner }}/hello-maven:latest .
        docker push ghcr.io/${{ github.repository_owner }}/hello-maven:${{ steps.version.outputs.version }}
        docker push ghcr.io/${{ github.repository_owner }}/hello-maven:latest

    - name: Create GitHub Release
      uses: softprops/action-gh-release@v1
      with:
        name: Release ${{ steps.version.outputs.version }}
        body: |
          ## Docker Image
          ```
          docker pull ghcr.io/${{ github.repository_owner }}/hello-maven:${{ steps.version.outputs.version }}
          ```

          ## Changes
          See commit history for details.
        draft: false
        prerelease: false
```

> **Note:** Add `permissions: packages: write` at the workflow or job level for GHCR access.

2. Create a release:

```bash
# Commit any pending changes
git add .
git commit -m "Prepare for release"
git push

# Create and push a version tag
git tag v1.0.0
git push origin v1.0.0
```

3. Check the Actions tab and the Releases page on GitHub.

### Verify Success

- [ ] Tagging triggers the release workflow
- [ ] Docker image is tagged with version number
- [ ] GitHub release is created automatically
- [ ] Release includes Docker pull instructions

---

## Going Further (Optional)

If you finish early and want more challenges:

### 1. Monorepo Support

Build only changed components:
```yaml
on:
  push:
    paths:
      - 'service-a/**'
jobs:
  build-service-a:
    # ...
```

### 2. Slack Notifications

```yaml
- name: Notify Slack
  uses: slackapi/slack-github-action@v1
  with:
    channel-id: 'builds'
    slack-message: "Build ${{ job.status }}"
  env:
    SLACK_BOT_TOKEN: ${{ secrets.SLACK_TOKEN }}
```

### 3. Security Scanning

```yaml
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myimage:latest'
    format: 'sarif'
    output: 'trivy-results.sarif'
```

### 4. Performance Testing

```yaml
- name: Run JMeter tests
  run: |
    wget https://downloads.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz
    tar -xzf apache-jmeter-5.6.3.tgz
    ./apache-jmeter-5.6.3/bin/jmeter -n -t tests/load-test.jmx
```

---

## Before Next Class

Come prepared to:

- Share your most interesting workflow customization
- Ask questions about anything that confused you
- Discuss: How would you deploy your Docker image to a server?

These CI/CD skills will be essential in **Week 4** when we deploy to the cloud!
