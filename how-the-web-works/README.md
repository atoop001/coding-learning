# How the Web Works — Learning Track

A self-paced track for understanding what actually happens between a browser and a server: the internet under the web, DNS, HTTP(S), the browser's rendering machinery, state and identity, APIs, hosting, and modern web architecture. The spine of the whole track is one question — **"what happens when you type a URL and press Enter?"** — introduced in Chapter 1 and answered completely, with your own measurements, by the end.

Everything is hands-on and Windows-friendly: the tools are the browser DevTools Network tab, `curl.exe`, `nslookup`, `ping`/`tracert`, and one-line Python local servers. No frameworks, no installs beyond what you already have.

**Pairs well with the `html-css` and `javascript` tracks** (and your Python track): this track explains the pipes those languages travel through. HTML/CSS skills are used to build the sites you'll serve and inspect; JavaScript's `fetch` appears from Chapter 8 onward; Python powers your local servers. You can run this track fully in parallel — Chapters 1–5 need almost no coding at all.

## Chapters (`learning-docs/`)

Study in order — each builds on the last.

| # | File | Covers |
|---|------|--------|
| 1 | `01-the-big-picture.md` | Clients, servers, URL anatomy, the full journey (the track's spine) |
| 2 | `02-the-internet-under-the-web.md` | IP addresses, packets, TCP vs UDP, ports, localhost |
| 3 | `03-dns-names-to-numbers.md` | Domains, record types, the resolution chain, caching/TTL, hosts-file experiment |
| 4 | `04-http-fundamentals.md` | Request/response anatomy, methods, status codes, headers; DevTools + curl fluency |
| 5 | `05-https-and-tls.md` | TLS conceptually, certificates and the chain of trust, the padlock's real meaning, mixed content |
| 6 | `06-inside-the-browser.md` | Parsing and the render pipeline, resource discovery, HTTP caching, cookies, localStorage |
| 7 | `07-state-and-identity.md` | Sessions, tokens, JWT, what "logged in" actually means, authn vs authz |
| 8 | `08-apis-and-rest.md` | What APIs are, JSON, REST conventions, pagination, versioning, rate limits, fetch and CORS |
| 9 | `09-servers-and-hosting.md` | What a web server is, static vs dynamic, where code runs, hosting, load balancers, CDNs |
| 10 | `10-modern-web-and-performance.md` | SPAs vs server-rendered vs hybrid, performance, dev/staging/prod, the complete request walkthrough |

Every chapter contains: Overview, Definitions & explanations (with ASCII diagrams), Hands-on examples (do them — they are the actual learning), Common misconceptions, and 3–5 practice exercises (no solutions provided — that's deliberate).

## Projects (`projects/`)

Guided investigation/build specs, easiest to hardest. No solution code anywhere — the specs give requirements, hints, and stretch goals.

| # | File | What you do | Do it after |
|---|------|-------------|-------------|
| 1 | `01-anatomy-of-a-page-load.md` | Document a real site's full load story from the DevTools Network tab | Chapters 1–4 (6 helps) |
| 2 | `02-dns-and-http-detective.md` | Profile five sites' DNS + HTTP infrastructure entirely from the terminal | Chapters 1–5 |
| 3 | `03-build-and-inspect.md` | Serve your own static site locally and reconcile every browser request with the server log | Chapters 1–6 |
| 4 | `04-api-field-guide.md` | Systematically map a public REST API with curl, then build a fetch-powered page on it | Chapters 4, 6–8 |
| 5 | `05-capstone-deploy-and-narrate.md` | Deploy a site publicly over HTTPS and write the evidence-backed journey of one request to it | All ten |

## Suggested cadence

A comfortable pace is **one chapter per week with its exercises, projects interleaved** — about 8–10 weeks total alongside your other tracks:

- **Weeks 1–2**: Chapters 1–2 (+ every hands-on example). Start poking at every site you visit with the Network tab.
- **Week 3**: Chapter 3, then **Project 1**.
- **Week 4**: Chapters 4–5, then **Project 2**.
- **Week 5**: Chapter 6, then **Project 3** (uses your html-css track output).
- **Week 6**: Chapter 7 — do the login-inspection examples on real accounts.
- **Week 7**: Chapter 8, then **Project 4** (uses your javascript track's fetch skills).
- **Week 8**: Chapter 9; start the capstone's Act One (deploy early, even ugly).
- **Weeks 9–10**: Chapter 10, then finish **Project 5** — the walkthrough document is the track's graduation piece.

Faster is fine; skipping hands-on sections is not. The commands and DevTools sessions *are* the course — the prose just explains what you saw.

## Ground rules for getting the most out of it

- Type every command yourself; don't paste without predicting the output first.
- Keep a running lab notebook (a markdown file) of commands, outputs, and surprises — Projects 1–5 all get easier if you do.
- When anything web-related breaks in your other tracks, come back here and ask: *which step of the journey failed?* That habit is the entire point.
