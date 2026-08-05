# Chapter 6: Inside the Browser — Parsing, Rendering, Caching, Storage

## Overview

The response has arrived. Now the most sophisticated program on your computer takes over: the browser. This chapter covers what happens *after* the bytes land — how HTML becomes pixels (the render pipeline, conceptually), how the browser discovers and fetches all the sub-resources a page needs, and the two superpowers that make the web feel fast and personal: **caching** (avoiding repeat downloads) and **client-side storage** (cookies, localStorage) that lets a stateless protocol remember you.

Why it matters: this is where your parallel HTML/CSS/JavaScript tracks physically execute. Understanding the pipeline explains *why* CSS goes in `<head>` and scripts go at the bottom (or use `defer`), why a second visit is so much faster than the first, and what all those DevTools tabs actually show.

## Definitions & Explanations

### From bytes to pixels: the render pipeline

When HTML arrives, the browser runs a pipeline (simplified but honest):

```
 HTML text ──parse──> DOM tree          (the page's structure in memory)
 CSS text  ──parse──> CSSOM             (all style rules in memory)
                        |
        DOM + CSSOM ──combine──> Render tree   (only visible elements, with styles)
                        |
                     Layout             (compute each element's size & position)
                        |
                     Paint              (fill in pixels: text, colors, images)
                        |
                     Composite          (layers assembled; GPU puts frame on screen)
```

- **DOM (Document Object Model)**: the live, in-memory tree built from your HTML. This is the *actual page* — what JavaScript reads and modifies. The HTML file was just the recipe; the DOM is the dish.
- **CSSOM**: same idea for CSS rules.
- **Layout (reflow)**: geometry math — where everything goes. Changing an element's size re-triggers this, which is why some JS/CSS operations are "expensive."
- **Paint & composite**: rasterizing to pixels and layering them.

This pipeline re-runs (in parts) whenever the DOM or styles change — every animation frame, every JS mutation. That's "rendering" for the rest of your career.

### Parsing discovers more work

As the parser walks the HTML, every `<link rel="stylesheet">`, `<script src>`, `<img>`, and font reference triggers *another HTTP request* (the full Chapter 1–5 journey each time). Two blocking rules shape all frontend performance advice:

- **CSS blocks rendering**: the browser won't paint until it has the stylesheets (to avoid flashing unstyled content). Hence CSS goes early, in `<head>`.
- **Classic `<script>` blocks parsing**: the parser stops, fetches, and executes the script before continuing (the script might `document.write` or expect the DOM built so far). Hence `defer`/`async` attributes and the old "scripts at the bottom" rule.

### HTTP caching: the fastest request is no request

Downloading the same logo 50 times would be absurd. The server attaches caching headers to responses; the browser keeps a local **HTTP cache** and obeys them:

- `Cache-Control: max-age=3600` — "this response is fresh for 3600 seconds; until then, reuse it without even asking." DevTools shows such hits as "(from disk cache)" or "(from memory cache)" — no network at all.
- `Cache-Control: no-cache` — "you may store it, but *revalidate* with me before each use."
- `Cache-Control: no-store` — "never store this" (banking pages, personal data).
- **Revalidation**: responses can carry a fingerprint (`ETag: "abc123"`) or timestamp (`Last-Modified`). On next use the browser asks: `If-None-Match: "abc123"` — and if unchanged, the server answers **`304 Not Modified`** with *no body*: "your copy is still good." A 304 costs a round trip but no download.

```
First visit:    GET /logo.png            -> 200 OK + 45 KB   (slow-ish)
Within max-age: (no request at all)      -> from disk cache  (instant)
After expiry:   GET /logo.png
                If-None-Match: "abc123"  -> 304 Not Modified (tiny)
```

The modern best-practice pattern you'll see everywhere: HTML is cached briefly or revalidated, while CSS/JS files get *hashed filenames* (`app.d41d8cd9.js`) plus `max-age=31536000, immutable` — cache forever, and deploy changes by changing the *filename*. This is "cache busting."

### Cookies: the browser's memory attached to a domain

