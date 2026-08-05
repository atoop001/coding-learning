# Chapter 5: HTTPS & TLS — Securing the Pipe

## Overview

Everything in Chapter 4 had an alarming property: it was plain text. Anyone positioned between you and the server — the coffee-shop Wi-Fi owner, your ISP, any router on the path — could read every request and response, including passwords, and could *modify* them in flight. **HTTPS** fixes this by running ordinary HTTP inside an encrypted, authenticated tunnel called **TLS** (Transport Layer Security).

This chapter explains what TLS actually guarantees (and what it doesn't), how the handshake works conceptually, what **certificates** are and why your browser trusts them, what the padlock really means, and what **mixed content** is. No heavy math — the goal is a working mental model good enough to reason about security warnings, deploy a site with HTTPS (your capstone), and never mistake the padlock for "this site is trustworthy."

## Definitions & Explanations

### The three guarantees of TLS

When you connect over HTTPS, TLS provides:

1. **Confidentiality** — the traffic is encrypted; eavesdroppers see gibberish.
2. **Integrity** — tampering is detectable; a modified message fails verification and is rejected.
3. **Authentication** — the server proves it really is `example.com`, not an impostor, via a certificate.

What observers on the network *can still see*: that you connected to `example.com`'s IP, roughly when, and how much data moved. What they *cannot* see: which pages/paths you visited, your cookies, form contents, or anything in the HTTP layer. The URL's hostname leaks (it's needed for routing and the handshake); the path and everything after it does not.

```
Plain HTTP:                          HTTPS:
  GET /medical-results  <- visible     [encrypted blob]   <- only endpoints
  Cookie: session=...   <- visible     [encrypted blob]      can read it
  <html>...             <- visible     [encrypted blob]
```

### The key exchange problem, and the two kinds of crypto

- **Symmetric encryption**: one shared key locks and unlocks. Fast — but how do two strangers agree on a key over a wire everyone can read?
- **Asymmetric (public-key) cryptography**: a mathematically-linked **key pair**. What one key encrypts, only the other decrypts; and holding the public key can verify signatures made by the private key. Slow, but solves the stranger problem: the server publishes its public key freely and guards its private key.

TLS combines them: asymmetric crypto is used briefly at the start to authenticate the server and agree on a fresh shared secret; then everything switches to fast symmetric encryption using that secret for the rest of the session.

### The TLS handshake (conceptually)

Right after the TCP connection opens (step [5] of Chapter 1's journey), before any HTTP flows:

```
CLIENT                                        SERVER
  |                                             |
  |-- ClientHello ---------------------------->  |
  |   "I speak TLS 1.3; here are my ciphers;    |
  |    I want the site named example.com"       |
  |                                             |
  |<-- ServerHello + CERTIFICATE ------------   |
  |   "TLS 1.3 it is. Here's my certificate     |
  |    proving I'm example.com; here's key      |
  |    material for our shared secret."         |
  |                                             |
  | [client verifies certificate against its    |
  |  trusted CA list; both sides derive the     |
  |  same session key from exchanged material]  |
  |                                             |
  |== everything from here on is encrypted ====>|
  |   GET / HTTP/1.1 ...   (normal HTTP inside) |
```

Details vary by version (TLS 1.3 streamlined this to fewer round trips), but the shape is stable: negotiate, authenticate, agree on a key, switch to symmetric encryption. The handshake costs extra round trips — one reason HTTPS setup latency exists and why connections are reused.

### Certificates and the chain of trust

A **certificate** is a signed digital document that says, roughly: *"The holder of this public key is the legitimate operator of the domain example.com. Signed, a Certificate Authority."*

- A **Certificate Authority (CA)** is an organization (Let's Encrypt, DigiCert, etc.) whose job is verifying domain control before signing. To get a cert for `example.com`, you must prove you control that domain — typically by serving a challenge file or creating a DNS record the CA checks.
- Your OS and browser ship with a **trust store**: ~100+ **root CA** certificates baked in. A site's certificate is trusted if it chains, signature by signature, up to one of those roots:

```
Root CA (in your browser's trust store, self-signed, heavily guarded)
   └── signs → Intermediate CA certificate
                  └── signs → example.com's certificate  (what the server presents)
```

- Certificates **expire** (typically ~90 days for Let's Encrypt, renewed automatically) and can be issued for one name, several names, or wildcards (`*.example.com`). The industry trend is toward even shorter lifetimes — options down to about **6 days** are rolling out — which only works at all because renewal is automated; the shorter the cert, the more the whole scheme depends on that automation never being forgotten.
- **Let's Encrypt** made certificates free and automated in 2015, which is why the web went from mostly-HTTP to mostly-HTTPS in under a decade — and why your capstone deployment will get HTTPS without paying anyone.

If any check fails — expired cert, name mismatch (cert says `example.com`, you asked for `exarnple.com`), or an untrusted signer — the browser throws the full-page warning you've seen (`NET::ERR_CERT_...`). That warning means "I cannot verify who you're talking to," which is exactly the condition an attacker impersonating a site would create. Take it seriously.

### What the padlock does NOT mean

The padlock (or, in current browsers, the mere *absence* of a "Not secure" label) means one thing: *the connection is encrypted and the server holds a valid certificate for this domain name*. It does **not** mean the site is honest, safe, or the site you intended. `paypa1-login-security.com` can get a perfectly valid certificate in minutes — the connection to the scammer is beautifully encrypted. Verify the *domain name itself*; the padlock only vouches for the pipe.

### Mixed content

An HTTPS page that loads sub-resources (scripts, images, stylesheets) over plain `http://` has **mixed content** — a hole in the armor:

- **Active mixed content** (scripts, stylesheets, iframes): an attacker who tampers with that one insecure script controls the whole "secure" page. Browsers **block** this outright.
- **Passive mixed content** (images, media): lower risk; browsers block or auto-upgrade these too in modern versions, and mark the page as not fully secure.

Practical consequences you'll hit in real life: after deploying a site to HTTPS, any hard-coded `http://` asset URLs will break or warn. Fixes: use `https://` URLs, or better, relative/protocol-less paths. Related hardening you'll meet: **HSTS** (a header telling browsers "only ever contact me via HTTPS") and the automatic `http→https` redirect virtually every site runs (you captured one in Chapter 4's redirect exercise).

### Other security headers, briefly

HSTS isn't the only response header doing defensive work — you don't need to master these yet, just recognize them when `curl -i` shows them:

- **`Content-Security-Policy` (CSP)** — tells the browser which *sources* are allowed to supply scripts, styles, images, etc. for this page, e.g. `Content-Security-Policy: script-src 'self'` permits scripts only from the page's own origin, blocking injected `<script>` tags from anywhere else — a major defense against XSS (Chapter 7).
- **`X-Content-Type-Options: nosniff`** — stops the browser from guessing ("sniffing") a resource's type from its content instead of trusting the server's declared `Content-Type`; without it, a file uploaded as an "image" that's actually script can sometimes get executed.

Worth a habit: `curl -i` any real site and skim its response headers for these — most large sites (GitHub, your bank) set several of them, most small ones set none.

## Hands-On Examples

### 1. Inspect a real certificate in the browser

1. Visit `https://en.wikipedia.org`.
2. Click the icon left of the URL (padlock or "tune" icon) → **Connection is secure** → **Certificate is valid** (wording varies by browser).
3. In the certificate viewer, find: **Issued to** (Common Name / SANs — which domains it covers), **Issued by** (the intermediate CA), validity dates, and the **Details/Hierarchy** tab showing the full chain up to a root. Note how many names one certificate can cover.

### 2. Watch the handshake with curl

```powershell
curl.exe -v https://example.com -o NUL
```

Read the `*` lines in the output. Expected fragments (versions/ciphers may differ):

```
* Connected to example.com (93.184.216.34) port 443
* schannel: ...  (or: TLS handshake, using TLSv1.3 / cipher ...)
* Server certificate:
*  subject: CN=example.com   (or similar)
*  issuer:  C=US; O=...; CN=...
> GET / HTTP/1.1
```

Everything before the first `>` happened *before any HTTP was spoken* — that's the TCP connect plus TLS handshake, steps [4] and [5] of the journey.

### 3. See certificate validation fail (safely)

The site `badssl.com` exists purely to demonstrate TLS failures:

- In the browser, visit each of: `https://expired.badssl.com`, `https://wrong.host.badssl.com`, `https://self-signed.badssl.com`. Read each warning page carefully — note the distinct error codes.
- With curl:

```powershell
curl.exe https://expired.badssl.com
```

Expected: an error like `curl: (60) SSL certificate problem: certificate has expired` and **no page content** — curl refuses, exactly as it should. Now observe the escape hatch and why it's dangerous:

```powershell
curl.exe -k https://expired.badssl.com -o NUL -s -w "%{http_code}"
```

`-k` disables verification — fine against a test site, a terrible habit anywhere real, because it reopens the impersonation hole TLS exists to close.

### 4. See mixed content blocked

Visit `https://mixed-script.badssl.com` with DevTools open (**Console** tab). Expected: a console message like *"Mixed Content: The page ... was loaded over HTTPS, but requested an insecure script ... This request has been blocked."* Check the Network tab — the insecure request shows as blocked.

### 5. Compare HTTP vs HTTPS availability

```powershell
curl.exe -i http://neverssl.com -s | Select-Object -First 5
curl.exe -i http://wikipedia.org -s | Select-Object -First 5
```

Expected: `neverssl.com` (kept deliberately on plain HTTP for testing captive Wi-Fi portals) answers `200` over HTTP; Wikipedia instead answers with a `301` pointing its `Location` at `https://...` — the standard modern pattern: plain HTTP exists only to redirect you to HTTPS.

### 6. Your local server has no HTTPS — notice it

Run `python -m http.server 8000` and open `http://localhost:8000`. Note the browser labels it "Not secure" (or shows no padlock) — plain HTTP. Browsers grant `localhost` special leniency (it's treated as a secure context since traffic never leaves your machine), but the label reminds you: your dev server speaks unencrypted HTTP, and that's fine locally and unacceptable publicly.

## Common Misconceptions

- **"The padlock means the site is trustworthy/legitimate."** It only means the connection to *whatever domain is in the URL bar* is encrypted and authenticated. Phishing sites have padlocks. Read the domain.
- **"HTTPS hides which sites I visit."** The destination (hostname and IP) is visible to your network; the *content and paths* are not. Full anonymity requires different tools (VPN, Tor), which just move the trust elsewhere.
- **"HTTPS is slow/expensive, only needed for banks."** Modern TLS overhead is negligible, certificates are free (Let's Encrypt), and browsers punish HTTP sites ("Not secure," blocked features like geolocation). HTTPS is simply the default for everything now.
- **"Encryption means authentication."** They're separate guarantees. Encryption without knowing *who's on the other end* would happily secure your conversation with an attacker. That's why certificates — the *authentication* half — matter so much, and why bypassing cert warnings defeats HTTPS entirely.
- **"HTTPS means the site handles my data safely."** TLS protects data *in transit* only. The server can still store your password in plain text, sell your data, or get hacked. The pipe is secure; the endpoints are on their own.
- **"A certificate error is probably just a glitch — click through."** A cert error is indistinguishable from an active attack by design. On a network you don't control, never click through on a site that matters.

## Practice Exercises

1. **Certificate field study.** Inspect certificates (via the browser viewer) for three sites: a big tech site, your bank or university, and a small personal blog. For each record: who issued it, validity period, and which domain names it covers. Which CA appears most? How long are the validity windows?
2. **badssl bingo.** Visit at least six different subdomains of `badssl.com` (browse `https://badssl.com` for the menu — try expired, wrong.host, self-signed, untrusted-root, rc4 or other weak-crypto ones your browser rejects). For each, note the exact browser error code and write one sentence on what real-world failure or attack it simulates.
3. **Handshake narration.** From memory, write the TLS handshake as a short dialogue script between Browser and Server, including the certificate check against the trust store and the switch to symmetric encryption. Then verify against this chapter and patch what you missed.
4. **Redirect + HSTS hunt.** Using `curl.exe -i http://<site>` on five well-known sites, record which redirect to HTTPS and with which status code. Then re-request the HTTPS version with `-i` and search the response headers for `strict-transport-security`. Which sites send it, and with what `max-age`?
5. **Threat-model a coffee shop.** Write a short paragraph: you're on open café Wi-Fi. For (a) an HTTP site and (b) an HTTPS site, list exactly what a malicious Wi-Fi operator could see and could alter in your traffic. End with which single browser warning, if clicked through, would collapse the HTTPS protections back to the HTTP case.
