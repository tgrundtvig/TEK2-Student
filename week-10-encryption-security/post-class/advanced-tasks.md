# Post-Class Advanced Tasks: Encryption + Security

**Estimated time: 1-2 hours total**

These tasks go deeper into the topics from class. They're independent — do them in any order. If you get stuck, check [hints.md](hints.md) for guidance.

---

## Task 1: GPG-Signed Git Commits (~30 minutes)

**Objective:** Configure Git to sign your commits with your GPG key, proving that commits really came from you.

### Part A: Find Your GPG Key ID

List your GPG keys with key IDs:

```bash
gpg --list-secret-keys --keyid-format=long
```

Look for the line starting with `sec`. The key ID is the part after the `/`:

```
sec   rsa2048/ABCDEF1234567890 2025-03-09 [SC]
                ^^^^^^^^^^^^^^^^
                This is your key ID
```

### Part B: Configure Git to Use Your GPG Key

```bash
git config --global user.signingkey ABCDEF1234567890
git config --global commit.gpgsign true
```

Replace `ABCDEF1234567890` with your actual key ID.

### Part C: Make a Signed Commit

Create a test repository and make a signed commit:

```bash
mkdir ~/gpg-test && cd ~/gpg-test
git init
echo "Signed commit test" > README.md
git add README.md
git commit -m "My first signed commit"
```

### Part D: Verify the Signature

```bash
git log --show-signature -1
```

You should see something like:

```
gpg: Signature made ...
gpg: Good signature from "Your Name <your@email>"
```

### Part E: See It on GitHub (Optional)

If you push a signed commit to GitHub and your GPG public key is uploaded to your GitHub account (Settings → SSH and GPG keys), GitHub shows a green "Verified" badge next to the commit.

### Verify Success

- [ ] Git is configured to sign commits with your GPG key
- [ ] You made a signed commit and verified the signature
- [ ] You understand why signed commits matter (proves authorship, prevents impersonation)

---

## Task 2: SSH Hardening on Your Server (~30 minutes)

**Objective:** Apply security best practices to SSH on your cloud server from Week 4.

If you no longer have your server, you can practice on a Docker container instead:

```bash
docker run -it --name ssh-lab ubuntu bash -c "apt-get update && apt-get install -y openssh-server && service ssh start && bash"
```

### Part A: Examine the SSH Configuration

```bash
cat /etc/ssh/sshd_config | grep -v "^#" | grep -v "^$"
```

This shows the active (non-commented) configuration lines.

### Part B: Identify Key Security Settings

Look for these settings and understand what they do:

| Setting | Recommended Value | Why |
|---------|-------------------|-----|
| `PermitRootLogin` | `no` | Prevent direct root login |
| `PasswordAuthentication` | `no` | Force key-based auth only |
| `PubkeyAuthentication` | `yes` | Enable key-based auth |
| `MaxAuthTries` | `3` | Limit brute-force attempts |
| `X11Forwarding` | `no` | Disable unless needed |

### Part C: Check Your Server's SSH Port

```bash
ss -tlnp | grep ssh
```

The default SSH port is 22. Note: some administrators change the port to reduce automated attacks, though this is "security through obscurity" and shouldn't be your only defense.

### Part D: Check Login Attempts (If on Real Server)

```bash
grep "Failed password" /var/log/auth.log | tail -20
```

If your server has been online for a while, you might see hundreds of failed login attempts from bots trying common passwords. **This is why key-based authentication matters.**

### Part E: Write Your Findings

Document:
1. What SSH settings are currently active on your server?
2. Which settings would you change for better security?
3. Why is key-based authentication superior to password authentication?

### Verify Success

- [ ] You can read and understand `sshd_config`
- [ ] You know the key SSH security settings and their recommended values
- [ ] You understand why password authentication should be disabled on public servers
- [ ] You can check for failed login attempts

Clean up if using Docker:

```bash
docker stop ssh-lab && docker rm ssh-lab
```

---

## Task 3: Explore Hashing in the Real World (~20 minutes)

**Objective:** See how hashing is used in real applications you already use.

### Part A: Git Commit Hashes

Every git commit ID is a SHA-1 hash:

```bash
cd ~/gpg-test  # or any git repo
git log --oneline -5
```

The short hash (`521c46c`) is just the first 7 characters of the full hash. View the full hash:

```bash
git log --format="%H" -1
```

Git hashes the commit content (message, author, timestamp, tree, parent) together. This means:
- Changing anything about a commit changes its hash
- Two repositories with the same commit hash have identical history up to that point

### Part B: Docker Image Digests

Docker images also use SHA-256 hashes:

```bash
docker pull ubuntu:latest
docker inspect ubuntu:latest | grep -i sha256 | head -3
```

When you see `sha256:abc123...` in Docker, that's a content hash of the image layer. It guarantees you're running exactly the same image, regardless of tags (which can be moved).

### Part C: Password Hashing

On Linux, passwords are stored as hashes in `/etc/shadow`. In a Docker container:

```bash
docker run --rm ubuntu bash -c "
  useradd -m testuser &&
  echo 'testuser:mypassword' | chpasswd &&
  grep testuser /etc/shadow
"
```

You'll see something like:

```
testuser:$6$randomsalt$longhashstring...:...
```

The `$6$` means SHA-512. The `randomsalt` is a salt (random value added before hashing to prevent pre-computed attacks). Even if two users have the same password, they'll have different hashes because of different salts.

### Verify Success

- [ ] You understand that git commit IDs are hashes
- [ ] You understand that Docker uses hashes for image integrity
- [ ] You understand how password hashing works (hash + salt)
- [ ] You can explain why salting prevents pre-computed attacks

---

## Task 4: Certificate Pinning and Security Headers (~25 minutes)

**Objective:** Explore advanced HTTPS security mechanisms that build on this week's encryption concepts.

### Part A: Inspect HTTPS Security Headers

```bash
curl -sI https://github.com | grep -i "strict\|content-security\|x-frame\|x-content"
```

Look for:
- **Strict-Transport-Security (HSTS)**: Forces browsers to always use HTTPS. The `max-age` value is in seconds.
- **Content-Security-Policy**: Controls what resources the page can load
- **X-Frame-Options**: Prevents clickjacking
- **X-Content-Type-Options**: Prevents MIME type sniffing

### Part B: Compare Security Headers

Try a few different sites and compare:

```bash
curl -sI https://google.com | grep -i "strict-transport"
curl -sI https://facebook.com | grep -i "strict-transport"
```

Which sites use HSTS? What are their `max-age` values?

### Part C: Understanding HSTS

HSTS is connected to certificates (Week 9) and encryption (this week):
1. Browser visits `https://example.com` for the first time
2. Server responds with `Strict-Transport-Security: max-age=31536000`
3. For the next year (31,536,000 seconds), the browser will **refuse** to connect via HTTP
4. Even if you type `http://example.com`, the browser automatically upgrades to HTTPS

This prevents **SSL stripping attacks** — where an attacker intercepts your initial HTTP connection and prevents the upgrade to HTTPS.

### Part D: Check Certificate Transparency

Certificate Transparency (CT) logs are public records of every TLS certificate issued. This helps detect rogue certificates:

```bash
openssl s_client -connect github.com:443 2>/dev/null | openssl x509 -noout -text | grep -A2 "CT Precertificate"
```

If you see CT-related extensions, the certificate is published in a transparency log.

### Verify Success

- [ ] You can check HTTPS security headers with `curl`
- [ ] You understand what HSTS does and why it matters
- [ ] You know that Certificate Transparency logs exist
- [ ] You can compare security practices across different websites
