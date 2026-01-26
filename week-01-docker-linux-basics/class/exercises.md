# Class Exercises: Docker + Linux Basics

Work through these exercises during class. Ask for help if you get stuck!

---

## Warm-up: Verify Setup (~5 minutes)

Quick check that everyone's pre-class setup is working.

### Docker
```bash
docker run --rm hello-world
```
You should see "Hello from Docker!"

### GitHub SSH
```bash
ssh -T git@github.com
```
You should see "Hi username! You've successfully authenticated..."

### Troubleshooting

**If Docker doesn't work:** Make sure Docker Desktop is running (Windows/Mac) or the Docker service is started (Linux).

**If GitHub SSH doesn't work:**
- Check that your key exists: `ls ~/.ssh/id_ed25519.pub`
- If not, generate one: `ssh-keygen -t ed25519 -C "your_email@example.com"`
- Add the key to GitHub: Settings → SSH and GPG keys → New SSH key

Once verified, move on to the exercises.

---

## Exercise 1: Linux Navigation and Files (~20 minutes)

### Goal
Practice Linux commands inside a Docker container.

### Setup

Start an Ubuntu container and keep it running:

```bash
docker run -it --name linux-practice ubuntu:22.04 bash
```

You should see a prompt like: `root@abc123:/#`

### Tasks

**Part A: Navigation** (~8 minutes)

1. Find out where you are:
   ```bash
   pwd
   ```
   What directory are you in? _______________

2. List what's in the current directory:
   ```bash
   ls
   ```

3. List with details (permissions, sizes, dates):
   ```bash
   ls -la
   ```

4. Navigate to the `/home` directory:
   ```bash
   cd /home
   pwd
   ```

5. Go back to root:
   ```bash
   cd /
   ```

6. Navigate to `/usr/bin` and list some commands:
   ```bash
   cd /usr/bin
   ls | head -20
   ```

**Part B: Files and Directories** (~10 minutes)

> **⏱️ If short on time:** Do steps 1-8, skip the rest.

1. Go to the `/tmp` directory (a good place for experiments):
   ```bash
   cd /tmp
   ```

2. Create a new directory:
   ```bash
   mkdir myproject
   ```

3. Navigate into it:
   ```bash
   cd myproject
   pwd
   ```

4. Create an empty file:
   ```bash
   touch readme.txt
   ```

5. Put some text in the file:
   ```bash
   echo "This is my first file in Linux" > readme.txt
   ```

6. Display the file contents:
   ```bash
   cat readme.txt
   ```

7. Create another file with multiple lines:
   ```bash
   echo "Line 1" > notes.txt
   echo "Line 2" >> notes.txt
   echo "Line 3" >> notes.txt
   ```
   Note: `>` overwrites, `>>` appends

8. Show the file:
   ```bash
   cat notes.txt
   ```

9. Copy a file:
   ```bash
   cp readme.txt readme_backup.txt
   ls
   ```

10. Rename a file:
    ```bash
    mv readme_backup.txt backup.txt
    ls
    ```

11. Create a subdirectory and move a file into it:
    ```bash
    mkdir backups
    mv backup.txt backups/
    ls backups/
    ```

12. Delete a file:
    ```bash
    rm notes.txt
    ls
    ```

**Part C: Quick Exploration** (~2 minutes)

1. Look at the system release information:
   ```bash
   cat /etc/os-release
   ```

2. Check disk space:
   ```bash
   df -h
   ```

### Self-check
- [ ] I can navigate using `cd` and verify with `pwd`
- [ ] I can create files and directories
- [ ] I understand the difference between `>` (overwrite) and `>>` (append)
- [ ] I can copy, move, and delete files

### Don't exit yet!
Keep this container running. We'll use it for comparison later.

---

## Exercise 2: Run a Database Container (~25 minutes)

### Goal
Run a MySQL database, create data, and understand container ephemerality.

### Setup

Open a **new terminal window** (keep your linux-practice container running).

### Tasks

**Part A: Start MySQL** (~5 minutes)

1. Run MySQL in background:
   ```bash
   docker run -d \
     --name mysql-class \
     -e MYSQL_ROOT_PASSWORD=student123 \
     -p 3306:3306 \
     mysql:8.0
   ```

2. Check it's running:
   ```bash
   docker ps
   ```

3. Wait for MySQL to initialize, then check logs:
   ```bash
   docker logs mysql-class
   ```

   **Look for this message:** `ready for connections`

   If you don't see it yet, wait 10-20 seconds and try again.

**Part B: Connect and Create Data** (~10 minutes)

1. Connect to MySQL inside the container:
   ```bash
   docker exec -it mysql-class mysql -uroot -pstudent123
   ```

2. You're now in the MySQL prompt. Try these commands:
   ```sql
   -- Show existing databases
   SHOW DATABASES;

   -- Create a new database
   CREATE DATABASE school;

   -- Verify it was created
   SHOW DATABASES;

   -- Exit MySQL
   exit
   ```

**Part C: Verify Data Persists (In Same Container)** (~3 minutes)

1. Reconnect to MySQL:
   ```bash
   docker exec -it mysql-class mysql -uroot -pstudent123
   ```

2. Check data is still there:
   ```sql
   SHOW DATABASES;
   ```

   You should see `school` in the list!

   ```sql
   exit
   ```

**Part D: Understand Ephemerality** (~7 minutes)

Now let's see what happens when we remove the container.

1. Stop and remove the container:
   ```bash
   docker stop mysql-class
   docker rm mysql-class
   ```

