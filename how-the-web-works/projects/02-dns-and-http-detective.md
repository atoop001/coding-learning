# Project 2: DNS & HTTP Detective

## Description

Command-line investigation work. Armed with `nslookup` and `curl`, you'll profile five real websites entirely from the terminal — no browser allowed until the final comparison step. For each site you'll uncover its DNS story (records, aliases, nameservers, signs of CDNs) and its HTTP front door (redirect chains, status codes, headers, server fingerprints), then compile your findings into a comparative "case file." The goal is fluency: by the end, `nslookup` and `curl` should feel like natural extensions of your hands, and you should be able to sketch a site's infrastructure without ever seeing its pixels.

Pick five targets with variety: one global giant (e.g. a big tech or media brand), one mid-size company, one university or government site, one small personal blog you like, and one site of your own choosing.

## Difficulty

**Beginner–Intermediate.** Estimated effort: 3–5 hours.

## Chapters Used

- Chapter 2 (IP addresses, ping/tracert, latency)
- Chapter 3 (DNS records, resolution chain, resolvers, TTLs)
- Chapter 4 (curl, methods, status codes, redirect chains, headers)
- Chapter 5 (HTTPS redirects, certificates via curl -v, HSTS)
- Chapter 9 (server fingerprints and CDN evidence — light preview)

## Requirements Checklist

Produce a case file (`detective-case-file.md`) containing, **for each of the five sites**:

- [ ] **DNS profile**: A and AAAA records; NS records (who runs their DNS); MX records (who handles their mail); any CNAME chain for the `www` name, followed link by link
- [ ] Results from querying **two different resolvers** (your default and a public one like `1.1.1.1`), noting whether answers differ and a hypothesis why
- [ ] **Latency**: average ping RTT, and the first three hops of a `tracert` (or a note if the site blocks ping — itself a finding)
- [ ] **HTTP front door**: what `http://` (plain) returns — full redirect chain hop by hop (status code + `Location` at each step) until the final 200
- [ ] **HTTPS profile**: from the final HTTPS response, at least six headers with explanations, specifically hunting for: `server`, anything cache/CDN-related (`age`, `x-cache`, `cf-ray`, etc.), `strict-transport-security`, and `content-type`
- [ ] **Certificate glimpse**: from `curl -v` output, the certificate's subject and issuer
- [ ] A one-paragraph **infrastructure verdict**: who seems to host it, whether a CDN is involved (cite your evidence), and anything surprising

And once, across the whole case file:

- [ ] A **comparison table**: all five sites × (IPv6 yes/no, mail provider, DNS host, CDN verdict, HSTS yes/no, server header)
- [ ] A short **methods appendix**: the exact commands you used, so a reader could reproduce any finding
- [ ] Two or three sentences on your **most interesting single discovery**

## Hints

- `nslookup -type=NS example.com` and `-type=MX` are your census tools; the *names* in the answers (e.g. `awsdns`, `cloudflare`, `google`) are themselves evidence.
- `curl.exe -i` shows one hop; following a chain manually (request the `Location` you got, repeat) teaches more than `-L` — but `-L` plus `-v` is fine once you've done one chain by hand.
- In PowerShell, pipe long curl output through `Select-String` to hunt headers: `curl.exe -sI https://site | Select-String "server|age|cache|strict"`.
- Mail hosted at `google.com`/`outlook.com` MX names tells you the company's email provider — detective work is mostly reading names carefully.
- Different IPs from different resolvers usually spells CDN (geographic DNS); identical single IPs everywhere suggests one origin server.
- If `tracert` shows `* * *` rows, that's routers declining to answer — note it and move on; the early hops still tell their story.

## Stretch Goals

- For one site, look up its IP's owner using a whois web service or `curl.exe https://ipinfo.io/<ip>` — does the network owner match your CDN verdict?
- Run the same DNS queries at two different times of day and diff the answers; explain any churn using TTLs.
- Add a sixth "site" that is actually a URL shortener link (e.g. any `bit.ly` link you make) and document its redirect chain as a bonus exhibit.
- Only now open the browser: load each site with the Network tab open and check your infrastructure verdicts against the Domain column and certificate viewer. Score your own detective work out of 5.
