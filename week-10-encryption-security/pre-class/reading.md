# Pre-Class Reading: Encryption + Security

**Estimated reading time: 30-45 minutes**

Read through this material before class. You don't need to memorize everything — focus on understanding the concepts. The exercises after will give you hands-on practice.

---

## Part 1: The Problem — Sharing Secrets Over a Public Network

Every time you visit a website, your data travels through many routers and networks. Anyone sitting on those networks could, in theory, read your traffic. You saw this yourself in Week 8 when you captured HTTP traffic in Wireshark — the entire request was readable in plain text.

HTTPS solved the readability problem. But *how* does the encryption actually work? To understand that, we need to start with two fundamental approaches to encryption.

---

## Part 2: Symmetric Encryption — One Key

### The Concept

Symmetric encryption is the oldest and most intuitive form of encryption. The same key is used to both encrypt and decrypt the message.

Think of it like a padlocked box where both the sender and receiver have identical copies of the same key:

```
Alice                                    Bob
  │                                       │
  ├── Has Key "K"                         ├── Has Key "K"
  │                                       │
  ├── Writes message: "Hello Bob"         │
  ├── Locks box with Key K                │
  ├── Sends locked box ──────────────────►│
  │                                       ├── Unlocks box with Key K
  │                                       ├── Reads: "Hello Bob"
```

### How It Works in Practice

The most common symmetric algorithm today is **AES** (Advanced Encryption Standard). When you encrypt a file with AES-256:

1. You provide a key (or a password that gets turned into a key)
2. AES processes the file in blocks of 128 bits
3. Each block goes through multiple rounds of substitution and shuffling
4. The output looks like random noise — but it's reversible with the same key

You don't need to understand the math. What matters is:
- **Same key encrypts and decrypts**
- **AES-256** means a 256-bit key — that's 2²⁵⁶ possible keys (a number with 77 digits)
- **It's fast** — your computer can encrypt gigabytes per second with AES

### The Problem with Symmetric Encryption

There's one huge problem: **how do you share the key?**

If Alice wants to send Bob an encrypted message, they both need the same key. But if Alice sends the key over the network... anyone watching the network can grab it. And then the encryption is useless.

```
Alice                                    Bob
  │                                       │
  ├── Sends key over network ────────────►│  ← Eve can see the key!
  ├── Sends encrypted message ───────────►│  ← Eve can decrypt it!
```

This is called the **key distribution problem**. It plagued cryptography for centuries. The solution came in the 1970s.

---

## Part 3: Asymmetric Encryption — The Key Pair

### The Breakthrough

What if you had two different keys — one that only encrypts, and one that only decrypts? Then you could share the encryption key publicly, and keep the decryption key private. Anyone could send you a secret message, but only you could read it.

This is exactly how **asymmetric encryption** (also called **public-key cryptography**) works:

- **Public key**: share with everyone. Used to *encrypt* messages to you.
- **Private key**: keep secret. Used to *decrypt* messages meant for you.

```
Alice                                    Bob
  │                                       │
  │                                       ├── Generates key pair:
  │                                       │     Public Key (share freely)
  │                                       │     Private Key (keep secret!)
  │                                       │
  │  ◄── Bob sends his PUBLIC key ────────┤  ← Eve can see this, doesn't help her
  │                                       │
  ├── Encrypts "Hello" with Bob's         │
  │   PUBLIC key                          │
  ├── Sends encrypted message ───────────►│  ← Eve sees gibberish
  │                                       ├── Decrypts with PRIVATE key
  │                                       ├── Reads: "Hello"
```

Eve can see Bob's public key and the encrypted message, but she can't decrypt it — only the private key can do that, and Bob never sent that over the network.

### The Math (Simplified)

You don't need to know the math, but the key insight is satisfying: asymmetric encryption relies on **mathematical problems that are easy in one direction but practically impossible to reverse**.

For RSA, this is the factoring problem:
- Easy: multiply two large prime numbers (3,571 × 7,919 = 28,274,749)
- Hard: given 28,274,749, find the two primes that produced it

