# Hints for Advanced Tasks

Use these hints if you get stuck. Try to solve problems on your own first!

---

## Task 1: Code Quality Checks

<details>
<summary>Hint 1: Checkstyle plugin not found</summary>

Make sure you added the plugin inside `<build><plugins>`:

```xml
<project>
  ...
  <build>
    <plugins>
      <!-- Your existing plugins -->

      <!-- Add checkstyle here -->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-checkstyle-plugin</artifactId>
        ...
      </plugin>
    </plugins>
  </build>
</project>
```

If you don't have a `<build>` section yet, create one.

</details>

<details>
<summary>Hint 2: JaCoCo not generating report</summary>

JaCoCo needs to run during the test phase. Make sure:

1. You have the `prepare-agent` execution (instruments the code)
2. You have the `report` execution (generates HTML report)
3. You run `mvn test` or `mvn package` (not just `mvn compile`)

Check if the report exists:
```bash
ls target/site/jacoco/
```

You should see `index.html` and other files.

</details>

<details>
<summary>Hint 3: Checkstyle has many violations</summary>

Google's checkstyle rules are strict. For learning, you can:

1. Set `failsOnError` to `false` to allow the build to continue:
   ```xml
   <configuration>
     <failsOnError>false</failsOnError>
   </configuration>
   ```

2. Use `continue-on-error: true` in the workflow step

3. Create a custom checkstyle.xml with relaxed rules

The point is to see the feedback, not necessarily pass all checks initially.

</details>

<details>
<summary>Hint 4: Can't download artifact</summary>

The artifact might take a moment to be available. Steps to access it:

1. Go to the workflow run page
2. Scroll down to "Artifacts" section
3. Click on "coverage-report" to download
4. Extract the ZIP and open `index.html`

Artifacts are deleted after 90 days by default.

</details>

---

## Task 2: Pull Request Workflow

<details>
<summary>Hint 1: Workflow doesn't trigger on PR</summary>

Check that your workflow file:

1. Is in `.github/workflows/` (exact path!)
2. Has the correct trigger:
   ```yaml
   on:
     pull_request:
       branches: [ main ]
   ```
3. The PR targets `main` branch

Also verify the YAML syntax - use a YAML validator if unsure.

</details>

<details>
<summary>Hint 2: GITHUB_STEP_SUMMARY not appearing</summary>

Make sure you're using the correct syntax:
```yaml
run: echo "## Title" >> $GITHUB_STEP_SUMMARY
```

Key points:
- Use `>>` (append), not `>` (overwrite)
- It's `$GITHUB_STEP_SUMMARY` (with underscore)
- Use markdown formatting for rich content

The summary appears on the workflow run page under the job.

</details>

<details>
<summary>Hint 3: Creating a PR from command line</summary>

If you have the GitHub CLI (`gh`) installed:
```bash
gh pr create --title "Test PR" --body "Testing PR workflow"
```

Otherwise, go to GitHub:
1. Navigate to your repository
2. Click "Pull requests" → "New pull request"
3. Select your feature branch as the source
4. Select `main` as the target
5. Click "Create pull request"

</details>

<details>
<summary>Hint 4: Understanding if conditions</summary>

```yaml
- name: On failure
  if: failure()      # Runs only if previous steps failed

- name: On success
  if: success()      # Runs only if all previous steps succeeded

- name: Always run
  if: always()       # Runs regardless of previous step status

- name: Conditional
  if: github.event_name == 'push'  # Custom condition
```

These conditions help create different outputs for different scenarios.

</details>

---

## Task 3: Environment-Specific Deployments

<details>
<summary>Hint 1: Output variable not working</summary>

The new syntax for setting outputs in GitHub Actions:
```yaml
- id: mystep
  run: echo "myvar=myvalue" >> $GITHUB_OUTPUT

- run: echo "${{ steps.mystep.outputs.myvar }}"
```

Common mistakes:
- Missing `id` on the step
- Using old `::set-output` syntax (deprecated)
- Typo in the output name

</details>

<details>
<summary>Hint 2: Branch comparison in workflow</summary>

To check which branch triggered the workflow:
```yaml
if: github.ref == 'refs/heads/main'
```

Note: It's `refs/heads/main`, not just `main`.

