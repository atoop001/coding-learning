# Project 5: Capstone — Deploy It, Secure It, Narrate Every Millisecond

## Description

The finale in two acts. **Act One**: take a small static site you've built (Project 3's site, upgraded, or something new that shows off your parallel-track skills) and put it on the public internet — hosted on GitHub Pages (or Netlify/Cloudflare Pages), served over HTTPS with a valid certificate, and verified from the outside with the full detective toolkit. **Act Two**: write the document this whole track has been training you for — *The Complete Journey of One Request to My Site* — a definitive, evidence-cited walkthrough of everything that happens when a stranger's browser loads your deployed page: DNS, TCP, TLS, HTTP, CDN, caching, parsing, rendering. Every claim in the narrative must be backed by something you actually observed with your own tools.

This is the artifact that proves the track took: a live URL plus a walkthrough you could hand to a fellow learner and genuinely teach them how the web works.

## Difficulty

**Advanced (integration of everything).** Estimated effort: 8–12 hours across several sessions.

## Chapters Used

All ten — explicitly:

- Chapters 1–2 (the journey frame; IPs, ports, latency measurements)
- Chapter 3 (DNS records of your live site; CNAMEs; optionally configuring a custom domain)
- Chapters 4–5 (HTTP inspection of your deployment; certificate, HSTS, HTTP→HTTPS redirect)
- Chapter 6 (caching behavior of your deployed assets; what the host's CDN adds)
- Chapter 7 (what state, if any, your site sets — and the auth involved in *deploying*, conceptually)
- Chapters 8–10 (a fetch-powered feature; hosting/CDN analysis; the full-journey narrative and a performance audit)

## Requirements Checklist

### Act One — Deploy

- [ ] A static site with at least three pages, styling, and at least one JavaScript-powered feature that fetches from a public API (Project 4 skills) — no broken links, no console errors
- [ ] Source in a Git repository pushed to GitHub (or equivalent)
- [ ] Deployed and publicly reachable at an HTTPS URL (GitHub Pages / Netlify / Cloudflare Pages)
- [ ] Plain-HTTP access verified to redirect to HTTPS, captured with curl (status code + `Location`)
- [ ] Certificate inspected (browser viewer): issuer, validity window, covered names — recorded in the report
- [ ] **Zero mixed content**: page loads with no mixed-content warnings in the console — checked and stated
- [ ] External verification pack, run against the live site and saved as output snippets: `nslookup` (A/AAAA/CNAME chain), `curl.exe -i` on the homepage, `ping` RTT from your machine, and one response-header hunt showing CDN/cache evidence (`server`, `age`, `x-cache`, etc.)
- [ ] A deployment log in the report: each step you took, in order, including at least one thing that went wrong and how you diagnosed it (a truthful "nothing went wrong" is suspicious — dig for the favicon-404-tier hiccups)

### Act Two — Narrate

Write `the-complete-journey.md` (aim: the depth of Chapter 10's walkthrough, but for *your* site, with *your* measurements):

- [ ] **Stage-by-stage narrative** of a stranger's first visit to your URL, covering: URL parsing → cache checks → DNS resolution chain (with your site's real records and who answers authoritatively) → TCP connection (real IP, real port, your measured RTT) → TLS handshake (your real certificate chain) → HTTP request/response (your real headers, quoted) → sub-resource discovery and fetching → parse/layout/paint → your JS running and its API fetch (a second full journey to a different host — narrate the contrast)
- [ ] Every stage cites its **evidence**: the command or DevTools observation that backs it, with the relevant snippet
- [ ] An **ASCII master diagram** of the whole journey specific to your deployment (your host, the platform's CDN, the API's host)
- [ ] A **caching epilogue**: what the second visit looks like (measured), and which response headers made it so
- [ ] A **performance audit**: Lighthouse run on the live site — scores recorded, plus your top two improvement opportunities and one sentence each on the chapter-concept behind them
- [ ] A closing **"what I still don't know" list** — five honest questions the track didn't answer; these are your syllabus for what comes next

## Hints

- GitHub Pages: a repository's Settings → Pages turns a branch into a site; the URL is `https://<username>.github.io/<repo>/`. HTTPS is automatic — notice *whose* certificate you get, and let that observation feed your CDN analysis.
- Deploy early, polish later: getting *any* page live on day one leaves the rest of the effort for inspection and writing, which is the real work here.
- Relative asset paths (`./style.css`, not `http://...` or `/style.css` if you're in a subpath) prevent both mixed content and the classic broken-CSS-on-Pages bug.
- Your Project 1 and Project 2 reports are templates: Act Two is those two skills aimed at your own property.
- The API fetch from your live HTTPS page must itself be HTTPS — an `http://` API URL is mixed *active* content and will be blocked; you learned exactly why.
- For the DNS resolution chain, `nslookup -type=NS` on the platform's domain shows you who's authoritative; narrate the chase root → TLD → those servers.
- Measure twice, write once: capture all evidence snippets into a scratch file first, then write the narrative around them.

## Stretch Goals

- **Custom domain**: register a cheap domain, add the CNAME/A records your platform requires, wait out the TTLs, and extend your narrative with the extra DNS hop and the new certificate that gets issued for it. (This is the single most instructive stretch goal in the track.)
- Ask a friend in another region (or an online ping/looking-glass tool) to measure RTT and `nslookup` answers to your site from far away — add a "view from elsewhere" sidebar comparing which CDN edge they hit versus you.
- Add a `404.html` page, verify the platform actually serves it with a real 404 status (curl proves it), and add the error path to your narrative.
- Run your verification pack as a scheduled check: a small PowerShell or Python script that curls your site, checks for status 200 and a substring, and prints PASS/FAIL — your first uptime monitor.
- Write a companion piece, one page, aimed at a non-programmer: the same journey with zero jargon. Explaining it simply is the final proof you understand it.
