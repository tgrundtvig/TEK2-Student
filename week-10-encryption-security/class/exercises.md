# Class Exercises: Encryption + Security

**Work through these exercises during class. Ask for help if you get stuck!**

---

## Warm-Up (~5 minutes)

Quick check — can you answer these from the pre-class reading?

1. What's the difference between symmetric and asymmetric encryption?
2. Why does TLS use *both* types?
3. Is hashing encryption? Why or why not?

Compare your answers with a classmate. If you disagree, discuss!

---

## Exercise 1: GPG Encrypted Messaging (~40 minutes)

**Goal:** Generate a GPG key pair, exchange public keys with a classmate, and send each other encrypted messages. This makes asymmetric encryption tangible — you'll experience the whole flow.

### Part A: Generate Your GPG Key Pair

Run this in your terminal:

```bash
gpg --full-generate-key
```

When prompted:
1. **Key type:** Choose `1` (RSA and RSA)
2. **Key size:** Type `2048`
3. **Expiration:** Type `0` (does not expire) — this is fine for a class exercise
4. **Real name:** Your actual name
5. **Email:** Your email address
6. **Passphrase:** Choose something you'll remember (or leave blank for this exercise)

Verify your key was created:

```bash
gpg --list-keys
```

You should see your name and email in the output.

### Part B: Export Your Public Key

```bash
gpg --export --armor "Your Name" > my-public-key.asc
```

Replace `"Your Name"` with the name you used when generating the key. Look at the exported key:

```bash
cat my-public-key.asc
```

You should see a block starting with `-----BEGIN PGP PUBLIC KEY BLOCK-----`.

### Part C: Exchange Public Keys

Find a partner. Exchange your public key files. You can:
- Share the file on a USB drive
- Email it to each other
- Copy-paste it via a chat message

Once you have your partner's public key file, import it:

```bash
gpg --import partner-public-key.asc
```

Verify it was imported:

```bash
gpg --list-keys
```

You should now see both your key and your partner's key.

### Part D: Encrypt a Message for Your Partner

Create a message:

```bash
echo "Hello from $(whoami)! This is our secret message." > for-partner.txt
```

Encrypt it for your partner (use their name or email):

```bash
gpg --encrypt --armor --recipient "Partner Name" for-partner.txt
```

This creates `for-partner.txt.asc`. Look at it:

```bash
cat for-partner.txt.asc
```

Encrypted! Only your partner's private key can decrypt this. Send this file to your partner.

### Part E: Decrypt Your Partner's Message

When you receive an encrypted file from your partner:

```bash
gpg --decrypt partner-message.txt.asc
```

You'll be prompted for your passphrase (if you set one). The decrypted message should appear in the terminal.

Think about what just happened:
- Your partner encrypted the message with YOUR public key
- Only YOUR private key (which never left your machine) can decrypt it
- Even if someone intercepted the encrypted file and your public key, they couldn't read the message

### Part F: Try to Decrypt Your Own Message (It Should Fail)

Try to decrypt the message you sent to your partner:

```bash
gpg --decrypt for-partner.txt.asc
```

You should see an error like:

```
gpg: decryption failed: No secret key
```

This makes sense — you encrypted it with your *partner's* public key, so only their private key can decrypt it. You don't have their private key.

### Self-check

- [ ] You generated a GPG key pair
- [ ] You exported your public key and imported your partner's
- [ ] You encrypted a message that only your partner could read
- [ ] You decrypted a message your partner encrypted for you
- [ ] You understand why you can't decrypt your own outgoing message

---

## Exercise 2: Digital Signatures with GPG (~20 minutes)

**Goal:** Sign a message so the recipient can verify it came from you and wasn't modified.

### Part A: Sign a Message

Create a file:

```bash
echo "I hereby certify that this assignment is my own work." > statement.txt
```

Sign it:

```bash
gpg --clearsign statement.txt
```

Look at the signed file:

```bash
cat statement.txt.asc
```

You should see something like:

```
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA512

I hereby certify that this assignment is my own work.
-----BEGIN PGP SIGNATURE-----

iQEzBAABCgAdFiEE...
...
-----END PGP SIGNATURE-----
```

Notice: the message is **readable** — it's signed, not encrypted. Anyone can read it, but the signature proves it came from you.

### Part B: Verify a Signature

Send the signed file to your partner. When you receive their signed file:

```bash
gpg --verify partner-statement.txt.asc
```

You should see:

```
gpg: Good signature from "Partner Name <partner@email.com>"
```

### Part C: Tamper with a Signed Message

Copy the signed file and change one character in the message (not in the signature block):

```bash
cp statement.txt.asc tampered.txt.asc
```

Edit `tampered.txt.asc` with a text editor and change a word. Then verify:

