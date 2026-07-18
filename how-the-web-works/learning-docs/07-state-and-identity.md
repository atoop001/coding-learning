# Chapter 7: State & Identity — What "Being Logged In" Actually Means

## Overview

HTTP is stateless: every request arrives at the server as a stranger. Yet you log in to a site once and it remembers you for weeks, across thousands of requests. This chapter demystifies that trick. You'll learn the difference between **authentication** and **authorization**, how **session cookies** turn stateless HTTP into a continuous logged-in experience, what **tokens** (including **JWT**) are and how they differ from sessions, why "log out" sometimes doesn't really log you out, and the two classic attacks (CSRF, XSS-driven cookie theft) that explain the cookie flags from Chapter 6.

Why it matters: identity is where the web's abstractions meet real consequences. Almost every app you'll ever build has login. Understanding the machinery — not just calling a library — is the difference between building it safely and copying incantations.

## Definitions & Explanations

### Authentication vs authorization

- **Authentication (authn)**: *who are you?* — proving identity (password, passkey, code).
- **Authorization (authz)**: *what may you do?* — permissions once identity is known.

HTTP's status codes mirror this (imperfectly named): `401` = authentication missing/failed; `403` = authenticated, but not authorized.

### The core problem, and the core trick

Logging in is one request (`POST /login` with credentials). Every *subsequent* request is, per HTTP, unrelated. The universal solution: after verifying your password once, the server hands the browser a **proof-of-identity value**, and the browser presents it with every future request. All login systems are variations on what that value is and where it's stored.

### Variation 1: Server-side sessions (the classic)

```
 1. POST /login  {username, password}
        |
 2. Server checks password hash. OK!
    Creates a session record in ITS OWN storage:
        sessions["7f3a9c..."] = {user_id: 42, created: ...}
    and replies:
        Set-Cookie: session=7f3a9c...; HttpOnly; Secure; SameSite=Lax
        |
 3. Browser automatically attaches to every later request:
        Cookie: session=7f3a9c...
        |
 4. On each request, server looks up "7f3a9c..." in its session store:
        found -> this is user 42       not found/expired -> 401, please log in
```

Key properties:

- The cookie value is a **meaningless random ID** — all real data lives server-side.
- **Logout is real**: the server deletes the session record; the ID becomes worthless instantly. Admins can revoke any session at will.
- The cost: the server must *store and look up* sessions — shared storage (e.g. Redis, a database) is needed once you run multiple server machines.
- The session ID is literally your identity while valid — anyone holding it *is you* to the server. Hence the armor: `HttpOnly` (scripts can't steal it), `Secure` (never over plain HTTP), `SameSite` (other sites can't ride along with it), expiry.

### Variation 2: Tokens — the proof carries its own data

Instead of a random ID pointing at server-side state, the server can hand out a **self-contained, signed token**: a value that *itself* says who you are, tamper-proofed by a cryptographic signature. The server then needs **no session storage** — it just verifies the signature on each request.

Tokens typically travel in a header rather than a cookie:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI0MiJ9.SflKx...
```

("Bearer" = whoever bears this token gets the access — like cash.) This is the norm for APIs (Chapter 8), mobile apps, and SPAs, because non-browser clients don't have automatic cookie machinery — they attach the header explicitly in code.

### JWT, concretely

The dominant token format is **JWT (JSON Web Token)** — three base64-encoded parts joined by dots:

```
 header            .  payload ("claims")                 .  signature
 {"alg":"HS256",      {"sub":"42",                          HMAC/RSA signature of
  "typ":"JWT"}         "name":"ada",                        header+payload using a
                       "exp":1767225600}                    key only the server holds
