# Chapter 10: Authentication & Sessions

## Overview

Almost every real backend needs to know *who* is making a request. This chapter covers authentication end to end: how to store passwords so a database leak isn't a catastrophe (bcrypt), how to build register and login routes that don't leak information to attackers, and how to keep a user "logged in" across requests using either **session cookies** or **JWTs** — including an honest comparison of the two, because the internet is full of one-sided advice on this. You'll also write auth middleware that protects routes, and implement logout for both models. Auth is the single most dangerous thing to get wrong in a backend — it guards everything else — and it's also a topic interviewers love, because it tests whether you understand HTTP's statelessness, cookies, hashing, and middleware all at once. Everything here builds directly on middleware (Chapter 6), validation and error handling (Chapter 8), and the database layer (Chapter 9).

## Definitions & Explanations

- **Authentication (authn)** — verifying *who* someone is. "This request comes from the user with email `alice@example.com`." Login is authentication.
- **Authorization (authz)** — verifying *what* an authenticated user may do. "Alice can edit *her* bookmarks, not Bob's." Authorization always comes *after* authentication; the two words look similar but are different problems.
- **Credential** — the secret a user presents to authenticate, usually a password. Your server must never store credentials in a recoverable form.
- **Hashing** — a one-way transformation: easy to compute `hash(password)`, computationally infeasible to reverse. Different from **encryption**, which is two-way (anything encrypted can be decrypted with the key). Passwords are hashed, never encrypted, precisely so that *nobody* — including you — can recover them.
- **Salt** — random data mixed into each password before hashing, so two users with the same password get different hashes, and precomputed "rainbow table" attacks fail. bcrypt generates and embeds a salt automatically; you never manage it yourself.
- **bcrypt** — a password-hashing algorithm designed to be *deliberately slow* and tunable via a **cost factor** (also called *rounds*). Cost 12 means the hashing work doubles 12 times; each +1 doubles the time. Slowness is the feature: it turns a billion-guesses-per-second cracking rig into a thousands-per-second one. General-purpose hashes like SHA-256 are *fast*, which is exactly why they're wrong for passwords.
- **Session** — server-side memory of a logged-in user. The server stores "session `<opaque-id>` belongs to user 42" and gives the browser only the opaque id, in a cookie. Every later request carries the cookie; the server looks the id up.
- **Cookie** — a small piece of data the server asks the browser to store (via the `Set-Cookie` response header) and which the browser automatically sends back on every subsequent request to that site. Cookies are the browser-native way to persist login state.
- **`httpOnly` cookie flag** — makes the cookie invisible to JavaScript (`document.cookie` can't read it). Essential for auth cookies: it means a cross-site-scripting (XSS) bug can't simply steal the session.
- **`secure` cookie flag** — the browser only sends the cookie over HTTPS. Always on in production.
- **`sameSite` cookie flag** — controls whether the cookie is sent on cross-site requests. `lax` (the modern default) or `strict` blocks most **CSRF** (cross-site request forgery) attacks, where another site tricks a logged-in browser into making requests to yours.
- **JWT (JSON Web Token)** — a self-contained, *signed* token: a base64-encoded header and payload (claims like user id and expiry) plus a cryptographic signature. The server can verify it wasn't tampered with *without storing anything* — that's "stateless" auth. Crucially, JWTs are signed, **not encrypted**: anyone holding one can read its payload.
- **Token revocation** — invalidating a credential before it expires. Trivial with sessions (delete the server-side record); genuinely hard with JWTs (the token remains valid until expiry unless you keep a server-side denylist — which reintroduces the state JWTs promised to remove).
- **Auth middleware** — an Express middleware that runs before protected route handlers, checks the session or token, attaches the user to `req`, and responds `401` if authentication fails.
- **401 vs 403** — `401 Unauthorized` really means *unauthenticated* ("I don't know who you are — log in"). `403 Forbidden` means *unauthorized* ("I know who you are, and the answer is no").

## Code Examples

Install what this chapter uses (PowerShell — same command in bash):

```powershell
npm install bcrypt express-session
```

### Hashing: the wrong ways, then bcrypt

```js
// ❌ CATASTROPHIC: plaintext. One database leak = every user's password,
// which they reuse on their email and bank.
db.prepare('INSERT INTO users (email, password) VALUES (?, ?)')
  .run(email, password);

// ❌ STILL BAD: fast hash, no salt. SHA-256 runs billions/sec on a GPU,
// and identical passwords produce identical hashes — crack one, crack all.
import { createHash } from 'node:crypto';
const hash = createHash('sha256').update(password).digest('hex');
```

```js
// ✅ bcrypt: slow on purpose, salt generated and stored inside the hash.
import bcrypt from 'bcrypt';

const COST = 12; // ~250ms per hash on typical hardware; raise as machines get faster

const passwordHash = await bcrypt.hash('correct horse battery staple', COST);
// "$2b$12$..." — the string embeds algorithm, cost, salt, and hash together.

// To check a login attempt, you never "decrypt" — you hash the attempt the
// same way and compare. bcrypt.compare does this, reading cost+salt from the
// stored hash:
const ok = await bcrypt.compare('correct horse battery staple', passwordHash); // true
const nope = await bcrypt.compare('hunter2', passwordHash);                    // false
```

Always use the async `bcrypt.hash`/`bcrypt.compare` (they run on a worker thread), never `hashSync`/`compareSync` in a server — a 250ms *synchronous* hash blocks the event loop for every other request (Chapter 3).

### Register

```js
// routes/auth.js
import { Router } from 'express';
import bcrypt from 'bcrypt';
import { z } from 'zod';
import * as users from '../data/users.js'; // your data layer from Chapter 9

const router = Router();

const registerSchema = z.object({
  email: z.string().email().toLowerCase(),
  password: z.string().min(10, 'password must be at least 10 characters'),
});

router.post('/register', async (req, res, next) => {
  try {
    const body = registerSchema.parse(req.body);   // 1. validate first, always

    if (users.findByEmail(body.email)) {           // 2. reject duplicates
      // 409 Conflict; keep the message boring — email enumeration is a real
      // attack, but on *register* you usually can't hide it (the UX needs it).
      return res.status(409).json({ error: 'email already registered' });
    }

    const passwordHash = await bcrypt.hash(body.password, 12); // 3. hash
    const user = users.create(body.email, passwordHash);       // 4. store hash only

    // 5. NEVER send the hash back. Shape the response by hand.
    res.status(201).json({ id: user.id, email: user.email });
  } catch (err) {
    next(err); // Zod errors -> your central error handler (Chapter 8)
  }
});
```

### Login — with session cookies

```js
// app.js
import session from 'express-session';

app.use(session({
  secret: process.env.SESSION_SECRET,  // from .env — NEVER hardcoded (Chapter 11)
  resave: false,                       // don't rewrite unchanged sessions
  saveUninitialized: false,            // no cookie until something is stored
  cookie: {
    httpOnly: true,                    // JS can't read it -> XSS can't steal it
    secure: process.env.NODE_ENV === 'production', // HTTPS-only in prod
    sameSite: 'lax',                   // blocks most CSRF
    maxAge: 1000 * 60 * 60 * 24 * 7,   // 7 days, in ms
  },
}));
```

```js
// routes/auth.js (continued)
router.post('/login', async (req, res, next) => {
  try {
    const { email, password } = loginSchema.parse(req.body);

    const user = users.findByEmail(email.toLowerCase());

    // ❌ Naive: different errors for "no such user" vs "wrong password"
    // tells an attacker exactly which emails have accounts.
    // ✅ One generic message for both failures:
    if (!user || !(await bcrypt.compare(password, user.password_hash))) {
      return res.status(401).json({ error: 'invalid email or password' });
    }
    // Note the order above: even when user is null we *could* run a dummy
    // compare to equalize timing. For a learning app the generic message is
    // the part that matters; know that timing equalization exists.

    req.session.userId = user.id;      // this is "logging in": store the id
    res.json({ id: user.id, email: user.email });
  } catch (err) {
    next(err);
  }
});

router.post('/logout', (req, res, next) => {
  // Sessions make logout trivial: destroy the server-side record.
  req.session.destroy((err) => {
    if (err) return next(err);
    res.clearCookie('connect.sid');    // express-session's default cookie name
    res.status(204).end();
  });
});
```

### Auth middleware and protected routes

```js
// middleware/require-auth.js
import * as users from '../data/users.js';

export function requireAuth(req, res, next) {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'authentication required' });
  }
  const user = users.findById(req.session.userId);
  if (!user) {
    // Session points at a deleted user — treat as logged out.
    return res.status(401).json({ error: 'authentication required' });
  }
  req.user = user; // handlers downstream can rely on req.user
  next();
}
```

```js
// Protect one route, a router, or everything below a line:
router.get('/me', requireAuth, (req, res) => {
  res.json({ id: req.user.id, email: req.user.email });
});

app.use('/api/bookmarks', requireAuth, bookmarksRouter); // whole resource

// Authorization is a separate check AFTER authentication:
router.delete('/bookmarks/:id', requireAuth, (req, res) => {
  const bookmark = bookmarks.findById(req.params.id);
  if (!bookmark) return res.status(404).json({ error: 'not found' });
  if (bookmark.user_id !== req.user.id) {
    return res.status(403).json({ error: 'forbidden' }); // known user, not allowed
  }
  bookmarks.remove(bookmark.id);
  res.status(204).end();
});
```

### The JWT alternative — and the honest tradeoffs

```js
// npm install jsonwebtoken
import jwt from 'jsonwebtoken';

// On login, instead of req.session.userId = ...:
const token = jwt.sign(
  { sub: user.id },                       // "subject" claim: whose token
  process.env.JWT_SECRET,                 // signing secret from env (Chapter 11)
  { expiresIn: '1h' }
);
res.json({ token });

// Auth middleware verifies the signature on every request:
export function requireAuth(req, res, next) {
  const header = req.headers.authorization ?? '';
  const token = header.startsWith('Bearer ') ? header.slice(7) : null;
  if (!token) return res.status(401).json({ error: 'authentication required' });
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = payload.sub;
    next();
  } catch {
    return res.status(401).json({ error: 'invalid or expired token' });
  }
}
```

|  | Session cookies | JWTs |
|---|---|---|
| Server state | Yes — session store | None (that's the point) |
| Logout / revoke | Instant: delete the session | Hard: token valid until expiry, unless you add a denylist (state again) |
| Browser storage | Cookie, `httpOnly` — XSS-resistant | Often `localStorage` — readable by any XSS payload |
| Scales across servers | Needs a shared store (e.g. Redis) | Any server with the secret can verify |
| Best fit | Same-site browser apps (your projects) | Service-to-service, mobile clients, third-party APIs |

For a browser app talking to your own Express backend — which is what you're building in this track — **sessions with `httpOnly` cookies are the simpler and safer default**. Reach for JWTs when a cookie genuinely can't work (native mobile apps, cross-domain APIs, microservices). "JWTs are more modern" is not a reason; instant revocation is a security feature you should be reluctant to give up.

## Common Pitfalls

1. **Storing passwords in plaintext or with a fast hash (MD5/SHA-256).** A leak exposes every password, and users reuse passwords everywhere. Correction: bcrypt (or argon2) with cost ≥ 12, always.
2. **Using `bcrypt.hashSync` in a request handler.** A quarter-second synchronous hash freezes the entire event loop — every concurrent user waits. Correction: `await bcrypt.hash(...)`; the async version runs off the main thread.
3. **Returning the password hash in API responses.** `res.json(user)` after a `SELECT *` ships the hash to the client. Correction: explicitly shape every response (`{ id, email }`) or exclude the column in the query; never spread a raw user row into JSON.
4. **Different error messages for "unknown email" and "wrong password".** This hands attackers a list of valid accounts. Correction: one generic `401 "invalid email or password"` for both.
5. **Hardcoding the session/JWT secret in code.** It ends up on GitHub, and anyone with it can forge logins. Correction: load it from an environment variable (Chapter 11) and use a long random value — placeholder in examples only, e.g. `SESSION_SECRET=REPLACE_ME_with_a_long_random_string`.
6. **Skipping the cookie flags.** Without `httpOnly` any XSS steals sessions; without `sameSite` you're open to CSRF; without `secure` the cookie travels over plain HTTP. Correction: set all three, as in the config above.
7. **Confusing 401 and 403 — and doing authorization checks nowhere.** Authenticated users editing *other users'* data is one of the most common real-world API bugs (OWASP calls it Broken Object Level Authorization). Correction: after `requireAuth`, every handler touching a resource must check ownership and return `403` (or `404` if you prefer not to reveal existence).

## Practice Exercises

1. Write a tiny standalone script (`node hash-play.js`) that hashes the same password twice with bcrypt and prints both hashes. Explain in a comment why they differ and why `bcrypt.compare` still returns `true` for both. Then time cost factors 8, 10, 12, and 14 with `console.time` and note the doubling.
2. Add `POST /api/auth/register` and `POST /api/auth/login` (session-based) to your bookmarks API from earlier chapters, with Zod validation, duplicate-email handling, and generic login errors. Verify with your REST client that the `Set-Cookie` header appears on login and that the response never contains a hash.
3. Write `requireAuth` middleware and protect all `/api/bookmarks` routes. Confirm: no cookie → 401; after login → 200; after `POST /api/auth/logout` → 401 again.
4. Add ownership: give bookmarks a `user_id`, make every list/read/update/delete scoped to `req.user.id`, and make sure user A gets a 403 or 404 (decide which, and write a comment defending the choice) when requesting user B's bookmark by id.
5. Rebuild exercise 3's protection using JWTs instead of sessions (issue on login, verify in middleware, send via `Authorization: Bearer`). Then write a short paragraph in a `NOTES.md`: what did logout require in each model, and which would you ship for a browser app, and why?
6. Deliberately break things and observe: log in, then restart your server. What happens to sessions stored in memory? Read the express-session docs on production stores and note in `NOTES.md` what you'd use instead of the default `MemoryStore`.
