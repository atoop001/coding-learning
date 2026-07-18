# Chapter 9: Servers & Hosting — Where Websites Actually Live

## Overview

You've been the client for eight chapters. Time to fully understand the other side: what a **web server** actually is, the crucial distinction between **static** and **dynamic** serving, where code runs (and whose computer it runs on), and how the pieces you now know — domain, DNS, server, certificates — assemble into a live public website. We close with **CDNs**, the globe-spanning cache layer that most of the web's bytes actually come from.

Why it matters: this is the deployment chapter. Your capstone project asks you to put a real site on the public internet with your own hands, and every choice there — GitHub Pages? A VPS? What DNS records? — comes from this material. It's also the chapter that turns "the cloud" from marketing fog into concrete machines.

## Definitions & Explanations

### A web server is a program

Strip away the mystique: a **web server** is a program that (1) listens on a port, (2) parses incoming HTTP requests, (3) decides what the response should be, (4) sends it. You wrote one in Chapter 1 with `python -m http.server`. Production-grade ones — **Nginx**, **Apache**, **Caddy**, IIS — do the same job with better performance, configuration, logging, and TLS handling.

The machine it runs on is unremarkable: a computer, always on, reachable at a public IP. "The cloud" is renting such computers (or slices of them) in someone's datacenter — AWS, Azure, Hetzner, and friends. A **VPS** (virtual private server) is the classic rentable unit: a virtual machine with a public IP that's all yours, for a few dollars a month.

### Static vs dynamic: the fundamental split

**Static serving**: the response already exists as a file; the server just reads and sends it.

```
GET /about.html
   server: open ./site/about.html, send bytes.  Same file for everyone, every time.
```

**Dynamic serving**: the response is *computed per request* by application code — typically consulting a **database** — then sent.

```
GET /dashboard
   server: run code -> who is this user (cookie)? -> query DB for THEIR data
           -> render HTML or JSON -> send.  Different for every user, every minute.
```

Consequences of the split:

| | Static | Dynamic |
|---|---|---|
| Examples | Portfolio, docs, blog (pre-built) | Webmail, shops, social feeds, APIs |
| Speed & scaling | Trivially fast; cache everywhere | Costs CPU + DB work per request |
| Can it break at 3am? | Barely — nothing to crash | Very much — code and DBs fail |
| Hosting cost | Often free (Pages, Netlify) | Real servers, real money |
| Personalization | None (until client-side JS adds it) | Total |

A typical real site is a **hybrid**: static assets (images, CSS, JS bundles) served statically and cached hard, dynamic endpoints (`/api/...`, logged-in pages) computed fresh. **Static site generators** (Hugo, Jekyll, Astro, Eleventy) exploit a clever middle path: run the "dynamic" code *once at build time*, producing plain files to host statically — how most blogs and docs sites work now.

### Where code runs: client side vs server side

Two computers can run your code, and the difference is architectural, not stylistic:

- **Server-side code** (Python/Flask/Django, Node/Express, Go, PHP...) runs on *your* machine in the datacenter. It can hold secrets (DB passwords, API keys), enforce rules, and must be *trusted* because users can't see or alter it. Users only ever see its *outputs*.
- **Client-side code** (the JavaScript you ship) runs in *each visitor's browser*. It's fully visible, modifiable, and skippable by any user — so it can never be your security boundary. Validation in the browser is a courtesy; validation on the server is the law.

A page's journey usually involves both: server-side code produced the HTML/JSON; client-side code animates and updates it. When you hear "backend" and "frontend," this is the physical reality underneath.

### Assembling a live website: all the pieces snap together

Here is the whole stack you've built up over nine chapters, as a deployment checklist:

```
 1. WRITE IT      HTML/CSS/JS files, or an app + database
 2. HOST IT       a server (or hosting service) with a public IP: 203.0.113.9
 3. NAME IT       rent yourname.dev from a registrar             (Ch 3)
 4. CONNECT NAME  at your DNS host, create records:              (Ch 3)
    TO HOST         A     yourname.dev      -> 203.0.113.9
                    CNAME www.yourname.dev  -> yourname.dev
 5. SECURE IT     obtain a TLS certificate (Let's Encrypt,       (Ch 5)
                  automatic on most platforms); redirect http->https
 6. VERIFY IT     nslookup the name, curl -i the URL,            (Ch 3-4)
                  check the padlock                              (Ch 5)
```

