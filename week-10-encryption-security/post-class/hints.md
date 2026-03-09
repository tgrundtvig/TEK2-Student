# Hints for Advanced Tasks

Try to work through each task on your own first! These hints are here if you get stuck.

---

## Task 1: GPG-Signed Git Commits

<details>
<summary>Hint 1: Finding your key ID</summary>

The key ID is the hexadecimal string on the `sec` line. It looks like this:

```
sec   rsa2048/ABCDEF1234567890 2025-03-09 [SC]
```

The part after the `/` is your key ID: `ABCDEF1234567890`. Copy this value and use it with `git config --global user.signingkey`.
</details>

<details>
<summary>Hint 2: GPG asks for passphrase but nothing happens</summary>

GPG needs a way to ask for your passphrase. If you're using a terminal without a GUI, you may need to set the GPG_TTY variable:

```bash
export GPG_TTY=$(tty)
```

Add this to your `~/.bashrc` to make it permanent. If you're using Git from an IDE, you may need `pinentry-mac` (macOS) or `pinentry-gnome3` (Linux) for the passphrase prompt to appear.
</details>

<details>
<summary>Hint 3: Commit says "error: gpg failed to sign the data"</summary>

This usually means Git can't find your GPG key or the key ID is wrong. Double-check:

```bash
gpg --list-secret-keys --keyid-format=long
git config --global user.signingkey
```

Make sure the key ID matches. Also verify that the email in your GPG key matches your Git email:

```bash
git config --global user.email
```
</details>

---

## Task 2: SSH Hardening

<details>
<summary>Hint 1: Where to find sshd_config</summary>

The SSH server configuration is at `/etc/ssh/sshd_config`. To see only active settings (no comments or blank lines):

```bash
grep -v "^#" /etc/ssh/sshd_config | grep -v "^$"
```

Many settings are commented out, which means they use the default value. Check `man sshd_config` to see what the defaults are.
</details>

<details>
<summary>Hint 2: Be careful disabling password authentication</summary>

Before setting `PasswordAuthentication no`, make absolutely sure your SSH key authentication works! If you disable passwords and your key doesn't work, you'll be locked out of your server.

Test in a separate terminal first:
```bash
ssh -i ~/.ssh/id_ed25519 user@your-server
```

If that works, you're safe to disable password auth.
</details>

<details>
<summary>Hint 3: No auth.log in Docker container</summary>

Docker containers don't have systemd logging by default. If you're practicing in Docker, you may not see `/var/log/auth.log`. This is expected — the auth.log exercise is most meaningful on your real cloud server from Week 4.
</details>

---

## Task 3: Hashing in the Real World

<details>
<summary>Hint 1: Understanding /etc/shadow format</summary>

The shadow file format is:

```
username:$algorithm$salt$hash:...
```

Algorithm identifiers:
- `$1$` = MD5 (old, weak)
- `$5$` = SHA-256
- `$6$` = SHA-512 (most common on modern Linux)

The salt is a random string added before hashing. Even if two users have the same password, different salts produce different hashes.
</details>

<details>
<summary>Hint 2: Why salting matters</summary>

Without salt, attackers could build a "rainbow table" — a pre-computed dictionary of password → hash. With salt, each password has a unique hash, so rainbow tables are useless. The attacker would need a separate rainbow table for every possible salt value.
</details>

---

## Task 4: Certificate Pinning and Security Headers

<details>
<summary>Hint 1: Not seeing security headers</summary>

Some sites redirect before sending security headers. Follow redirects with:

```bash
curl -sIL https://example.com | grep -i "strict-transport"
```

The `-L` flag follows redirects. Also note that HSTS headers are only sent over HTTPS — you won't see them on HTTP responses.
</details>

<details>
<summary>Hint 2: Understanding HSTS max-age</summary>

The `max-age` value is in seconds:
- 31536000 = 1 year
- 63072000 = 2 years
- 15768000 = 6 months

`includeSubDomains` means the policy applies to all subdomains too. `preload` means the site is included in browsers' built-in HSTS list — the browser knows to use HTTPS even before the first visit.
</details>

<details>
<summary>Hint 3: Certificate Transparency not showing</summary>

Not all certificates show CT extensions in the same way. Some embed the CT proof in the certificate itself (SCT extension), others deliver it during the TLS handshake. If `grep` doesn't find it in the certificate text, the CT proof might be delivered via the TLS handshake instead.
</details>

---

## General Troubleshooting

**`gpg: command not found`** — Install GPG. On Ubuntu/Debian: `sudo apt install gnupg`. On macOS: `brew install gnupg`. On Windows: install [Gpg4win](https://gpg4win.org/).

**`Permission denied` when running chmod** — You need to own the file or be root. Use `sudo chmod` if needed.

**`openssl s_client` hangs** — It's waiting for input. Press Ctrl+C to exit, or pipe from `/dev/null`: `openssl s_client -connect host:443 < /dev/null`.

**Docker commands fail** — Make sure Docker is running. On Linux: `sudo systemctl start docker`. On macOS/Windows: start Docker Desktop.

**`ssh-keygen` shows different key type** — If you see `RSA` instead of `ED25519`, you may have generated an RSA key in Week 4. That's fine — the concepts are the same. Both are asymmetric key pairs.

---

## Still Stuck?

- [ ] Re-read the relevant section in [reading.md](../pre-class/reading.md)
- [ ] Check the [quick-reference.md](../quick-reference.md) for command syntax
- [ ] Search the error message online — Stack Overflow often has the answer
- [ ] Ask in class or on the course forum