```bash
gpg --verify tampered.txt.asc
```

You should see:

```
gpg: BAD signature from "Your Name"
```

**One changed character** invalidates the signature. This is integrity verification in action.

### Self-check

- [ ] You signed a message with your private key
- [ ] Your partner verified your signature with your public key
- [ ] Modifying the signed message caused verification to fail
- [ ] You understand the difference: signing ≠ encrypting

---

## Exercise 3: SSH Key Deep Dive (~20 minutes)

**Goal:** Examine your SSH keys with a deeper understanding of asymmetric cryptography.

### Part A: Key Anatomy

Look at your SSH keys:

```bash
cat ~/.ssh/id_ed25519.pub
```

The format is:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI[base64-encoded-key] user@host
│            │                                               │
└── Algorithm └── The actual public key (base64-encoded)     └── Comment
```

### Part B: Key Fingerprint

```bash
ssh-keygen -l -f ~/.ssh/id_ed25519.pub
```

The fingerprint is a **SHA-256 hash** of the public key. This is what you see when connecting to a new server:

```
The authenticity of host 'server (192.168.1.1)' can't be established.
ED25519 key fingerprint is SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz.
Are you sure you want to continue connecting (yes/no)?
```

When you type "yes", the server's fingerprint gets saved to `~/.ssh/known_hosts`. Future connections check this fingerprint to detect if someone is impersonating the server (a **man-in-the-middle attack**).

### Part C: Visualize Your Key

```bash
ssh-keygen -lv -f ~/.ssh/id_ed25519.pub
```

You'll see a piece of "randomart" — a visual representation of your key. This makes it easier for humans to spot if a key has changed (you'd see a different picture).

### Part D: Compare Ed25519 and RSA

You generated an RSA key in the pre-class exercises and you have an Ed25519 SSH key. Let's compare:

| | Ed25519 (your SSH key) | RSA 2048 (from pre-class) |
|---|---|---|
| **Key size** | 256 bits | 2048 bits |
| **Security level** | ~128-bit equivalent | ~112-bit equivalent |
| **Speed** | Faster | Slower |
| **Key file size** | Smaller | Larger |

Ed25519 achieves better security with a smaller key because it uses elliptic curve math instead of prime factoring. That's why `ssh-keygen -t ed25519` is the modern recommendation.

### Self-check

- [ ] You understand the three parts of a public key file (algorithm, key, comment)
- [ ] You know that the key fingerprint is a hash
- [ ] You understand what `known_hosts` protects against
- [ ] You can explain why Ed25519 is preferred over RSA for SSH

---

## Exercise 4: Linux File Permissions (~25 minutes)

**Goal:** Understand Linux file permissions — your first line of defense on any server.

### Part A: Understanding Permission Strings

Run this on a Docker container to have a clean environment:

```bash
docker run -it --name permissions-lab ubuntu bash
```

Inside the container, create some test files:

```bash
mkdir /lab && cd /lab
echo "public info" > public.txt
echo "private secret" > private.txt
echo '#!/bin/bash\necho "I run!"' > script.sh
```

Look at the permissions:

```bash
ls -la
```

You should see something like:

```
-rw-r--r-- 1 root root  12 ... public.txt
-rw-r--r-- 1 root root  15 ... private.txt
-rw-r--r-- 1 root root  28 ... script.sh
```

The permission string `-rw-r--r--` breaks down as:

```
 -  rw-  r--  r--
 │   │    │    │
 │   │    │    └── Others: read only
 │   │    └─────── Group: read only
 │   └──────────── Owner: read + write
 └──────────────── Type: - = file, d = directory
