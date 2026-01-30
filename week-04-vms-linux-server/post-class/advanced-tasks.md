# Post-class Advanced Tasks

**Estimated time: 1-2 hours**

These tasks will make your deployment production-ready. They're optional but highly recommended for a complete setup.

If you get stuck, check the [hints.md](hints.md) file - but try on your own first!

---

## Task 1: Configure a Custom Domain (~30 minutes)

### Objective

Access your application via a custom domain instead of an IP address.

### Background

IP addresses work, but they're hard to remember and look unprofessional. A domain name like `myapp.example.com` is much better.

### Options for Getting a Domain

1. **Buy a domain** (~$10-15/year): Namecheap, Google Domains, etc.
2. **Use a free subdomain service**: Afraid.org FreeDNS, DuckDNS
3. **Use your school's domain** (if available)

### Instructions

#### Step 1: Get a domain or subdomain

For this example, we'll assume you have `myapp.example.com`.

#### Step 2: Create a DNS A record

In your domain provider's DNS settings:

| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | myapp | YOUR_SERVER_IP | 300 |

Or for the root domain:

| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | @ | YOUR_SERVER_IP | 300 |

#### Step 3: Wait for DNS propagation

DNS changes can take up to 48 hours, but usually complete within minutes.

Test with:
```bash
# On your local machine
nslookup myapp.example.com
# or
dig myapp.example.com
```

#### Step 4: Verify access

Once DNS is propagated:
```
http://myapp.example.com
```

### Verify Success
- [ ] DNS record created
- [ ] Domain resolves to your server IP
- [ ] Can access application via domain name

### Challenge Extension
- Configure multiple subdomains for different applications
- Set up a CNAME record instead of an A record

---

## Task 2: Configure HTTPS with Let's Encrypt (~30 minutes)

### Objective

Secure your application with free SSL/TLS certificates.

### Background

HTTPS encrypts traffic between users and your server. It's required for:
- User trust
- SEO (Google ranks HTTPS sites higher)
- Modern browser features (service workers, geolocation, etc.)

**Let's Encrypt** provides free, automated SSL certificates.

### Prerequisites
- A domain name pointing to your server (Task 1)
- Ports 80 and 443 open in firewall

### Instructions

#### Step 1: Install Certbot

SSH into your server:

```bash
sudo apt update
sudo apt install -y certbot
```

#### Step 2: Stop your container temporarily

Certbot needs port 80 for verification:

```bash
docker stop myapp
```

#### Step 3: Get a certificate

```bash
sudo certbot certonly --standalone -d myapp.example.com
```

Follow the prompts:
- Enter your email
- Agree to terms
- Certificate will be saved to `/etc/letsencrypt/live/myapp.example.com/`

#### Step 4: Set up HTTPS with nginx

Install nginx as a reverse proxy:

```bash
sudo apt install -y nginx
```

Create configuration file:

```bash
sudo nano /etc/nginx/sites-available/myapp
```

Add this configuration:

```nginx
server {
    listen 80;
    server_name myapp.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name myapp.example.com;

    ssl_certificate /etc/letsencrypt/live/myapp.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

#### Step 5: Enable the site

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

# Remove default site
sudo rm /etc/nginx/sites-enabled/default

# Test configuration
sudo nginx -t

# Reload nginx
sudo systemctl reload nginx
```

#### Step 6: Update your container

Start your container on a different port (nginx handles 80/443):

```bash
docker run -d \
  --name myapp \
  -p 8080:8080 \
  --restart unless-stopped \
  ghcr.io/YOUR_USERNAME/hello-maven:latest
```

#### Step 7: Set up auto-renewal

Let's Encrypt certificates expire after 90 days. Set up auto-renewal:

```bash
# Test renewal
sudo certbot renew --dry-run

# The certbot package sets up a systemd timer automatically
sudo systemctl status certbot.timer
```

### Verify Success
- [ ] Certificate obtained successfully
- [ ] Nginx is configured and running
- [ ] `https://myapp.example.com` works
- [ ] HTTP redirects to HTTPS
- [ ] Browser shows padlock icon

