# Hints for Advanced Tasks

Try to solve the tasks on your own first! Use these hints only when you're truly stuck.

---

## Task 1: Full HTTP Conversation Analysis

<details>
<summary>Hint 1: I can't find the HTTP packets on loopback</summary>

Make sure you're capturing on the correct interface:

- **Linux:** Select `lo` (loopback)
- **macOS:** Select `Loopback: lo0`
- **Windows:** Select `Npcap Loopback Adapter` or `Adapter for loopback traffic capture`

If you don't see a loopback interface on Windows, you may need to reinstall Npcap with the "Support raw 802.11 traffic" and "Support loopback traffic" options checked.

Alternatively, you can capture on your regular interface and make requests to httpbin.org instead of localhost.

</details>

<details>
<summary>Hint 2: How to follow a TCP stream</summary>

1. Apply the filter: `tcp.port == 8080`
2. Find any packet in the HTTP conversation
3. Right-click on it → **Follow** → **TCP Stream**
4. A new window opens showing the conversation:
   - **Red text**: Data sent by the client (your HTTP request)
   - **Blue text**: Data sent by the server (the HTTP response)

You can change the display format at the bottom of the stream window (ASCII, hex, etc.).

</details>

<details>
<summary>Hint 3: Counting packets</summary>

Remove all filters to see every packet. A typical HTTP conversation looks like:

```
1. SYN          (client → server)
2. SYN-ACK      (server → client)
3. ACK          (client → server)
4. HTTP GET     (client → server)
5. ACK          (server → client)
6. HTTP 200 OK  (server → client)
7. ACK          (client → server)
8. FIN-ACK      (one side)
9. FIN-ACK      (other side)
10. ACK         (final acknowledgment)
```

That's about 10 packets for a single request/response. The overhead of TCP (handshake + acknowledgments + close) is the price of reliable delivery.

</details>

<details>
<summary>Hint 4: Who initiates the close?</summary>

Filter for `tcp.flags.fin==1` to find FIN packets. The first FIN packet shows who initiated the connection close.

For HTTP/1.1 with `curl`, the server often initiates the close after sending the response. But either side can close first.

</details>

---

## Task 2: HTTP Headers Exploration

<details>
<summary>Hint 1: How to send custom headers with curl</summary>

Use `-H` (can be used multiple times):

```bash
curl -H "Header-Name: Header-Value" -H "Another: Value" URL
```

Common headers to try:
- `User-Agent`: Identifies your client
- `Accept`: What response format you want
- `Accept-Language`: What language you prefer
- `Authorization`: Authentication credentials
- `X-Custom-Header`: Any custom data

</details>

<details>
<summary>Hint 2: Why do browsers send so many headers?</summary>

Browsers automatically send many headers that `curl` doesn't:

- `Accept-Language: en-US,en;q=0.9` — your preferred languages
- `Accept-Encoding: gzip, deflate, br` — compression formats you support
- `Connection: keep-alive` — keep the TCP connection open for more requests
- `Sec-Fetch-Mode: navigate` — security-related metadata
- `Cache-Control` — caching preferences

These headers help servers optimize responses for your specific browser and preferences.

</details>

---

## Task 3: Cookie Tracking Investigation

<details>
<summary>Hint 1: curl cookie flags explained</summary>

- `-c filename` — **Save** cookies to a file (cookies received via `Set-Cookie`)
- `-b filename` — **Send** cookies from a file (sends them via `Cookie` header)
- `-b "name=value"` — Send a specific cookie directly

To both save and send in the same request:
```bash
curl -b cookies.txt -c cookies.txt URL
```

</details>

<details>
<summary>Hint 2: Finding cookies in Wireshark</summary>

Filter for `http` and look through the packets:

- **Set-Cookie** appears in response headers (server → client). Look in the HTTP response packet details under the headers section.
- **Cookie** appears in request headers (client → server). Look in HTTP request packet details.

You can also use the filter: `http.cookie` to find requests that include cookies, or `http.set_cookie` to find responses that set cookies.

</details>

<details>
<summary>Hint 3: What does the cookie file look like?</summary>

The cookie file format (Netscape format) has columns:

```
domain  flag  path  secure  expiration  name  value
```

Example:
```
localhost  FALSE  /  FALSE  0  session_id  user42_authenticated
```

- `domain`: Which site the cookie belongs to
- `path`: Which paths on the site it applies to
- `secure`: Whether it's only sent over HTTPS
- `expiration`: When it expires (0 = session cookie, deleted when browser closes)
- `name` and `value`: The actual cookie data

</details>

---

## Task 4: Comparing Real HTTP and HTTPS Traffic

