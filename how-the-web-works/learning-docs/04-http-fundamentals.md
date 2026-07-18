# Chapter 4: HTTP Fundamentals — The Language of the Web

## Overview

Once DNS has found the server and TCP has connected to it, client and server need a shared language. That language is **HTTP** (HyperText Transfer Protocol) — and the wonderful secret of the web is that it's *readable text*. A request is a few lines of text; a response is a few lines of text plus content. Learn to read those lines and the web stops being magic.

This chapter covers the exact anatomy of requests and responses, the HTTP **methods** (GET, POST, and friends), **status codes** (200, 404, 301, 500...), and **headers** — the metadata that powers caching, cookies, content negotiation and more. You'll inspect real traffic two ways: the browser DevTools **Network tab** and **curl**, the two tools you'll use for the rest of this track and the rest of your career.

## Definitions & Explanations

### The request/response cycle

HTTP is strictly turn-based: the client sends one **request**, the server returns one **response**. The server never speaks first and (in plain HTTP) never sends more than asked.

```
CLIENT                                          SERVER
  |                                               |
  |---- REQUEST ------------------------------->  |
  |     "GET /products/42 ... "                   |
  |                                               | (reads file / runs code)
  |<--- RESPONSE -------------------------------  |
  |     "200 OK ... <html>...</html>"             |
  |                                               |
```

### Anatomy of a request

A raw HTTP request is plain text with a fixed shape:

```
GET /products/42?ref=home HTTP/1.1        <- request line: METHOD PATH VERSION
Host: shop.example.com                    <- headers: Name: value, one per line
User-Agent: Mozilla/5.0 ...
Accept: text/html
Accept-Language: en-US
Cookie: session=abc123
                                          <- blank line ends the headers
{"optional": "body goes here"}            <- body (only for some methods)
```

- **Request line**: the method (what kind of action), the path (which resource), the protocol version.
- **Headers**: key-value metadata. `Host` is required — it tells a server hosting many sites *which* site you mean.
- **Body**: data you're sending *to* the server (form contents, JSON). GET requests have no body; POST/PUT usually do.

### Anatomy of a response

```
HTTP/1.1 200 OK                           <- status line: VERSION CODE REASON
Content-Type: text/html; charset=utf-8    <- headers describing the response
Content-Length: 5321
Cache-Control: max-age=600
Set-Cookie: session=abc123; HttpOnly
                                          <- blank line
<!doctype html>                           <- body: the actual content
<html> ...
```

- **Status line**: protocol version, a three-digit **status code**, and a human-readable reason phrase.
- **Headers**: describe the body (`Content-Type`, `Content-Length`), give caching instructions, set cookies, and much more.
- **Body**: the payload — HTML, JSON, an image's bytes, anything.

### Methods: the verbs

The method declares your *intent* toward the resource at the path:

| Method | Meaning | Body? | Safe? | Idempotent? |
|---|---|---|---|---|
| **GET** | Read/fetch the resource | No | Yes | Yes |
| **POST** | Submit data / create something / "do a thing" | Yes | No | No |
| **PUT** | Replace the resource with this body | Yes | No | Yes |
| **PATCH** | Partially update the resource | Yes | No | No |
| **DELETE** | Remove the resource | Rarely | No | Yes |
| **HEAD** | Like GET but return headers only | No | Yes | Yes |
| **OPTIONS** | "What am I allowed to do here?" (used by browsers for CORS) | No | Yes | Yes |

Two vocabulary words worth internalizing:

- **Safe** = doesn't change anything on the server. Browsers freely prefetch/retry safe requests.
- **Idempotent** = doing it twice has the same effect as once. `DELETE /order/7` twice still leaves order 7 deleted; `POST /orders` twice may create two orders — which is why browsers warn "resubmit form?" on refresh after a POST.

Everyday browsing is almost entirely GET (typing URLs, clicking links, loading images) plus POST (submitting forms, logging in). The rest shine in APIs (Chapter 8).

### Status codes: the server's one-glance verdict

The first digit tells the story:

- **1xx — informational** (rare in daily life).
- **2xx — success.** `200 OK` (here you go), `201 Created` (made the thing, common on POST), `204 No Content` (done; nothing to show).
- **3xx — redirection.** `301 Moved Permanently` and `302 Found` say "go ask this other URL instead" — the new URL rides in the `Location` header, and browsers follow it automatically. `304 Not Modified` says "your cached copy is still good" (Chapter 6).
- **4xx — client's fault.** `400 Bad Request` (malformed), `401 Unauthorized` (who are you? — really means *unauthenticated*), `403 Forbidden` (I know who you are; still no), `404 Not Found`, `429 Too Many Requests` (slow down).
- **5xx — server's fault.** `500 Internal Server Error` (server code crashed), `502 Bad Gateway` / `503 Service Unavailable` / `504 Gateway Timeout` (infrastructure between you and the app is unhappy).

The 4xx/5xx split is your first debugging fork: 4xx → fix the request; 5xx → the server (or its operators) must fix something.

### Headers worth knowing on sight

| Header | Direction | What it does |
|---|---|---|
| `Host` | request | Which site on this server you want (mandatory) |
| `User-Agent` | request | What browser/client is asking |
| `Accept` | request | What content types you can handle |
| `Authorization` | request | Credentials/tokens (Chapter 7) |
| `Cookie` / `Set-Cookie` | req / resp | State between requests (Chapters 6–7) |
| `Content-Type` | both | Format of the body: `text/html`, `application/json`, `image/png` |
| `Content-Length` | both | Body size in bytes |
| `Location` | response | Where a redirect points |
| `Cache-Control`, `ETag` | response | Caching rules (Chapter 6) |

