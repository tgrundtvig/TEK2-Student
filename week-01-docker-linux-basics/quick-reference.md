# Week 1 Quick Reference Card

Print this page or keep it open while working!

---

## Docker Commands

| Command | What it does | Example |
|---------|--------------|---------|
| `docker run IMAGE` | Create & start NEW container | `docker run ubuntu:22.04` |
| `docker run -it IMAGE bash` | Interactive shell in NEW container | `docker run -it ubuntu:22.04 bash` |
| `docker run -d IMAGE` | Run in background (detached) | `docker run -d nginx:1.25` |
| `docker run -p 8080:80 IMAGE` | Map port (your:container) | `docker run -p 8080:80 nginx:1.25` |
| `docker run --name NAME IMAGE` | Give container a name | `docker run --name web nginx:1.25` |
| `docker run --rm IMAGE` | Auto-remove when done | `docker run --rm alpine:3.19 echo hi` |
| `docker exec -it NAME bash` | Shell into EXISTING container | `docker exec -it web bash` |
| `docker ps` | List running containers | |
| `docker ps -a` | List ALL containers | |
| `docker stop NAME` | Stop a container | `docker stop web` |
| `docker rm NAME` | Remove a container | `docker rm web` |
| `docker logs NAME` | View container output | `docker logs web` |
| `docker images` | List downloaded images | |

### The Most Important Distinction!

```
docker run  = Create NEW container (like opening a new restaurant)
docker exec = Enter EXISTING container (like walking into an open restaurant)
```

---

## Linux Commands

| Command | What it does | Example |
|---------|--------------|---------|
| `pwd` | Where am I? | |
| `ls` | What's here? | |
| `ls -la` | Detailed list with hidden files | |
| `cd /path` | Change directory | `cd /home` |
| `cd ..` | Go up one level | |
| `cd ~` | Go to home directory | |
| `cat file` | Show file contents | `cat /etc/os-release` |
| `touch file` | Create empty file | `touch notes.txt` |
| `mkdir dir` | Create directory | `mkdir myproject` |
| `cp src dest` | Copy file | `cp file.txt backup.txt` |
| `mv src dest` | Move/rename | `mv old.txt new.txt` |
| `rm file` | Delete file | `rm notes.txt` |
| `rm -r dir` | Delete directory | `rm -r myproject` |
| `echo "text" > file` | Write to file (overwrite) | `echo "hello" > hi.txt` |
| `echo "text" >> file` | Append to file | `echo "more" >> hi.txt` |

---

## Keyboard Shortcuts

| Shortcut | What it does |
|----------|--------------|
| **Tab** | Autocomplete (the most useful one!) |
| **Up Arrow** | Previous command |
| **Ctrl+C** | Cancel/stop current command |
| **Ctrl+L** | Clear screen |

---

## Common Flags Explained

| Flag | Meaning |
|------|---------|
| `-d` | Detached (background) |
| `-it` | Interactive + Terminal (for shell access) |
| `-p 8080:80` | Port mapping (your computer:container) |
| `--name web` | Name the container "web" |
| `--rm` | Remove container when it stops |
| `-e VAR=value` | Set environment variable |
| `-v /path:/path` | Mount a volume (folder) |

---

## Windows Path Syntax

The `$(pwd)` syntax for "current directory" varies by shell:

| Shell | Syntax |
|-------|--------|
| Mac/Linux/Git Bash | `$(pwd)` |
| PowerShell | `${PWD}` |
| CMD | `%cd%` |

---

## Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| "command not found" | Is Docker Desktop running? |
| "Cannot connect to Docker daemon" | Start Docker Desktop |
| "port already in use" | Use different port: `-p 8081:80` |
| Container exits immediately | Add `-it` for interactive, or `-d` for background |
| "permission denied" (Linux) | Run `sudo usermod -aG docker $USER` then log out/in |

---

## Remember

1. **Containers are disposable** - don't fear deleting them
2. **Data disappears** when you remove a container
3. **Use specific tags** - `nginx:1.25` not `nginx:latest`
4. **Read error messages** - Docker's errors are usually helpful!
