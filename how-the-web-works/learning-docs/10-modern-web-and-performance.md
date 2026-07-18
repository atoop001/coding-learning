# Chapter 10: The Modern Landscape — Architectures, Performance, and the Full Journey

## Overview

You now hold every individual piece: packets, DNS, HTTP, TLS, rendering, state, APIs, servers, CDNs. This final chapter assembles them into the questions working developers actually argue about: **where should HTML be built** (server-rendered pages vs single-page apps vs the hybrids that now dominate), **what makes sites fast or slow** (the performance model that falls straight out of Chapters 2–6), and **how real teams ship** (dev → staging → production). It ends with the payoff of the whole track: the complete, no-hand-waving walkthrough of one request — the Chapter 1 diagram, now with every box filled in.

## Definitions & Explanations

### Who builds the HTML? Three architectures

**Server-rendered (traditional / MPA — multi-page app).** Every navigation is a full request; the server (Python, PHP, Rails...) assembles complete HTML each time.

```
click link -> GET /products/42 -> server renders full HTML -> browser replaces page
```

Strengths: simple mental model, fast first paint, works without JS, trivially crawlable. Cost: every click repeats the full journey and redraws everything; interactivity between navigations needs sprinkled JS anyway.

**Single-page application (SPA).** The server sends one near-empty HTML shell plus a large JavaScript bundle *once*. From then on, JS intercepts navigation, fetches **JSON from APIs** (Chapter 8), and rewrites the DOM (Chapter 6) itself — the page never truly reloads.

```
first visit -> shell + big JS bundle -> JS builds the page
click link  -> fetch /api/products/42 (JSON) -> JS redraws part of the DOM
              (URL changes via the History API; no document request)
```

Strengths: app-like fluidity, rich state, only data crosses the wire after boot. Costs: heavy first load, blank-page-until-JS, SEO and accessibility require extra care, and the browser's Network tab shifts from documents to a stream of `fetch/XHR` rows. React, Vue, Angular built their reputations here.

**The hybrid mainstream (SSR + hydration, static generation).** Modern frameworks (Next.js, Nuxt, SvelteKit, Astro) blend the two: the *first* page is rendered to full HTML on the server (or pre-built at deploy time — Chapter 9's static generation) for instant display, then JavaScript **hydrates** it — attaches listeners and takes over subsequent navigation SPA-style. Most large sites you use are somewhere on this spectrum, not at either pole.

How to tell what you're looking at: view-source (Chapter 6). Full readable content in the source → server-rendered or pre-built; a near-empty `<div id="root">` → client-rendered SPA; full content *plus* a big JSON blob of the same data embedded in a `<script>` tag → SSR with hydration.

### Performance: latency arithmetic you already know

Web performance is mostly Chapters 2–6 doing arithmetic:

- **Round trips are the currency.** DNS (~1 RTT) + TCP (~1) + TLS (~1) + request/response (~1+) before *any* HTML arrives — on a 100 ms RTT, that's 400 ms of pure physics. Then CSS/JS discovered in the HTML each cost more trips. Fewer, smaller, earlier requests win.
- **Weight matters after latency.** Megabyte bundles and unoptimized images dominate download time, especially on mobile networks.
- **Blocking matters most of all.** Render-blocking CSS and parser-blocking JS (Chapter 6) stand between arrived bytes and visible pixels.

The standard levers, each mapping to a chapter you've done: cache aggressively with hashed filenames (Ch 6); serve from CDN edges (Ch 9); compress bodies — gzip/brotli, the `Content-Encoding` header — and images properly; minify and split JS; `defer` scripts; reuse connections (HTTP/2 multiplexes many requests over one connection; HTTP/3/QUIC over UDP cuts handshake trips — Chapter 2's trade-offs, renegotiated).

Measurement vocabulary (all visible in DevTools): **TTFB** (time to first byte — server + network think-time), **FCP/LCP** (first/largest contentful paint — when things appear), **CLS** (layout shift — things jumping). Chrome's **Lighthouse** tab bundles these into an audit; the Network tab's throttling dropdown ("Slow 4G") lets you feel your site the way a phone on a train does.

### From laptop to the world: environments and deployment

