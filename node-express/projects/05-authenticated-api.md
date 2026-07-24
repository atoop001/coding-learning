# Project 5: Authenticated Notes API

## Description

Build a personal-notes API where data belongs to *someone*. Users register with an email and password, log in, and from then on every note they create, read, update, or delete is theirs alone — two logged-in users hitting the same endpoints see completely different data, and a logged-out visitor sees nothing but `401`s.

This is the project where security stops being a chapter and becomes muscle memory: passwords are hashed with bcrypt (never stored, never logged, never returned), authentication is enforced by middleware rather than per-route good intentions, login is rate-limited, security headers come from helmet, and every secret and environment-specific value lives in environment variables — with a committed `.env.example` documenting them and the real `.env` ignored by git.

You must choose your session mechanism: **cookie-based sessions or JWTs**. Both are acceptable. As part of the project you'll write a short justification of your choice — not because one answer is right, but because "I can defend this tradeoff" is exactly what interviews and code reviews demand.

## Difficulty & Estimated Effort

Advanced− — 10–14 hours.

## Chapters Used

- Chapter 9: Databases from Node
- Chapter 10: Authentication & sessions
- Chapter 11: Security & configuration

(Plus everything from chapters 5–8 and 13 in the background — layered structure, validation, and centralized errors are assumed from here on.)

## Requirements

**Foundation & config**

- [ ] Layered project structure (as in Project 4) with SQLite; `users` and `notes` tables in the schema, with `notes` holding a foreign key to `users`
- [ ] All configuration (port, session/JWT secret, bcrypt cost, rate-limit numbers) comes from environment variables loaded via dotenv, read in one config module — no `process.env` scattered through the codebase
- [ ] `.env` is git-ignored; `.env.example` is committed with every variable named and a placeholder value like `REPLACE_ME` (never anything that looks like a real secret)
- [ ] The app refuses to start with a clear message if a required secret is missing

**Registration**

- [ ] `POST /api/auth/register` validates email format and a minimum password policy (length at least; document what you chose and why)
- [ ] Passwords are hashed with bcrypt before storage; the plain password exists only in the request's lifetime
- [ ] The database enforces email uniqueness (unique constraint), and a duplicate registration returns a clean, consistent error — decide deliberately how specific that error message should be
- [ ] No response, log line, or error message ever contains a password or a hash — audit for this explicitly

**Login, session mechanism, logout**

- [ ] A short paragraph (in the README or a `DECISIONS.md`) justifying your choice of sessions vs. JWT for *this* app, naming at least one concrete tradeoff you accepted
- [ ] `POST /api/auth/login` verifies credentials with bcrypt's comparison function and establishes the session (sets the session cookie, or issues the token)
- [ ] Failed login returns `401` with an identical message and timing-of-response behavior whether the email was unknown or the password wrong
- [ ] If cookies: `httpOnly` set, `sameSite` considered, `secure` flagged for production; if JWT: an expiry is set, the secret comes from env, and you've decided (and documented) where the client should store it
- [ ] `POST /api/auth/logout` works, and you can explain what it actually does in your chosen mechanism (destroy server session vs. what "logout" even means for a JWT)
- [ ] `GET /api/auth/me` returns the current user (id, email — never the hash) or `401`

**Protected, per-user data**

- [ ] An auth middleware — one function, applied to every `/api/notes` route — rejects unauthenticated requests with `401` in the standard error shape and attaches the current user for downstream handlers
- [ ] Full CRUD on `/api/notes` (title, body, timestamps), validated, with the centralized error handler from previous projects
- [ ] Every notes query is scoped to the authenticated user *in the SQL/repository layer* — list only lists yours, and fetching another user's note by its ID returns `404` (decide why `404` rather than `403`, and document it)
- [ ] Prove isolation: register two users, create notes as each, and verify neither can list, read, update, or delete the other's notes by any route — including guessing IDs

**Hardening**

- [ ] helmet is applied
- [ ] Rate limiting protects `POST /api/auth/login` (and register) with tight limits; the rest of the API may have a looser general limit
- [ ] All auth-relevant inputs are validated before any database or bcrypt work happens
- [ ] Manual end-to-end pass recorded in the README: register → login → create/read/update/delete notes → logout → verify protected routes now 401 — with the tool you used (REST client or `Invoke-RestMethod` with a session variable)

## Hints

- Chapter 10's session-vs-JWT comparison is the input for your justification paragraph. A useful forcing question: "what happens the moment I need to revoke access — and how much do I care for this app?"
- Testing cookie-based auth from PowerShell: `Invoke-RestMethod -SessionVariable s` on login, then `-WebSession $s` on later calls. For JWTs, capture the token and send it in an `Authorization: Bearer` header. If this feels fiddly, a REST client (VS Code extension, Insomnia, Postman) is a fine choice.
- bcrypt's compare function exists so you never compare hashes yourself. If you find yourself hashing the login attempt and string-comparing, stop and reread Chapter 10 — you're missing what the salt does.
- "Scoped in the repository layer" means functions like `findNoteForUser(noteId, userId)` — the `WHERE` clause carries both. If the scoping only happens in an `if` inside a controller, one forgotten check equals a data breach. Make the unsafe query impossible to call.
- The identical-error requirement for login exists to prevent user enumeration. Think through *all* the places behavior could differ: message text, status code, and how long the response takes when the email doesn't exist (does bcrypt still run?).
- Rate limiting: `express-rate-limit` is the standard choice. Read its docs on what "per" means (per IP) and think about what limits are strict enough to matter without locking yourself out while testing. Env-configure the numbers so dev and prod can differ.
- Generate your session/JWT secret with something like `node -e "console.log(crypto.randomBytes(32).toString('hex'))"` locally, put it in `.env`, and never in a file that's committed.

## Stretch Goals

- Add password change (requires the current password) and think through what should happen to existing sessions/tokens when it succeeds.
- Add note sharing: an owner can mark a note public and it becomes readable (only) at a public URL — this forces you to separate "authenticated" from "authorized."
- If you chose JWTs: implement a refresh-token flow, and revisit your justification paragraph — did the complexity change your opinion? If you chose sessions: move session storage out of memory into SQLite and explain why the default memory store is dev-only.
- Add account lockout or exponential backoff after repeated failed logins for one account (distinct from IP rate limiting), and consider the denial-of-service tradeoff it introduces.
- Add an admin role that can list users (never hashes) — role checks as a second layer of middleware after authentication.
