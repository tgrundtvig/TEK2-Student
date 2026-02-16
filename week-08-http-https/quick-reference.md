# Week 8 Quick Reference Card

Print this page or keep it open while working!

---

## HTTP Methods

| Method | Purpose | Has Body? | Example |
|--------|---------|-----------|---------|
| **GET** | Read/retrieve | No | Load a web page, fetch data |
| **POST** | Create | Yes | Submit form, create resource |
| **PUT** | Update/replace | Yes | Update user profile |
| **DELETE** | Remove | No | Delete an account |

CRUD mapping: **C**reate=POST, **R**ead=GET, **U**pdate=PUT, **D**elete=DELETE

---

## HTTP Status Codes

| Code | Name | Meaning |
|------|------|---------|
| **200** | OK | Success |
| **201** | Created | Resource created (after POST) |
| **301** | Moved Permanently | Redirect to new URL |
| **302** | Found | Temporary redirect |
| **400** | Bad Request | Malformed request |
| **401** | Unauthorized | Not logged in |
| **403** | Forbidden | Logged in but no permission |
| **404** | Not Found | URL doesn't exist |
| **500** | Internal Server Error | Server bug |

Families: 2xx = success, 3xx = redirect, 4xx = your fault, 5xx = server's fault

---

## Common HTTP Headers

### Request Headers (Client → Server)

| Header | Example | Purpose |
|--------|---------|---------|
| `Host` | `Host: example.com` | Which site you want |
| `User-Agent` | `User-Agent: Mozilla/5.0` | Your browser/tool |
| `Accept` | `Accept: text/html` | What formats you accept |
| `Content-Type` | `Content-Type: application/json` | Format of your request body |
| `Cookie` | `Cookie: session_id=abc123` | Stored data sent to server |

### Response Headers (Server → Client)

| Header | Example | Purpose |
|--------|---------|---------|
| `Content-Type` | `Content-Type: text/html` | Format of response body |
| `Content-Length` | `Content-Length: 1256` | Size of response in bytes |
| `Set-Cookie` | `Set-Cookie: session_id=abc123` | Ask browser to store a cookie |
| `Location` | `Location: https://example.com` | Redirect destination (with 3xx) |

---

## curl Commands

| Command | What It Does |
|---------|-------------|
| `curl URL` | GET request, show response body |
| `curl -v URL` | Verbose — show full request + response |
| `curl -I URL` | HEAD request — show headers only |
| `curl -X POST URL` | Send a POST request |
| `curl -X PUT URL` | Send a PUT request |
| `curl -X DELETE URL` | Send a DELETE request |
| `curl -d "data" URL` | Send data in request body (implies POST) |
| `curl -H "Key: Value" URL` | Add a custom header |
| `curl -L URL` | Follow redirects |
| `curl -b "name=value" URL` | Send a cookie |

### Common Combinations

```bash
# POST JSON data
curl -X POST -H "Content-Type: application/json" -d '{"name":"Alice"}' URL

# POST form data
curl -d "username=alice&password=secret" URL

# See headers + follow redirects
curl -v -L URL
```

---

## httpbin Endpoints (for exercises)

| Endpoint | Method | What It Does |
|----------|--------|-------------|
| `/get` | GET | Returns details about your GET request |
| `/post` | POST | Returns details about your POST request |
| `/put` | PUT | Returns details about your PUT request |
| `/delete` | DELETE | Returns details about your DELETE request |
| `/status/:code` | any | Returns the specified status code |
| `/headers` | GET | Returns the headers you sent |
| `/cookies/set?name=value` | GET | Sets a cookie |
| `/cookies` | GET | Returns cookies you sent |
| `/html` | GET | Returns HTML content |
| `/ip` | GET | Returns your IP address |

---

## Wireshark HTTP Filters

| Filter | What It Shows |
|--------|--------------|
| `http` | All HTTP traffic |
| `http.request` | HTTP requests only |
| `http.response` | HTTP responses only |
| `http.request.method == "GET"` | Only GET requests |
| `http.request.method == "POST"` | Only POST requests |
| `http.response.code == 200` | Only 200 OK responses |
| `http.response.code >= 400` | Only error responses |
| `http.host == "localhost"` | Traffic to a specific host |
| `tcp.port == 8080` | All traffic on port 8080 |
| `tls` | TLS/HTTPS handshake traffic |
| `tls.handshake` | TLS handshake packets only |

### Follow a Conversation

Right-click any HTTP packet → **Follow** → **TCP Stream** to see the full request/response as readable text.

---

## Browser Developer Tools

| Action | Shortcut |
|--------|----------|
| Open dev tools | **F12** or **Ctrl+Shift+I** |
| Go to Network tab | Click "Network" after opening |
| Clear requests | **Ctrl+L** or click clear button |
| Filter requests | Use the filter bar (e.g., type "fetch" or "xhr") |

### What to Look For in Network Tab

1. **Status column**: 200, 301, 404, etc.
2. **Type column**: document, script, stylesheet, image, xhr/fetch
3. **Size column**: How large each response is
4. Click a request → **Headers** tab → see request/response headers
5. Click a request → **Response** tab → see the response body
6. Click a request → **Cookies** tab → see cookies sent and received

---

## URL Structure

```
https://www.example.com:443/search?q=docker&lang=en
│       │               │   │      │
scheme  host            port path   query parameters
```

Default ports (omitted from URLs):
- `http` → port 80
- `https` → port 443

---

## HTTP vs HTTPS in Wireshark

| | HTTP | HTTPS |
|--|------|-------|
| Port | 80 | 443 |
| TCP handshake | Visible | Visible |
| TLS handshake | N/A | Visible (ClientHello/ServerHello) |
| Request/response | **Readable** | **Encrypted** |
| Passwords | **Exposed!** | Protected |

---

## Remember

1. **HTTP is plain text** — you can read it in Wireshark
2. **HTTPS encrypts** the HTTP content — Wireshark sees only noise
3. **GET** reads data (in URL), **POST** creates data (in body)
4. **Status codes**: 2xx good, 4xx your fault, 5xx server's fault
5. **Cookies** are just headers (`Set-Cookie` / `Cookie`)
6. **Browser dev tools (F12)** show everything your browser sends and receives
7. **`curl -v`** shows the full HTTP conversation in your terminal
