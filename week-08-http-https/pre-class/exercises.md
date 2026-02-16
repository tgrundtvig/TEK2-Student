# Pre-class Exercises: Your First HTTP Requests

**Estimated time: 30-60 minutes**

Complete these exercises before class. If you get stuck, note where you had problems — we'll address them at the start of class.

> **Tip:** If something isn't working, don't spend more than 10 minutes on it. Note the error message and move on — we'll troubleshoot together in class.

---

## Exercise 1: HTTP with curl (~15 minutes)

### Goal

Use `curl` to make HTTP requests and see the full request/response cycle — headers, status codes, and all.

### Step 1.1: A simple GET request

```bash
curl http://example.com
```

This downloads the HTML of example.com and prints it to your terminal. That HTML is the **response body** — the same content your browser would render as a web page.

### Step 1.2: See the full conversation with -v

```bash
curl -v http://example.com
```

The `-v` (verbose) flag shows everything: the TCP connection, the HTTP request your computer sent, and the HTTP response the server returned.

Look for these sections in the output:

```
* Connected to example.com (93.184.216.34) port 80     ← TCP connection
> GET / HTTP/1.1                                         ← Your HTTP request
> Host: example.com                                      ← Request headers
> User-Agent: curl/7.81.0
> Accept: */*
>
< HTTP/1.1 200 OK                                        ← Server's response
< Content-Type: text/html; charset=UTF-8                 ← Response headers
< Content-Length: 1256
<
<!DOCTYPE html>...                                        ← Response body (HTML)
```

Lines starting with `>` are what **you sent**. Lines starting with `<` are what **the server sent back**.

### Step 1.3: See only the headers

```bash
curl -I http://example.com
```

The `-I` flag sends a HEAD request — it asks for only the headers, no body. This is useful for checking status codes and content types without downloading the whole page.

### Step 1.4: Try a URL that doesn't exist

```bash
curl -v http://example.com/this-page-does-not-exist
```

Look for the status code in the response. You should see `404 Not Found`.

### Self-check

- [ ] You can see the HTTP request and response with `curl -v`
- [ ] You can identify the status code in the response (200, 404)
- [ ] You understand that `>` lines are your request and `<` lines are the server's response
- [ ] You can see the request headers (Host, User-Agent, Accept)

---

## Exercise 2: HTTP Status Codes (~10 minutes)

### Goal

See different HTTP status codes in action using `curl`.

### Step 2.1: Redirects

```bash
curl -v http://google.com 2>&1 | grep "< HTTP"
```

You should see a `301 Moved Permanently` — Google redirects from `http://` to `https://`.

Now follow the redirect automatically:

```bash
curl -v -L http://google.com 2>&1 | grep "< HTTP"
```

The `-L` flag tells curl to follow redirects. You should see the `301` followed by a `200 OK`.

### Step 2.2: Different methods

```bash
# GET request (default)
curl -v -X GET http://example.com 2>&1 | grep "< HTTP"

# HEAD request (headers only)
curl -v -X HEAD http://example.com 2>&1 | grep "< HTTP"
```

### Self-check

- [ ] You saw a 301 redirect from http://google.com
- [ ] You used `-L` to follow redirects
- [ ] You understand that `-X` changes the HTTP method

---

## Exercise 3: Browser Developer Tools (~20 minutes)

### Goal

Learn to use the Network tab in your browser's developer tools — a skill you'll use constantly as a web developer.

### Step 3.1: Open developer tools

Open your browser (Chrome, Firefox, or Edge) and press **F12** (or right-click → Inspect). Click the **Network** tab.

### Step 3.2: Capture a page load

1. With the Network tab open, type `http://example.com` in the address bar and press Enter
2. You should see a list of requests appear in the Network tab
3. Click on the first request (`example.com` or `/`)

### Step 3.3: Explore the request details

After clicking the request, you'll see tabs/sections showing:

