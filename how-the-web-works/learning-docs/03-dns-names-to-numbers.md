# Chapter 3: DNS — Turning Names into Numbers

## Overview

Packets are addressed to IP numbers, but humans type names like `wikipedia.org`. The **Domain Name System (DNS)** is the world-wide, distributed phone book that translates one into the other. It runs step [3] of the journey from Chapter 1, and it runs *before anything else can happen* — no DNS answer, no connection, no website.

This chapter covers how domain names are structured, who actually answers a DNS query (spoiler: it's a chain of servers, not one), the main record types you'll meet, and how caching makes it all fast. You'll interrogate DNS yourself with `nslookup`, and hijack a domain name on your own machine using the hosts file — the oldest trick in networking.

Why it matters: DNS is involved in every request, it's a classic source of outages ("it's always DNS"), and when you eventually deploy a site with your own domain (the capstone project), configuring DNS records is a task *you* will perform by hand.

## Definitions & Explanations

### Anatomy of a domain name

Domain names are hierarchical, read **right to left**:

```
        www.example.co.uk.
        \_/ \_____/ \__/\/ \
         |     |      |  |  +-- the invisible root (usually omitted)
         |     |      |  +----- top-level domain (TLD): uk
         |     |      +-------- second-level: co  (a UK convention)
         |     +--------------- the registered domain: example
         +--------------------- subdomain: www
```

- **TLD (top-level domain)**: `com`, `org`, `dev`, `uk`, `io`... managed by registries.
- **Registered domain**: the part you rent (yes, rent — annually) from a **registrar** like Namecheap or Cloudflare: `example.com`.
- **Subdomains**: anything the domain owner invents to the left: `www`, `api`, `blog`, `mail`. Owners can create unlimited subdomains for free — they control that zone.

`www` is *just a subdomain*, by convention pointed at the same place as the bare domain. It has no special technical meaning.

### The resolution chain: who actually answers?

No single computer knows every domain. Resolution is a delegation chase:

```
 Your browser: "What's the IP of www.example.com?"
      |
      v
 [1] STUB RESOLVER (built into your OS)
      |  checks local cache & hosts file first; else asks -->
      v
 [2] RECURSIVE RESOLVER (your ISP's, or a public one like 1.1.1.1 / 8.8.8.8)
      |  does the real legwork, caches aggressively:
      |
      |--> [3] ROOT SERVER:      "Ask the .com servers, here they are"
      |--> [4] TLD SERVER (.com): "Ask example.com's nameservers, here they are"
      |--> [5] AUTHORITATIVE SERVER for example.com:
      |         "www.example.com is 93.184.216.34"  <- the actual answer
      |
      v
 Answer flows back, cached at every layer, browser connects to 93.184.216.34
```

Key roles:

- **Stub resolver**: the tiny client in your OS. Checks the **hosts file** and a local cache, then forwards to a recursive resolver.
- **Recursive resolver**: the workhorse. Usually run by your ISP, or you can use public ones: Cloudflare `1.1.1.1`, Google `8.8.8.8`, Quad9 `9.9.9.9`. It performs the full chase and caches results.
- **Root servers**: 13 named server clusters (hundreds of physical machines) that know where every TLD's servers are.
- **TLD servers**: know which **authoritative nameservers** handle each registered domain under them.
- **Authoritative nameservers**: hold the actual records for a domain. When you deploy a site, *these* are the servers you'll configure (via your registrar or DNS host's dashboard).

### Caching and TTL

The full chase happens rarely. Every answer carries a **TTL (time to live)** in seconds — how long it may be cached. Your browser caches answers, your OS caches, the recursive resolver caches. A record with TTL 3600 might be served from cache for up to an hour after it changes — which is why DNS changes "take time to propagate." (Nothing is pushed anywhere; the world is just waiting for caches to expire.)

### Record types you'll actually meet

A domain's zone holds typed records:

| Type | Maps | Example use |
|---|---|---|
| **A** | name → IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | name → IPv6 address | `example.com → 2606:2800:...` |
| **CNAME** | name → *another name* (alias) | `www.example.com → example.com`, or `blog.you.dev → yoursite.github.io` |
| **MX** | domain → mail servers | where email for `@example.com` goes |
| **TXT** | name → arbitrary text | domain-ownership verification, email anti-spoofing (SPF/DKIM) |
| **NS** | domain → its authoritative nameservers | the delegation glue |

CNAME matters for your future: services like GitHub Pages have you point a CNAME at *their* hostname, so they can change their IPs without you updating anything.

### The hosts file: DNS's local override

Before DNS existed, every machine kept a text file mapping names to addresses. That file still exists and is checked **before any DNS query**. On Windows:

```
C:\Windows\System32\drivers\etc\hosts
```

A line like `127.0.0.1  myapp.local` makes that name resolve to your own machine — for your machine only. Developers use it to test sites under realistic hostnames, block domains, or preview a site on a new server before switching real DNS.

## Hands-On Examples

### 1. Basic lookups with nslookup

```powershell
nslookup wikipedia.org
```

Expected shape:

```
Server:  UnKnown          <- or your router/ISP resolver name
Address:  192.168.1.1     <- WHO answered (your recursive resolver)

Non-authoritative answer:  <- i.e., served from a resolver, likely cached
Name:    wikipedia.org
Addresses:  2620:0:862:ed1a::1    <- AAAA (IPv6)
            198.35.26.96          <- A (IPv4)
```