With the prime numbers used in real RSA keys (hundreds of digits long), no computer on Earth can factor them in a reasonable time.

### The Trade-Off

Asymmetric encryption solves the key distribution problem, but it has a downside: **it's slow**. About 1000x slower than AES for bulk data. You wouldn't want to encrypt a large file download with RSA.

So which do you use? Both. And that's exactly what TLS does.

---

## Part 4: How TLS Uses Both — The Handshake Explained

Remember the TLS handshake you saw in Wireshark (Week 8)? You saw ClientHello and ServerHello messages. Now you can understand what's actually happening inside them:

```
Step 1:  Browser ──► Server
         "ClientHello: I support these encryption methods"

Step 2:  Server ──► Browser
         "ServerHello: Let's use AES-256"
         "Here's my certificate (contains my PUBLIC key)"

Step 3:  Browser verifies certificate chain of trust (Week 9)

Step 4:  Browser generates a random symmetric key (for AES)

Step 5:  Browser encrypts the symmetric key with the server's
         PUBLIC key (asymmetric — only server can decrypt)

Step 6:  Server decrypts with its PRIVATE key
         Now both sides have the same symmetric key!

Step 7:  All HTTP traffic encrypted with symmetric AES key (fast!)
```

**This is why TLS uses both:**
- Asymmetric encryption (RSA) to safely exchange a symmetric key
- Symmetric encryption (AES) for the actual data — because it's 1000x faster

The asymmetric part only happens during the handshake. After that, everything uses the fast symmetric key.

### Connecting to What You've Seen

| What you saw | Week | Now you understand |
|-------------|------|--------------------|
| HTTP traffic readable in Wireshark | 8 | No encryption at all |
| HTTPS traffic shows only "noise" | 8 | Encrypted with symmetric AES key |
| TLS handshake (ClientHello/ServerHello) | 8 | Exchanging the symmetric key using asymmetric encryption |
| Certificate contains a "public key" | 9 | That's the asymmetric public key used in Step 5 |
| Chain of trust (Root CA → Server) | 9 | Ensures you're encrypting to the *real* server's public key |

---

## Part 5: Digital Signatures — Asymmetric in Reverse

There's another powerful use of key pairs: **digital signatures**. Instead of encrypting with the public key, you *sign* with the private key:

```
Signing (private key):
  Message + Private Key ──► Signature

Verifying (public key):
  Message + Signature + Public Key ──► Valid or Invalid
```

Because only the private key holder can create the signature, anyone with the public key can verify:
1. **Who sent it** (authentication) — only the private key holder could sign
2. **It wasn't modified** (integrity) — changing the message invalidates the signature

This is how:
- **TLS certificates** are signed by Certificate Authorities (Week 9)
- **SSH authentication** works (Week 4) — your "cryptographic proof" was a digital signature
- **Git commits** can be signed to prove authorship

Remember in Week 4 when you connected to your server with SSH? The server challenged your client to prove it had the private key. Your SSH client signed a challenge with your private key (`~/.ssh/id_ed25519`), and the server verified it with your public key (`~/.ssh/authorized_keys`). That was a digital signature.

---

## Part 6: Hashing — The One-Way Function

Hashing looks like encryption but is fundamentally different: **there is no key and no way to reverse it**.

A hash function takes any input and produces a fixed-size output (called a **hash**, **digest**, or **fingerprint**):

```
"Hello"         ──► SHA-256 ──► 185f8db32271fe25f561a6fc938b2e26...  (64 hex chars)
"Hello!"        ──► SHA-256 ──► 334d016f755cd6dc58c53a86e183882f...  (completely different!)
A 10 GB video   ──► SHA-256 ──► e3b0c44298fc1c149afbf4c8996fb924...  (still 64 hex chars)
```

Key properties:
- **Fixed output size**: SHA-256 always produces 256 bits (64 hex characters), regardless of input size
- **Deterministic**: Same input always gives the same hash
- **Avalanche effect**: Change one bit of input, and roughly half the output bits change
- **One-way**: You cannot reconstruct the input from the hash
- **Collision-resistant**: It's practically impossible to find two different inputs with the same hash