HTTP is stateless (Chapter 4), so the browser provides state: a **cookie** is a small named value that a server asks the browser to store, which the browser then **automatically attaches to every future request to that same site**.

```
Response:  Set-Cookie: theme=dark; Max-Age=31536000; Path=/
                       session=abc123; HttpOnly; Secure; SameSite=Lax

Every later request to this site:
           Cookie: theme=dark; session=abc123
```

Attributes that matter:

| Attribute | Meaning |
|---|---|
| `Max-Age` / `Expires` | How long to keep it. Absent → a *session cookie*, gone when the browser closes. |
| `Domain`, `Path` | Which requests it rides along with. |
| `Secure` | Only ever send over HTTPS. |
| `HttpOnly` | JavaScript cannot read it (protects login cookies from malicious scripts). |
| `SameSite` | Whether it's sent on requests *initiated by other sites* (defense against CSRF; `Lax` is the modern default). |

Cookies are scoped **per site** — `evil.com` can never read `bank.com`'s cookies. Their automatic-attachment behavior is what makes logins work (Chapter 7) and also what makes cross-site tracking possible (third-party cookies — a cookie set by a domain other than the one in the address bar, typically an ad or analytics network embedded on many sites). Browser policy on this differs by vendor and keeps shifting: Safari and Firefox block third-party cookies by default; Chrome has flip-flopped on deprecating them and still allows them by default as of this writing. Don't take this paragraph's word for it — check each browser's current behavior before relying on it.

### Web storage: localStorage and friends

For data that *doesn't* need to travel to the server on every request, JavaScript gets storage APIs:

- **`localStorage`** — key/value strings, per-site, persistent (~5–10 MB). `localStorage.setItem("theme", "dark")`.
- **`sessionStorage`** — same API, but per-tab and cleared when the tab closes.
- **IndexedDB / Cache Storage** — bigger, structured storage for serious offline apps (know they exist).

Cookies vs localStorage in one line: **cookies are sent to the server automatically on every request; localStorage never leaves the browser unless your code sends it.** Cookies are for things the *server* needs each time (who you are); localStorage is for things only the *page* needs (UI preferences, drafts).

## Hands-On Examples

### 1. Watch the resource waterfall

1. DevTools → **Network**, cache disabled. Load `https://www.wikipedia.org`.
2. Study the **waterfall** column: the document finishes first, *then* bursts of CSS/JS/images begin — because the parser had to read the HTML to discover them. Hover a bar to see its phase breakdown (queueing, DNS, connect, TLS, waiting/TTFB, download): the entire Chapters 2–5 journey, itemized per request.
3. Sort by Size and by Time. What's the heaviest resource? The slowest?

### 2. See the DOM ≠ the HTML

1. On any page, view the raw HTML: `view-source:https://example.com` in the address bar — this is the text that arrived over the network.
2. Now open DevTools → **Elements**: this is the live DOM. On `example.com` they match closely; on a JS-heavy site (try a Twitter/X or YouTube page) the source is a near-empty shell while Elements shows thousands of nodes — JavaScript built the page after load. Keep this contrast in mind for Chapter 10 (SPAs).
3. In the **Console**, run `document.title = "I changed the DOM"` — watch the tab title change. The HTML file, wherever it lives on the server, is untouched.

### 3. Observe caching states: 200 → cache hit → 304

1. DevTools → Network. Make sure "Disable cache" is **unchecked**.
2. Hard-refresh once (`Ctrl+Shift+R`) on a real site — everything is `200`.
3. Click a link on the site, or navigate away and back (normal navigation, not refresh). Many rows now show "(from memory cache)" / "(from disk cache)" in the Size column — zero network.
4. Normal-refresh (`Ctrl+R`, or F5): watch for `304 Not Modified` rows — revalidations. Click one and inspect the request's `If-None-Match` header and the empty response.
5. Note the three tiers you just saw: full download (200), free (cache), cheap check (304).

### 4. Caching headers with curl

```powershell
curl.exe -I https://en.wikipedia.org/static/images/icons/wikipedia.png
```

Expected headers to hunt for: `cache-control` (note the `max-age`), `etag` or `last-modified`, and `age` (how long a CDN has been holding this copy). Then simulate the browser's revalidation — copy the ETag value into a conditional request:

