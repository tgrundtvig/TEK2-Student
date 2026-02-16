# Class Exercises: HTTP/HTTPS

Work through these exercises during class. Ask for help if you get stuck!

---

## Warm-up: Verify Pre-class Work (~5 minutes)

Quick check that everyone's tools are ready.

### Check httpbin

```bash
docker ps
```

You should see the `httpbin` container running. If not:

```bash
docker start httpbin
```

Or if you never created it:

```bash
docker run -d --name httpbin -p 8080:80 kennethreitz/httpbin
```

Verify with:

```bash
curl http://localhost:8080/get
```

You should see a JSON response.

### Check Wireshark

Open Wireshark — make sure it launches and shows network interfaces.

### Self-check

- [ ] httpbin container is running on port 8080
- [ ] `curl http://localhost:8080/get` returns JSON
- [ ] Wireshark opens and shows interfaces

---

## Exercise 1: See HTTP in Wireshark (~25 minutes)

### Goal

Capture HTTP traffic in Wireshark and read a real HTTP request and response in plain text. This is the "aha moment" — seeing that HTTP is just text.

### Part A: Start a Wireshark capture

1. Open Wireshark
2. Select the **Loopback** interface:

| Platform | Interface Name |
|----------|---------------|
| Linux | `lo` (loopback) |
| macOS | `Loopback: lo0` |
| Windows | `Adapter for loopback traffic capture` or `Npcap Loopback Adapter` |

> **Why loopback?** The httpbin container is on `localhost`. Traffic to `localhost` goes through the loopback interface, not your Wi-Fi or Ethernet card.

3. Double-click the interface to start capturing

### Part B: Make an HTTP request

With Wireshark capturing, run:

```bash
curl http://localhost:8080/get
```

### Part C: Stop and filter

1. Click the **Stop** button (red square) in Wireshark
2. In the filter bar, type: `http` and press Enter

You should see HTTP packets. Look for:
- A packet with `GET /get HTTP/1.1` in the Info column — that's your request
- A packet with `HTTP/1.1 200 OK` — that's the server's response

### Part D: Read the HTTP request

Click on the `GET /get` packet. In the detail pane (middle), expand **Hypertext Transfer Protocol**. You'll see:

```
GET /get HTTP/1.1\r\n
Host: localhost:8080\r\n
User-Agent: curl/7.81.0\r\n
Accept: */*\r\n
\r\n
```

**This is exactly what your computer sent to the server.** Plain text. You can read every header.

### Part E: Read the HTTP response

Click on the `HTTP/1.1 200 OK` packet. Expand **Hypertext Transfer Protocol**. You'll see:

```
HTTP/1.1 200 OK\r\n
Content-Type: application/json\r\n
Content-Length: 250\r\n
...
```

And in the bottom pane, you can see the response body — the JSON data that httpbin returned.

### Part F: Follow the TCP Stream

Right-click on any of the HTTP packets → **Follow** → **TCP Stream**

A window opens showing the **entire HTTP conversation** in readable text — your request in one color, the server's response in another. This is what's inside those TCP packets you saw in Week 5.

### Self-check

- [ ] You captured HTTP traffic on the loopback interface
- [ ] You found the GET request in Wireshark
- [ ] You found the 200 OK response
- [ ] You read the HTTP headers in the packet details
- [ ] You used "Follow TCP Stream" to see the full conversation

---

## Exercise 2: HTTP Methods with the REST API (~30 minutes)

### Goal

Use all four HTTP methods (GET, POST, PUT, DELETE) against the httpbin API and see how each one works.

### Part A: GET request

```bash
curl http://localhost:8080/get
```

The response shows what httpbin received:

```json
{
  "args": {},
  "headers": {
    "Accept": "*/*",
    "Host": "localhost:8080",
    "User-Agent": "curl/7.81.0"
  },
  "url": "http://localhost:8080/get"
}
```

httpbin echoes back your headers, your URL, and any arguments you sent.

### Part B: POST request with JSON

```bash
curl -X POST http://localhost:8080/post \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "role": "student"}'
```

Look at the response. You should see a `"json"` field containing the data you sent:

```json
{
  "json": {
    "name": "Alice",
    "role": "student"
  },
  ...
}
```