Real teams run the same app in several **environments**:

```
 DEV        your machine: localhost, fake data, instant edits      (Ch 1-2)
   |  git push / code review
 STAGING    a private clone of production: real-ish data, final checks
   |  deploy (often automatic — CI/CD)
 PROD       the real site, real users, real consequences           (Ch 9)
```

- **dev**: `http://localhost:5173`, hot reload, debug logging — the world you already inhabit.
- **staging** (a.k.a. test/preview): catches "works on my machine" — differences in HTTPS, real DNS, production-like data, environment variables. Modern static hosts auto-create a **preview URL per pull request**, which is staging democratized.
- **prod**: guarded by the deploy pipeline (**CI/CD** — automated build, test, deploy on merge), monitored by logs and alerts, rolled back when a release goes wrong.

Configuration that differs between environments (database URLs, API keys) lives in **environment variables**, never in code — which is also how secrets stay out of repositories (Chapter 9's trusted-server principle).

### The full journey — every box filled in

The Chapter 1 spine, replayed with everything you now know. Scenario: first-ever visit to `https://shop.example.com/products/42`.

```
[URL]    Browser parses: scheme https -> port 443; host shop.example.com; path /products/42.
[Cache]  HTTP cache: no entry (first visit). No service worker. Proceed to network.
[DNS]    Stub resolver: hosts file? no. OS cache? no. -> recursive resolver
         -> (root) -> (.com TLD) -> example.com's authoritative servers
         -> CNAME to cdn-provider.net -> A record for the NEAREST edge: 203.0.113.50.
         Every answer cached with its TTL.
[TCP]    Three-way handshake with 203.0.113.50:443. Browser's end: ephemeral port 51834.
[TLS]    ClientHello (naming shop.example.com) -> certificate chain -> verified against
         root store -> keys agreed -> encrypted channel. (HTTP/2 negotiated here too.)
[Request]GET /products/42 HTTP/2
         host: shop.example.com | accept: text/html | accept-encoding: br
         cookie: session=7f3a9c... (set on an earlier login — the server will know us)
[Edge]   CDN edge: HTML marked no-cache -> MISS -> forwards to origin.
[Origin] Load balancer -> app server #2 -> session store: 7f3a9c = user 42
         -> database: product 42 -> template rendered into HTML for THIS user.
[Response] 200 OK | content-type: text/html | content-encoding: br
         cache-control: no-cache | set-cookie: (session refreshed)
         ...compressed HTML body streams back through edge to browser...
[Parse]  HTML -> DOM. Parser discovers: /app.9f8e2c.css, /app.4d1a7b.js (defer),
         product images -> EACH repeats the journey (DNS cached, connection REUSED,
         so mostly just requests). CSS blocks first paint; deferred JS doesn't.
[Cache2] Hashed CSS/JS arrive with cache-control: max-age=31536000, immutable ->
         next visit, zero network for them. Edge already had them: X-Cache: HIT.
[Render] CSSOM + DOM -> render tree -> layout -> paint -> composite. Pixels.
[JS]     Deferred bundle runs, hydrates the page, then fetch("/api/recommendations")
         -> 200, application/json -> DOM updated in place. The page is now a program.
```

Read that top to bottom once more, slowly. Ten chapters ago it was one mysterious sentence — "the page loads." Now every line is a system you've poked with your own tools. That transformation is the point of this track; the walkthrough is also exactly the artifact you'll produce, for a real site, in the capstone project.

## Hands-On Examples

### 1. Classify sites by architecture

For each of `https://en.wikipedia.org`, `https://gmail.com` (or any web app you use), and a modern brand site of your choice:

1. Open `view-source:` — full content, empty shell, or content + embedded JSON blob?
2. Network tab, then click an internal link: does a new *document* request appear (server-rendered) or only `fetch/XHR` rows (SPA navigation)?
3. Settle each site into: MPA / SPA / hybrid, with your evidence.

### 2. Feel a slow connection

DevTools → Network → throttling dropdown → **Slow 4G**, cache disabled. Load a heavy site and watch the waterfall crawl: DNS/connect/TLS phases now visibly cost, the render-blocking CSS delays first paint, images trickle. Then re-enable cache and reload — experience the Chapter 6 payoff at full force. Turn throttling off afterwards.

### 3. Run a Lighthouse audit

DevTools → **Lighthouse** tab → Performance, Desktop → Analyze. Read your scores for FCP, LCP, CLS and the "Opportunities" list — every suggestion (compress images, eliminate render-blocking resources, use HTTP/2...) should now map to a chapter in this track. Run it against `example.com` too, and note what a near-perfect score looks like and why (tiny, static, no JS).

### 4. Spot compression and protocol versions

```powershell
curl.exe -sI -H "Accept-Encoding: gzip, br" https://en.wikipedia.org/wiki/Main_Page | Select-String "content-encoding|age|cache"
curl.exe -sI --http2 https://www.cloudflare.com | Select-Object -First 3
```

Expected: `content-encoding: gzip` (or `br`) proving body compression; and a first line reading `HTTP/2 200` — you're watching protocol negotiation results. In DevTools you can add a **Protocol** column (right-click column headers) and catch `h2` and `h3` in the wild.

### 5. A taste of push: watch a live connection stay open

Plain request/response isn't the whole modern story — pages can hold connections open for live updates (**WebSockets**, server-sent events). Open a site with live content (a live sports/score page, or `web.whatsapp.com` if you use it) → Network tab → **WS** filter. If a row appears, click it and watch the Messages sub-tab: frames flowing both ways over one long-lived connection — the exception to "the server never speaks first" that you were promised in Chapter 1.

## Common Misconceptions

- **"SPAs are the modern way; server-rendered pages are legacy."** The industry swung to SPAs, hit the costs (slow first loads, SEO, complexity), and swung back to hybrids. Choose per project: content sites lean server/static; apps lean SPA; frameworks now let you mix per page.
- **"Performance is about faster servers."** TTFB is one slice. Most perceived slowness is round trips, bytes, and blocking — fixed by caching, CDNs, compression, and unblocking the render path, not bigger machines.
- **"My fast experience is everyone's experience."** You test on fiber with a warm cache near the servers. First-visit users on mobile networks live in your Slow-4G throttled world. Measure there.
- **"Works on my machine ⇒ works in production."** Dev differs from prod in HTTPS, DNS, data volume, env vars, caching layers, and concurrency. That's the entire reason staging environments exist. Distrust any deploy that skipped one.
- **"The page finishes loading."** Modern pages keep fetching, rendering, and holding sockets open indefinitely. "Loaded" is a milestone (and a fuzzy one — which metric?), not a final state.
- **"Now I know how the web works."** You know the load-bearing 90%: the request's full journey and every layer it crosses. Beyond lie deeper strata (HTTP/3 internals, BGP routing, browser engine internals, WebRTC...) — but they all attach to the skeleton you now own, and you know exactly where each would attach.

## Practice Exercises

1. **Architecture field report.** Survey six sites you actually use. For each: MPA/SPA/hybrid verdict with view-source and Network-tab evidence, plus one sentence on whether you think the choice fits the site's job. Present as a table.
2. **Performance autopsy.** Pick one noticeably slow site. Using the throttled Network waterfall and a Lighthouse run, identify its three biggest costs (e.g., render-blocking scripts, huge hero image, long TTFB, no compression) and write a prioritized three-item fix list, citing the header or waterfall evidence for each.
3. **Optimize your own site.** Take a local multi-page site from earlier chapters and apply at least three levers: resize/compress images, add `defer` to scripts, inline or minify CSS, reduce request count. Measure before/after with the Network tab's request count / transferred bytes / finish time (throttled, cache disabled) and report the deltas.
4. **The walkthrough, from memory.** The track's final exam-by-yourself: without looking at this chapter, write the complete journey of `https://app.example.com/inbox` for a logged-in user on a first visit of the day — every layer from URL parsing to rendered pixels and a follow-up API fetch. Then diff against this chapter's walkthrough and list what you missed.
5. **Environment design.** For a hypothetical small web app (you + one friend, a Flask API + static frontend), design the environments: what runs where for dev, what serves as staging, how code moves between them, and which five configuration values must differ per environment. Half a page, boxes and arrows encouraged.