```

### Part B: Changing Permissions with Octal Notation

```bash
chmod 600 private.txt
chmod 755 script.sh
ls -la
```

You should see:

```
-rw------- 1 root root  15 ... private.txt
-rwxr-xr-x 1 root root  28 ... script.sh
```

The numbers:
- `600` = owner read+write (6=4+2), group nothing (0), others nothing (0)
- `755` = owner rwx (7=4+2+1), group rx (5=4+1), others rx (5=4+1)

### Part C: Why This Matters for Security

Try creating a user and testing permissions:

```bash
apt-get update && apt-get install -y sudo
useradd -m student
```

Switch to the new user and try to read the private file:

```bash
su - student
cat /lab/private.txt
```

You should see:

```
cat: /lab/private.txt: Permission denied
```

But the public file works:

```bash
cat /lab/public.txt
```

Exit back to root:

```bash
exit
```

**This is exactly how SSH protects your private key:** `chmod 600 ~/.ssh/id_ed25519` means only you can read it.

### Part D: Checking for Security Issues

Still in the container, run:

```bash
chmod 777 private.txt
ls -la private.txt
```

`777` means everyone can read, write, and execute. This is a **security problem** for any sensitive file. SSH will actually refuse to use a private key with permissions like this.

Fix it:

```bash
chmod 600 private.txt
```

### Part E: Exit and Clean Up

```bash
exit
```

```bash
docker rm permissions-lab
```

### Self-check

- [ ] You can read a permission string like `-rwxr-xr--`
- [ ] You understand octal permission numbers (600, 644, 755, 777)
- [ ] You know why SSH private keys must be `chmod 600`
- [ ] You saw that file permissions prevent unauthorized users from reading files

---

## Exercise 5: Security Challenge — Find the Vulnerabilities (~30 minutes)

**Goal:** Find security issues in a deliberately misconfigured Docker container. This is a mini security audit.

### Part A: Start the Vulnerable Container

```bash
docker run -d --name vuln-lab -p 8080:80 ubuntu bash -c '
  apt-get update && apt-get install -y nginx sudo > /dev/null 2>&1

  # Create some users
  useradd -m admin
  useradd -m developer
  echo "admin:admin123" | chpasswd
  echo "developer:dev" | chpasswd

  # Create sensitive files with bad permissions
  echo "DB_PASSWORD=SuperSecret123" > /etc/app-secrets.env
  chmod 644 /etc/app-secrets.env

  echo "-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA0Z3VS5JJcds3xfn/ygWyF8PbnGy0AHB7MhgHcTz6sE2I2yPB
THIS_IS_A_FAKE_KEY_FOR_EDUCATIONAL_PURPOSES_ONLY
-----END RSA PRIVATE KEY-----" > /etc/ssl/private/server.key
  chmod 644 /etc/ssl/private/server.key

  # Create a password file
  echo "root:toor" > /var/www/.passwords
  chmod 777 /var/www/.passwords

  # Create a world-writable config
  echo "server_name vuln-lab;" > /etc/nginx/conf.d/app.conf
  chmod 666 /etc/nginx/conf.d/app.conf

  # Start services
  nginx
  tail -f /dev/null
'
```

Wait a few seconds for the setup to complete:

```bash
docker exec vuln-lab ls /etc/app-secrets.env
```

If you see the path, the container is ready.

### Part B: Audit the Container

Enter the container:

```bash
docker exec -it vuln-lab bash
```

Your mission: **find as many security issues as you can.** Here are things to check:

1. **World-readable secrets:**
   ```bash
   find / -name "*.env" -o -name "*.key" -o -name ".passwords" 2>/dev/null | xargs ls -la
   ```

2. **World-writable files:**
   ```bash
   find /etc -perm -o=w -type f 2>/dev/null
   ```

3. **Weak passwords:**
   ```bash
   cat /var/www/.passwords
   ```

4. **Private key permissions:**
   ```bash
   ls -la /etc/ssl/private/
   ```

5. **Check who can read the app secrets:**
   ```bash
   su - developer -c "cat /etc/app-secrets.env"
   ```

### Part C: Fix the Issues

For each issue you found, apply the fix:

```bash
# Fix secret file permissions
chmod 600 /etc/app-secrets.env
chown root:root /etc/app-secrets.env

# Fix private key permissions
chmod 600 /etc/ssl/private/server.key

# Fix password file
chmod 600 /var/www/.passwords

# Fix writable config
chmod 644 /etc/nginx/conf.d/app.conf
```

Verify the developer can no longer read the secrets:

```bash
su - developer -c "cat /etc/app-secrets.env"
```

You should now see `Permission denied`.

### Part D: Write a Mini Security Report

Write down:
1. How many issues did you find?
2. What was the worst one? Why?
3. What's the general principle? (Think about "least privilege")

Exit and clean up:

```bash
exit
docker stop vuln-lab && docker rm vuln-lab
```

### Self-check

- [ ] You found at least 4 security issues in the container
- [ ] You fixed the issues using `chmod` and `chown`
- [ ] You verified that the fixes work (developer can no longer read secrets)
- [ ] You understand the principle of "least privilege"

---

## Summary

| Topic | What You Did | Key Tool |
|-------|-------------|----------|
| GPG encryption | Exchanged encrypted messages with a partner | `gpg` |
| Digital signatures | Signed and verified messages, saw tamper detection | `gpg --sign/--verify` |
| SSH key anatomy | Examined keys, fingerprints, and known_hosts | `ssh-keygen` |
| File permissions | Read permission strings, used `chmod` with octal | `chmod`, `ls -la` |
| Security audit | Found and fixed vulnerabilities in a container | `find`, `chmod`, `chown` |

**Key takeaway:** Encryption and security aren't just theory — they're practical tools you use every day. Your SSH keys are asymmetric key pairs, your file permissions are access control, and TLS combines everything to keep the web secure.