For pull requests, use:
```yaml
if: github.base_ref == 'main'
```

</details>

<details>
<summary>Hint 3: Creating the develop branch</summary>

```bash
# From main branch
git checkout main
git pull

# Create and switch to develop
git checkout -b develop

# Push the new branch
git push -u origin develop
```

Now you have two branches that trigger different tags.

</details>

<details>
<summary>Hint 4: Multiple tags for same image</summary>

You can tag and push multiple times:
```bash
docker build -t user/image:latest .
docker tag user/image:latest user/image:v1.0.0
docker tag user/image:latest user/image:abc123
docker push user/image --all-tags
```

Or with the build command directly:
```yaml
docker build -t user/image:tag1 -t user/image:tag2 .
```

</details>

---

## Task 4: Scheduled Workflows

<details>
<summary>Hint 1: Cron syntax help</summary>

```
┌───────────── minute (0-59)
│ ┌───────────── hour (0-23, UTC!)
│ │ ┌───────────── day of month (1-31)
│ │ │ ┌───────────── month (1-12)
│ │ │ │ ┌───────────── day of week (0-6, Sunday=0)
│ │ │ │ │
* * * * *
```

Examples:
- `0 2 * * *` - 2:00 AM UTC daily
- `30 8 * * 1-5` - 8:30 AM UTC, Monday-Friday
- `0 0 1 * *` - Midnight UTC, first of each month
- `*/30 * * * *` - Every 30 minutes

