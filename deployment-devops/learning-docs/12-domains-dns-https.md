# Chapter 12: Domains, DNS & HTTPS

## Overview

Everything you've deployed so far lives at a platform-issued address — `something.onrender.com`, `yourname.github.io` — which works fine and screams "hobby project." The final upgrade is a domain of your own: `yourproject.com`, pointing wherever you deploy, surviving platform migrations, presentable on a résumé. Getting there means finally meeting DNS — the internet's name-resolution system, which the `how-the-web-works` track introduced conceptually and which you'll now *operate*: buying a domain, creating A and CNAME records, understanding TTL and propagation (the reason DNS changes "take a while" and the source of a thousand premature panics), and letting your platform handle HTTPS via Let's Encrypt. This chapter is shorter on new theory than most — DNS in practice is a small number of record types and a lot of waiting — but the practical craft here, cheap as it is, is what makes every project you ship from now on look and feel real.

## Definitions & Explanations

**Domain name** — a human-readable name (`example.com`) leased — not bought outright — from the DNS system via a registrar, renewed annually. Anatomy right-to-left: **TLD** (`.com`, `.dev`, `.io`), your registered name (`example`), and optional **subdomains** on the left (`www.example.com`, `api.example.com`). You control all subdomains of a name you register, free, via records — one domain can front your portfolio, your API, and every future project.