**Headers tab:**
- **Request URL**: The full URL
- **Request Method**: GET
- **Status Code**: 200 OK
- **Request Headers**: What your browser sent (Host, User-Agent, Accept, Cookie, etc.)
- **Response Headers**: What the server sent back (Content-Type, Set-Cookie, etc.)

**Response/Preview tab:**
- The actual HTML content the server returned

### Step 3.4: Visit a more complex site

1. Clear the Network tab (click the clear button or press Ctrl+L)
2. Visit `https://github.com` or another site you use
3. Watch how many requests appear — each one is a separate HTTP request for HTML, CSS, JavaScript, images, fonts, etc.

**Count the requests.** A modern web page might make 50-100+ HTTP requests to fully load!

### Step 3.5: Find an interesting header

Browse through the request headers on a site you visit regularly. Look for:
- `Cookie` — data being sent to the server
- `Authorization` — authentication tokens
- `Content-Type` — what format the data is in
- `Cache-Control` — how long the browser caches the response

### Self-check

- [ ] You can open the developer tools Network tab (F12)
- [ ] You can see HTTP requests as a page loads
- [ ] You can click a request and see its headers, status code, and response body
- [ ] You noticed how many requests a single web page makes
- [ ] You found at least one interesting header

---

## Exercise 4: Start the httpbin Container (~10 minutes)

### Goal

Pull and start the Docker container we'll use for class exercises. Doing this now saves time in class.

### Step 4.1: Pull the httpbin image

httpbin is a testing service that echoes back everything about the HTTP requests you send it. It's perfect for learning HTTP.

```bash
docker pull kennethreitz/httpbin
```

### Step 4.2: Start the container

```bash
docker run -d --name httpbin -p 8080:80 kennethreitz/httpbin
```

### Step 4.3: Verify it works

Open your browser and go to [http://localhost:8080](http://localhost:8080). You should see the httpbin web interface — a page listing all available API endpoints.

Also test with curl:

```bash
curl http://localhost:8080/get
```

You should see a JSON response showing details about your GET request (headers, IP, etc.).

### Step 4.4: Leave it running

Keep the httpbin container running — we'll use it in class. If you need to stop and start it later:

```bash
# Stop
docker stop httpbin

# Start again
docker start httpbin
```

### Self-check

- [ ] httpbin container is running (`docker ps` shows it)
- [ ] You can access http://localhost:8080 in your browser
- [ ] `curl http://localhost:8080/get` returns a JSON response

---

## Troubleshooting

### curl: command not found

**Windows:** curl is included in Windows 10+. Use PowerShell or CMD. If it doesn't work, try `curl.exe` instead of `curl`.

**Linux:** Install with `sudo apt install curl`

**macOS:** curl is pre-installed.

### "Port 8080 is already in use" when starting httpbin

Another container or application is using port 8080. Either:
- Stop the other container: `docker ps` to find it, then `docker stop <name>`
- Use a different port: `docker run -d --name httpbin -p 9090:80 kennethreitz/httpbin` (then use 9090 instead of 8080)

### Browser developer tools won't open

- **Chrome/Edge:** Press F12, or Ctrl+Shift+I, or right-click → Inspect
- **Firefox:** Press F12, or Ctrl+Shift+I, or Menu → Web Developer → Network
- **Safari:** Enable Developer menu first: Preferences → Advanced → "Show Develop menu"

---

## Before Class Checklist

**Tools:**

- [ ] `curl` works in your terminal
- [ ] Browser developer tools open (F12 → Network tab)
- [ ] httpbin Docker container is running on port 8080

**Knowledge:**

- [ ] You've used `curl -v` to see a full HTTP request/response
- [ ] You can identify status codes (200, 301, 404)
- [ ] You've explored the Network tab in your browser's dev tools
- [ ] You understand the difference between request headers and response headers

**If you're stuck on anything, don't worry!** Note down the specific error or step, and we'll troubleshoot together at the start of class.

---

**Next step**: Bring your laptop to class with Docker running, Wireshark installed, and the httpbin container running!
