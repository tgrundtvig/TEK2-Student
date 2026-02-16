# Pre-class Reading: HTTP and HTTPS

**Estimated reading time: 45-60 minutes**

---

## What's Inside Those TCP Packets?

In Week 5, you captured TCP traffic in Wireshark. You saw the three-way handshake (SYN → SYN-ACK → ACK), you saw source and destination ports, and you may have noticed some data packets flowing after the handshake.

But what was *in* those data packets?

When you ran `curl http://example.com` in Week 5, here's what actually happened inside that TCP connection:

```
Your computer ──TCP handshake──► example.com:80
Your computer ──"GET / HTTP/1.1\r\nHost: example.com\r\n\r\n"──► example.com
example.com  ──"HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n<html>..."──► Your computer
```

That's **HTTP** — HyperText Transfer Protocol. It's plain text riding on top of TCP. And this week, you'll see every character of it in Wireshark.

---

## What is HTTP?

HTTP is the protocol your browser uses to communicate with web servers. It was designed in 1991, and its simplicity is what made the web possible.

### The Key Idea: Request and Response

HTTP follows a simple pattern:

1. The **client** (your browser or `curl`) sends a **request**
2. The **server** (nginx, Apache, your Spring Boot app) sends back a **response**
3. That's it. One request, one response. Then the conversation is over.

```
┌──────────┐                              ┌──────────┐
│  Client  │── HTTP Request ────────────►│  Server  │
│ (browser)│                              │ (nginx)  │
│          │◄──────────────── HTTP Response│          │
└──────────┘                              └──────────┘
```

Every web page you've ever loaded follows this pattern. When a page has images, CSS, and JavaScript, your browser sends a *separate* HTTP request for each one.

### HTTP is Plain Text

This is the most important thing to understand about HTTP: **it's human-readable text**. Unlike binary protocols, you can read an HTTP request like a letter:

```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
```

And the response:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1256

<!DOCTYPE html>
<html>
  <head><title>Example Domain</title></head>
  <body>
    <h1>Example Domain</h1>
    ...
  </body>