### Part C: POST request with form data

```bash
curl -X POST http://localhost:8080/post \
  -d "username=alice&password=secret123"
```

Look at the response. The data appears in the `"form"` field:

```json
{
  "form": {
    "password": "secret123",
    "username": "alice"
  },
  ...
}
```

This is what happens when you submit an HTML form — the data is sent as `key=value` pairs.

### Part D: PUT request

```bash
curl -X PUT http://localhost:8080/put \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice Updated", "role": "admin"}'
```

The response shows the JSON you sent in the `"json"` field. In a real API, this would update an existing resource.

### Part E: DELETE request

```bash
curl -X DELETE http://localhost:8080/delete
```

The response confirms the DELETE was received. In a real API, this would remove a resource.

### Part F: Status codes

httpbin can return any status code you want:

```bash
# 404 Not Found
curl -v http://localhost:8080/status/404 2>&1 | grep "< HTTP"

# 500 Internal Server Error
curl -v http://localhost:8080/status/500 2>&1 | grep "< HTTP"

# 301 Redirect
curl -v http://localhost:8080/status/301 2>&1 | grep "< HTTP"

# 418 I'm a Teapot (yes, this is a real status code!)
curl -v http://localhost:8080/status/418 2>&1 | grep "< HTTP"
```

### Self-check

- [ ] You made GET, POST, PUT, and DELETE requests
- [ ] You sent JSON data in a POST request
- [ ] You sent form data in a POST request
- [ ] You can trigger different status codes
- [ ] You understand the difference between JSON and form-encoded data

---

## Exercise 3: Browser Developer Tools Deep Dive (~20 minutes)

### Goal

Use the browser's Network tab to inspect real HTTP traffic — a skill you'll use daily as a developer.

### Part A: Inspect httpbin in your browser

1. Open your browser and press **F12** to open developer tools
2. Click the **Network** tab
3. Navigate to `http://localhost:8080`
4. Watch the requests appear in the Network tab

### Part B: Examine a request

1. Click on the main request (the first one, usually `localhost` or `/`)
2. Look at the **Headers** tab:
   - Find the **Request Method** (GET)
   - Find the **Status Code** (200)
   - Scroll down to see **Request Headers** and **Response Headers**
3. Click the **Response** tab to see the HTML that was returned

### Part C: Make API calls from the browser

1. Clear the Network tab (Ctrl+L)
2. In your browser's address bar, go to: `http://localhost:8080/get`
3. In the Network tab, click the request
4. Look at the **Response** tab — you'll see the JSON response from httpbin

### Part D: See a redirect

1. Clear the Network tab
2. Navigate to `http://google.com` (not https)
3. In the Network tab, look at the first request — its status should be **301**
4. The second request shows where you were redirected to

### Part E: Count resources on a real page

1. Clear the Network tab
2. Visit a site you use regularly (GitHub, YouTube, etc.)
3. Look at the bottom of the Network tab — it shows the total number of requests and total data transferred
4. Note how many requests a single page load makes

### Self-check

- [ ] You inspected an HTTP request in the browser's Network tab
- [ ] You found the status code, request method, and headers
- [ ] You saw a 301 redirect from http://google.com
- [ ] You counted the requests on a real web page

---

## Exercise 4: Cookies in Action (~20 minutes)

### Goal

See how cookies work — how the server sets them and how the browser sends them back.

### Part A: Set a cookie with httpbin

```bash
curl -v http://localhost:8080/cookies/set?theme=dark 2>&1 | grep -i "set-cookie\|cookie"
```

Look for the `Set-Cookie` header in the response. The server is asking your browser to store `theme=dark`.

> **Note:** `curl` doesn't automatically store cookies by default (browsers do). We'll fix that in Part B.

### Part B: Store and send cookies with curl

```bash
# Set a cookie and save it to a file
curl -c cookies.txt http://localhost:8080/cookies/set?theme=dark -L

# Send the saved cookie back
curl -b cookies.txt http://localhost:8080/cookies
```

The second command should show:

```json
{
  "cookies": {
    "theme": "dark"
  }
}
```

The server set a cookie, `curl` stored it in `cookies.txt`, and then sent it back. This is exactly what your browser does automatically.

