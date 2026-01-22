# Post-class Advanced Tasks

**Estimated time: 1-2 hours**

These tasks will deepen your understanding of Docker and Linux. Try to complete them before next week's class.

If you get stuck, check the [hints.md](hints.md) file - but try on your own first!

---

## Important: Volume Mounts Explained

Some of these tasks use **volume mounts** (the `-v` flag). We'll cover this properly in Week 2, but here's a quick preview:

```bash
docker run -v "/path/on/your/computer:/path/in/container" IMAGE
```

This connects a folder on your computer to a folder inside the container:
- Changes you make on your computer appear inside the container
- Files created in the container (in that folder) appear on your computer

**Example:**
```bash
docker run -v "$(pwd)":/app python:3.11 python /app/script.py
```

This mounts your current directory to `/app` in the container.

### Windows Users - Important!

The `$(pwd)` syntax only works on Mac/Linux. On Windows, use:

| Shell | Syntax |
|-------|--------|
| PowerShell | `${PWD}` |
| CMD | `%cd%` |
| Git Bash | `$(pwd)` (works!) |

**Example for PowerShell:**
```powershell
docker run -v "${PWD}:/app" -w /app python:3.11 python script.py
```

---

## Task 1: Docker Hub Exploration

### Objective
Explore Docker Hub and run three images you haven't used before.

### Instructions