```

Three facts that beginners get wrong, in order of importance:

1. **JWTs are readable by anyone.** Base64 is encoding, not encryption — paste any JWT into a decoder and read the payload. Never put secrets in one.
2. **JWTs are unforgeable (if verified).** Change one character of the payload and the signature no longer matches; without the server's signing key you cannot produce a valid signature. That's the entire security model.
3. **JWTs are hard to revoke.** The server keeps no record, so a stolen or logged-out token remains *valid until its `exp` time*. Real systems mitigate with short-lived **access tokens** (minutes) paired with longer-lived **refresh tokens** that can be revoked server-side — reintroducing a little state to get the kill switch back.

### Sessions vs tokens at a glance

| | Server-side session | Signed token (JWT) |
|---|---|---|
| Server storage | Required (session store) | None for verification |
| Instant revocation / logout | Yes — delete the record | No — valid until expiry (needs workarounds) |
| Contents visible to holder | No (opaque ID) | Yes (readable claims) |
| Natural transport | Cookie (automatic) | `Authorization` header (explicit) |
| Sweet spot | Traditional server-rendered sites | APIs, mobile, third-party access, microservices |

Neither is "modern vs legacy" — they're trade-offs, and many real systems combine them.

### Third-party login and OAuth (one paragraph's worth)

"Sign in with Google/GitHub" is **OAuth**/OpenID Connect: instead of giving Site A a password, you authenticate *with Google*, which then hands Site A a token asserting your identity (and only the permissions you approved). You'll implement this someday via a library; conceptually it's the token model with a trusted third party doing the authentication step.

### Why those cookie flags exist: two attacks in brief

- **CSRF (cross-site request forgery)**: a malicious page makes *your browser* send a request to `bank.com` — and cookies historically rode along automatically, so the bank saw a validly-authenticated request you never intended. Defenses: `SameSite` cookies (the modern default) and anti-CSRF tokens embedded in forms.
- **Cookie theft via XSS**: if an attacker gets JavaScript running on a site (cross-site scripting), `document.cookie` would hand them your session — unless the cookie is `HttpOnly`, which is exactly why login cookies always are, and why storing auth tokens in `localStorage` (fully script-readable) is widely criticized.

You don't need to master these attacks now — you need to recognize that every odd-looking security flag is a scar from a real one.

## Hands-On Examples

### 1. Watch a real login set a session cookie

1. Open an incognito/InPrivate window (clean cookie slate), DevTools open → Network tab, check **Preserve log**.
2. Go to any site you have an account on (GitHub works well) and log in.
3. Find the login request (method **POST**; filter by "Doc" or search "session"/"login"). Inspect it:
   - **Payload/Request body**: your credentials went here (over HTTPS — recall Chapter 5).
   - **Response headers**: find `set-cookie` — the moment identity was granted.
4. Click any *later* request and confirm the `cookie` request header now carries that value. That silent header is your "logged in" state, on every single request.

### 2. Prove the cookie IS the login

1. Logged in on the site, open DevTools → Application → Cookies.
2. Delete the session cookie(s) (right-click → delete; on GitHub, `user_session`).
3. Refresh the page. Expected: you're logged out. Nothing about "you" changed — no logout request was sent; the browser simply lost the proof. Being "logged in" is precisely "holding a valid session cookie."

### 3. Cookie round-trip with curl

httpbin lets you watch both directions:

```powershell
curl.exe -i "https://httpbin.org/cookies/set?demo=hello"
```

Expected: a `set-cookie: demo=hello` response header (and a redirect). But curl, unlike a browser, forgets cookies by default — prove it:

```powershell
curl.exe -s https://httpbin.org/cookies
```

Expected: `{"cookies": {}}`. Now give curl a "cookie jar" so it behaves like a browser:

```powershell
curl.exe -s -c cookies.txt "https://httpbin.org/cookies/set?demo=hello" -L
curl.exe -s -b cookies.txt https://httpbin.org/cookies
```

Expected: `{"cookies": {"demo": "hello"}}`. `-c` saves cookies, `-b` sends them — you've manually recreated the browser's automatic session machinery. Delete `cookies.txt` afterwards.

### 4. Send a bearer token by hand

```powershell
curl.exe -s https://httpbin.org/bearer -w "\n%{http_code}"
```

Expected: `401` — no token, no entry. Now present one:

```powershell
curl.exe -s -H "Authorization: Bearer my-test-token-123" https://httpbin.org/bearer
```

Expected: `{"authenticated": true, "token": "my-test-token-123"}`. (httpbin accepts any token — real servers verify signatures/lookups — but the *mechanics* are exactly this: a header you attach explicitly.)

### 5. Decode a JWT and fail to forge it

1. Go to a JWT debugger (e.g. `jwt.io`) and study the sample token: flip between the encoded string and the decoded header/payload. Confirm you can read every claim — no key required.
2. Edit the payload in the debugger (change the name or `sub`). Watch the encoded token change — and note the signature is now only valid because the debugger *re-signs with the displayed secret*. Without knowing a server's real secret, your edited token would be rejected. Readable ≠ forgeable.
3. Never paste a *real* production token into any website — treat tokens like passwords.

## Common Misconceptions

- **"The server remembers I'm logged in."** With sessions, the server remembers a record *only because your browser presents the matching ID every time*; with JWTs, the server remembers nothing at all. Lose the cookie/token and you are instantly a stranger.
- **"401 means I lack permission."** 401 = not (successfully) authenticated; 403 = authenticated but not authorized. The names are historical accidents; the distinction is real.
- **"JWTs are encrypted."** Standard JWTs are *signed, not encrypted* — anyone can read the payload. The signature prevents modification, not reading.
- **"Tokens are the modern replacement for sessions."** They solve different problems. Sessions give instant revocation and opacity at the cost of server storage; tokens give statelessness at the cost of revocation. Grown-up systems often use short tokens + revocable refresh tokens — a hybrid.
- **"Logging out destroys my authentication everywhere."** With server sessions, yes (record deleted). With pure JWTs, "logout" often just deletes the client's copy — an already-stolen token keeps working until expiry. Check the `exp` before assuming.
- **"HTTPS makes login secure, full stop."** HTTPS secures transit only. Session fixation, XSS token theft, CSRF, weak password storage — all live *inside* the endpoints, past the encrypted pipe. Transport security and application security are separate jobs.

## Practice Exercises

1. **Login lifecycle diary.** In an incognito window with Network "Preserve log" on, perform a full cycle on a real site: load login page → submit credentials → browse two pages → log out. Document, request by request: where credentials traveled, where the session cookie was set, where it was sent, and what the logout request/response actually did (was the cookie cleared? how — look at `set-cookie`?).
2. **Cookie flag audit.** For the session cookie of two different sites, record all attributes (Expires, HttpOnly, Secure, SameSite, Domain, Path) from the Application tab. For each flag present, write one sentence naming the attack or failure it prevents; for each flag *absent*, note what that omission would risk.
3. **Session vs token decision memo.** You're designing (a) a server-rendered forum, and (b) a public API consumed by mobile apps. In half a page, choose sessions or tokens for each and justify with at least three properties from the comparison table.
4. **JWT dissection.** Take the standard sample JWT from a debugger site, split it on the dots yourself, and base64-decode the first two parts (Python: `base64.urlsafe_b64decode(part + "==")`). Confirm you recover the header and claims JSON without any secret. Write two sentences on why the third part stops you from minting your own admin token.
5. **curl session simulation.** Using only curl with `-c`/`-b` against `httpbin.org` (endpoints under `/cookies` and `/bearer`), script (in a `.ps1` or just a saved sequence) a mini "login flow": acquire a cookie, prove it's sent on a second request, then separately access a bearer-protected endpoint with and without the header, capturing the 401 vs 200 difference.