</html>
```

No magic. No binary encoding. Just text. This is what makes HTTP easy to learn, easy to debug, and — as we'll see — easy to spy on (which is why HTTPS exists).

---

## Anatomy of a URL

Before we look at HTTP requests, let's break down the URLs you type every day:

```
https://www.example.com:443/search?q=docker&lang=en
│       │               │   │      │
scheme  host            port path   query parameters
```

| Part | Example | Meaning |
|------|---------|---------|
| **Scheme** | `https` | Which protocol to use (http or https) |
| **Host** | `www.example.com` | Which server to connect to |
| **Port** | `443` | Which port (usually omitted — 80 for HTTP, 443 for HTTPS) |
| **Path** | `/search` | Which resource on the server |
| **Query** | `?q=docker&lang=en` | Extra data sent as key=value pairs |

When you type `http://localhost:8080` in your browser:
- Scheme: `http` (plain text, not encrypted)
- Host: `localhost` (127.0.0.1 — your own computer)
- Port: `8080` (where Docker is listening)
- Path: `/` (the root page, implied when you don't type one)

---

## HTTP Methods — What Do You Want to Do?

Every HTTP request includes a **method** that tells the server what action you want:

| Method | Purpose | Example | Analogy |
|--------|---------|---------|---------|
| **GET** | Read/retrieve data | Load a web page, fetch user list | "Show me the menu" |
| **POST** | Create new data | Submit a form, create a new user | "I'd like to place an order" |
| **PUT** | Update/replace data | Update a user's profile | "Change my order to this instead" |
| **DELETE** | Remove data | Delete a user account | "Cancel my order" |

### GET — The Most Common Method

When you type a URL in your browser and press Enter, it sends a GET request. GET means "give me this resource."

```
GET /api/users HTTP/1.1
Host: myapp.com
```

GET requests:
- Should never change data on the server
- Can be bookmarked and cached
- Send data only through the URL (query parameters)

### POST — Sending Data to the Server

When you submit a login form or create a new account, your browser sends a POST request. The data travels in the **request body**:

```
POST /api/login HTTP/1.1
Host: myapp.com
Content-Type: application/x-www-form-urlencoded

username=alice&password=secret123
```

See that? The username and password are right there in the request body. In plain text. Over HTTP (not HTTPS), anyone capturing the network traffic can read them.

### PUT and DELETE

These work like POST but signal different intentions:

```
PUT /api/users/42 HTTP/1.1
Host: myapp.com
Content-Type: application/json

{"name": "Alice", "email": "alice@example.com"}
```

```
DELETE /api/users/42 HTTP/1.1
Host: myapp.com
```

If you're building REST APIs in your programming courses, you'll use all four methods. They map directly to **CRUD** operations:

| CRUD | HTTP Method |
|------|-------------|
| **C**reate | POST |
| **R**ead | GET |
| **U**pdate | PUT |
| **D**elete | DELETE |

---

## HTTP Status Codes — What Happened?

Every HTTP response starts with a **status code** — a three-digit number that summarizes what happened:

### The Five Families

| Range | Category | Meaning |
|-------|----------|---------|
| **1xx** | Informational | "Hold on, I'm working on it" |
| **2xx** | Success | "Here you go!" |
| **3xx** | Redirection | "It's not here, try over there" |
| **4xx** | Client Error | "You made a mistake" |
| **5xx** | Server Error | "I made a mistake" |

### The Ones You'll See Most Often

| Code | Name | When You See It |
|------|------|-----------------|
| **200** | OK | Everything worked. The response contains what you asked for. |
| **201** | Created | A POST request successfully created a new resource. |
| **301** | Moved Permanently | The page moved to a new URL. Your browser follows automatically. |
| **302** | Found (Redirect) | Temporary redirect. Common after login ("redirect to dashboard"). |
| **400** | Bad Request | Your request was malformed. Check your syntax. |
| **401** | Unauthorized | You need to log in first. |
| **403** | Forbidden | You're logged in, but you don't have permission. |
| **404** | Not Found | The URL doesn't exist. Everyone knows this one. |
| **500** | Internal Server Error | The server crashed. A bug in the server code. |

When you visit a web page and it loads normally, the status code was `200 OK`. You never see it in the browser, but it's there in every response.

---

## HTTP Headers — The Metadata

Every HTTP request and response includes **headers** — lines of metadata that provide extra information. They look like `Key: Value` pairs:

### Request Headers (Client → Server)

```
GET / HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html, application/json
Accept-Language: en-US,en
Cookie: session_id=abc123
```

| Header | Purpose |
|--------|---------|
| `Host` | Which website you want (a server can host multiple sites) |
| `User-Agent` | What browser/tool you're using |
| `Accept` | What content types you can handle |
| `Cookie` | Data the server asked you to store earlier |

### Response Headers (Server → Client)

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1256
Set-Cookie: session_id=abc123; Path=/; HttpOnly
Cache-Control: max-age=3600
```

| Header | Purpose |
|--------|---------|
| `Content-Type` | What format the response is in (HTML, JSON, image, etc.) |
| `Content-Length` | How many bytes the response body contains |
| `Set-Cookie` | Asks the browser to store a cookie |
| `Cache-Control` | How long the browser can cache this response |

You don't need to memorize all headers. The important thing is understanding that HTTP requests and responses are more than just the URL and the body — they carry metadata that controls behavior.

---

## Cookies — How Servers Remember You

HTTP is **stateless** — each request is independent. The server doesn't inherently know that two requests came from the same person. So how does a website keep you logged in?

**Cookies.**

Here's how they work:

```
Step 1: You log in
─────────────────────────────────────────────────────
POST /login HTTP/1.1
Host: myapp.com

username=alice&password=secret123

Step 2: Server responds with a cookie
─────────────────────────────────────────────────────
HTTP/1.1 200 OK
Set-Cookie: session_id=abc123xyz; Path=/

Step 3: Your browser stores the cookie and sends it with EVERY future request
─────────────────────────────────────────────────────
GET /dashboard HTTP/1.1
Host: myapp.com
Cookie: session_id=abc123xyz

Step 4: Server reads the cookie and knows it's Alice
```

Cookies are just HTTP headers. There's no magic — it's text being sent back and forth. You'll see them clearly in Wireshark.

---

## Query Parameters and POST Bodies — How Data Travels

There are two main ways to send data in an HTTP request:

### Query Parameters (in the URL)

```
GET /search?q=docker+tutorial&page=2 HTTP/1.1
Host: google.com
```

- Data is visible in the URL
- Used with GET requests
- Good for search queries, filters, page numbers
- Shows up in browser history and bookmarks
- **Visible in Wireshark** (and in server logs, browser history, bookmarks)

### Request Body (in the body)

```
POST /api/users HTTP/1.1
Host: myapp.com
Content-Type: application/json

{"name": "Alice", "email": "alice@example.com"}
```

- Data is in the body, not the URL
- Used with POST, PUT, and PATCH requests
- Good for forms, login credentials, file uploads
- Doesn't show up in browser history
- **Still visible in Wireshark over HTTP!** (only encrypted with HTTPS)

The content type tells the server how to read the body:

| Content-Type | Format | Example |
|-------------|--------|---------|
| `application/json` | JSON | `{"name": "Alice"}` |
| `application/x-www-form-urlencoded` | Form data | `name=Alice&email=alice%40example.com` |
| `multipart/form-data` | File uploads | (binary + text sections) |

---

## HTTPS — Why Encryption Matters

Everything you've read so far about HTTP has one massive problem: **it's all plain text**. Anyone between you and the server — your Wi-Fi network, your ISP, a hacker on public Wi-Fi — can read every byte.

That includes:
- Your passwords
- Your cookies (which could be used to impersonate you)
- Your personal data
- The content of every page you visit

### What HTTPS Adds

HTTPS is **HTTP over TLS** (Transport Layer Security). It adds an encryption layer between TCP and HTTP:

```
HTTP:   TCP → HTTP (plain text)
HTTPS:  TCP → TLS (encryption) → HTTP (encrypted)
```

With HTTPS:
- The TCP handshake happens normally (SYN, SYN-ACK, ACK — visible in Wireshark)
- Then a **TLS handshake** happens (you can see it in Wireshark, but can't read the content)
- After that, all HTTP data is encrypted — Wireshark shows only gibberish

### What You'll See in Wireshark

| | HTTP (port 80) | HTTPS (port 443) |
|--|----------------|-------------------|
| TCP handshake | Visible | Visible |
| TLS handshake | N/A | Visible (ClientHello, ServerHello) |
| HTTP request | **Readable** (GET /index.html...) | **Encrypted** (unreadable) |
| HTTP response | **Readable** (200 OK, HTML...) | **Encrypted** (unreadable) |
| Passwords | **Visible!** | **Protected** |

In the class exercises, you'll see this difference with your own eyes. It's the single best argument for why HTTPS matters.

### How Does the Encryption Work?

We won't go deep into this now — Week 9 covers certificates (how you trust a server is who it claims to be) and Week 10 covers encryption (how the actual scrambling works). For now, just know:

1. The browser and server agree on an encryption method during the TLS handshake
2. They exchange keys so only they can read the data
3. All HTTP traffic after that is encrypted
4. Even if someone captures the packets, they see only encrypted noise

---

## Putting It All Together

Let's trace a full web request, combining what you learned in Week 5 with this week's HTTP knowledge:

```
Step 1: You type http://myapp.com/users in your browser

Step 2: TCP three-way handshake (Week 5)
        Your IP:54321 ──SYN──► myapp.com:80
        Your IP:54321 ◄──SYN-ACK── myapp.com:80
        Your IP:54321 ──ACK──► myapp.com:80

Step 3: HTTP request (this week!)
        GET /users HTTP/1.1
        Host: myapp.com
        User-Agent: Mozilla/5.0
        Accept: text/html
        Cookie: session_id=abc123

Step 4: HTTP response
        HTTP/1.1 200 OK
        Content-Type: text/html
        Content-Length: 4523

        <html>... user list ...</html>

Step 5: TCP connection close (FIN → ACK → FIN → ACK)
```

The TCP layer handles delivery (reliable, in-order). The HTTP layer handles meaning (what do you want? here's the answer). You'll see both layers in Wireshark, stacked on top of each other.

---

## Self-Check Questions

Before moving to the exercises, make sure you can answer these:

1. What HTTP method does your browser use when you type a URL and press Enter?
2. What's the difference between a 404 and a 500 status code?
3. Where does data travel in a GET request vs a POST request?
4. What is a cookie and why do websites need them?
5. Why can you read HTTP traffic in Wireshark but not HTTPS traffic?

<details>
<summary>Click to reveal answers</summary>

1. **GET** — typing a URL in the browser always sends a GET request.

2. **404** means the client asked for something that doesn't exist (client's mistake — wrong URL). **500** means the server encountered an error while processing (server's bug).

3. In a **GET** request, data travels in the URL as query parameters (`?key=value`). In a **POST** request, data travels in the request body (below the headers).

4. A **cookie** is a small piece of data the server asks your browser to store (via `Set-Cookie` header) and send back with every future request (via `Cookie` header). Websites need them because HTTP is stateless — without cookies, the server wouldn't know you're logged in.

5. **HTTP** sends everything as plain text on top of TCP — Wireshark can read it directly. **HTTPS** encrypts the HTTP data using TLS before sending it over TCP — Wireshark sees only encrypted bytes.

</details>

---

## Further Reading (Optional)

- [MDN: An Overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) — Mozilla's comprehensive HTTP reference
- [HTTP Status Codes](https://httpstatuses.com/) — Complete list with explanations

---

**Next step**: Continue to [exercises.md](exercises.md) to make your first HTTP requests with `curl` and explore browser developer tools.