1. Go to [Docker Hub](https://hub.docker.com) and browse the "Explore" section

2. Find and run **three different images** that interest you. Some suggestions:
   - `httpd:2.4` - Apache web server
   - `mongo:7` - MongoDB database
   - `mariadb:11` - MySQL-compatible database
   - `rabbitmq:3` - Message broker
   - `postgres:16` - PostgreSQL database
   - `grafana/grafana:latest` - Monitoring dashboards

3. For each image, document:
   - Image name and tag you used
   - The `docker run` command you used
   - What the container does
   - Any interesting observations

### Example Documentation

```
Image: httpd:2.4
Command: docker run -d -p 8080:80 httpd:2.4
Purpose: Apache HTTP Server - serves web pages
Observations: Similar to nginx but with different configuration approach.
              Default page says "It works!"
              Configuration files are in /usr/local/apache2/conf/
```

### Verify Success
For each image:
- [ ] Container starts without errors (`docker ps` shows it running)
- [ ] You understand what the container does
- [ ] You documented your findings

---

## Task 2: Python Script in a Container

### Objective
Run a Python script inside a Docker container without installing Python on your computer.

### Instructions

1. Create a file called `analysis.py` on your computer with this content:

```python
#!/usr/bin/env python3
"""Simple data analysis script"""

import sys
import statistics

def analyze_numbers(numbers):
    """Analyze a list of numbers and return statistics."""
    return {
        'count': len(numbers),
        'sum': sum(numbers),
        'mean': statistics.mean(numbers),
        'median': statistics.median(numbers),
        'min': min(numbers),
        'max': max(numbers),
        'stdev': statistics.stdev(numbers) if len(numbers) > 1 else 0
    }

def main():
    # Sample data
    data = [23, 45, 67, 89, 12, 34, 56, 78, 90, 11, 33, 55, 77, 99, 21]

    print("=" * 50)
    print("Python Data Analysis in Docker")
    print("=" * 50)
    print(f"\nData: {data}")
    print(f"\nPython version: {sys.version}")

    stats = analyze_numbers(data)
    print("\nStatistics:")
    for key, value in stats.items():
        if isinstance(value, float):
            print(f"  {key}: {value:.2f}")
        else:
            print(f"  {key}: {value}")

if __name__ == "__main__":
    main()
```

2. Run the script using Docker:

**Mac/Linux/Git Bash:**
```bash
docker run --rm -v "$(pwd)":/app -w /app python:3.11 python analysis.py
```

**Windows PowerShell:**
```powershell
docker run --rm -v "${PWD}:/app" -w /app python:3.11 python analysis.py
```

**Windows CMD:**
```cmd
docker run --rm -v "%cd%":/app -w /app python:3.11 python analysis.py
```

**Breaking down the command:**
- `--rm` - Remove container when done (clean up automatically)
- `-v "$(pwd)":/app` - Mount current directory to /app in container
- `-w /app` - Set working directory to /app inside container
- `python:3.11` - Use Python 3.11 image
- `python analysis.py` - Run our script

3. Modify the script to analyze different data and run it again.

### Challenge Extension
- Try running the same script with Python 3.9 and Python 3.12
- Are there any differences in output?

### Verify Success
- [ ] Script runs and shows statistics output
- [ ] You see "Python version: 3.11.x" in the output
- [ ] You understand what each flag in the command does

---

## Task 3: Serve Your Own HTML Page

### Objective
Create a custom HTML page and serve it with nginx.

### Instructions

1. Create a project directory:
```bash
mkdir my-website
cd my-website
```

2. Create an `index.html` file with this content (or create your own!):

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Docker Website</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        h1 { color: #0066cc; }
        .container {
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
        }
        .skill-list {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-top: 20px;
        }
        .skill {
            background: #0066cc;
            color: white;
            padding: 5px 15px;
            border-radius: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Week 1: Docker + Linux</h1>
        <p>This page is being served from an nginx container!</p>

        <h2>What I Learned</h2>
        <ul>
            <li>How to run Docker containers</li>
            <li>The difference between images and containers</li>
            <li>Basic Linux commands</li>
            <li>How to mount files into containers</li>
        </ul>

        <h2>Skills Acquired</h2>
        <div class="skill-list">
            <span class="skill">docker run</span>
            <span class="skill">docker ps</span>
            <span class="skill">docker exec</span>
            <span class="skill">Linux CLI</span>
            <span class="skill">nginx</span>
        </div>

        <p style="margin-top: 30px; color: #666;">
            Created by: [Your Name]<br>
            Date: [Today's Date]
        </p>
    </div>
</body>
</html>
```

3. Run nginx with your custom page:

**Mac/Linux/Git Bash:**
```bash
docker run -d -p 8080:80 -v "$(pwd)":/usr/share/nginx/html:ro --name my-web nginx:1.25
```

**Windows PowerShell:**
```powershell
docker run -d -p 8080:80 -v "${PWD}:/usr/share/nginx/html:ro" --name my-web nginx:1.25
```

**Breaking down the new parts:**
- `-v "$(pwd)":/usr/share/nginx/html:ro` - Mount current directory to nginx's web root
- `:ro` - Read-only (nginx shouldn't modify our files)

4. Open http://localhost:8080 in your browser

5. **While the container is running**, modify your HTML file and refresh the browser. What happens?

6. Customize the page with your own content!

7. Clean up when done:
```bash
docker stop my-web
docker rm my-web
```

### Challenge Extension
- Add more pages (about.html, contact.html)
- Add links between pages
- Add images to your site

### Verify Success
- [ ] You can see your custom page at http://localhost:8080
- [ ] Changes to the HTML file appear immediately when you refresh
- [ ] You understand why changes appear immediately (volume mount!)

---

## Task 4: Container Investigation

### Objective
Learn to inspect and understand running containers.

### Instructions

1. Start a MySQL container:
```bash
docker run -d --name mysql-inspect -e MYSQL_ROOT_PASSWORD=secret mysql:8.0
```

2. Wait for it to start (check with `docker logs mysql-inspect` for "ready for connections")

3. Inspect the container:
```bash
docker inspect mysql-inspect
```

This returns a lot of JSON! Answer these questions by finding the information:

**Questions to answer:**
1. What is the container's IP address? (Look in NetworkSettings.IPAddress)
2. What environment variables are set? (Look in Config.Env)
3. What ports are exposed? (Look in Config.ExposedPorts)
4. What is the container's hostname?
5. When was the container created?

**Tip:** You can use this to extract specific fields:
```bash
docker inspect mysql-inspect --format '{{.NetworkSettings.IPAddress}}'
```

4. Use `docker exec` to explore inside:
```bash
docker exec -it mysql-inspect bash
```

Inside, find:
- Where are MySQL's data files stored? (hint: `ls /var/lib/mysql`)
- What's in the MySQL configuration? (hint: `cat /etc/mysql/my.cnf`)

5. Check resource usage:
```bash
docker stats mysql-inspect --no-stream
```

6. Clean up:
```bash
docker stop mysql-inspect
docker rm mysql-inspect
```

### Verify Success
- [ ] You found all 5 pieces of information from `docker inspect`
- [ ] You explored inside the container with `docker exec`
- [ ] You saw resource usage with `docker stats`

---

## Task 5: Document Your Learning

### Objective
Create a personal reference document.

### Instructions

Create a Markdown file called `docker-cheatsheet.md` with:

1. **Commands you've learned** - organized by category
2. **Common mistakes** you made and how you fixed them
3. **Tips** you discovered
4. **Questions** you still have

### Template to start with:

```markdown
# My Docker Cheatsheet

## Container Lifecycle
| Command | Description | Example |
|---------|-------------|---------|
| docker run | Create and start NEW container | docker run -it ubuntu:22.04 bash |
| docker exec | Run command in EXISTING container | docker exec -it NAME bash |
| ... | ... | ... |

## Useful Flags
| Flag | Meaning | When to use |
|------|---------|-------------|
| -d | Detached/background | Services like nginx, mysql |
| -it | Interactive terminal | When you need a shell |
| ... | ... | ... |

## Linux Commands
| Command | Description |
|---------|-------------|
| pwd | Print working directory |
| ... | ... |

## Mistakes I Made
1. Forgot -it flag and container exited immediately
2. ...

## Tips I Learned
1. Use Tab to autocomplete commands
2. ...

## Questions for Next Week
1. How do I keep data when a container is removed?
2. ...
```

### Verify Success
- [ ] Cheatsheet has at least 10 Docker commands
- [ ] Cheatsheet has at least 10 Linux commands
- [ ] You documented at least 2 mistakes and how to fix them

---

## Submission

No formal submission required, but come to next week's class prepared to:
- Share one interesting thing you discovered
- Ask questions about anything that confused you
- Show your custom website if you completed Task 3

---

## Going Further (Optional)

If you finish early and want more challenges:

1. **Try Docker Desktop GUI**: If you've been using the command line, explore Docker Desktop's graphical interface. It can show you running containers, images, and resource usage visually.

2. **Read about Dockerfiles**: Preview what's coming in Week 2 by reading [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

3. **Explore Alpine Linux**: Run `docker run -it alpine:3.19 sh` and explore how different it is from Ubuntu. Can you install packages? (hint: `apk add`)

4. **Container logs**: Run a container that produces output (like mysql) and explore:
   ```bash
   docker logs mysql-inspect           # All logs
   docker logs --follow mysql-inspect  # Follow live (Ctrl+C to stop)
   docker logs --tail 50 mysql-inspect # Last 50 lines
   ```

5. **Check disk usage**: See how much space Docker is using:
   ```bash
   docker system df
   ```