### Part C: Multiple cookies

```bash
# Set several cookies
curl -c cookies.txt http://localhost:8080/cookies/set?theme=dark -L
curl -b cookies.txt -c cookies.txt http://localhost:8080/cookies/set?lang=en -L
curl -b cookies.txt -c cookies.txt http://localhost:8080/cookies/set?user=alice -L

# See all cookies
curl -b cookies.txt http://localhost:8080/cookies
```

### Part D: Cookies in the browser

1. Open your browser and go to `http://localhost:8080/cookies/set?session_id=abc123`
2. Open developer tools (F12) → **Application** tab (Chrome/Edge) or **Storage** tab (Firefox)
3. In the left sidebar, click **Cookies** → `http://localhost:8080`
4. You should see the `session_id` cookie listed

Now go to `http://localhost:8080/cookies` — the server reads the cookie your browser sent and displays it.

### Part E: Clean up

```bash
rm -f cookies.txt
```

### Self-check

- [ ] You set a cookie using httpbin's `/cookies/set` endpoint
- [ ] You stored and sent cookies with `curl -c` and `curl -b`
- [ ] You can see cookies in the browser's developer tools
- [ ] You understand that cookies are just HTTP headers (`Set-Cookie` / `Cookie`)

---

## Exercise 5: Query Parameters and POST Bodies (~15 minutes)

### Goal

See the two ways data travels in HTTP requests: in the URL (query parameters) and in the body.

### Part A: Query parameters

```bash
curl "http://localhost:8080/get?search=docker&page=2&sort=date"
```

Look at the `"args"` field in the response:

```json
{
  "args": {
    "page": "2",
    "search": "docker",
    "sort": "date"
  },
  ...
}
```

The query parameters from the URL are parsed and shown. This is how search engines, filters, and pagination work.

### Part B: POST body — form data

```bash
curl -X POST http://localhost:8080/post \
  -d "search=docker&page=2&sort=date"
```

Now the same data appears in the `"form"` field instead of `"args"`. The data traveled in the request body, not the URL.

### Part C: See the difference in Wireshark

1. Start a Wireshark capture on the loopback interface
2. Run both commands:

```bash
curl "http://localhost:8080/get?secret=in-the-url"
curl -X POST http://localhost:8080/post -d "secret=in-the-body"
```

3. Stop the capture and filter for `http`
4. Find both requests:
   - The GET request: you can see `?secret=in-the-url` right in the request line
   - The POST request: the data `secret=in-the-body` is in a separate section below the headers

Both are visible in Wireshark! Over plain HTTP, neither method hides data. Only HTTPS protects the content.

### Self-check

- [ ] You sent data via query parameters (GET)
- [ ] You sent data via request body (POST)
- [ ] You saw both in Wireshark — both are plain text
- [ ] You understand that POST bodies are NOT hidden without HTTPS

---

## Exercise 6: The Security Demo — Passwords in Plain Text (~20 minutes)

### Goal

See why HTTPS matters by capturing a "login" over HTTP and reading the password in Wireshark.

### Part A: Capture the "login"

1. Start a Wireshark capture on the loopback interface
2. Run this command (simulating a login form submission):

```bash
curl -X POST http://localhost:8080/post \
  -d "username=alice&password=SuperSecret123!"
```

3. Stop the Wireshark capture

### Part B: Find the password

1. Filter for `http.request.method == "POST"`
2. Click on the POST request
3. In the packet details, expand **Hypertext Transfer Protocol** and then **HTML Form URL Encoded**
4. You should see:
   - `username=alice`
   - `password=SuperSecret123!`

**There it is.** The password, in plain text, visible to anyone capturing traffic on the network. On public Wi-Fi, a coffee shop, an airport — anyone running Wireshark could see this.

### Part C: Now try HTTPS

1. Start a new Wireshark capture — this time on your **Wi-Fi or Ethernet** interface (not loopback)
2. Run:

```bash
curl -X POST https://httpbin.org/post \
  -d "username=alice&password=SuperSecret123!"
```

> **Note:** This uses `httpbin.org` (the public HTTPS version), not your local container.

3. Stop the capture

### Part D: Try to find the password