### What Hashing Is Used For

**Password storage**: Websites don't store your password — they store its hash. When you log in, they hash what you typed and compare. If the database is stolen, attackers get hashes, not passwords.

**File integrity**: Download a file and compute its hash. Compare to the hash published by the author. If they match, the file wasn't tampered with.

**Digital signatures**: You don't sign the entire message — you hash it first, then sign the hash. This is faster and works for any size message.

**Git commits**: Every commit hash you've seen (`521c46c...`) is a SHA-1 hash of the commit content. Change one character, and you get a different commit hash.

### Hashing Is NOT Encryption

This is a common confusion. Here's the difference:

| | Encryption | Hashing |
|---|-----------|---------|
| **Reversible?** | Yes (with the key) | No (one-way) |
| **Key needed?** | Yes | No |
| **Output size** | Same as input | Fixed (e.g., 256 bits) |
| **Purpose** | Confidentiality | Integrity/verification |

---

## Part 7: Putting It All Together

Here's how everything connects across the weeks:

```
Week 4: SSH Keys
  └── Asymmetric key pair (Ed25519)
  └── Authentication via digital signature

Week 8: HTTPS in Wireshark
  └── Saw encrypted traffic (symmetric AES)
  └── Saw TLS handshake messages

Week 9: Certificates
  └── Certificate contains public key (asymmetric)
  └── Chain of trust = chain of digital signatures

Week 10: This Week
  └── Symmetric encryption: AES (the fast one)
  └── Asymmetric encryption: RSA/Ed25519 (the key pair)
  └── TLS handshake: uses asymmetric to exchange symmetric key
  └── Hashing: one-way, for integrity and passwords
  └── Digital signatures: asymmetric in reverse
```

The key insight: **these aren't separate topics — they're all parts of the same system**. Every time you visit an HTTPS website, symmetric encryption, asymmetric encryption, hashing, and digital signatures all work together in the TLS handshake.

---

## Self-Check Questions

Test your understanding before moving on to the exercises.

<details>
<summary>1. Why can't you just use symmetric encryption for everything?</summary>

You need a safe way to share the key. If you send the symmetric key over the network, anyone watching can grab it. Asymmetric encryption solves this by letting you encrypt the key with a public key that's safe to share openly.
</details>

<details>
<summary>2. Why can't you just use asymmetric encryption for everything?</summary>

It's too slow — about 1000x slower than symmetric encryption for bulk data. That's why TLS uses asymmetric only for the handshake (to exchange a symmetric key), then switches to symmetric AES for the actual data.
</details>

<details>
<summary>3. What's the difference between encryption and hashing?</summary>

Encryption is reversible (with the key) — it's for confidentiality. Hashing is one-way (no key) — it's for verification and integrity. You can decrypt ciphertext back to plaintext, but you cannot "un-hash" a hash back to the original input.
</details>

<details>
<summary>4. When you SSH into your server (Week 4), what kind of cryptography proves your identity?</summary>

A digital signature. Your SSH client signs a challenge from the server using your private key (`~/.ssh/id_ed25519`). The server verifies the signature using your public key (from `~/.ssh/authorized_keys`). This is asymmetric cryptography used for authentication.
</details>

<details>
<summary>5. In the TLS handshake, why does the browser generate the symmetric key (not the server)?</summary>

The browser generates a random symmetric key and encrypts it with the server's public key from the certificate. Only the server can decrypt it with its private key. This way, the symmetric key is never sent in the clear — it's protected by asymmetric encryption.
</details>

---

## Further Reading (Optional)

- [How HTTPS Works (comic)](https://howhttps.works/) — a fun illustrated guide to TLS
- [Computerphile: Public Key Cryptography](https://www.youtube.com/watch?v=GSIDS_lvRv4) — clear video explanation
- [Why Passwords Should Be Hashed](https://auth0.com/blog/hashing-passwords-one-way-road-to-security/) — practical password security