2. Verify it's gone:
   ```bash
   docker ps -a | grep mysql-class
   ```
   (Should show nothing)

3. Start a fresh MySQL container (same command as before):
   ```bash
   docker run -d \
     --name mysql-class \
     -e MYSQL_ROOT_PASSWORD=student123 \
     -p 3306:3306 \
     mysql:8.0
   ```

4. Wait for it to start (check with `docker logs mysql-class`), then check for our database:
   ```bash
   docker exec -it mysql-class mysql -uroot -pstudent123 -e "SHOW DATABASES;"
   ```

   **What happened to the `school` database?** _______________

### Key Insight

The database disappeared because:
- `docker rm` removes the container AND its data
- The new `docker run` creates a completely NEW container
- It's like demolishing a building and constructing a new one - nothing is preserved

**How do we solve this?** We'll learn about **volumes** in Week 2!

### Self-check
- [ ] I successfully ran MySQL in a container
- [ ] I created a database and verified it existed
- [ ] I understand why the data disappeared when I removed the container
- [ ] I understand: `docker run` = NEW container, `docker rm` = delete everything

---

## Exercise 3: Explore Different Images (~20 minutes)

> **⏱️ If short on time:** Just do Parts A and B, skip the rest.

### Goal
Practice running various Docker images and understand image differences.

### Tasks

**Part A: Compare Alpine and Ubuntu** (~8 minutes)

1. Check image sizes:
   ```bash
   docker images | grep -E "alpine|ubuntu"
   ```

   | Image | Size |
   |-------|------|
   | ubuntu:22.04 | _______ |
   | alpine:3.19 | _______ |

2. Run Alpine:
   ```bash
   docker run -it alpine:3.19 sh
   ```
   Note: Alpine uses `sh`, not `bash`

3. Try to use bash:
   ```bash
   bash
   ```
   What happened? _______________

4. Check the OS:
   ```bash
   cat /etc/os-release
   exit
   ```

**Part B: Run a Python Container** (~7 minutes)

1. Run Python interactively:
   ```bash
   docker run -it python:3.11 python
   ```

2. You're now in a Python REPL. Try:
   ```python
   print("Hello from Docker!")
   2 + 2
   import sys
   sys.version
   exit()
   ```

3. Run a one-liner (no need to enter the container):
   ```bash
   docker run --rm python:3.11 python -c 'print("Hello from Python in Docker!")'
   ```

   Note the `--rm` flag - it automatically removes the container when done.

**Part C: Run Redis** (~5 minutes - OPTIONAL)

1. Start Redis in background:
   ```bash
   docker run -d --name redis-test redis:7
   ```

2. Connect to Redis CLI:
   ```bash
   docker exec -it redis-test redis-cli
   ```

3. Try some commands:
   ```
   SET greeting "Hello, Redis!"
   GET greeting
   SET counter 1
   INCR counter
   INCR counter
   GET counter
   exit
   ```

4. Clean up:
   ```bash
   docker stop redis-test
   docker rm redis-test
   ```

**Part D: See Your Container History**

```bash
docker ps -a
```

How many containers have you created today? _______________

### Clean Up (Optional)

Remove all stopped containers:
```bash
docker container prune
```

### Self-check
- [ ] I understand that different images have different sizes and tools
- [ ] I ran Python inside a container without installing Python
- [ ] I know the `--rm` flag auto-removes containers
- [ ] I know how to clean up old containers

---

## Bonus Exercise: Create a Simple Web Page (If Time Permits)

### Goal
Serve a custom HTML page using nginx.

### Tasks

1. First, let's see the default nginx page structure. Start nginx:
   ```bash
   docker run -d --name web-bonus -p 8080:80 nginx:1.25
   ```

2. Look at where nginx serves files from:
   ```bash
   docker exec -it web-bonus ls /usr/share/nginx/html
   ```

3. Look at the default page:
   ```bash
   docker exec -it web-bonus cat /usr/share/nginx/html/index.html
   ```

4. Replace it with our own content:
   ```bash
   docker exec -it web-bonus sh -c 'echo "<h1>Hello from Week 1!</h1><p>I made this in Docker class.</p>" > /usr/share/nginx/html/index.html'
   ```

5. Visit http://localhost:8080 in your browser

6. What do you see? _______________

7. Clean up:
   ```bash
   docker stop web-bonus
   docker rm web-bonus
   ```

**Note:** This change is temporary - if we remove and recreate the container, the default page returns. In Week 2, we'll learn how to make permanent changes with Dockerfiles and volumes.

---

## Final Cleanup

Before leaving class, clean up your containers:

```bash
# Stop all running containers
docker stop $(docker ps -q) 2>/dev/null

# Remove stopped containers
docker container prune -f

# See what's left
docker ps -a
```

---

## Summary

Today you practiced:

| Skill | Commands Used |
|-------|---------------|
| Linux navigation | `pwd`, `cd`, `ls` |
| File operations | `cat`, `touch`, `mkdir`, `cp`, `mv`, `rm`, `echo` |
| Running containers | `docker run` with `-d`, `-it`, `-p`, `-e`, `--name`, `--rm` |
| Managing containers | `docker ps`, `docker stop`, `docker rm`, `docker logs` |
| Entering containers | `docker exec -it CONTAINER bash` |
| Exploring images | `docker images`, different base images |

**Key takeaways:**
1. Containers are isolated, disposable environments - experiment freely!
2. `docker run` = NEW container, `docker exec` = EXISTING container
3. Data inside containers disappears when the container is removed
4. Always use specific image tags (e.g., `mysql:8.0`, not `mysql:latest`)