```powershell
curl.exe -i -H "If-None-Match: \"<paste-etag-here>\"" https://en.wikipedia.org/static/images/icons/wikipedia.png
```

Expected: `HTTP/2 304` (or `HTTP/1.1 304 Not Modified`) with no body — you just performed a revalidation by hand.

### 5. Inspect and forge cookies

1. Visit any site where you have an account (or just `https://github.com`). DevTools → **Application** tab → **Cookies** in the sidebar.
2. Examine the table: names, values, Domain, Path, Expires, and the HttpOnly / Secure / SameSite flag columns. Find at least one HttpOnly cookie — try `document.cookie` in the Console and confirm the HttpOnly one is *missing* from the output.
3. See cookies being *set*: Network tab, click the document request, look for `set-cookie` in response headers (often easiest in an incognito window on first visit).

### 6. Play with localStorage

On any page, in the Console:

```js
localStorage.setItem("experiment", "hello from chapter 6");
localStorage.getItem("experiment");
```

Close the tab entirely, reopen the same site, run `localStorage.getItem("experiment")` — still there. Check Application → **Local storage** to see it in the UI. Then confirm scoping: open a *different* site and run the same `getItem` — `null`, because storage is per-origin. Clean up with `localStorage.removeItem("experiment")`.

## Common Misconceptions

- **"The browser displays the HTML file."** The browser displays the *DOM* — a live structure initially built from HTML but freely rewritten by JavaScript. View-source and Elements can differ wildly.
- **"Refreshing gets a completely fresh copy."** A normal refresh happily serves cached and 304-revalidated resources. That's why "did you hard-refresh?" (`Ctrl+Shift+R`) is a real debugging question — and why "clear your cache" fixes stale-site problems.
- **"Cookies are files sites store on your PC / are inherently spyware."** A cookie is a scoped key-value string with an expiry, unreadable by other sites. The tracking reputation comes from one specific pattern — *third-party* cookies set by ad networks embedded across many sites — and how aggressively that pattern is restricted depends entirely on which browser you're using (see above); it's not a settled, universal fact.
- **"localStorage is a secure place for secrets."** Any JavaScript running on the page — including injected malicious scripts — can read it. Sensitive tokens generally belong in HttpOnly cookies precisely because scripts *can't* touch those.
- **"Caching just makes things stale and buggy."** Done right (hashed filenames + long max-age + revalidated HTML), caching is *the* biggest performance win on the web and never serves stale code. Bugs come from caching without a busting strategy.
- **"The page loads, then it's done."** Rendering is continuous: every JS change re-triggers parts of style/layout/paint. A "loaded" page is a running program, not a finished document.

## Practice Exercises

1. **Pipeline storyboard.** Draw the render pipeline (parse → DOM/CSSOM → render tree → layout → paint → composite) and annotate where these external files enter and what they block: a stylesheet in `<head>`, a classic `<script>` in `<head>`, the same script with `defer`, an `<img>`.
2. **Cache census.** Pick a media-heavy site. With caching enabled, visit, navigate away, return, and normal-refresh. Using the Network tab's Size and Status columns, count requests in each class: fresh 200s, memory/disk cache hits, and 304s. Present as a small table with your interpretation.
3. **Header archaeology.** With `curl.exe -I`, fetch (a) a site's main HTML page and (b) one of its fingerprinted static assets (find a `.css`/`.js` URL with a hash in its name via DevTools). Compare their `cache-control` values and explain, in two sentences, why they differ.
4. **Cookie audit.** In the Application tab on three sites you use, count total cookies and classify each: session vs persistent, first-party vs third-party (check the Domain column), HttpOnly or not. Which site is heaviest? Any cookie whose purpose you can guess from its name?
5. **Build a remembering page.** Using your HTML/JS track skills: a local page (served via `python -m http.server`) with a button that toggles dark mode and stores the choice in `localStorage`, restoring it on load. Then write one paragraph: why is localStorage the right tool here rather than a cookie — and in what variant of this feature would a cookie become the right tool?
