# Hints for Advanced Tasks

Use these hints if you get stuck. Try to solve problems on your own first!

---

## Task 1: Configure a Custom Domain

<details>
<summary>Hint 1: DNS changes not taking effect</summary>

DNS propagation can take time. To check if it's working:

```bash
# Check with specific DNS server
nslookup myapp.example.com 8.8.8.8

# Check DNS propagation globally
# Visit: https://www.whatsmydns.net/
```

While waiting, you can test locally by editing `/etc/hosts`:
```bash
# Add to /etc/hosts on your local machine
YOUR_SERVER_IP myapp.example.com
```

Remove this line after DNS propagates!

</details>

<details>
<summary>Hint 2: Domain works but application doesn't</summary>

Check:

1. **Firewall allows port 80/443:**
   ```bash
   sudo ufw status
   sudo ufw allow 80
   sudo ufw allow 443
   ```

2. **Container is running and mapped to right port:**
   ```bash
   docker ps
   # Should show 0.0.0.0:80->8080/tcp or similar
   ```

3. **Application responds locally:**
   ```bash
   curl http://localhost:80
   ```

</details>

<details>
<summary>Hint 3: A record vs CNAME</summary>

- **A record**: Points domain to IP address
  - Use for: Your own server with static IP
  - Example: `myapp -> 64.23.145.101`

- **CNAME record**: Points domain to another domain
  - Use for: Pointing to a service that might change IP
  - Example: `myapp -> myserver.provider.com`

For your VM with a fixed IP, use an A record.

</details>

---

## Task 2: HTTPS with Let's Encrypt

<details>
<summary>Hint 1: Certbot verification fails</summary>

Certbot needs to verify you own the domain. Common issues:

1. **Port 80 is blocked:**
   ```bash
   sudo ufw allow 80
   ```

2. **Something else is using port 80:**
   ```bash
   sudo ss -tulpn | grep :80
   # Stop the service using port 80
   docker stop myapp  # if Docker container
   sudo systemctl stop nginx  # if nginx
   ```

3. **DNS not pointed correctly:**
   ```bash
   nslookup myapp.example.com
   # Should return your server IP
   ```

Try again with verbose output:
```bash
sudo certbot certonly --standalone -d myapp.example.com -v
```

</details>

<details>
<summary>Hint 2: Nginx configuration errors</summary>

Check nginx syntax:
```bash
sudo nginx -t
```

Common issues:

1. **Missing semicolon** - Every directive needs `;`
2. **Wrong certificate path** - Check exact path with:
   ```bash
   sudo ls /etc/letsencrypt/live/
   ```
3. **Duplicate server_name** - Each domain should only be in one server block

View nginx error log:
```bash
sudo tail -f /var/log/nginx/error.log
```

</details>

<details>
<summary>Hint 3: Proxy not forwarding correctly</summary>

If nginx connects but app doesn't respond:

1. **Check container is running:**
   ```bash
   docker ps
   ```

2. **Check container port mapping:**
   ```bash
   # Container should be on 8080 (or whatever you configured)
   docker run -d -p 8080:8080 ...
   ```

3. **Test direct connection:**
   ```bash
   curl http://localhost:8080
   ```

4. **Check nginx proxy_pass:**
   ```nginx
   location / {
       proxy_pass http://localhost:8080;  # Match your container port
   }
   ```

</details>

<details>
<summary>Hint 4: Certificate renewal issues</summary>

Test renewal:
```bash
sudo certbot renew --dry-run
```

If it fails because port 80 is in use:

**Option A:** Use webroot instead of standalone:
```bash
sudo certbot certonly --webroot -w /var/www/html -d myapp.example.com
```

**Option B:** Configure nginx to handle ACME challenge:
```nginx
location /.well-known/acme-challenge/ {
    root /var/www/html;
}
```

Check renewal timer is active:
```bash
sudo systemctl status certbot.timer
```

</details>

---

## Task 3: Set Up Basic Monitoring

<details>
<summary>Hint 1: UptimeRobot not detecting downtime</summary>

Check what UptimeRobot sees:

1. **Monitor the right URL:**
   - Include `http://` or `https://`
   - Check for typos in domain

2. **Check expected response:**
   - Default expects HTTP 200
   - If your app returns different status, configure it

3. **Test manually:**
   ```bash
   curl -I https://myapp.example.com
   # Look at HTTP status code
   ```

</details>

<details>
<summary>Hint 2: Health check script not running</summary>

Troubleshoot cron:

1. **Check crontab is saved:**
   ```bash
   crontab -l
   ```

2. **Check script permissions:**
   ```bash
   chmod +x ~/check-health.sh
   ```

3. **Test script manually:**
   ```bash
   ~/check-health.sh
   echo $?  # Should be 0 for success
   ```

4. **Check cron logs:**
   ```bash
   sudo grep CRON /var/log/syslog
   ```

5. **Use full paths in cron:**
   ```
   */5 * * * * /usr/bin/bash /home/username/check-health.sh
   ```

</details>

<details>
<summary>Hint 3: Docker stats not showing</summary>

If `docker stats` doesn't show containers:

1. **Check containers are running:**
   ```bash
   docker ps
   ```

2. **Check Docker is running:**
   ```bash
   sudo systemctl status docker
   ```