<details>
<summary>Hint 1: I can't find HTTP packets for the HTTPS request</summary>

That's the whole point! With HTTPS, the HTTP data is encrypted inside TLS. You won't find any `http` packets for HTTPS connections.

What you CAN see:
- TCP handshake (SYN, SYN-ACK, ACK) — filter: `tcp.flags.syn==1`
- TLS handshake (ClientHello, ServerHello) — filter: `tls.handshake`
- Encrypted data packets — filter: `tcp.port == 443`

What you CANNOT see:
- The HTTP method (GET, POST, etc.)
- The URL path
- The request/response headers
- The response body (HTML, JSON, etc.)
- Any data in POST requests (passwords, form data, etc.)

</details>

<details>
<summary>Hint 2: Filling in the comparison table</summary>

Here's what a completed table might look like:

| What | HTTP | HTTPS |
|------|------|-------|
| Destination IP | Visible (e.g., 93.184.216.34) | Visible (same IP) |
| Destination Port | 80 | 443 |
| TCP handshake | Visible | Visible |
| HTTP method (GET) | Visible in Wireshark | Hidden (encrypted) |
| URL path | Visible in Wireshark | Hidden (encrypted) |
| Response HTML | Readable in Wireshark | Hidden (encrypted) |
| TLS handshake | N/A | Visible (ClientHello, ServerHello) |

Key insight: Even with HTTPS, an observer can see **which server** you're connecting to (IP address), but NOT **what you're doing** on that server.

</details>

---

## Task 5: Build Your HTTP Knowledge Document

<details>
<summary>Hint 1: Good analogies for HTTP</summary>

Some analogies that work well:

- **HTTP is like a restaurant**: You (client) look at the menu (URL), place an order (GET request), and the kitchen (server) sends back your food (response). The status code is like the waiter saying "Here you go" (200) or "We don't have that" (404) or "The kitchen caught fire" (500).

- **HTTP is like sending letters**: Each request is a letter with a specific question. Each response is a reply. You have to send a new letter for every question — the server doesn't remember your previous letters (stateless). Cookies are like writing "customer #42" on every letter so the server can look up your history.

</details>

<details>
<summary>Hint 2: Connecting HTTP to your programming courses</summary>

Think about:
- When you build a REST API in Java/Spring, your controller methods handle GET, POST, PUT, DELETE
- When your API returns `ResponseEntity.ok()`, it's sending a 200 status code
- When you add `@RequestBody` to a parameter, you're reading the POST body
- When you add `@RequestParam`, you're reading query parameters
- When your frontend calls `fetch('/api/users')`, it's sending a GET request
- When you handle form submission, the browser sends a POST with the form data

</details>

---

## General Troubleshooting

<details>
<summary>httpbin container is not running</summary>

Start it:
```bash
docker start httpbin
```

Or recreate it:
```bash
docker run -d --name httpbin -p 8080:80 kennethreitz/httpbin
```

If port 8080 is busy:
```bash
docker run -d --name httpbin -p 9090:80 kennethreitz/httpbin
```
Then use port 9090 instead of 8080 in all commands.

</details>

<details>
<summary>Wireshark shows no loopback interface (Windows)</summary>

Windows may not show a loopback interface by default. Options:

1. Reinstall Npcap and check "Support loopback traffic" during installation
2. Alternatively, use your real network interface and make requests to `httpbin.org` instead of `localhost`:
   ```bash
   curl http://httpbin.org/get
   ```
   Then filter in Wireshark for `http.host == "httpbin.org"`

</details>

<details>
<summary>curl on Windows shows different output</summary>

Windows PowerShell has its own `curl` alias that behaves differently. Use one of these:

```powershell
# Use curl.exe explicitly
curl.exe -v http://localhost:8080/get

# Or use Invoke-WebRequest (PowerShell native)
Invoke-WebRequest http://localhost:8080/get
```

For the best experience, use Git Bash or WSL on Windows.

</details>

<details>
<summary>"Connection refused" when using curl</summary>

This means nothing is listening on that port. Check:

1. Is the httpbin container running? `docker ps`
2. Is it mapped to the right port? Look for `0.0.0.0:8080->80/tcp`
3. Are you using the right port in your curl command?

</details>

---

## Still Stuck?

1. **Read the error message carefully** — HTTP errors are usually descriptive
2. **Check that httpbin is running** — `docker ps`
3. **Try the request in your browser first** — if `http://localhost:8080/get` works in the browser, curl should work too
4. **Make sure you're capturing on the right Wireshark interface** — loopback for localhost, Wi-Fi/Ethernet for external sites
5. **Ask in class** — bring your questions to the next session!