1. Filter for `tls` to see the TLS handshake
2. You can see `ClientHello` and `ServerHello` packets — the encryption negotiation
3. Now filter for `http` — **you won't find any HTTP packets!** The HTTP data is encrypted inside TLS
4. Filter for `tcp.port == 443` to see all traffic to the HTTPS server — it's all encrypted gibberish

**The password is protected.** Even though you're on the same machine, you can't read the HTTP data because it's encrypted.

### Part E: The TLS handshake (preview of Week 9-10)

Filter for `tls.handshake` on the HTTPS capture. You'll see:

1. **ClientHello**: Your computer says "I want to connect securely. Here are the encryption methods I support."
2. **ServerHello**: The server says "Let's use this encryption method. Here's my certificate."
3. **Key exchange**: Both sides agree on encryption keys
4. After this, all data is encrypted

Don't worry about understanding the details — that's Weeks 9 and 10. The important thing is: **you can see the handshake, but you can't read the data.**

### Self-check

- [ ] You captured a POST with username/password over HTTP
- [ ] You found the password in plain text in Wireshark
- [ ] You captured the same POST over HTTPS
- [ ] You confirmed that the password is NOT visible over HTTPS
- [ ] You saw the TLS handshake (ClientHello, ServerHello)
- [ ] You understand why HTTPS matters for any site that handles sensitive data

---

## Exercise 7: Putting It All Together (~15 minutes)

### Goal

Capture a complete HTTP session and identify every layer you've learned about.

### Part A: One clean capture

1. Start Wireshark on the loopback interface
2. Run:

```bash
curl http://localhost:8080/html
```

3. Stop the capture

### Part B: Identify the layers

Remove any filters and find the conversation. For each packet, identify:

| What to Find | Where to Look |
|--------------|---------------|
| TCP three-way handshake | First three packets: SYN, SYN-ACK, ACK |
| HTTP GET request | After handshake: `GET /html HTTP/1.1` |
| HTTP 200 response | Server's reply: `HTTP/1.1 200 OK` |
| HTML content | Response body in the last data packet |
| TCP connection close | FIN packets at the end |

### Part C: Count the layers

Click on the HTTP response packet. In the detail pane, count the layers from bottom to top:

```
Ethernet / Loopback      (Layer 2 — physical delivery)
Internet Protocol v4     (Layer 3 — IP addresses, routing)
Transmission Control     (Layer 4 — ports, reliable delivery)
Hypertext Transfer       (Layer 7 — HTTP request/response)
```

This is the network stack in action. Each layer adds its own information, and Wireshark shows you all of them.

### Self-check

- [ ] You identified the TCP handshake, HTTP request, HTTP response, and TCP close
- [ ] You can see all four layers in a single packet
- [ ] You understand how HTTP sits on top of TCP, which sits on top of IP

---

## Final Cleanup

Remove the httpbin container:

```bash
docker stop httpbin && docker rm httpbin
```

> **Note:** If you want to keep it for the post-class exercises, leave it running!

---

## Summary

| Skill | What You Practiced | Connection to Earlier Weeks |
|-------|-------------------|---------------------------|
| Wireshark HTTP capture | Read HTTP requests/responses in Wireshark | Week 5: TCP handshake, now + HTTP content |
| curl with methods | GET, POST, PUT, DELETE against a REST API | Programming courses: building REST APIs |
| Browser dev tools | Inspect network traffic, headers, cookies | Every web development task from now on |
| Cookies | Set, send, and inspect cookies | Authentication in web applications |
| Query params & POST bodies | Two ways data travels in HTTP | Forms, search, API design |
| HTTP vs HTTPS | Plaintext vs encrypted in Wireshark | Security for every web application |

**Key takeaways:**

1. HTTP is **plain text** — you can read every byte in Wireshark
2. HTTP methods (GET/POST/PUT/DELETE) map to CRUD operations
3. Cookies are just headers — `Set-Cookie` creates them, `Cookie` sends them back
4. Browser developer tools (F12 → Network) show everything your browser does
5. Over HTTP, passwords and data are **visible** to anyone on the network
6. HTTPS encrypts everything — Wireshark sees only the TLS handshake, not the HTTP data
7. The network layers stack: Ethernet → IP → TCP → HTTP (visible in Wireshark)