Use [crontab.guru](https://crontab.guru/) to help create expressions.

</details>

<details>
<summary>Hint 2: Scheduled workflow not running</summary>

Scheduled workflows:
1. Only run on the **default branch** (usually `main`)
2. May be delayed during high GitHub load
3. Are disabled after 60 days of repo inactivity

To test, use `workflow_dispatch` for manual triggering:
```yaml
on:
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch:  # Allows manual trigger
```

Then click "Run workflow" in the Actions tab.

</details>

<details>
<summary>Hint 3: Dependency updates command</summary>

Maven has built-in plugins for checking updates:

```bash
# Check for dependency updates
mvn versions:display-dependency-updates

# Check for plugin updates
mvn versions:display-plugin-updates

# Check for property updates
mvn versions:display-property-updates
```

These commands show which dependencies have newer versions available.

</details>

<details>
<summary>Hint 4: Timezone considerations</summary>

GitHub Actions cron uses **UTC** timezone, not your local time.

To run at 9 AM Copenhagen time (CET/CEST):
- Winter (CET, UTC+1): `0 8 * * *` (8 AM UTC = 9 AM CET)
- Summer (CEST, UTC+2): `0 7 * * *` (7 AM UTC = 9 AM CEST)

For simplicity, many teams just use a UTC time and accept the variation.

</details>

---

## Task 5: Optimize Docker Build

<details>
<summary>Hint 1: Buildx action fails</summary>

Make sure you have the setup step BEFORE the build step:
```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Build and push
  uses: docker/build-push-action@v5
  ...
```

The order matters! Buildx must be set up first.

</details>

<details>
<summary>Hint 2: Cache not working</summary>

GitHub Actions cache (type=gha) requirements:
1. Must be in a GitHub Actions environment
2. Cache is scoped to the branch (PRs share with their base branch)
3. Cache has a 10GB limit per repository

If cache isn't restoring, check:
```yaml
cache-from: type=gha
cache-to: type=gha,mode=max
```

The `mode=max` exports all layers, not just the final ones.

</details>

<details>
<summary>Hint 3: Understanding buildx vs regular build</summary>

Regular Docker build:
```bash
docker build -t myimage .
docker push myimage
```

Docker Buildx (used by build-push-action):
- Builds and pushes in one step
- Supports multi-platform builds
- Better caching options
- Can build without loading to local daemon

That's why we use the action instead of raw commands.

</details>

<details>
<summary>Hint 4: First build is slow</summary>

The first build after adding caching will still be slow because:
1. No cache exists yet
2. All layers must be built
3. Cache must be uploaded

The second build should be significantly faster as it:
1. Downloads the existing cache
2. Reuses unchanged layers
3. Only rebuilds changed layers

Look for "importing cache manifest" in the logs.

</details>

---

## Task 6: Release Workflow

<details>
<summary>Hint 1: Tag not triggering workflow</summary>

Check your trigger configuration:
```yaml
on:
  push:
    tags:
      - 'v*'
```

Creating and pushing tags:
```bash
# Create tag locally
git tag v1.0.0

# Push the tag to GitHub
git push origin v1.0.0

# Or push all tags
git push --tags
```

The tag must be pushed to GitHub to trigger the workflow.

</details>

<details>
<summary>Hint 2: Extracting version from tag</summary>

The tag is in `GITHUB_REF` as `refs/tags/v1.0.0`.

To extract just the version number:
```bash
# Remove the refs/tags/v prefix
version=${GITHUB_REF#refs/tags/v}
# Result: 1.0.0
```

Use it in a step:
```yaml
- id: version
  run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

- run: echo "Version is ${{ steps.version.outputs.version }}"
```

</details>

<details>
<summary>Hint 3: Release action permissions</summary>

The `softprops/action-gh-release` action needs write permissions.

If you get permission errors, add to your workflow:
```yaml
permissions:
  contents: write
```

Or check your repository settings → Actions → General → Workflow permissions.

</details>

<details>
<summary>Hint 4: Deleting and recreating tags</summary>

If you need to redo a release:

```bash
# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin :refs/tags/v1.0.0

# Recreate tag
git tag v1.0.0
git push origin v1.0.0
```

Note: This will also delete the GitHub release associated with that tag.

</details>

---

## General Troubleshooting

<details>
<summary>Workflow syntax errors</summary>

YAML is sensitive to indentation. Common issues:

```yaml
# Wrong - inconsistent indentation
steps:
  - name: Step 1
    run: echo "hello"
   - name: Step 2    # Wrong! One space off
    run: echo "world"

# Correct
steps:
  - name: Step 1
    run: echo "hello"
  - name: Step 2
    run: echo "world"
```

Use a YAML validator or the GitHub workflow editor which shows errors.

</details>

<details>
<summary>Secrets not available</summary>

Check that:
1. Secret name matches exactly (case-sensitive)
2. Secret is defined in repository settings
3. Workflow is in the same repository as secrets
4. For forks: secrets aren't available by default

Reference secrets with:
```yaml
${{ secrets.MY_SECRET }}
```

Never print secrets in logs!

</details>

<details>
<summary>GHCR authentication issues</summary>

If you have trouble pushing to GitHub Container Registry:

1. Check your workflow has `permissions: packages: write`
2. Verify repository Settings → Actions → Workflow permissions is "Read and write"
3. Ensure image names are lowercase (GHCR requires lowercase)
4. Check `registry: ghcr.io` is specified in login action

**Note:** If using Docker Hub instead, you'll need to create secrets for `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`.

</details>

<details>
<summary>Workflow runs out of time</summary>

By default, jobs have a 6-hour timeout. You can set a lower limit:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 30    # Fail after 30 minutes
```

This prevents runaway jobs from consuming all your Actions minutes.

</details>

<details>
<summary>Understanding workflow run context</summary>

Useful context variables:
```yaml
${{ github.sha }}           # Full commit SHA
${{ github.ref }}           # refs/heads/main or refs/tags/v1.0.0
${{ github.ref_name }}      # main or v1.0.0 (short form)
${{ github.event_name }}    # push, pull_request, schedule, etc.
${{ github.actor }}         # Username who triggered
${{ github.repository }}    # owner/repo
${{ runner.os }}            # Linux, Windows, or macOS
```

Use these to make workflows dynamic.

</details>

---

## Still Stuck?

1. **Check the Actions tab** - Error messages are usually descriptive

2. **Look at the raw logs** - Click on a failed step to see details

3. **Test locally when possible:**
   ```bash
   # Test Maven commands
   docker run --rm -v "$(pwd)":/app -w /app maven:3.9-eclipse-temurin-17 mvn clean package

   # Test Docker builds
   docker build -t test .
   ```

4. **Search the error** - GitHub Actions documentation and Stack Overflow

5. **Simplify** - Remove parts of the workflow until it works, then add back

6. **Ask in class** - Bring your workflow file and error message
