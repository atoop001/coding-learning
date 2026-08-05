# Chapter 11: Security & Configuration

## Overview

This chapter is about running an API that faces the real internet, where every endpoint gets probed by bots within minutes of going live. Two themes intertwine. **Configuration**: secrets and environment-specific settings (database paths, ports, session secrets) must live outside your code, in environment variables — you'll learn the `.env` convention, dotenv, and Node's built-in `--env-file`. **Security hardening**: sensible HTTP headers with helmet, CORS configured deliberately instead of copy-pasted, rate limiting so a script can't hammer your login endpoint, and a recap of input handling through the lens of the **OWASP API Security Top 10** — the industry's list of how APIs actually get broken. None of this is optional polish; the difference between a hobby project and a hireable portfolio piece is often exactly this chapter. Everything here assumes the middleware model from Chapter 6 and the auth flows from Chapter 10.

## Definitions & Explanations

- **Environment variable** — a key-value pair provided by the operating system to a process when it starts. In Node they appear on **`process.env`** (always as strings). They exist *outside* your code, so the same code can run with different settings in development, testing, and production.
- **`.env` file** — a plain-text file of `KEY=value` lines in your project root, used to set environment variables during local development. It is a *convenience for your machine only* and **must be gitignored** — it's where secrets live.
- **`.env.example`** — a committed template listing every variable the app needs, with placeholder values (`SESSION_SECRET=REPLACE_ME`). New developers copy it to `.env` and fill in real values. The example file documents; the real file configures.
- **dotenv** — the classic npm package that reads `.env` into `process.env`. Node 22 has this built in: `node --env-file=.env server.js` — no dependency needed, which is what this track uses.
- **Secret** — any value that grants access if leaked: session secrets, API keys, database passwords. Secrets never appear in code, in git history, or in logs.
- **Config module** — a single file that reads all environment variables in one place, validates them, applies defaults, and exports a typed object. The rest of the app imports `config`, never touches `process.env` directly.
- **helmet** — Express middleware that sets a bundle of security-related HTTP response headers with safe defaults (see examples below). One line of insurance against a class of browser-side attacks.
- **Same-origin policy** — the browser rule that JavaScript on `https://site-a.com` may not read responses from `https://site-b.com`. An **origin** is the exact combination of scheme + host + port; `http://localhost:5173` and `http://localhost:3000` are *different origins*.
- **CORS (Cross-Origin Resource Sharing)** — the mechanism by which a *server* opts in to letting specific other origins read its responses, via headers like `Access-Control-Allow-Origin`. CORS *relaxes* the same-origin policy; it is not itself a security feature you "add" — it's a hole you deliberately open, as narrowly as possible.
- **Preflight request** — for "non-simple" requests (e.g. `PUT`, `DELETE`, or anything with a `Content-Type: application/json` header), the browser first sends an `OPTIONS` request asking "may I?". The server's CORS headers in the response decide whether the real request is sent. This is why a misconfigured API "works in Postman but fails in the browser" — Postman doesn't enforce the same-origin policy.
- **Credentialed request** — a cross-origin request that includes cookies. Browsers refuse to combine `Access-Control-Allow-Origin: *` with credentials; you must name the exact origin and set `Access-Control-Allow-Credentials: true`. This is why "just use `*`" breaks session-based auth.
- **CSRF (Cross-Site Request Forgery)** — an attack where a *different* site tricks a logged-in user's browser into firing a request at yours, riding along on the cookie the browser attaches automatically. Chapter 10's `sameSite: 'lax'` (or `'strict'`) cookie flag is your main defense: it tells the browser not to send the cookie on requests that originate from another site, which covers this track's shape — a same-site single-page app talking to its own API — almost completely. You'd still reach for a dedicated **CSRF token** (a random value the server issues, the frontend echoes back on every state-changing request, and the server verifies matches) in two cases: requests that must work from an actual cross-site context, like a plain HTML form posting from another domain into yours; or supporting older browsers that don't honor `sameSite` reliably. Libraries in the `csrf-csrf`-style family exist for exactly that — recognition level for now, since `sameSite` alone covers everything you're building in this track.

