# Post-class Advanced Tasks: HTTP/HTTPS

**Estimated time: 1-2 hours**

These tasks will deepen your understanding of HTTP. Try to complete them before next week's class.

If you get stuck, check the [hints.md](hints.md) file — but try on your own first!

---

## Task 1: Full HTTP Conversation Analysis (~30 minutes)

### Objective

Capture and analyze a complete HTTP conversation in Wireshark — from TCP handshake to connection close — and document every step.

### Instructions

#### Part A: Capture the conversation

Make sure httpbin is running:

```bash
docker start httpbin 2>/dev/null || docker run -d --name httpbin -p 8080:80 kennethreitz/httpbin
```

1. Open Wireshark and start capturing on the **loopback** interface
2. Run:

```bash
curl http://localhost:8080/get
```

3. Stop the capture

#### Part B: Follow the TCP stream

1. Filter for `tcp.port == 8080`
2. Right-click on any packet → **Follow** → **TCP Stream**
3. In the stream window, you'll see the complete conversation:
   - Your HTTP request (one color)
   - The server's HTTP response (another color)

Copy the contents of this stream — you'll need it for Part C.

#### Part C: Document the conversation

Create a file called `http-analysis.md` and fill in:

**1. TCP handshake:**
- What were the three handshake packets? (SYN, SYN-ACK, ACK)
- What was your source port?
- What was the destination port?

**2. HTTP request:**
- What method was used?
- What path was requested?
- List all request headers

**3. HTTP response:**
- What was the status code?
- What was the Content-Type?
- How large was the response body (Content-Length)?

**4. Connection close:**
- Who initiated the close (client or server)?
- How many FIN/ACK packets were there?

#### Part D: Calculate the total packets

Remove all filters and count the total packets in this conversation:
- Handshake packets (3)
- HTTP request packets (1+)
- HTTP response packets (1+)
- Close packets (2-4)

How many packets does it take just to make one simple HTTP request?

### Verify Success

- [ ] You followed a TCP stream and read the HTTP conversation
- [ ] You documented the handshake, request, response, and close
- [ ] You counted the total packets in a single HTTP request/response

---

## Task 2: HTTP Headers Exploration (~25 minutes)

### Objective

Understand how HTTP headers control behavior by experimenting with different headers.

### Instructions

#### Part A: Custom User-Agent

```bash
# Pretend to be a mobile browser
curl -H "User-Agent: Mobile-Phone/1.0" http://localhost:8080/headers

# Pretend to be a bot
curl -H "User-Agent: MyBot/1.0" http://localhost:8080/headers
```

Look at the `"User-Agent"` field in the response. Many websites serve different content based on this header (e.g., mobile vs desktop layout).

#### Part B: Content negotiation with Accept

```bash
# Ask for JSON
curl -H "Accept: application/json" http://localhost:8080/get

# Ask for HTML
curl -H "Accept: text/html" http://localhost:8080/html
```

The `Accept` header tells the server what format you want. Some APIs return different formats based on this header.

#### Part C: Custom headers

```bash
curl -H "X-Student-Name: Alice" -H "X-Course: TEK2" http://localhost:8080/headers
```

Headers starting with `X-` are custom headers. You can send any headers you want — the server may or may not care about them. httpbin echoes them back so you can see them.

#### Part D: Headers in the browser

1. Open browser dev tools (F12) → Network tab
2. Visit `http://localhost:8080/get`
3. Click the request and look at **Request Headers**
4. Compare these headers with what `curl` sends — the browser sends many more headers (User-Agent, Accept-Language, Accept-Encoding, etc.)

### Verify Success

- [ ] You sent custom headers with `curl -H`
- [ ] You understand that User-Agent identifies the client
- [ ] You compared browser headers vs curl headers
- [ ] You can add custom X- headers to requests

---

## Task 3: Cookie Tracking Investigation (~25 minutes)

### Objective

Understand how cookies enable session tracking by building a multi-step session with httpbin.

### Instructions

#### Part A: Simulate a login session