3. **Permissions issue:**
   ```bash
   # Either use sudo
   sudo docker stats

   # Or add user to docker group
   sudo usermod -aG docker $USER
   # Log out and back in
   ```

</details>

---

## Task 4: Multi-Container Deployment

<details>
<summary>Hint 1: Services can't communicate</summary>

Docker Compose creates a network automatically. Services reference each other by name:

```yaml
# In app service
environment:
  - DATABASE_URL=postgresql://postgres:password@db:5432/mydb
                                              # ↑ This is the service name
```

Check network:
```bash
docker network ls
docker network inspect myapp_default
```

Test connectivity:
```bash
# Get into app container
docker compose exec app sh

# Try to reach database
ping db
```

</details>

<details>
<summary>Hint 2: Database data lost on restart</summary>

You need a volume for persistence:

```yaml
services:
  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data  # Named volume

volumes:
  postgres_data:  # Declare the volume
```

Check volume exists:
```bash
docker volume ls
```

**Don't use** bind mounts on the host for databases (permission issues).

</details>

<details>
<summary>Hint 3: Compose file syntax errors</summary>

Validate your file:
```bash
docker compose config
```

Common issues:

1. **Indentation** - YAML is picky about spaces
2. **Version string** - Use quotes: `version: '3.8'`
3. **Port mapping** - Use quotes for ports like `"80:80"`
4. **Environment variables** - Either list or map syntax:

```yaml
# List syntax
environment:
  - VAR1=value1
  - VAR2=value2

# Map syntax
environment:
  VAR1: value1
  VAR2: value2
```

</details>

<details>
<summary>Hint 4: Deployment workflow failing</summary>

SSH action issues:

1. **Key format:**
   ```yaml
   key: ${{ secrets.SSH_PRIVATE_KEY }}
   # Make sure secret includes entire key including header/footer:
   # -----BEGIN OPENSSH PRIVATE KEY-----
   # ...
   # -----END OPENSSH PRIVATE KEY-----
   ```

2. **Working directory:**
   ```yaml
   script: |
     cd ~/myapp  # Make sure this directory exists!
     docker compose pull
   ```

3. **Test SSH manually:**
   ```bash
   ssh -i deploy_key user@server "docker compose -f ~/myapp/docker-compose.yml ps"
   ```

4. **Check Docker Compose path:**
   ```bash
   # On server, verify compose file exists
   ls -la ~/myapp/docker-compose.yml
   ```

</details>

---

## General SSH Troubleshooting

<details>
<summary>"Permission denied (publickey)"</summary>

This means SSH key authentication failed.

1. **Check key is in authorized_keys on server:**
   ```bash
   # From another session or console
   cat ~/.ssh/authorized_keys
   ```

2. **Check key permissions on server:**
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

3. **Check you're using the right key locally:**
   ```bash
   ssh -i ~/.ssh/deploy_key -v user@server
   ```

4. **Check SSH config isn't overriding:**
   ```bash
   cat ~/.ssh/config
   ```

</details>

<details>
<summary>"Connection refused" on port 22</summary>

SSH service isn't running or accessible.

1. **Check SSH is running (need console access from provider):**
   ```bash
   sudo systemctl status sshd
   ```

2. **Check firewall:**
   ```bash
   sudo ufw status
   # Port 22 should be ALLOW
   ```

3. **Check provider's firewall:**
   - Some providers have a separate security group/firewall
   - Check in provider dashboard

</details>

<details>
<summary>"Connection timed out"</summary>

Can't reach the server at all.

1. **Check IP address is correct**
2. **Check server is running in provider dashboard**
3. **Check from different network (phone hotspot?)**
4. **Try pinging:**
   ```bash
   ping YOUR_SERVER_IP
   ```

</details>

---

## Docker Troubleshooting

<details>
<summary>Container keeps restarting</summary>

Check logs:
```bash
docker logs myapp
docker logs myapp --tail 100
```

Common causes:
- Application crash (check error message)
- Missing environment variable
- Port already in use
- Out of memory

</details>

<details>
<summary>"Port already in use"</summary>

Find what's using the port:
```bash
sudo ss -tulpn | grep :80
```

Either stop that service or use a different port:
```bash
docker run -d -p 8080:8080 ...  # Use 8080 instead of 80
```

</details>

<details>
<summary>Can't pull from GHCR</summary>

Authentication issue:

1. **Check you're logged in:**
   ```bash
   docker login ghcr.io -u YOUR_GITHUB_USERNAME
   ```

2. **Check token has correct scope:**
   - Token needs `read:packages` permission
   - Go to GitHub → Settings → Developer Settings → Personal access tokens

3. **Check image name is correct:**
   ```bash
   # Must be lowercase!
   ghcr.io/username/imagename:tag
   ```

</details>

---

## Still Stuck?

1. **Check logs** - Most services log errors somewhere:
   ```bash
   sudo journalctl -xe  # System journal
   docker logs container_name  # Container logs
   sudo tail -f /var/log/nginx/error.log  # Nginx errors
   ```

2. **Test components separately:**
   - Can you SSH in? (network/auth)
   - Can you run docker commands? (docker working)
   - Can you pull images? (GHCR auth)
   - Can container start? (application issues)

3. **Simplify and rebuild:**
   - Start with basic working state
   - Add one thing at a time
   - Test after each change

4. **Search the error** - Copy exact error messages into search

5. **Ask for help** - Bring specific error messages to class