### Challenge Extension
- Add HTTP/2 support to nginx
- Configure HSTS (HTTP Strict Transport Security)
- Get an A+ rating on [SSL Labs Test](https://www.ssllabs.com/ssltest/)

---

## Task 3: Set Up Basic Monitoring (~25 minutes)

### Objective

Monitor your server and get notified when something goes wrong.

### Background

Without monitoring, you won't know if your application goes down until users complain. Basic monitoring should track:
- Server health (CPU, memory, disk)
- Application availability
- Response times

### Instructions

#### Option A: Simple Uptime Monitoring with UptimeRobot (Easiest)

1. Go to [UptimeRobot](https://uptimerobot.com/) and create a free account
2. Add a new monitor:
   - Monitor Type: HTTP(s)
   - URL: `https://myapp.example.com`
   - Monitoring Interval: 5 minutes
3. Set up email alerts
4. Done! You'll get emails when your site goes down.

#### Option B: Server Metrics with htop and journalctl

SSH into your server and monitor in real-time:

```bash
# Live process monitor
htop

# View system logs
sudo journalctl -f

# View Docker container logs
docker logs -f myapp

# View resource usage
docker stats
```

#### Option C: Set Up Simple Health Check Endpoint

Add a health endpoint to your application. For a Java/Spring application:

```java
@RestController
public class HealthController {
    @GetMapping("/health")
    public String health() {
        return "OK";
    }
}
```

Then monitor this endpoint with UptimeRobot.

#### Option D: Create a Monitoring Script

Create a script on your server:

```bash
nano ~/check-health.sh
```

```bash
#!/bin/bash

# Check if Docker is running
if ! systemctl is-active --quiet docker; then
    echo "Docker is not running!"
    exit 1
fi

# Check if container is running
if ! docker ps | grep -q myapp; then
    echo "Container 'myapp' is not running!"
    exit 1
fi

# Check HTTP response
HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080)
if [ "$HTTP_STATUS" != "200" ]; then
    echo "Application returned status $HTTP_STATUS"
    exit 1
fi

echo "All checks passed"
```

Make it executable and test:

```bash
chmod +x ~/check-health.sh
./check-health.sh
```

Add to crontab to run every 5 minutes:

```bash
crontab -e
```

Add this line:
```
*/5 * * * * /home/username/check-health.sh >> /home/username/health.log 2>&1
```

### Verify Success
- [ ] Monitoring is configured
- [ ] Can see server metrics
- [ ] Alerts are set up (email or other)
- [ ] Tested by stopping container briefly

---

## Task 4: Multi-Container Deployment with Compose (~30 minutes)

### Objective

Deploy multiple containers (e.g., app + database) using Docker Compose.

### Background

Real applications often need multiple services:
- Web application
- Database
- Cache (Redis)
- Reverse proxy

Docker Compose manages multiple containers together.

### Instructions

#### Step 1: Install Docker Compose

It's usually included with Docker, but verify:

```bash
docker compose version
```

#### Step 2: Create docker-compose.yml

On your server, create a directory for your project:

```bash
mkdir -p ~/myapp
cd ~/myapp
nano docker-compose.yml
```

```yaml
version: '3.8'

services:
  app:
    image: ghcr.io/YOUR_USERNAME/hello-maven:latest
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/mydb
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

#### Step 3: Start the services

```bash
docker compose up -d
```

#### Step 4: Verify all services are running

```bash
docker compose ps
docker compose logs
```

#### Step 5: Update deployment workflow

Update your GitHub Actions to use Compose:

```yaml
- name: Deploy to server
  uses: appleboy/ssh-action@v1.0.0
  with:
    host: ${{ secrets.SERVER_IP }}
    username: ${{ secrets.SERVER_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      cd ~/myapp

      # Login and pull latest
      echo ${{ secrets.GHCR_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      docker compose pull

      # Restart services
      docker compose up -d

      # Clean up
      docker image prune -f
```

### Verify Success
- [ ] docker-compose.yml created
- [ ] All services start with `docker compose up`
- [ ] Services can communicate (app connects to db)
- [ ] Deployment workflow updated

### Challenge Extension
- Add Redis for caching
- Add nginx as a reverse proxy in the Compose file
- Configure environment-specific compose files (dev, staging, prod)

---

## Going Further (Optional)

If you finish early and want more challenges:

### 1. Set Up SSH Hardening

```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config

# Disable password authentication (SSH key only)
PasswordAuthentication no

# Disable root login
PermitRootLogin no

# Restart SSH
sudo systemctl restart sshd
```

### 2. Configure Fail2ban

Protect against brute-force attacks:

```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### 3. Set Up Log Rotation

Prevent logs from filling disk:

```bash
# Docker logs are rotated by daemon.json
sudo nano /etc/docker/daemon.json
```

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Restart Docker:
```bash
sudo systemctl restart docker
```

### 4. Set Up Backup

Backup important data:

```bash
# Backup Docker volumes
docker run --rm -v myapp_postgres_data:/data -v $(pwd):/backup alpine tar czf /backup/db-backup.tar.gz /data
```

---

## Before Next Week

Come prepared to share:

1. Your deployment URL (IP or domain)
2. Any interesting challenges you solved
3. Questions about production deployments

These skills are directly applicable to real-world DevOps work!
