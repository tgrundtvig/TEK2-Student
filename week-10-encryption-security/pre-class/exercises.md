# Pre-Class Exercises: Encryption + Security

**Estimated time: 45-60 minutes**

Work through these exercises before class. They give you hands-on experience with the concepts from the reading. If you get stuck on any exercise for more than 10 minutes, note where you had problems and move on.

---

## Exercise 1: Symmetric Encryption with OpenSSL

**Goal:** Encrypt and decrypt a file using AES symmetric encryption.

### 1.1 Create a test file

Create a simple text file to work with:

```bash
echo "This is a secret message for Week 10" > secret.txt
cat secret.txt
```

### 1.2 Encrypt the file with AES-256

```bash
openssl enc -aes-256-cbc -salt -pbkdf2 -in secret.txt -out secret.enc
```

You'll be prompted for a password. Choose something simple for this exercise (e.g., `test123`). You'll need to type it twice.

### 1.3 Look at the encrypted file

```bash
cat secret.enc
```

You should see binary noise — unreadable characters. The file is encrypted. You can also compare sizes:

```bash
ls -la secret.txt secret.enc
```

### 1.4 Decrypt the file

```bash
openssl enc -aes-256-cbc -d -pbkdf2 -in secret.enc -out decrypted.txt
```

Enter the same password you used to encrypt. Then verify:

```bash
cat decrypted.txt
```

You should see your original message: `This is a secret message for Week 10`

### 1.5 Try the wrong password

```bash
openssl enc -aes-256-cbc -d -pbkdf2 -in secret.enc -out wrong.txt
```

Enter a wrong password. You should see an error like:

```
bad decrypt
```

This confirms: **without the right key, the data is unrecoverable**.

### Self-check

- [ ] You can encrypt a file with `openssl enc`
- [ ] The encrypted file is unreadable binary noise
- [ ] Decryption with the correct password restores the original
- [ ] Decryption with a wrong password fails

---

## Exercise 2: Asymmetric Encryption with RSA

**Goal:** Generate an RSA key pair and use it to encrypt and decrypt a message.

### 2.1 Generate a private key

```bash
openssl genrsa -out private.pem 2048
```

You should see output like:

```
Generating RSA private key, 2048 bit long modulus
..........+++++
..+++++
e is 65537 (0x010001)
```

### 2.2 Extract the public key

```bash
openssl rsa -in private.pem -pubout -out public.pem
```

### 2.3 Look at both keys

```bash
cat private.pem
```

You'll see a long block starting with `-----BEGIN RSA PRIVATE KEY-----`. This is your private key — in a real scenario, you'd never share this.

```bash
cat public.pem
```

A shorter block starting with `-----BEGIN PUBLIC KEY-----`. This one is safe to share.

### 2.4 Encrypt a message with the public key

First, create a short message (RSA can only encrypt small amounts of data directly):

```bash
echo "Hello, this is encrypted with RSA!" > message.txt
openssl rsautl -encrypt -pubin -inkey public.pem -in message.txt -out message.enc
```

### 2.5 Verify it's encrypted

```bash
xxd message.enc | head -5
```

You should see hex bytes — completely unreadable.

### 2.6 Decrypt with the private key

```bash
openssl rsautl -decrypt -inkey private.pem -in message.enc -out message.dec
cat message.dec
```

You should see: `Hello, this is encrypted with RSA!`

### 2.7 Try decrypting with the public key (it should fail)

```bash
openssl rsautl -decrypt -pubin -inkey public.pem -in message.enc
```

This will fail. **Only the private key can decrypt what the public key encrypted.**

### Self-check

- [ ] You generated an RSA key pair (private.pem and public.pem)
- [ ] You encrypted a message with the public key
- [ ] You decrypted it with the private key
- [ ] Decrypting with the public key failed — confirming the asymmetric principle

---

## Exercise 3: Hashing and the Avalanche Effect

**Goal:** See how hashing works and why even a tiny change completely changes the hash.

### 3.1 Hash a string

```bash
echo -n "Hello" | sha256sum
```

You should see:

```
185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969  -
```

The `-n` flag on `echo` prevents a trailing newline. Without it, you'd get a different hash (because the input is different!).

### 3.2 Hash a slightly different string

```bash
echo -n "Hello!" | sha256sum
```