Note "Non-authoritative": the answer came via a resolver's cache, not straight from Wikipedia's own nameservers.

### 2. Query specific record types

```powershell
nslookup -type=MX gmail.com      # where Gmail's email is delivered
nslookup -type=TXT example.com   # text records
nslookup -type=NS example.com    # who is authoritative for this domain
nslookup -type=CNAME www.github.com
```

For the MX query expect several lines like `gmail.com MX preference = 5, mail exchanger = gmail-smtp-in.l.google.com` — lower preference = tried first.

### 3. Ask a different resolver

Compare your default resolver's answer with Cloudflare's:

```powershell
nslookup example.com
nslookup example.com 1.1.1.1
```

The trailing `1.1.1.1` says "send this query to that resolver instead." For big sites you may get *different IPs* from different resolvers or at different times — large sites answer with varying, often geographically-tuned addresses (a taste of CDNs, Chapter 9).

### 4. Watch a CNAME chain

```powershell
nslookup www.github.com
```

You'll likely see it resolve through an alias (`www.github.com → github.com`) before reaching final A records. Try a site hosted on a platform, e.g. any `*.github.io` project site with a custom domain, to see longer chains.

### 5. Inspect and flush your DNS cache

```powershell
ipconfig /displaydns | Select-Object -First 40   # peek at cached entries
ipconfig /flushdns                                # clear it
```

Expected after flush: `Successfully flushed the DNS Resolver Cache.` This is a standard first-aid move when DNS behaves oddly.

### 6. The hosts file experiment (the classic)

You'll override a domain locally. This requires Administrator rights.

1. Open Notepad **as Administrator** (Start → type Notepad → right-click → Run as administrator).
2. File → Open → paste path `C:\Windows\System32\drivers\etc\hosts` (switch the file-type dropdown to "All Files").
3. Add this line at the bottom, then save:

   ```
   127.0.0.1   myfirstsite.test
   ```

4. Flush caches: `ipconfig /flushdns`, and restart your browser (it caches DNS too).
5. Start a local server in a folder with an `index.html`: `python -m http.server 80` (port 80 so no `:port` is needed; if 80 is blocked/taken, use 8000 and browse to `http://myfirstsite.test:8000`).
6. Visit `http://myfirstsite.test` — your page appears under a real-looking domain name that exists *only on your machine*.
7. Verify with `nslookup myfirstsite.test` — interestingly, nslookup may *fail* while the browser works: nslookup queries DNS servers directly and skips the hosts file, proving the hosts file is an OS-level, pre-DNS override. (`ping myfirstsite.test` *does* use the hosts file — try both.)
8. **Clean up**: remove the line from the hosts file and save. Leaving stale overrides causes deeply confusing bugs later.

## Common Misconceptions

- **"There's one big DNS server somewhere."** DNS is a distributed hierarchy: root → TLD → authoritative, with caching resolvers in front. No single machine knows everything; no single failure kills it all.
- **"DNS changes 'propagate' like a broadcast."** Nothing is pushed. Old answers simply live in caches until their TTL expires. Lower the TTL *before* a planned change and the switchover looks near-instant.
- **"www is technically special."** It's an ordinary subdomain that convention points at the website. `www.example.com` and `example.com` can host entirely different things.
- **"Buying a domain means owning it forever."** Domains are leased annually via registrars. Miss the renewal and someone else can register it — this has burned major companies.
- **"DNS gives you the page."** DNS only returns addresses (and other records). It's the address lookup *before* the trip; HTTP is the trip. A working `nslookup` plus a broken website means the problem lies beyond DNS.
- **"My browser always uses the OS resolver."** Modern browsers may use DNS-over-HTTPS to a resolver of their own choosing, bypassing OS settings — one reason browser behavior and `nslookup` can disagree.

## Practice Exercises

1. **Resolution chain narration.** Write out, step by step in your own words, everything that happens DNS-wise when a freshly-restarted computer looks up `api.weather.gov` for the first time. Name each actor (stub, recursive, root, TLD, authoritative) and what it contributes.
2. **Record safari.** For three domains of your choice (one big tech company, one small personal site, one university), collect their A, AAAA, MX, and NS records with `nslookup -type=...`. Present findings in a markdown table and note anything odd (missing AAAA? many MX entries? third-party nameservers like Cloudflare?).
3. **Resolver comparison.** Query the same popular domain (e.g. `www.amazon.com`) against your default resolver, `1.1.1.1`, and `8.8.8.8`. Record the IPs returned. Are they the same? Form a hypothesis for why or why not.
4. **Hosts-file dev setup.** Extend the hands-on experiment: create *two* fake domains in your hosts file (`site-a.test`, `site-b.test`), both pointing at `127.0.0.1`, and serve two different folders on two ports. Document the limitation you hit (why can't the hosts file alone send the two names to two different ports?). Clean up afterwards.
5. **TTL stakeout.** Using `nslookup -debug example.com` (which shows TTLs), query the same domain twice a minute apart via your default resolver. Watch the TTL value count down between queries — explain what that countdown tells you about where the answer came from.