```bash
# Step 1: "Log in" — server sets a session cookie
curl -c session.txt -L http://localhost:8080/cookies/set?session_id=user42_authenticated

# Step 2: Visit a "protected" page — send the cookie
curl -b session.txt http://localhost:8080/cookies

# Step 3: "Add item to cart" — server adds another cookie
curl -b session.txt -c session.txt -L http://localhost:8080/cookies/set?cart=item_567

# Step 4: Check all cookies
curl -b session.txt http://localhost:8080/cookies
```

The final response should show both cookies:

```json
{
  "cookies": {
    "cart": "item_567",
    "session_id": "user42_authenticated"
  }
}
```

This is how online shopping carts work — cookies track your session and your selections.

#### Part B: Read the cookie file

```bash
cat session.txt
```

Look at the format. Each line represents one cookie with its domain, path, expiration, and value. This is what your browser stores internally.

#### Part C: Capture cookies in Wireshark

1. Start Wireshark on loopback
2. Run the login simulation from Part A again
3. Stop the capture
4. Filter for `http`
5. Find the `Set-Cookie` headers in the responses and the `Cookie` headers in the requests

You can see the entire session tracking mechanism in plain text.

#### Part D: Clean up

```bash
rm -f session.txt
```

### Verify Success

- [ ] You simulated a multi-step session using cookies
- [ ] You examined the cookie file format
- [ ] You saw Set-Cookie and Cookie headers in Wireshark
- [ ] You understand how cookies enable session tracking

---

## Task 4: Comparing Real HTTP and HTTPS Traffic (~20 minutes)

### Objective

Capture traffic to a real website over both HTTP and HTTPS and compare what's visible.

### Instructions

#### Part A: HTTP capture

1. Start Wireshark on your **Wi-Fi or Ethernet** interface (not loopback)
2. Run:

```bash
curl http://example.com
```

3. Stop the capture
4. Filter for `http`
5. Find the GET request and the 200 response
6. Right-click → Follow → TCP Stream
7. You should see the full HTML content of example.com in the stream

#### Part B: HTTPS capture

1. Start a new Wireshark capture on the same interface
2. Run:

```bash
curl https://example.com
```

3. Stop the capture
4. Try filtering for `http` — **no results!** The HTTP is encrypted inside TLS
5. Filter for `tls` — you'll see the TLS handshake
6. Filter for `tcp.port == 443` — you'll see encrypted data packets, but can't read them

#### Part C: Document the difference

Write down what you can see in Wireshark for each:

| What | HTTP | HTTPS |
|------|------|-------|
| Destination IP | | |
| Destination Port | | |
| TCP handshake | | |
| HTTP method (GET) | | |
| URL path | | |
| Response HTML | | |
| TLS handshake | | |

### Verify Success

- [ ] You captured HTTP traffic and read the HTML in Wireshark
- [ ] You captured HTTPS traffic and confirmed you cannot read the content
- [ ] You documented what is visible and what is hidden in each case
- [ ] You understand the practical security difference

---

## Task 5: Build Your HTTP Knowledge Document (~15 minutes)

### Objective

Create a personal reference that connects HTTP concepts to your earlier work and upcoming courses.

### Instructions

Create a file called `http-notes.md` with:

#### 1. HTTP in My Own Words

Explain HTTP to a friend who doesn't know programming. What is it? How does it work? Use your own analogies.

#### 2. Three Things That Now Make Sense

List at least three things from earlier weeks or your programming courses that now make more sense. Examples:
- "Now I understand what happens when my browser loads a page"
- "REST APIs use GET/POST/PUT/DELETE because..."
- "When my app returned a 500 error, it meant..."

#### 3. Questions for Upcoming Weeks

List questions you still have. These will likely be covered:
- Week 9: DNS & Certificates (how does the browser know to trust HTTPS?)
- Week 10: Encryption (how does the encryption actually work?)

### Verify Success

- [ ] You created a personal HTTP reference document
- [ ] You connected HTTP to your programming experience
- [ ] You identified questions for future weeks

---

## Clean Up

Remove the httpbin container and any temporary files:

```bash
docker stop httpbin 2>/dev/null && docker rm httpbin 2>/dev/null
rm -f cookies.txt session.txt
```
