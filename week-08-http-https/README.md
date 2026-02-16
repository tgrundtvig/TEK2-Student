# Week 8: HTTP/HTTPS

## Overview

In Week 5, you looked at network traffic in Wireshark and saw TCP handshakes, IP addresses, and ports. But we never looked at what's actually *inside* those TCP packets. This week, we open them up.

Every time you visit a website, submit a form, or call an API, your browser is speaking **HTTP** — a simple, text-based protocol that rides on top of TCP. And the amazing part? It's all readable plain text. You'll see it with your own eyes in Wireshark.

Then we'll look at **HTTPS** — the encrypted version — and see why it matters.

## Learning Objectives

By the end of this week, you should be able to:

1. **Explain** how HTTP works as a request/response protocol on top of TCP
2. **Use** `curl` to make HTTP requests and inspect headers, methods, and status codes
3. **Identify** HTTP traffic in Wireshark and read requests/responses in plaintext
4. **Compare** HTTP and HTTPS traffic in Wireshark and explain why encryption matters
5. **Use** browser developer tools to inspect network traffic
6. **Demonstrate** how cookies, query parameters, and POST bodies travel over HTTP

## Time Allocation

| Phase | Duration | Focus |
|-------|----------|-------|
| Pre-class | 1-2 hours | Reading about HTTP + `curl` and browser dev tools exploration |
| Class | 3.5 hours | Wireshark HTTP capture, REST API exercises, HTTPS comparison, security demo |
| Post-class | 1-2 hours | Advanced Wireshark analysis + deeper HTTP exploration |

## Prerequisites

- Completed Week 5 (Networking Basics — TCP, ports, Wireshark installation)
- Wireshark installed and working
- Docker installed and running

## Materials

| File | Description |
|------|-------------|
| [quick-reference.md](quick-reference.md) | One-page cheat sheet (print this!) |
| [pre-class/reading.md](pre-class/reading.md) | HTTP protocol, methods, status codes, headers |
| [pre-class/exercises.md](pre-class/exercises.md) | Hands-on with `curl` and browser developer tools |
| [class/exercises.md](class/exercises.md) | Wireshark HTTP analysis, REST API, HTTPS comparison |
| [post-class/advanced-tasks.md](post-class/advanced-tasks.md) | Deep HTTP analysis challenges |
| [post-class/hints.md](post-class/hints.md) | Hints for advanced tasks |

## Key Concepts

- **HTTP**: HyperText Transfer Protocol — a text-based protocol for requesting and delivering web content
- **HTTPS**: HTTP over TLS — the same protocol, but encrypted so nobody can read it in transit
- **HTTP Method**: What you want to do (GET = read, POST = create, PUT = update, DELETE = remove)
- **Status Code**: The server's response summary (200 = OK, 404 = not found, 500 = server error)
- **Header**: Metadata sent with requests and responses (content type, cookies, authentication)
- **Cookie**: A small piece of data the server asks your browser to store and send back with future requests

## What's Next

In Week 9, we'll explore DNS (how `google.com` gets translated to an IP address) and certificates (the trust system that makes HTTPS work). You'll understand the full chain: DNS lookup → TCP connection → TLS handshake → HTTP request.
