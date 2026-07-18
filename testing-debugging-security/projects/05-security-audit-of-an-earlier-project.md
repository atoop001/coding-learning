# Project 5: Security Audit of One of Your Earlier Projects

## Description

Time to point the security lens at your *own* real code. Choose a web project you built in an earlier track — ideally a Flask or Express/Node app with forms, a database, or user accounts (a to-do app, a blog, a booking tool, a small API). You will threat-model it, run it through a structured security-audit checklist, document every finding with severity, and *fix the findings* — each fix locked in with a regression test where possible. This is defensive work end to end: you are hardening software you own. If you don't have a suitable past project, build a small intentionally-naive Flask CRUD app first (a public "guestbook + admin" is perfect) and audit that.

The deliverable is threefold: a threat model, an `AUDIT.md` findings report, and a hardened codebase with tests.

## Difficulty

**Advanced.** Estimated effort: 10–14 hours.

## Chapters used

- Chapter 10 — Security Mindset & Threat Basics (threat model, trust boundaries, IDOR)
- Chapter 11 — Common Web Vulnerabilities (XSS, SQLi, CSRF, OWASP Top 10 map)
- Chapter 12 — Secrets, Auth & Safe Practices (hashing, secrets, HTTPS, dependency audit)
- Chapter 9 — Systematic Debugging & Prevention (regression tests for each fix)
- Chapter 6 — Integration Testing (Flask `test_client` to prove fixes)

## Requirements checklist

**Threat model (`THREAT-MODEL.md`)**
- [ ] Assets, entry points, three realistic attackers, and top risks — using Chapter 10's template
- [ ] Every trust boundary in the app named explicitly (browser→server, server→DB, server→browser, server→third-parties)

**Audit (`AUDIT.md`) — one entry per finding: location, category (map to OWASP Top 10), severity, why it's exploitable, the fix**
- [ ] **Injection sweep:** every SQL query classified parameterized vs concatenated; every place user data reaches HTML classified escaped vs raw (`| safe`, `innerHTML`, `dangerouslySetInnerHTML`)
- [ ] **Access control sweep:** every route listed with its required auth; every by-ID lookup checked for IDOR (can changing the ID reach another user's data?)
- [ ] **CSRF sweep:** every state-changing route checked for token protection and correct method (no state-changing GETs)
- [ ] **Secrets sweep:** grep the repo *and its git history* for hardcoded keys/passwords/URLs; check `.gitignore` covers `.env`
- [ ] **Auth sweep:** password storage (plaintext? fast hash? bcrypt?); login error messages (do they leak which field failed?); session cookie flags (`HttpOnly`/`Secure`/`SameSite`); `SECRET_KEY` strength and source
- [ ] **Config sweep:** debug mode off for "production"? default credentials anywhere? verbose error pages leaking stack traces to users?
- [ ] **Dependencies:** `npm audit` / `pip-audit` run, output captured, findings triaged (severity + runtime-vs-dev + decision)

**Remediation**
- [ ] Every High/Critical finding fixed; each Medium/Low either fixed or explicitly accepted with a written reason
- [ ] Injection fixes locked with regression tests (e.g., `' OR '1'='1` returns nothing; a `<script>` comment renders escaped) via `test_client`
- [ ] At least one IDOR fix proven by a test: user A requests user B's resource → 403/404
- [ ] Password storage migrated to bcrypt if it wasn't already, with the Project-6-style tests (stored value ≠ password; equal passwords → different hashes)
- [ ] Secrets moved to `.env` + environment; `.env.example` added; any historically-committed secret marked for rotation in the report
- [ ] `AUDIT.md` closes with a before/after summary table: finding, severity, status

## Hints

- Audit with the checklist, not vibes — coverage is the whole value. An empty section ("no raw SQL found") is a real, reportable result, not a skipped step.
- Grep is your friend for the sweeps: search for query-building patterns, `| safe` / `innerHTML`, `request.args`/`request.form`, `SECRET_KEY`, and obvious secret prefixes (`sk_`, `AKIA`, `password=`). Chapter 11's "escape hatches" list is your grep target list.
- Reproduce before you fix (Chapter 7): actually send `' OR '1'='1` to your own login, actually post a `<script>` comment and watch it fire *locally*, before and after — proof beats belief, and it's your app so it's fully in-scope.
- Rank severity by realistic impact × ease: an unauthenticated SQLi on the login is Critical; a reflected XSS reachable only by an already-logged-in admin is lower. Say *why* in each entry.
- Don't gold-plate into paranoia — a localhost hobby app doesn't need a WAF. Fix the real holes (the OWASP-mapped ones), note the theoretical ones, move on. Judgment is part of the skill.
- Keep the app runnable throughout; fix in small commits so a regression test can accompany each.

## Stretch goals

- Add security headers (`Content-Security-Policy`, `X-Content-Type-Options`, etc.) and explain in the report what each mitigates.
- Add login rate-limiting and a test that the 6th rapid attempt is throttled.
- Set up a pre-commit secret scanner (e.g., gitleaks) and show it catching a deliberately-planted fake key before commit.
- Re-run the entire audit checklist against a *second* past project and compare which mistakes recur — those are your personal patterns to design against.