**Registrar** — a company accredited to lease you domains: Cloudflare Registrar (at-cost pricing, no upsell — a fine default), Namecheap, Porkbun, Google-successor Squarespace. Realistic cost: $10–15/year for a `.com`; "$1 first year" specials often hide $30 renewals — always check the *renewal* price. Two settings worth confirming at purchase: WHOIS privacy on (default at good registrars — keeps your home address out of the public database) and registrar-account 2FA (whoever controls the registrar account controls the domain — it's the master key over everything else in this chapter).

**DNS (Domain Name System)** — the distributed database mapping names to values, queried by every browser before any connection. The practical hierarchy: your machine asks a **resolver** (your ISP's, or public ones like 1.1.1.1), which walks root → TLD servers → your domain's **authoritative nameservers** — the ones holding *your records*, hosted wherever your registrar (or Cloudflare, etc.) provides. "Managing DNS" means editing records at whoever runs your authoritative nameservers.

**DNS record** — one entry in your zone: a *name*, a *type*, a *value*, and a *TTL*. The types that cover 95% of real work:
- **A record** — name → IPv4 address. `example.com → 203.0.113.10`. Used when your target is a stable IP (a VPS, or a platform that gives you a static IP for apex domains). (**AAAA** is the IPv6 twin.)
- **CNAME record** — name → *another name*. `www.example.com → myapp.onrender.com`. The alias record, and the workhorse for PaaS deployments: the platform's name can re-resolve to new IPs whenever *they* rearrange infrastructure, and your record never changes. Two rules: the value is a name, never an IP; and a plain CNAME traditionally can't sit at the **apex** (see below).
- **TXT record** — name → arbitrary text. How you *prove domain ownership* to platforms (GitHub Pages custom-domain verification, Google Search Console) and how email authentication works (SPF/DKIM/DMARC live here — recognize them; configuring email is its own rabbit hole).
- Recognize-only tier: **MX** (mail routing — set only if you attach email), **NS** (which nameservers are authoritative — normally your registrar's or Cloudflare's; changing NS moves your whole zone).

**Apex / root domain** — `example.com` itself, no subdomain. The DNS standard historically forbids a CNAME there (it conflicts with the NS/SOA records that must also exist at the apex), which is why platforms hand you either an A-record IP for the apex, or why DNS hosts invented **CNAME flattening / ALIAS / ANAME** records — apex records that *act* like CNAMEs (Cloudflare flattens automatically). This one arcane fact explains most "why is www easy but the bare domain weird?" confusion.

**TTL (Time To Live)** — each record's cache lifetime in seconds: resolvers worldwide may serve their cached copy that long before re-asking. This is **propagation** demystified: nothing is "spreading" — caches are simply *expiring at different moments*. Consequences: changes appear at different times for different people (your phone on cellular may see the new value while your desktop's resolver still caches the old one); and the pro move before any planned change is lowering TTL (e.g., to 300) a day early, so the eventual switch converges in minutes instead of hours.

**HTTPS / TLS certificate** — the padlock: a certificate proves to browsers that the server speaks for your domain, and TLS encrypts the traffic. **Let's Encrypt** (and ZeroSSL etc.) issue certificates free and automatedly — which is why every modern platform (GitHub Pages, Netlify, Vercel, Render, Fly) offers "add custom domain → we obtain and renew the certificate." The mechanics under the hood: the platform answers an ACME **challenge** — proving control of the domain via a well-known HTTP path or a DNS TXT record — gets a ~90-day certificate, and renews it forever. Your entire job is pointing DNS correctly and clicking "add domain"; certificates only become *your* problem on a raw VPS (where tools like Caddy or certbot automate the same ACME dance).

**Wildcard certificate** — one certificate covering `*.example.com` (every subdomain at that level). Platforms sometimes use these behind the scenes; if you self-host one, note that ACME issues wildcards only via the DNS challenge (you prove control by TXT record, not by serving a file).

**Domain/registrar lock & expiry protection** — registrar features guarding the domain itself: transfer lock (prevents the domain being moved to another registrar without you unlocking it — on by default at good registrars, leave it on) and auto-renew. Combined with account 2FA these are the "don't lose the master key" trio.

**WHOIS / RDAP** — the public registry of who registered each domain. Without privacy protection your name, email, and address are published there and scraped by spammers within days; with it (free at good registrars) the registrar's proxy details show instead. Look a domain up once (`whois example.com` in WSL2, or any web WHOIS) to see what the world sees.

**Redirects — the two you always configure:**
- **www ↔ apex**: pick one canonical form (`example.com` *or* `www.example.com`) and 301-redirect the other to it — both must *work*, one must *win*, or you split bookmarks, analytics, and search ranking across two "sites." Platforms and registrars offer redirect features for whichever form isn't primary.
- **HTTP → HTTPS**: every platform does this via a checkbox ("Force HTTPS" / automatic). Verify it: `http://` must 301 to `https://`. (**HSTS** is the header that tells browsers "never even try HTTP again" — recognize it; platforms often set it.)
- **301 vs 302**: permanent vs temporary. Canonicalization redirects are 301s — browsers and search engines cache the permanence, which is what you want (and why you should be *sure* before shipping a 301).

## Code Examples

Your DNS toolbox from PowerShell — no installs needed:

```powershell
# What does this name resolve to, right now, per my resolver?
Resolve-DnsName example.com -Type A
Resolve-DnsName www.example.com -Type CNAME

# Ask a SPECIFIC public resolver (bypasses your local cache — the propagation-check move):
Resolve-DnsName example.com -Type A -Server 1.1.1.1
Resolve-DnsName example.com -Type A -Server 8.8.8.8
# Two resolvers disagreeing = propagation in progress. Not broken. Waiting is the fix.

# See TTLs counting down in the answers; whose nameservers are authoritative:
Resolve-DnsName example.com -Type NS
# (On servers/WSL2 the equivalents are `dig example.com A` and `dig @1.1.1.1 example.com` —
#  same information, Unix spelling. Web-based checkers like dnschecker.org poll ~30
#  resolvers worldwide at once, which makes propagation visible on one screen.)
```

The two canonical wiring patterns, as you'd enter them in any DNS dashboard:

```text
# Pattern 1 — PaaS/static host (Render, Netlify, Vercel, GitHub Pages):
Type    Name    Value                          TTL
CNAME   www     <your-app>.onrender.com.       300     # www -> platform's name
A       @       203.0.113.10                   300     # apex -> the IP the platform
                                                       #   TELLS you to use (dashboard docs)
# "@" is dashboard shorthand for the apex. If your DNS host supports CNAME
# flattening/ALIAS at the apex, you may use the platform hostname for both — nicer.

# Pattern 2 — your own VPS (the Chapter 2 side-quest world):
A       @       <your-server-ip>               300
CNAME   www     example.com.                   300     # www aliases the apex
```

Choosing the domain itself — two minutes of naming pragmatism before you shop:

```text
- .com if available and affordable; .dev (HTTPS-enforced by the browser — neat)
  and .io/.app are respectable for developer projects. Exotic TLDs are fine for
  play but check the renewal price — some spike after year one.
- Short, typeable, no hyphens if you can help it, nothing you'd hesitate to
  say aloud in an interview.
- For a portfolio+projects setup, one good domain and generous subdomains
  (project1.you.dev, api.you.dev) beats buying a domain per project — cheaper
  and it accumulates presence in one place.
```

The end-to-end custom-domain flow (Render spelling; Netlify/Vercel/GitHub Pages differ only in dashboard geography):

```text
1. Platform dashboard → your service → Settings → Custom Domains → add example.com
   (and www.example.com). It responds with EXACTLY the records it wants — copy those,
   not a tutorial's; IPs and targets vary by platform and change over the years.
2. DNS dashboard → create those records, TTL 300 while experimenting.
3. Wait. Check with Resolve-DnsName against 1.1.1.1 until the new values appear.
4. Platform detects correct DNS → runs the ACME challenge → issues the certificate.
   ("Certificate pending" for 5–30 minutes after DNS resolves is NORMAL.)
5. Enable/verify Force HTTPS and the www->apex (or reverse) redirect.
```

What a finished small-portfolio zone actually looks like — a worked example to check your dashboard against:

```text
# Zone for example.com after this chapter + a Project 5 API + email left unconfigured:
Type    Name       Value                              TTL     Why it exists
-----   ----       -----                              ---     -------------
A       @          216.24.57.1                        3600    apex -> platform's static IP
CNAME   www        myapp.onrender.com.                3600    www -> platform hostname
CNAME   api        my-api.onrender.com.               3600    the deployed backend
TXT     @          "verification=abc123..."           3600    proved ownership to the platform
NS      @          (registrar/Cloudflare's pair)      —       who's authoritative — don't touch
# Nothing else. Every record has a one-line justification — that's the audit
# standard from pitfall 7. TTLs back up to 3600 now that the experimenting era ended.
```

Watching a lookup happen, hop by hop — run once, and DNS stops being abstract:

```powershell
# In WSL2 (dig traces better than Resolve-DnsName for this one):
dig +trace example.com A

# Output walks the actual hierarchy:
#   . (root servers)          -> "ask the .com servers, here they are"
#   com. (TLD servers)        -> "ask example.com's nameservers, here they are"
#   example.com. (YOUR zone)  -> "A = 216.24.57.1" — the authoritative answer
# Every resolver in the world runs this walk (then caches it, per TTL).
```

Verifying the finished job like a professional — each layer separately:

```powershell
# DNS layer — right values, from a neutral resolver?
Resolve-DnsName example.com -Server 1.1.1.1
Resolve-DnsName www.example.com -Server 1.1.1.1

# Redirect layer — http and the non-canonical host BOTH 301 to the canonical https?
curl.exe -sI http://example.com | Select-String "HTTP|Location"
curl.exe -sI https://www.example.com | Select-String "HTTP|Location"
# (curl.exe, not curl — bare `curl` is a PowerShell alias for Invoke-WebRequest)

# TLS layer — valid certificate, right names, sane expiry?
curl.exe -svo NUL https://example.com 2>&1 | Select-String "subject|expire|issuer"
# Browser padlock -> certificate viewer shows the same, pointier-clickier.

# App layer — and does the actual site answer?
curl.exe -s https://example.com/health
```

The registrar-checkout checklist — thirty seconds of settings that prevent every domain horror story:

```text
Before paying:
  [ ] renewal price checked (not just year-one price)
  [ ] WHOIS privacy included and ON
After paying:
  [ ] auto-renew ON, payment method that won't expire soon
  [ ] transfer lock ON (usually default)
  [ ] 2FA on the registrar account — this account outranks everything it points at
  [ ] renewal date in your calendar anyway (belt and suspenders)
While experimenting:
  [ ] TTLs at 300 — raise to 3600 when the zone stabilizes
```

And checking whether HSTS is in play on your finished setup:

```powershell
curl.exe -sI https://example.com | Select-String "strict-transport-security"
# Present -> browsers that have seen it will refuse plain HTTP for this domain
# for max-age seconds. Great hardening; also why you test redirects BEFORE
# enabling long max-age values — HSTS makes HTTPS mistakes sticky.
```

## Common Pitfalls

1. **Panicking (or re-editing everything) during propagation.** You change a record, your browser still shows the old site, so you "fix" more things — now converging on a config you no longer understand, while caches expire on their own schedule. Correction: one change at a time; verify against `1.1.1.1`/`8.8.8.8` directly rather than your own cached resolver; lower TTL *before* planned changes; give it the old TTL's worth of patience before touching anything else.

2. **Putting an IP in a CNAME, or a CNAME at the apex (where unsupported).** Both are category errors the dashboard may accept and DNS will break on quietly. Correction: CNAME values are *names*; the apex takes an A record (or your host's ALIAS/flattening feature). If the platform's instructions distinguish apex vs www setups — and they all do — follow *their* table verbatim.

3. **Configuring only one of apex and www.** Half your visitors type the other one; for them, your site is simply down, and you'll rarely hear about it. Correction: both resolve, one canonical, 301 between them — then verify both with curl, because "I always type www" is not a test plan.

4. **Copying record values (especially IPs) from tutorials instead of the platform dashboard.** GitHub Pages' A-record IPs, Render's apex IP, Vercel's targets — all platform-specific and all subject to change over the years; a stale tutorial wires your domain to nowhere. Correction: the platform's own custom-domain page is the sole source of truth for values; tutorials are for understanding *shape*.

5. **Certificate anxiety — or certificate neglect.** Anxiety: "DNS is right but the padlock's broken!" — five minutes after the change, while ACME issuance is still in flight. Neglect: hand-managing a cert on a VPS and letting it expire in 90 days, the classic self-hosting faceplant. Correction: platforms need only patience (pending → issued, minutes); self-hosted needs *automated* renewal (Caddy does it by default; certbot installs a timer) plus, ideally, your Chapter 11 uptime monitor watching the HTTPS URL — it will catch expiry before users do.

6. **Losing the domain itself.** Auto-renew off + expired card + ignored registrar emails = your domain on the open market, with your project's links and reputation attached (and re-buying from a squatter is a hostage negotiation). Correction: auto-renew on, payment method current, registrar 2FA on, and the renewal date in your calendar anyway. The domain is the one piece of your stack that's a *lease with a deadline*.

7. **DNS scatter after experiments.** Six months of trying things leaves ghost records — an A record to a dead VPS, TXT verifications for services you left — some of which are security liabilities (a CNAME to a deprovisioned platform app can enable *subdomain takeover*: someone claims that platform name and now serves content on your subdomain). Correction: audit your zone twice a year; every record should have a one-line justification you can state; delete the ones that don't.

## Practice Exercises

1. **Dry-run resolution safari.** Without owning anything yet: pick three real sites (a big one, a PaaS-hosted project you know, a GitHub Pages site) and map their DNS with `Resolve-DnsName`: apex A/AAAA (or flattened CNAME?), www CNAME chain (follow it hop by hop to the platform), NS records, TTLs. Write two sentences per site on the hosting story the records tell.

2. **Buy and configure a domain.** Choose a registrar (check *renewal* pricing, WHOIS privacy, 2FA), buy one domain you'd be happy having on a résumé, and before pointing it anywhere: inventory the default records it comes with (parking pages, registrar MX) and decide which survive. Set TTLs to 300 while you're in the experimenting era.

3. **Full custom-domain deployment.** Point the domain (apex *and* www) at your Project 1 static site or Project 5 API: platform custom-domain flow, records per its instructions, propagation watched via a neutral resolver until issuance completes. Then run the complete four-layer verification battery (DNS, redirects, TLS, app) from this chapter and paste the outputs into your notes.

4. **Propagation observed, TTL felt.** With everything working: note the current TTL, then change where www points (any harmless target), and record timestamps — when 1.1.1.1 sees it, when 8.8.8.8 does, when your own browser does. Change it back. Compare the timings against the TTL and write the one-paragraph explanation you'd give a junior who says "DNS is being flaky."

5. **Subdomain expansion.** Add `api.` (pointing at a deployed backend) or `staging.` (at a second service instance) under the same domain — records, platform domain-add, certificate, verification. Reflect: what did this cost? (Nothing.) This is why one domain serves a whole portfolio.

6. **Break-and-read drill.** Deliberately sabotage one layer at a time and *learn each failure's face*: delete the www CNAME (browser error? which one?), point the apex at a wrong IP (what happens — timeout, wrong site, cert warning?), request a subdomain with no record at all. Fix each, verify each. Production DNS issues will now look like old acquaintances instead of emergencies.
