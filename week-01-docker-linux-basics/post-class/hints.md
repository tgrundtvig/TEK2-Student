# Hints for Advanced Tasks

Use these hints if you get stuck. Try to solve problems on your own first!

---

## Windows Users: Path Syntax Quick Reference

Throughout these hints, you'll see `$(pwd)` which only works on Mac/Linux/Git Bash.

| Your Shell | Use This Instead |
|------------|------------------|
| PowerShell | `${PWD}` |
| CMD | `%cd%` |
| Git Bash | `$(pwd)` (works!) |

**How to tell which shell you're using:**
- PowerShell prompt looks like: `PS C:\Users\Name>`
- CMD prompt looks like: `C:\Users\Name>`
- Git Bash prompt looks like: `Name@COMPUTER MINGW64 ~`

---

## Task 1: Docker Hub Exploration

<details>
<summary>Hint 1: Finding the right run command</summary>

Most images on Docker Hub have a "How to use this image" section. Look for it on the image's page.

For example, the `mongo` page shows:
```bash
docker run -d mongo
```
</details>

<details>
<summary>Hint 2: Images that need configuration</summary>

Some images require environment variables. Look for required variables in the documentation.

Common patterns:
- Databases often need `_ROOT_PASSWORD` or `_PASSWORD`
- Web apps might need `_PORT` or `_HOST`

Example for MongoDB with authentication:
```bash
docker run -d -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=secret mongo
```
</details>

<details>
<summary>Hint 3: Exploring inside the container</summary>

After starting a container, explore it:
```bash
docker exec -it <container_name> sh
# or
docker exec -it <container_name> bash
```

Some minimal images only have `sh`, not `bash`.
</details>

---

## Task 2: Python Script in a Container

<details>
<summary>Hint 1: Volume mount not working</summary>

On Windows with PowerShell, use:
```powershell
docker run --rm -v ${PWD}:/app -w /app python:3.11 python analysis.py
```

On Windows with CMD, use:
```cmd
docker run --rm -v %cd%:/app -w /app python:3.11 python analysis.py
```

On Mac/Linux:
```bash
docker run --rm -v "$(pwd)":/app -w /app python:3.11 python analysis.py
```
</details>

<details>
<summary>Hint 2: File not found error</summary>

Make sure:
1. You're in the same directory as your `analysis.py` file
2. The filename matches exactly (Linux is case-sensitive)
3. The file was saved (not just open in editor)

Check with:
```bash
ls -la analysis.py
```
</details>

<details>
<summary>Hint 3: Trying different Python versions</summary>

Just change the tag:
```bash
docker run --rm -v "$(pwd)":/app -w /app python:3.9 python analysis.py
docker run --rm -v "$(pwd)":/app -w /app python:3.12 python analysis.py
```

You can see all available tags on the Python Docker Hub page.
</details>

---

## Task 3: Serve Your Own HTML Page

<details>
<summary>Hint 1: nginx shows "403 Forbidden"</summary>

This usually means:
1. The directory doesn't have an `index.html` file
2. File permissions don't allow reading

Check your file exists:
```bash
ls -la index.html
```

Make sure the filename is exactly `index.html` (lowercase).
</details>

<details>
<summary>Hint 2: Volume mount path issues</summary>

The path before the colon is your computer, after is the container:
```
-v "$(pwd)":/usr/share/nginx/html
     │                    │
     │                    └── Container path (nginx's web root)
     └── Your computer's current directory
```

If your files are in a subdirectory:
```bash
docker run -d -p 8080:80 -v "$(pwd)/my-website":/usr/share/nginx/html nginx
```
</details>

<details>
<summary>Hint 3: Changes not showing in browser</summary>

Try:
1. Hard refresh: Ctrl+Shift+R (Windows/Linux) or Cmd+Shift+R (Mac)
2. Clear browser cache
3. Open in incognito/private window
4. Check if the container is still running: `docker ps`
</details>

<details>
<summary>Hint 4: Adding more pages</summary>

Create additional HTML files in the same directory:
```
my-website/
├── index.html
├── about.html
└── contact.html
```

Link between them in HTML:
```html
<a href="about.html">About</a>
<a href="contact.html">Contact</a>
```

Access via:
- http://localhost:8080/
- http://localhost:8080/about.html
- http://localhost:8080/contact.html
</details>

---

## Task 4: Container Investigation

<details>
<summary>Hint 1: Finding IP address in inspect output</summary>

The output is JSON. Look for:
```json
"NetworkSettings": {
    "IPAddress": "172.17.0.x",
    ...
}
```

Or use a filter:
```bash
docker inspect mysql-inspect --format '{{.NetworkSettings.IPAddress}}'
```
</details>

<details>
<summary>Hint 2: Filtering inspect output</summary>

Use `--format` to extract specific fields:

```bash
# IP Address
docker inspect mysql-inspect --format '{{.NetworkSettings.IPAddress}}'

# Environment variables
docker inspect mysql-inspect --format '{{.Config.Env}}'

# Exposed ports
docker inspect mysql-inspect --format '{{.Config.ExposedPorts}}'

# Hostname
docker inspect mysql-inspect --format '{{.Config.Hostname}}'

# Created time
docker inspect mysql-inspect --format '{{.Created}}'
```
</details>

<details>
<summary>Hint 3: Finding MySQL data directory</summary>

Inside the container:
```bash
docker exec -it mysql-inspect bash
ls -la /var/lib/mysql/
```

You'll see database directories and MySQL system files.
</details>

<details>
<summary>Hint 4: docker stats shows nothing</summary>

Make sure the container is running:
```bash
docker ps
```

If it exited, check logs to see why:
```bash
docker logs mysql-inspect
```
</details>

---

## General Troubleshooting

<details>
<summary>Container exits immediately</summary>

Some containers exit when they finish their task. To keep them running:

1. Run interactively: `docker run -it image bash`
2. Run with a long-running command: `docker run -d image sleep 3600`
3. Check if the image is designed to run a service (nginx, mysql) or a one-shot command (hello-world)
</details>

<details>
<summary>Port already in use</summary>

Error: `Bind for 0.0.0.0:8080 failed: port is already allocated`

Solution: Use a different port:
```bash
docker run -d -p 8081:80 nginx  # Use 8081 instead
```

Or find and stop what's using the port:
```bash
# See what's running
docker ps

# Stop the conflicting container
docker stop <container_id>
```
</details>

<details>
<summary>Permission denied on Linux</summary>

If you get "permission denied" when running docker:
```bash
sudo usermod -aG docker $USER
```
Then log out and log back in.

Alternatively, prefix commands with `sudo`:
```bash
sudo docker run hello-world
```
</details>

<details>
<summary>"No such file or directory" inside container</summary>

Remember:
1. Container filesystem is separate from your computer
2. Files only appear in container if you mount them with `-v`
3. Paths in container might be different from your computer
</details>

---

## Still Stuck?

1. **Read the error message carefully** - Docker's errors usually explain the problem
2. **Check Docker Hub documentation** - Most images have usage examples
3. **Google the error** - Someone else has probably had the same issue
4. **Ask in class** - Bring your questions to the next session!