`Content-Type` values are **MIME types** (`type/subtype`). Browsers trust this header, not the file extension, to decide how to treat a response.

### HTTP is stateless

Each request stands alone: the protocol itself has no memory of previous requests. The server doesn't inherently know that request #2 came from the same person as request #1. Everything that *feels* stateful — shopping carts, logins — is built by attaching identifying data (usually cookies) to every request. This single design fact explains most of Chapter 7; park it in your head now.

## Hands-On Examples

### 1. Read a full request/response in DevTools

1. Open DevTools (`F12`) → **Network** tab → check "Disable cache".
2. Visit `https://developer.mozilla.org`.
3. Click the first (document) row. In **Headers**:
   - Find the **General** section: request URL, method (`GET`), status (`200`).
   - Expand **Response Headers**: locate `content-type` — confirm `text/html`.
   - Expand **Request Headers**: find `user-agent`, `accept`, `accept-language`, and any `cookie`.
4. Click the **Response** sub-tab to see the raw HTML body.
5. Now use the filter buttons (Doc / CSS / JS / Img / Font / Fetch) to see how many requests of each type this one page triggered.

### 2. curl: requests by hand

`-i` shows response headers; `-s` silences the progress bar.

```powershell
curl.exe -i https://example.com
```

Expected shape:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Cache-Control: max-age=...
...

<!doctype html>
<html>...
```

Headers only, no body (a HEAD request):

```powershell
curl.exe -I https://example.com
```

### 3. Watch a redirect happen

```powershell
curl.exe -i http://github.com
```

Expected:

```
HTTP/1.1 301 Moved Permanently
Location: https://github.com/
...
```

No page — just an instruction to go elsewhere (note: HTTP → HTTPS). Add `-L` to make curl follow redirects like a browser does, and `-v` to see each hop:

```powershell
curl.exe -sL -o NUL -v http://github.com 2>&1 | Select-String "^< HTTP|^> GET|Location"
```

### 4. Trigger status codes on demand

`httpbin.org` is a free service built for exactly this:

```powershell
curl.exe -i https://httpbin.org/status/404
curl.exe -i https://httpbin.org/status/500
curl.exe -i https://httpbin.org/status/301
```

Check the status line of each. Then see your own request reflected back as JSON:

```powershell
curl.exe -s https://httpbin.org/get
```

Expected: a JSON document whose `headers` object shows exactly what you sent — including a `User-Agent` of `curl/...`, proving you looked nothing like a browser.

### 5. Send a POST with a body

```powershell
curl.exe -s -X POST https://httpbin.org/post -H "Content-Type: application/json" -d "{\"name\":\"me\",\"level\":1}"
```

Expected: JSON echoing your submission under `"json": {"name": "me", "level": 1}`. Dissect the command: `-X POST` sets the method, `-H` adds a header, `-d` supplies the body. You just did manually what every login form does.

### 6. Talk to your own server

Run `python -m http.server 8000` in a folder with an `index.html`, then:

```powershell
curl.exe -i http://localhost:8000/
curl.exe -i http://localhost:8000/does-not-exist.html
```

Expected: a `200` with your HTML, then a `404` — and watch the server's console log both requests. You're seeing both ends of HTTP at once.

## Common Misconceptions

- **"URLs point at files."** URLs point at *resources* — the server decides what a path means. `/products/42` usually triggers code and a database query; no `42` file exists anywhere.
- **"404 means the site is down."** 404 means *this path* wasn't found on a perfectly functioning server. "Down" looks like 5xx, a timeout, or a connection refusal.
- **"401 means you lack permission."** Despite the name "Unauthorized," 401 means *unauthenticated* (you haven't proven who you are). 403 is the actual "you're known, and denied."
- **"POST is secure, GET is not."** Both travel identically. POST keeps data out of the URL (so out of history/logs), but only **HTTPS** (Chapter 5) makes either private in transit.
- **"The browser and server keep a conversation going."** HTTP is stateless; every request is a stranger until proven otherwise by cookies/tokens. (TCP connections do get *reused* for performance, but that's plumbing — each HTTP request is still logically independent.)
- **"Headers are exotic edge-case stuff."** Headers are load-bearing on every single request: caching, cookies, auth, content types, redirects, compression, CORS — all headers.

## Practice Exercises

1. **Annotated capture.** Load any medium-sized site with DevTools open. Pick its main document request and write out, in a markdown file, every request header and every response header with a one-line plain-English explanation of each. Look up any header you don't recognize (MDN's HTTP headers reference).
2. **Status code hunt.** Using real sites and `curl.exe -i` (not httpbin), capture in the wild: a 200, a 301 or 302, a 304 (hint: DevTools with cache *enabled*, reload), a 404, and any 4xx or 5xx of your choice. Record the URL, the status line, and one interesting response header from each.
3. **Redirect chain mapping.** Find a URL that redirects at least twice (try `http://` + a bare famous domain, e.g. `http://google.com` vs its country/`www` variants). Using `curl.exe -iL` or `-v`, document each hop: status code and `Location` at every step, ending at the final 200.
4. **Method experiments.** Against `https://httpbin.org`, send one request each with GET, POST, PUT, PATCH, DELETE (endpoints `/get`, `/post`, `/put`, `/patch`, `/delete`), including a JSON body where appropriate. Save each command and its response; note how the echoed data differs between methods.
5. **Be the weird client.** Use curl's `-H` to impersonate and mutate: send a request to `https://httpbin.org/get` with a custom `User-Agent` of your own invention, an `Accept-Language` of `fr-FR`, and a made-up header `X-Student: yourname`. Confirm in the echoed JSON that all three arrived. What does this tell you about how much servers can trust request headers?