Managed platforms (GitHub Pages, Netlify, Cloudflare Pages) collapse steps 2 and 5 into "push your files"; the domain and DNS steps remain yours. Notice there's no magic left in this list — you have personally used every tool it mentions.

### Behind one URL: more than one machine

At scale, "the server" is a fiction maintained for your convenience:

```
                        ┌────────────────────────────────────┐
 you ──> DNS ──> LOAD   │  app server 1                      │
                BALANCER┼─ app server 2  ──> database        │
                (one IP)│  app server 3      cache, queues…  │
                        └────────────────────────────────────┘
```

A **load balancer** (or **reverse proxy** — Nginx again, or a cloud service) receives all traffic on the public IP and fans requests out to interchangeable app servers. This is why statelessness (Chapters 4, 7, 8) is prized: any server can take any request if no conversation state is trapped on one box. The 502/504 errors from Chapter 4 are this layer speaking: the front door is up, but the app servers behind it aren't answering.

### CDNs: moving the content to the user

Chapter 2 taught you the latency floor: distance costs milliseconds, physics is non-negotiable. A **CDN (Content Delivery Network)** — Cloudflare, Akamai, Fastly, CloudFront — beats it by copying your content to hundreds of **edge servers** worldwide and answering each user from the nearest one:

```
 Without CDN:  Tokyo user ──────────(180 ms)──────────> your Virginia server
 With CDN:     Tokyo user ──(8 ms)──> Tokyo edge cache ──(only on cache miss)──> Virginia
```

How your request lands on the nearest edge: **DNS** (of course). The CDN answers DNS queries with different IPs depending on where the query comes from — one hostname, many locations. This is why Chapter 3's exercise showed big sites resolving differently from different resolvers.

CDNs natively serve static/cacheable content (the `Cache-Control` headers from Chapter 6 tell edges what they may keep and for how long) and increasingly also provide TLS, DDoS absorption, and even code-at-the-edge. The `Age`, `X-Cache`, or `CF-Cache-Status` response headers are the CDN's fingerprints — you'll hunt them below. A huge fraction of everything you've ever downloaded came from a CDN edge, not the site's own machines.

## Hands-On Examples

### 1. Build a real static server experience locally

Create a folder with `index.html`, a `style.css`, an image, and a second page `about.html` linking back. Serve it:

```powershell
python -m http.server 8000
```

Now act as your own operations team:

- Browse it; watch the server console log each request line (`"GET /style.css HTTP/1.1" 200`) — this is a web server **access log**, the same artifact (richer in production) that powers all analytics and debugging.
- `curl.exe -i http://localhost:8000/about.html` — confirm `Content-Type: text/html`.
- `curl.exe -i http://localhost:8000/style.css` — note the server chose `text/css` from the file extension: static servers map extensions to MIME types.
- Request a missing file and watch the 404 appear in *both* curl and the access log.

### 2. Serve something dynamic — feel the difference

A minimal dynamic server, using only Python's standard library. Save as `dyn.py` and run `python dyn.py`:

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import datetime

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        body = f"<h1>Hello!</h1><p>Server time: {datetime.datetime.now()}</p><p>You asked for: {self.path}</p>"
        self.send_response(200)
        self.send_header("Content-Type", "text/html; charset=utf-8")
        self.end_headers()
        self.wfile.write(body.encode())

HTTPServer(("127.0.0.1", 8000), Handler).serve_forever()
```

Refresh `http://localhost:8000` a few times: the time changes — **no file anywhere contains this page**; it's computed per request. Try `http://localhost:8000/anything/at/all` — every path "exists," because code, not the filesystem, decides. That's the entire static/dynamic distinction, live. (`Ctrl+C` to stop.)

### 3. Fingerprint real servers and CDNs

The `Server` header and CDN cache headers reveal infrastructure:

```powershell
curl.exe -sI https://en.wikipedia.org | Select-String "server|cache|age"
curl.exe -sI https://www.netflix.com | Select-String -Pattern "server|via|cache"
curl.exe -sI https://github.com | Select-String "server"
```

Expected finds: Wikipedia showing `server: ...` plus cache-hit headers and an `age:` (their CDN's been holding that copy for that many seconds); others showing `cloudflare`, `AmazonS3`, `Vercel`, `nginx`, or — commonly — nothing informative (hiding the header is a mild hardening habit). Collect `Server` headers from five sites and see how few origins the modern web has.

### 4. Spot the CDN in DNS

```powershell
nslookup www.reddit.com
nslookup www.reddit.com 1.1.1.1
```

Expected: CNAME chains ending in CDN-owned names (e.g. `...fastly.net` or `...cloudflare...`) and possibly different IPs per resolver — geographic DNS steering, observed first-hand. Try the same on two more big brands and note which CDN each uses.

### 5. Watch one page draw from many servers

DevTools → Network on a major news site. Add the **Domain** column (right-click the column headers). One "page" pulls from: the site's own domain, one or more CDN/static-asset domains, and a crowd of third parties (analytics, ads, fonts). Count distinct domains. Each was a full DNS + TCP + TLS + HTTP journey. "A website" is a federation.

## Common Misconceptions

- **"A server is special hardware."** It's a program on an ordinary always-on computer with a reachable address. Your laptop served pages in Chapter 1; production differs in reliability and scale, not kind.
- **"The cloud is somewhere ethereal."** It's other people's datacenters: physical buildings of physical machines you rent by the hour. Every "serverless" function still runs on a server — you just don't manage it.
- **"Websites live at their domain."** Domains are pointers (Chapter 3). The site lives on whatever machines DNS currently points at — which the owner can swap silently, and which a CDN multiplies into hundreds of locations.
- **"My JavaScript validation protects my server."** Client-side code runs on machines you don't control and can be bypassed with curl in one line. Every rule that matters must be enforced server-side; the browser copy is UX only.
- **"Static sites are toy sites."** Static hosting powers massive documentation sites, blogs, and marketing properties, is nearly unbreakable, and — via static site generators and client-side `fetch` to APIs — can look and feel as rich as anything. Choosing static when you can is an engineering virtue.
- **"One URL = one computer."** Between load balancers, server pools, and CDN edges, a popular URL may be answered by any of thousands of machines. HTTP's statelessness is what makes that swap invisible.

## Practice Exercises

1. **Access log analysis.** Serve a multi-file static site locally and browse every page with a normal (cache-enabled) browser. Copy the console access log into a file and annotate it: group the lines by page-view, mark cached/304 requests, and explain any request you didn't expect (favicon hunts count).
2. **Static or dynamic? — an evidence-based survey.** For five sites you use, judge which parts are static vs dynamic and *prove it with headers or behavior*: look for `age`/cache-hit headers (static/CDN evidence), personalized content (dynamic evidence), or identical vs changing responses across two curl fetches. Write one line of evidence per verdict.
3. **Extend the dynamic server.** Modify `dyn.py` so that (a) the path `/time` returns JSON (`Content-Type: application/json`) with the current time, (b) any other path returns a 404 with a custom HTML message, and (c) every request prints a one-line log with method, path, and status. Verify all three behaviors with curl. (This is server-side routing — the heart of every web framework.)
4. **Infrastructure dossier.** Pick one big site and compile everything externally observable about its hosting: DNS records and CNAME chains (`nslookup`), apparent CDN(s), `Server`/cache headers, redirect behavior (`curl -i http://...`), and certificate issuer (browser cert viewer). Present as a half-page profile.
5. **Deployment dry run (paper only).** You have a finished static portfolio and a rented domain. Write the complete, ordered checklist to make `https://www.yourname.dev` live on GitHub Pages *or* a generic static host: every DNS record with type and target, where the certificate comes from, how `www` vs bare domain is handled, and the three verification commands you'd run at the end. (You'll execute this for real in the capstone.)
