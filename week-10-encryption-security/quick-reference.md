# Week 10 Quick Reference Card

## Symmetric Encryption (One Key)

```
Same key encrypts and decrypts:

  Plaintext ──► [ AES encrypt with key ] ──► Ciphertext
  Ciphertext ──► [ AES decrypt with key ] ──► Plaintext
```

| Command | What It Does |
|---------|-------------|
| `openssl enc -aes-256-cbc -salt -pbkdf2 -in file.txt -out file.enc` | Encrypt a file (prompts for password) |
| `openssl enc -aes-256-cbc -d -pbkdf2 -in file.enc -out file.txt` | Decrypt a file |

**Key fact:** AES-256 = 256-bit key = 2²⁵⁶ possible keys. Fast enough for bulk data.

---

## Asymmetric Encryption (Key Pair)

```
Public key encrypts, private key decrypts:

  Plaintext ──► [ Encrypt with PUBLIC key ] ──► Ciphertext
  Ciphertext ──► [ Decrypt with PRIVATE key ] ──► Plaintext

Digital signatures work in reverse:

  Message ──► [ Sign with PRIVATE key ] ──► Signature
  Signature ──► [ Verify with PUBLIC key ] ──► Valid/Invalid
```

| Command | What It Does |
|---------|-------------|
| `openssl genrsa -out private.pem 2048` | Generate RSA private key |
| `openssl rsa -in private.pem -pubout -out public.pem` | Extract public key |
| `openssl rsautl -encrypt -pubin -inkey public.pem -in msg.txt -out msg.enc` | Encrypt with public key |
| `openssl rsautl -decrypt -inkey private.pem -in msg.enc -out msg.txt` | Decrypt with private key |

---

## How TLS Uses Both

```
1. Browser ──► Server: ClientHello (supported ciphers)
2. Server ──► Browser: ServerHello + Certificate (contains public key)
3. Browser verifies certificate chain (Week 9)
4. Browser generates random symmetric key
5. Browser encrypts symmetric key with server's PUBLIC key  ◄── Asymmetric
6. Server decrypts with its PRIVATE key
7. Both sides now have the same symmetric key
8. All HTTP traffic encrypted with symmetric key              ◄── Symmetric (fast!)
```

**Why both?** Asymmetric is too slow for bulk data. Symmetric is fast but needs a safe way to share the key. Solution: use asymmetric to exchange a symmetric key.

---

## Hashing (One-Way)

```
Input ──► [ SHA-256 ] ──► Fixed-size hash (256 bits)
                          (cannot reverse!)
```

| Command | What It Does |
|---------|-------------|
| `sha256sum file.txt` | Hash a file |
| `echo -n "hello" \| sha256sum` | Hash a string |
| `sha256sum file1 file2` | Hash multiple files |

**Key facts:**
- Same input always produces the same hash
- Change one bit of input → completely different hash (avalanche effect)
- Used for: password storage, file integrity, digital signatures

---

## GPG (GNU Privacy Guard)

| Command | What It Does |
|---------|-------------|
| `gpg --full-generate-key` | Generate a GPG key pair |
| `gpg --list-keys` | List public keys |
| `gpg --list-secret-keys` | List your private keys |
| `gpg --export -a "Name" > public.key` | Export public key |
| `gpg --import public.key` | Import someone's public key |
| `gpg --encrypt --recipient "Name" file.txt` | Encrypt for someone |
| `gpg --decrypt file.txt.gpg` | Decrypt a file |
| `gpg --sign file.txt` | Sign a file |
| `gpg --verify file.txt.gpg` | Verify a signature |

---

## SSH Key Anatomy

```
~/.ssh/
├── id_ed25519          ← Private key (NEVER share!)
├── id_ed25519.pub      ← Public key (safe to share)
├── known_hosts         ← Server fingerprints you've accepted
└── authorized_keys     ← (on server) Public keys allowed to connect
```

| Command | What It Does |
|---------|-------------|
| `ssh-keygen -t ed25519` | Generate Ed25519 key pair |
| `ssh-keygen -l -f ~/.ssh/id_ed25519.pub` | Show key fingerprint |
| `ssh-keygen -lv -f ~/.ssh/id_ed25519.pub` | Show fingerprint + visual art |
| `cat ~/.ssh/id_ed25519.pub` | View your public key |

---

## Linux Security Basics

| Command | What It Does |
|---------|-------------|
| `ls -la` | Show file permissions |
| `chmod 600 file` | Owner read+write only |
| `chmod 755 file` | Owner rwx, group+others rx |
| `chmod +x script.sh` | Add execute permission |
| `chown user:group file` | Change file owner |
| `sudo command` | Run as root |
| `ss -tlnp` | Show listening TCP ports |
| `ss -ulnp` | Show listening UDP ports |
| `who` | Show logged-in users |
| `last` | Show recent logins |

### Permission Numbers

```
r = 4    w = 2    x = 1

chmod 754 file
  │││
  ││└── others: r+x (4+1=5)... wait:
  │└─── group:  r+x (4+1=5)
  └──── owner:  rwx (4+2+1=7)

Common patterns:
  600 = owner read+write (private keys!)
  644 = owner rw, others read
  700 = owner only, full access
  755 = owner rwx, others read+execute
```

---

## Remember

1. **Symmetric** = one key, fast, for bulk data (AES)
2. **Asymmetric** = key pair, slower, for key exchange and signing (RSA, Ed25519)
3. **TLS** uses asymmetric to exchange a symmetric key, then symmetric for speed
4. **Hashing** is NOT encryption — it's one-way and irreversible
5. **Your SSH private key** is an asymmetric private key — never share it, `chmod 600`
6. **Digital signatures** = sign with private key, verify with public key (reverse of encryption)
7. **File permissions** are your first line of defense on Linux — get them right
