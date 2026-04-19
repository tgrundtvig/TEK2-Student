# Pre-Class Reading: DNS & Certificates

**Estimated time: 45-60 minutes**

> **Video library for this week** — each is self-contained, pick what you need:
>
> 1. 🎬 [What DNS is — the internet's phone book](https://tek2.apps.tobiasgrundtvig.dk/week-09/what-is-dns/) (2:53) — You type google.com, not 142.250.74.46
> 2. 🎬 [The DNS hierarchy — root, TLD, authoritative](https://tek2.apps.tobiasgrundtvig.dk/week-09/dns-hierarchy/) (3:04) — DNS isn't one giant server in a basement somewhere
> 3. 🎬 [DNS record types — A, AAAA, CNAME, MX, NS, TXT](https://tek2.apps.tobiasgrundtvig.dk/week-09/dns-record-types/) (3:34) — DNS doesn't just map a name to one IP
> 4. 🎬 [dig — reading DNS output like a pro](https://tek2.apps.tobiasgrundtvig.dk/week-09/dig-essentials/) (2:46) — dig is the practical tool for inspecting DNS
> 5. 🎬 [DNS TTL and caching — why changes take time](https://tek2.apps.tobiasgrundtvig.dk/week-09/dns-ttl-and-caching/) (3:28) — Every DNS record has a TTL — time to live — that says how long answers can be cached
> 6. 🎬 [TLS certificate anatomy — what's actually inside the padlock](https://tek2.apps.tobiasgrundtvig.dk/week-09/tls-certificate-anatomy/) (3:09) — Click the padlock and you see 'Issued to: github.com'
> 7. 🎬 [The chain of trust — root, intermediate, server](https://tek2.apps.tobiasgrundtvig.dk/week-09/chain-of-trust/) (3:33) — A server cert alone isn't trusted — your browser has never heard of the issuing company
> 8. 🎬 [Let's Encrypt and ACME — free automated certificates](https://tek2.apps.tobiasgrundtvig.dk/week-09/lets-encrypt-and-acme/) (3:44) — Ten years ago, HTTPS certificates cost money and had to be renewed by hand
> 9. 🎬 [The full HTTPS chain — DNS, TCP, TLS, HTTP](https://tek2.apps.tobiasgrundtvig.dk/week-09/full-https-chain/) (4:04) — Type github.com
>
---


## From HTTP to "But How Does It Find the Server?"

In Week 8, you typed `curl http://localhost:8080/get` and saw HTTP requests and responses in plain text. You opened Wireshark and watched your browser talk to a web server. But we always used `localhost` or IP addresses directly.

In real life, you type `google.com` — not `142.250.74.46`. Something has to translate that name into an IP address before any TCP handshake can happen. That something is **DNS**.

And once your browser connects, something has to prove that the server is *really* Google, not someone pretending to be Google. That something is a **certificate**.

This week ties together everything from Weeks 5 and 8: DNS lookup → TCP connection → TLS handshake → HTTP request.

---

## Part 1: DNS — The Internet's Phone Book

### The Problem

Imagine if you had to remember phone numbers for everyone you wanted to call. No contacts app, no search — just raw numbers. That's what the internet would be like without DNS. Every website would require you to memorize an IP address:

- `142.250.74.46` instead of `google.com`
- `140.82.121.4` instead of `github.com`
- `104.16.249.249` instead of `cloudflare.com`

Not practical. So in 1983, Paul Mockapetris invented DNS — a system that translates human-friendly names into machine-friendly IP addresses.

### How DNS Works (The Simple Version)

When you type `github.com` in your browser:

```
Your Browser              DNS Resolver              github.com's IP
    │                         │                          │
    ├── "What's the IP        │                          │
    │    for github.com?" ───►│                          │
    │                         ├── (looks it up) ────────►│
    │                         │◄─── "140.82.121.4" ──────┤
    │◄── "140.82.121.4" ──────┤                          │
    │                         │                          │
    ├── TCP handshake to 140.82.121.4 ──────────────────►│
    ├── TLS handshake ──────────────────────────────────►│
    ├── GET / HTTP/1.1 ─────────────────────────────────►│
```

Your browser asks a **DNS resolver** (usually provided by your ISP or a public service like Google's `8.8.8.8`), and the resolver does the work of finding the answer.

### The DNS Hierarchy

DNS isn't a single giant phone book. It's a distributed system organized like a tree:

```
                    . (root)
                    │
        ┌───────────┼───────────┐
        │           │           │
       com.        dk.        org.
        │           │           │
    ┌───┴───┐      kea.dk    wikipedia.org
    │       │
 google   github
  .com     .com
```

There are three levels of servers involved:

1. **Root servers**: Know where to find the TLD servers (there are 13 root server addresses, run by different organizations worldwide)
2. **TLD servers**: Know where to find nameservers for domains under `.com`, `.dk`, `.org`, etc.
3. **Authoritative nameservers**: Have the actual records for a specific domain (e.g., "github.com is at 140.82.121.4")

### How DNS Resolution Actually Works

When your resolver doesn't have the answer cached, it follows the hierarchy step by step. Let's trace a lookup for `www.github.com`:

```
Step 1: Ask a root server
        "Where can I find info about .com domains?"
        Root says: "Ask the .com TLD server at 192.5.6.30"

Step 2: Ask the .com TLD server
        "Where can I find info about github.com?"
        TLD says: "Ask github.com's nameserver at dns1.p08.nsone.net"

Step 3: Ask github.com's authoritative nameserver
        "What's the IP for www.github.com?"
        Nameserver says: "It's a CNAME to github.com, which is at 140.82.121.4"
```

This is called **recursive resolution** — your resolver walks down the tree until it gets the answer. You can actually watch this happen with `dig +trace` (we'll do this in class).

### DNS Record Types

DNS doesn't just map names to IP addresses. A domain can have many different types of records:

| Record | What It Stores | Example |
|--------|---------------|---------|
| **A** | IPv4 address | `github.com → 140.82.121.4` |
| **AAAA** | IPv6 address | `github.com → 2606:50c0:8000::64` |
| **CNAME** | Alias to another name | `www.github.com → github.com` |
| **MX** | Mail server | `gmail.com → gmail-smtp-in.l.google.com` |
| **NS** | Authoritative nameserver | `github.com → dns1.p08.nsone.net` |
| **TXT** | Free text (verification, SPF) | `"v=spf1 include:_spf.google.com"` |

**CNAME** is particularly important — it's like a redirect. When you look up `www.github.com`, DNS says "that's actually just `github.com`" and then gives you the A record for `github.com`.

**MX** records are how email works. When you send an email to `user@gmail.com`, your mail server looks up the MX record for `gmail.com` to find where to deliver it.

**TXT** records have many uses. Companies put verification codes in TXT records to prove they own a domain. SPF records (a type of TXT record) tell other mail servers which IPs are allowed to send email for that domain.

### DNS Caching and TTL

If your browser had to walk the entire DNS tree for every single request, the internet would be painfully slow. That's why DNS responses include a **TTL** (Time To Live) — a number in seconds that says "you can cache this answer for this long."

```
github.com.    60    IN    A    140.82.121.4
               ^^
               TTL: cache this for 60 seconds
```

Caching happens at multiple levels:

1. **Your browser** caches DNS results
2. **Your operating system** caches DNS results
3. **Your DNS resolver** (ISP or Google/Cloudflare) caches results
4. **The TLD and root servers** are themselves cached

A common TTL is 300 seconds (5 minutes) or 3600 seconds (1 hour). Some sites use very low TTLs (60 seconds) so they can change IPs quickly. Others use high TTLs (86400 = 24 hours) because their IP rarely changes.

**Why this matters**: If you change your server's IP address, the old IP might still be cached worldwide for up to the old TTL. That's why you sometimes hear "DNS changes can take up to 24-48 hours to propagate" — it's really just caches expiring.

---

## Part 2: Certificates — Proving You Are Who You Say You Are

### The Problem

DNS gets you to the right IP address. But how do you know the server at that IP address is actually who it claims to be?

Think about it: when you type `bank.dk` in your browser, DNS translates it to an IP address. But what if:
- Someone hacked the DNS server and pointed `bank.dk` to their own server?
- Someone is sitting on the same Wi-Fi and redirecting your traffic?
- Your ISP is compromised?

Without certificates, you'd have no way to tell the real `bank.dk` from a fake one. Your browser would happily send your password to an imposter.

### What Is a Certificate?

A **TLS certificate** (often just called an "SSL certificate" — the old name) is a digital document that says:

> "I am `github.com`. Here is my public key. This has been verified and signed by DigiCert Inc."

It contains:
- **Subject**: The domain name(s) the certificate covers (e.g., `github.com`, `*.github.com`)
- **Issuer**: Who signed it (a Certificate Authority)
- **Public key**: Used for encryption during the TLS handshake
- **Validity period**: When the certificate expires
- **Signature**: The CA's cryptographic signature proving they issued it

### The Chain of Trust

Here's the clever part. Your browser doesn't just trust any certificate. It verifies a **chain of trust**:

```
Root CA (DigiCert Global Root G2)           ← Pre-installed in your browser/OS
    │                                          Your browser trusts ~100 root CAs
    │ signed
    ▼
Intermediate CA (DigiCert SHA2 Extended)    ← Signed by the root CA
    │
    │ signed
    ▼
Server Certificate (github.com)             ← Signed by the intermediate CA
                                               This is what the server sends you
```

**Why does this work?**

1. Your browser/OS comes with ~100 pre-installed **root CA certificates** from trusted organizations (DigiCert, Let's Encrypt, Comodo, etc.)
2. These root CAs sign **intermediate CA** certificates
3. The intermediate CAs sign **server certificates** for individual websites
4. When a server sends its certificate, your browser walks up the chain to see if it leads to a trusted root

**Why intermediate CAs?** Security. Root CA certificates are incredibly valuable — if one is compromised, thousands of websites are affected. So root CAs stay offline and only sign intermediate certificates. The intermediates handle day-to-day certificate issuance.

### How Your Browser Verifies a Certificate

When you visit `https://github.com`, the TLS handshake (which you saw briefly in Week 8) includes certificate verification:

```
1. Your browser connects to github.com:443
2. Server sends: "Here's my certificate (and the intermediate CA cert)"
3. Browser checks:
   ✓ Is the domain name correct? (certificate says "github.com")
   ✓ Is it expired? (valid until 2025-03-15)
   ✓ Is the intermediate CA signed by a trusted root? (yes, DigiCert root)
   ✓ Has the certificate been revoked? (no)
4. All checks pass → show the padlock icon
5. TLS handshake continues → encrypted connection established
```

If any check fails, your browser shows a scary warning page. You've probably seen one — "Your connection is not private" or "NET::ERR_CERT_AUTHORITY_INVALID". That's your browser saying "I can't verify the chain of trust."

### Let's Encrypt — Free Certificates for Everyone

Until 2015, getting a TLS certificate cost money (typically $50-200/year) and required manual paperwork. Many small websites just used HTTP because the cost and effort weren't worth it.

Then **Let's Encrypt** launched — a free, automated Certificate Authority. It:
- Issues certificates for free
- Automates the process (no paperwork, no waiting)
- Issues certificates valid for 90 days (shorter = more secure)
- Uses the **ACME protocol** for automatic renewal

Let's Encrypt has issued billions of certificates and is a major reason why HTTPS usage went from ~40% of web traffic in 2015 to over 90% today.

**How does Let's Encrypt verify you own a domain?** Through automated challenges:
1. You request a certificate for `example.com`
2. Let's Encrypt says: "Put this specific file at `http://example.com/.well-known/acme-challenge/xyz123`"
3. You put the file there (your ACME client does this automatically)
4. Let's Encrypt fetches the file — if it's there, you control the domain
5. Certificate issued!

This is called the **HTTP-01 challenge**. There's also a **DNS-01 challenge** that asks you to create a specific TXT record — useful for wildcard certificates.

### Self-Signed Certificates

You can create your own certificate without a CA. This is called a **self-signed certificate**. It works for encryption (the traffic is still encrypted), but your browser will show a warning because it can't verify the chain of trust — there's no trusted CA in the chain.

Self-signed certificates are fine for:
- Development and testing
- Internal tools and services
- Learning (like we'll do in exercises)

They're **not** fine for public websites — users would see scary warnings and (rightly) not trust the site.

---

## Putting It All Together

Now you can see the complete chain from Week 5 through Week 9:

```
1. You type "github.com" in your browser

2. DNS Resolution (this week)
   Browser → DNS resolver → root → .com TLD → github.com nameserver
   Answer: 140.82.121.4

3. TCP Handshake (Week 5)
   SYN → SYN-ACK → ACK
   Connection established to 140.82.121.4:443

4. TLS Handshake (this week + Week 8)
   ClientHello → ServerHello + Certificate
   Browser verifies certificate chain of trust
   Keys exchanged → encrypted tunnel ready

5. HTTP Request (Week 8)
   GET / HTTP/1.1 (encrypted inside TLS)
   Server responds with the web page

6. You see github.com in your browser!
```

Every single step here is something you've seen or will see in Wireshark. That's the power of understanding the stack from the bottom up.

---

## Self-Check Questions

Before class, make sure you can answer these:

<details>
<summary>1. What happens if DNS is down?</summary>

You can't resolve domain names to IP addresses, so websites won't load when accessed by name. However, if you know the IP address directly (e.g., `142.250.74.46`), you could still connect. That's why DNS is sometimes called "the internet's single point of failure" — even though the system is heavily distributed and redundant.
</details>

<details>
<summary>2. What's the difference between an A record and a CNAME record?</summary>

An **A record** maps a domain name directly to an IPv4 address (e.g., `github.com → 140.82.121.4`). A **CNAME record** maps a domain name to another domain name — it's an alias (e.g., `www.github.com → github.com`). The CNAME is then resolved to get the actual IP address.
</details>

<details>
<summary>3. Why do we need intermediate CAs? Why not just have root CAs sign everything?</summary>

Root CA private keys are incredibly valuable. If a root CA key is compromised, every certificate it ever signed is at risk. By keeping root CAs offline and using intermediate CAs for daily operations, the damage from a compromise is limited. An intermediate CA can be revoked without affecting the root, and a new intermediate can be created.
</details>

<details>
<summary>4. What does the padlock icon in your browser actually mean?</summary>

It means: (1) the server presented a valid TLS certificate, (2) the certificate was signed by a trusted CA chain, (3) the certificate hasn't expired, and (4) the connection is encrypted. It does **not** mean the website is safe or trustworthy — a phishing site can have a valid certificate too. It only proves the server's identity and encryption.
</details>

<details>
<summary>5. What is TTL and why does it matter?</summary>

TTL (Time To Live) is the number of seconds a DNS response can be cached. A TTL of 3600 means "you can use this answer for 1 hour before asking again." It matters because if you change a server's IP address, the old address stays cached until the TTL expires — which is why DNS changes don't take effect immediately everywhere.
</details>

---

## Further Reading (Optional)

- [How DNS Works (comic)](https://howdns.works/) — visual explanation of DNS resolution
- [Let's Encrypt: How It Works](https://letsencrypt.org/how-it-works/) — the ACME protocol explained
- [What is a TLS Certificate?](https://www.cloudflare.com/learning/ssl/what-is-an-ssl-certificate/) — Cloudflare's explanation