You should see a completely different hash:

```
334d016f755cd6dc58c53a86e183882f8ec14f52fb05345887c8a5edd42c87b7  -
```

One character changed (`!` added), but the hash is totally different. This is the **avalanche effect**.

### 3.3 Hash a file

```bash
sha256sum secret.txt
```

This gives you a hash of the file you created in Exercise 1.

### 3.4 Modify the file and hash again

```bash
echo "This is a secret message for Week 10!" > secret2.txt
sha256sum secret.txt secret2.txt
```

Compare the two hashes. They should be completely different, even though the messages differ by just one character (the `!`).

### 3.5 Verify a file hasn't changed

```bash
sha256sum secret.txt > checksum.txt
cat checksum.txt
sha256sum -c checksum.txt
```

You should see:

```
secret.txt: OK
```

This is how downloaded files are verified — the publisher provides a hash, you compute it locally, and compare.

### Self-check

- [ ] You can hash a string with `sha256sum`
- [ ] You saw the avalanche effect — one character changes the entire hash
- [ ] You understand that hashing is one-way (you can't "un-hash")
- [ ] You verified a file's integrity using a checksum

---

## Exercise 4: Examining Your SSH Keys

**Goal:** Look at the SSH keys you created in Week 4 with new understanding.

### 4.1 Check your SSH key files

```bash
ls -la ~/.ssh/
```

You should see at least:
- `id_ed25519` — your private key
- `id_ed25519.pub` — your public key

If you don't have SSH keys, generate them:

```bash
ssh-keygen -t ed25519
```

### 4.2 Look at your public key

```bash
cat ~/.ssh/id_ed25519.pub
```

You'll see something like:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... user@hostname
```

This is the key that goes in `~/.ssh/authorized_keys` on your server.

### 4.3 Check the key fingerprint

```bash
ssh-keygen -l -f ~/.ssh/id_ed25519.pub
```

You should see something like:

```
256 SHA256:AbCdEfGhIjKlMnOpQrStUvWxYz user@hostname (ED25519)
```

This fingerprint is a **hash** of your public key. When you first connected to your server in Week 4 and saw "The authenticity of host... can't be established. ED25519 key fingerprint is SHA256:...", that was the server's public key fingerprint.

### 4.4 Check file permissions

```bash
ls -la ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub
```

You should see:
- Private key: `-rw-------` (600) — only you can read it
- Public key: `-rw-r--r--` (644) — anyone can read it

**This is security in action:** the private key must be `600` (owner read+write only). SSH will actually refuse to use a private key that has too-open permissions.

### Self-check

- [ ] You understand that your SSH key pair is an asymmetric key pair
- [ ] The public key goes on the server, the private key stays on your machine
- [ ] The key fingerprint is a hash of the public key
- [ ] The private key has strict file permissions (600)

---

## Clean Up

Remove the test files you created:

```bash
rm -f secret.txt secret.enc decrypted.txt wrong.txt secret2.txt checksum.txt
rm -f message.txt message.enc message.dec
rm -f private.pem public.pem
```

---

## Before Class Checklist

**Tools Ready:**
- [ ] `openssl` works (you used it in exercises above)
- [ ] `sha256sum` works (or `shasum -a 256` on macOS)
- [ ] Docker installed and running
- [ ] You have SSH keys in `~/.ssh/`

**Knowledge:**
- [ ] You understand symmetric vs asymmetric encryption
- [ ] You know how TLS uses both (asymmetric handshake, then symmetric data transfer)
- [ ] You understand that hashing is one-way — not encryption
- [ ] You can connect SSH keys to asymmetric cryptography

---

## Troubleshooting

**`openssl: command not found`** — Install OpenSSL. On Ubuntu/Debian: `sudo apt install openssl`. On macOS, it's pre-installed. On Windows, use Git Bash (which includes OpenSSL) or WSL.

**`sha256sum: command not found`** — On macOS, use `shasum -a 256` instead. On Windows, use Git Bash or WSL.

**`openssl rsautl` gives a deprecation warning** — Newer versions of OpenSSL prefer `openssl pkeyutl`. The commands still work, but if you see a warning, it's safe to ignore for this exercise.

**SSH key has wrong permissions** — Fix with: `chmod 600 ~/.ssh/id_ed25519`