- **Rate limiting** — capping how many requests a client (usually per IP) may make in a time window, answering with **`429 Too Many Requests`** beyond it. Essential on login routes to blunt password-guessing (**brute force** / **credential stuffing** — trying leaked email+password pairs from other sites' breaches).
- **OWASP** — the Open Worldwide Application Security Project, a nonprofit whose "Top 10" lists catalog the most common real-world vulnerability classes. The **OWASP API Security Top 10** is the one to internalize for backend work.

## Code Examples

### Environment variables and `.env` (Node 22 built-in)

```powershell
# PowerShell: set a variable for one command
$env:PORT = "4000"; node server.js
# (bash equivalent: PORT=4000 node server.js)
```

`.env` in the project root (gitignored!):

```ini
# .env — real values, NEVER committed
PORT=3000
DATABASE_PATH=./data/app.db
SESSION_SECRET=REPLACE_ME_with_64_random_characters
NODE_ENV=development
```

`.env.example` (committed — placeholders only):

```ini
# .env.example — copy to .env and fill in
PORT=3000
DATABASE_PATH=./data/app.db
SESSION_SECRET=REPLACE_ME
NODE_ENV=development
```

`.gitignore` must contain:

```
.env
```

Run with the built-in loader (no dotenv package needed on Node 22):

```json
// package.json
{
  "scripts": {
    "dev": "node --watch --env-file=.env server.js",
    "start": "node server.js"
  }
}
```

In production you *don't* use a `.env` file — the hosting platform injects real environment variables — which is why `start` omits the flag. (If you ever need the package instead: `npm install dotenv` and `import 'dotenv/config'` as the first import.)

### A config module: read env in ONE place

```js
// ❌ Naive: process.env sprinkled everywhere. Typos fail silently
// (process.env.SESION_SECRET is just undefined), and everything is a string.
app.listen(process.env.PORT); // undefined? "3000" the string? Who knows.
```

```js
// ✅ config.js — the only file allowed to touch process.env
const required = (name) => {
  const value = process.env[name];
  if (!value) throw new Error(`Missing required env var: ${name}`);
  return value;
};

export const config = {
  port: Number(process.env.PORT ?? 3000),      // numbers converted explicitly
  databasePath: process.env.DATABASE_PATH ?? './data/app.db',
  sessionSecret: required('SESSION_SECRET'),   // no default for secrets — crash loudly
  env: process.env.NODE_ENV ?? 'development',
  isProduction: process.env.NODE_ENV === 'production',
};
```

Crashing at startup with "Missing required env var: SESSION_SECRET" is a *feature*: the alternative is a server that runs fine until the first login, then fails mysteriously — or worse, runs with a guessable default secret.

### helmet

```powershell
npm install helmet
```

```js
import helmet from 'helmet';
app.use(helmet()); // early in the stack, before routes
```

One call sets a bundle of headers, including: `Content-Security-Policy` (restricts where scripts/styles may load from — the big anti-XSS header), `X-Content-Type-Options: nosniff` (stops browsers guessing content types), `Strict-Transport-Security` (forces future visits over HTTPS), and removes `X-Powered-By: Express` (no free reconnaissance for attackers). For a pure JSON API the defaults just work; if you later serve a frontend from Express (Chapter 14) you may need to tune the CSP.

### CORS, deliberately

```powershell
npm install cors
```

```js
import cors from 'cors';

// ❌ The Stack Overflow special: allows EVERY website on the internet to
// call your API from the browser. And it silently breaks cookie auth anyway,
// because browsers reject '*' for credentialed requests.
app.use(cors());

// ✅ Name exactly who may call you, and whether cookies are allowed:
app.use(cors({
  origin: 'http://localhost:5173',  // your React dev server — exact origin, or an array
  credentials: true,                // allow cookies; requires a named origin, not '*'
}));
```

Two things to keep straight: (1) CORS protects *browser users*, not your server — curl and Postman ignore it entirely, so it is **not** access control; auth is (Chapter 10). (2) If the browser console shows a CORS error, the fix belongs on the **server** — no frontend code can override the server's headers, and any tutorial telling the frontend to "disable CORS" is teaching you to disable your users' safety locks.

### Rate limiting

```powershell
npm install express-rate-limit
```

```js
import rateLimit from 'express-rate-limit';

// A general limit for the whole API...
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  limit: 300,                // per IP per window
  standardHeaders: 'draft-8', // send RateLimit headers so clients can back off
  legacyHeaders: false,
});
app.use('/api', apiLimiter);

// ...and a MUCH stricter one for login, where each request is a password guess:
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  limit: 10,
  message: { error: 'too many login attempts, try again later' },
});
app.use('/api/auth/login', loginLimiter);
```

The default in-memory store resets per process — fine for one server, needs a shared store (Redis) when you scale out. Also note this limits per *IP*: a real credential-stuffing defense adds per-*account* throttling, which you can build yourself as an exercise.

### Input handling recap — the whole pipeline

```js
// Every write endpoint should look like this by now:
router.post('/bookmarks', requireAuth, async (req, res, next) => {
  try {
    const data = bookmarkSchema.parse(req.body);        // Zod: shape + types (Ch. 8)
    const bookmark = bookmarks.create(req.user.id, data); // parameterized SQL (Ch. 9)
    res.status(201).json(toPublicBookmark(bookmark));   // explicit response shape
  } catch (err) { next(err); }
});
```

Validation (Zod) stops malformed data, parameterized queries stop SQL injection, and explicit response shaping stops accidental data leaks. There is no single "sanitize" magic wand — safety comes from doing the right thing at each boundary. If you ever *render user input as HTML*, that's a separate escaping problem (XSS); JSON APIs mostly sidestep it, but your future frontend must treat all API data as text, not HTML.

### The OWASP API mindset

Skim the full OWASP API Security Top 10; these map straight onto what you've built:

| OWASP item | Plain English | Your defense |
|---|---|---|
| Broken Object Level Authorization | User A reads/edits user B's data by changing an id in the URL | Ownership checks on every resource (Ch. 10) |
| Broken Authentication | Weak passwords, unhashed storage, no rate limit on login | bcrypt, session flags, `loginLimiter` |
| Excessive Data Exposure | `res.json(row)` ships hashes and internal fields | Explicit response shaping |
| Unrestricted Resource Consumption | No limits: huge bodies, unlimited requests | `express.json({ limit: '100kb' })`, rate limiting |
| Injection | User input interpreted as SQL/commands | Parameterized queries, always (Ch. 9) |
| Security Misconfiguration | Default secrets, stack traces in responses, missing headers | Config module that crashes on missing secrets; central error handler that hides internals; helmet |

The mindset behind the list: **every request is hostile until proven otherwise**, and each layer assumes the layer before it failed.

## Common Pitfalls

1. **Committing `.env` to git.** Once a secret touches git history it's compromised forever — deleting the file later doesn't remove old commits, and bots scan GitHub for exactly this. Correction: add `.env` to `.gitignore` *before* creating it; commit only `.env.example` with `REPLACE_ME` placeholders. If a real secret ever lands in history, rotate (change) the secret — don't just delete the file.
2. **Forgetting `process.env` values are always strings.** `if (process.env.FEATURE_ON)` is true even when the value is the string `"false"`, and `PORT` is `"3000"` not `3000`. Correction: convert and validate once, in the config module.
3. **`app.use(cors())` with no options, everywhere, forever.** It allows every origin, breaks credentialed cookie requests anyway, and usually gets cargo-culted to production. Correction: named `origin` + `credentials: true`, driven from config so dev and prod differ.
4. **Treating CORS as access control.** "Only my frontend's origin is allowed, so my API is private" — false; anything outside a browser ignores CORS. Correction: CORS controls *browser reading*, authentication controls *access*. You need both, for different reasons.
5. **No rate limit on login.** An attacker gets unlimited free guesses at every user's password. Correction: a strict per-window limiter on auth routes specifically, plus the general API limiter.
6. **Leaking internals through error responses.** Stack traces, SQL error text, or `"user not found"` vs `"wrong password"` all feed attackers. Correction: central error handler (Chapter 8) returns generic messages in production and logs the details server-side instead.
7. **Different behavior in dev and prod because config is scattered.** `secure: true` cookies break local HTTP dev, so beginners comment security out — and ship it commented out. Correction: make settings *conditional on config* (`secure: config.isProduction`), never on commenting code in and out.

## Practice Exercises

1. Retrofit your bookmarks API: create `config.js` as the only file reading `process.env`, a `.env` file (gitignored) and a `.env.example` (committed), and run via `node --watch --env-file=.env server.js`. Delete `SESSION_SECRET` from `.env` and confirm the app refuses to start with a clear error.
2. Add helmet, then compare `curl.exe -i http://localhost:3000/api/bookmarks` (note: `curl.exe`, not PowerShell's `curl` alias) before and after. List each new header in a `NOTES.md` with a one-line explanation of what it protects against — look up any you don't recognize.
3. Configure CORS for a frontend at `http://localhost:5173` with credentials. Then write down (before testing!) your prediction of what happens when a page on a *different* port calls your API with `fetch(..., { credentials: 'include' })` — then build a 10-line HTML page that does it, and check the browser console and Network tab (find the preflight `OPTIONS` request).
4. Add the two-tier rate limiting shown above, then write a small Node script that fires 15 rapid login attempts with `fetch` in a loop and prints each status code. Confirm the 429s, and inspect the `RateLimit-*` response headers.
5. Audit your API against the OWASP table in this chapter: for each of the six rows, write one sentence in `NOTES.md` naming the exact file/middleware in *your* code that addresses it — or a TODO if nothing does. Fix at least one TODO.
6. Cap request body size with `express.json({ limit: '10kb' })`, send a bigger payload, and observe the error. Then make your central error handler turn that specific error into a clean `413 Payload Too Large` JSON response.
