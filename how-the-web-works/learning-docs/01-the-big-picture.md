# Chapter 1: The Big Picture — What Happens When You Press Enter

## Overview

Before diving into any single technology, you need a map of the whole territory. This chapter answers the question every web developer should be able to answer in depth by the end of this track:

> **"What happens when you type `https://example.com` into your browser and press Enter?"**

That single question is the spine of this entire track. Every later chapter zooms into one segment of the journey you'll sketch here. By the end of this chapter you'll have a rough but *correct* mental model; by the end of the track, each fuzzy box in that model will be filled in with real detail.

Why this matters: when something breaks — a page won't load, an API call fails, a login stops working — the developers who debug it fastest are the ones who know *where in the journey* to look. "Is this a DNS problem, a network problem, a server problem, or a browser problem?" is the first question of every web debugging session.

## Definitions & Explanations

### Client and server

- **Client**: any program that *asks* for something over a network. Your browser is a client. So is `curl`, a mobile app, or a Python script using `requests`.
- **Server**: any program that *listens* for requests and answers them. Despite the name, a server is software, not (only) hardware. A "server machine" is just a computer that runs server programs.

The web is built on this **request/response** pattern: the client always initiates, the server always responds. Servers do not spontaneously push pages at you (with narrow exceptions you'll meet in Chapter 10).

```
   CLIENT (your browser)                    SERVER (a program on another computer)
   +------------------+   -- request -->    +---------------------+
   | "GET me the page |                     | Listens on port 443 |
   |  at example.com" |   <-- response --   | Sends back HTML     |
   +------------------+                     +---------------------+
```

### The URL, dissected

A URL (Uniform Resource Locator) is an address with several parts, each of which matters:

```
https://www.example.com:443/products/shoes?color=red&size=10#reviews
\___/   \_______________/\_/\_____________/\_______________/\______/
scheme        host       port     path          query        fragment
```

- **Scheme** (`https`): *how* to talk — the protocol. `http`, `https`, `ftp`, `mailto` are all schemes.
- **Host** (`www.example.com`): *who* to talk to — a name that must be translated into an IP address (Chapter 3).
- **Port** (`443`): *which door* on that machine. Usually invisible because schemes have defaults: 80 for `http`, 443 for `https` (Chapter 2).
- **Path** (`/products/shoes`): *what* you want from that server.
- **Query string** (`?color=red&size=10`): extra parameters, as key=value pairs joined by `&`.
- **Fragment** (`#reviews`): a position *within* the page. Never sent to the server — the browser handles it alone.

### The journey, step by step (the track's spine)

Here is the whole trip. Do not worry about fully understanding each step yet — the chapter numbers show where each one gets its own deep dive.

```
 You press Enter on https://www.example.com/
      |
 [1]  Browser parses the URL                      (this chapter)
      |
 [2]  Browser checks its caches                   (Ch 6)
      |  - "Do I already have this page?"
      |
 [3]  DNS lookup: www.example.com -> 93.184.216.34   (Ch 3)
      |
 [4]  TCP connection opened to 93.184.216.34:443     (Ch 2)
      |
 [5]  TLS handshake: encryption negotiated,          (Ch 5)
      |  server proves its identity with a certificate
      |
 [6]  Browser sends an HTTP request:                 (Ch 4)
      |     GET / HTTP/1.1
      |     Host: www.example.com
      |
 [7]  Server receives it, runs code or reads a file, (Ch 9)
      |  builds a response
      |
 [8]  Server sends an HTTP response:                 (Ch 4)
      |     HTTP/1.1 200 OK  + headers + HTML body
      |
 [9]  Browser parses HTML, discovers it needs more   (Ch 6)
      |  files (CSS, JS, images) -> repeats steps
      |  2-8 for EACH of them
      |
 [10] Browser renders pixels to your screen          (Ch 6)
      |
 [11] JavaScript runs, may fetch even more data      (Ch 8, 10)
      |  from APIs
      v
 Page is "loaded"
```

Two big insights hide in this diagram:

1. **A "page load" is many requests, not one.** A typical news homepage makes 50–150 separate requests: one for HTML, then dozens for stylesheets, scripts, images, fonts, and data.
2. **Every request repeats the same core steps.** Learn the pipeline once and you understand every request your browser will ever make.

### Where does "the internet" fit in?

People say "the web" and "the internet" interchangeably, but they're different layers:

- **The internet** is the global network of connected computers and the rules (protocols) for moving data between them: IP, TCP, UDP. It carries *anything* — email, video calls, online games, the web.
- **The web (World Wide Web)** is one application *built on top of* the internet: documents and data linked together, fetched with HTTP, displayed by browsers.

```
+---------------------------------------------+
|  The Web (HTTP, HTML, browsers, servers)    |   <- Chapters 4-10
+---------------------------------------------+
|  The Internet (IP, TCP/UDP, DNS, routers)   |   <- Chapters 2-3
+---------------------------------------------+
|  Physical stuff (fiber, Wi-Fi, cables)      |   <- mentioned, not covered
+---------------------------------------------+
```

Email uses the internet but not the web. The web cannot exist without the internet. Keep the layers separate in your head and half of networking confusion disappears.

## Hands-On Examples

You'll do all of these on Windows. Open **PowerShell** (press Win, type "powershell", Enter) for the terminal parts.

### 1. Watch a page load in DevTools

1. Open Chrome or Edge. Press `F12` (or `Ctrl+Shift+I`) to open DevTools.
2. Click the **Network** tab.
3. Check "Disable cache" (so you see the real, uncached journey).
4. Navigate to `https://example.com` (a deliberately tiny site).
5. Observe: you'll see just a couple of rows — the HTML document and maybe a favicon request.
6. Now navigate to a real site like `https://www.wikipedia.org`. Watch the rows pour in: document, CSS, JS, images, fonts.
7. Click the very first row (the document). Look at the **Headers** panel — you are looking at a real HTTP request and response. You'll learn to read every line of it in Chapter 4.

### 2. Fetch a page without a browser

The browser is not magic — it's just one client. Prove it with `curl`, which ships with Windows 10/11:

```powershell
curl.exe https://example.com
```

(Note the `.exe` — in PowerShell plain `curl` is an alias for a different command.)

Expected output: raw HTML, starting roughly like:

```html
<!doctype html>
<html>
<head>
    <title>Example Domain</title>
...
```

That HTML is *exactly* what the browser received in your DevTools experiment. The browser's only extra job is turning it into pixels.

### 3. See the name-to-number translation

```powershell
nslookup example.com
```

Expected output (addresses may differ):

```
Server:  <your router or ISP resolver>
Address:  192.168.1.1

Non-authoritative answer:
Name:    example.com
Addresses:  93.184.216.34
```

That number is where your request actually went. Chapter 3 explains everything on this screen.

### 4. Be a server yourself (30 seconds)

You're learning Python in a parallel track — here's your first server:

```powershell
cd ~
python -m http.server 8000
```

Then open `http://localhost:8000` in your browser. You are now both the client *and* the server: your browser (client) is requesting a directory listing from a Python program (server) on your own machine. Watch the PowerShell window — each browser request prints a log line. Press `Ctrl+C` to stop it.

## Common Misconceptions

- **"The web and the internet are the same thing."** No — the internet is the transport network; the web is one application riding on it. Email, video calls, and games also use the internet without touching the web.
- **"A server is a big special computer."** A server is a *program* that listens and responds. You just ran one on your laptop with a single Python command. Big companies run server programs on powerful machines, but the concept is identical.
- **"Typing a URL fetches 'the website' in one go."** A page load is dozens of separate request/response cycles. The first response (HTML) is a *shopping list* that triggers all the rest.
- **"The browser has a special connection to websites."** The browser speaks plain, documented protocols (HTTP over TCP). Any program — curl, Python, your future code — can do exactly what it does.
- **"The server sends the page to my screen."** The server sends *text* (HTML/CSS/JS/data). Your browser, running on your machine, does all the work of turning that text into a rendered page. Rendering is local.

## Practice Exercises

1. **Draw the journey from memory.** Close this document and sketch (on paper or in a text file) the numbered steps from URL to rendered page. Compare with the diagram above and note which steps you forgot — those chapters deserve extra attention later.
2. **URL dissection drill.** Break these URLs into scheme / host / port / path / query / fragment (write your answers in a markdown file): `https://developer.mozilla.org/en-US/docs/Web/HTTP?utm_source=newsletter#syntax`, `http://localhost:8000/index.html`, `https://api.github.com/users/octocat/repos?per_page=5`.
3. **Request counting.** Using the DevTools Network tab (cache disabled), load three sites of different complexity — `example.com`, a news site, and a web app like `github.com`. Record for each: total number of requests, total transferred bytes, and load time (all shown in the Network tab's status bar). Write two sentences on what surprised you.
4. **Client zoo.** Fetch `https://example.com` three different ways: in the browser, with `curl.exe`, and with Python (`python -c "import urllib.request; print(urllib.request.urlopen('https://example.com').read().decode()[:300])"`). Confirm all three receive the same HTML. What does this tell you about where "the website" actually lives?
5. **Serve your own page.** Create a folder containing a simple `index.html` (use your HTML track skills), run `python -m http.server 8000` in that folder, and load it at `http://localhost:8000`. Then open DevTools Network tab and reload — identify which request delivered your HTML and what status code it got.
