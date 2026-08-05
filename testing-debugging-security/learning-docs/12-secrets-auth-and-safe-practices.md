# Chapter 12: Secrets, Auth & Safe Practices

## Overview

The final chapter covers the operational side of defensive security: how to store passwords so a database leak isn't a catastrophe, how to keep secrets (API keys, tokens, database URLs) out of your code and your git history, why HTTPS is non-negotiable, and how to keep your dependencies from becoming the hole in your fence. These practices are unglamorous, mostly mechanical — and they're the difference between an incident being "we rotated a key" versus "we leaked every user's password." Everything here becomes part of your default project setup from now on, in every track.

## Definitions & Explanations

**Password hashing** — never store passwords; store a *hash*: the output of a one-way function applied to the password. At login you hash the attempt and compare hashes. If the database leaks, attackers get hashes, not passwords. But not any hash will do:
- **Fast hashes (MD5, SHA-256) are wrong for passwords.** They're designed to be fast — so attackers can try billions of guesses per second against a leaked hash. They also map equal passwords to equal hashes, so precomputed tables ("rainbow tables") crack common passwords instantly.
- **A salt** — a random value stored alongside each hash and mixed into it — defeats precomputed tables and makes identical passwords hash differently.
- **Slow, salted, tunable algorithms — bcrypt, scrypt, argon2 — are correct.** They're deliberately expensive (tens of milliseconds per attempt), turning "billions of guesses/second" into "thousands," and they handle salting for you. The libraries below make the right thing the easy thing.
- Current OWASP guidance ranks them: **Argon2id is the preferred choice where your stack/hosting supports it**; **bcrypt is the battle-tested, universally-available fallback** — still correct, just older and less tunable against modern GPU attacks.
- Related non-negotiables: never log passwords; never email passwords; password *reset* (a fresh token) not password *recovery* (you can't recover what you never stored — a site that emails your old password is storing plaintext: red flag).

**Secrets** — anything that grants access: API keys, database URLs with credentials, session-signing keys, OAuth client secrets. The cardinal sin is hardcoding them in source, because source gets copied, shared, and pushed. **Git makes this permanent**: a secret committed once lives in history forever, even after you delete it in a later commit — anyone with the repo can check out the old commit. Bots scrape public GitHub for leaked keys *within minutes* of a push. If a secret ever touches a commit, consider it burned: **rotate it** (revoke and reissue) — deleting the file is not enough.

**Environment variables & `.env` files** — the standard solution: code reads secrets from the process environment (`os.environ`, `process.env`); locally you keep them in a `.env` file that is loaded at startup and — critically — **listed in `.gitignore`** so it can never be committed. You commit a `.env.example` with the variable *names* but placeholder values, so collaborators know what to configure.

**HTTPS/TLS** — encryption between browser and server. Without it, anyone on the network path (the café Wi-Fi, the ISP) can read *and modify* traffic — including passwords and session cookies in transit. Rules: production is HTTPS-only (hosting platforms and Let's Encrypt make certificates free and automatic); session cookies get the `Secure` flag (HTTPS-only) alongside `HttpOnly` and `SameSite` from Chapter 11; `http://localhost` during development is fine.

**Dependency vulnerabilities** — your app includes thousands of lines you didn't write; a known vulnerability (CVE) in any of them is a known vulnerability in your app, and scanners actively exploit popular ones. Defenses: (1) **audit tools** — `npm audit` and `pip-audit` check your dependency tree against vulnerability databases; (2) **lockfiles** (`package-lock.json`, `requirements.txt` with pinned versions) so builds are reproducible and updates are deliberate; (3) update *regularly in small steps* rather than rarely in terrifying leaps; (4) **install skepticism** — typosquatting (malicious `requsts` impersonating `requests`) is real, so check names and download counts before installing.

**Auth vocabulary** — **authentication** (who are you?) vs **authorization** (what may you do?) vs **session management** (how do we remember between requests?). Beginner rules: use your framework's session machinery (signed cookies with a strong `SECRET_KEY` from the environment) rather than inventing your own; rate-limit login attempts; use generic failure messages ("invalid username or password") so attackers can't enumerate which usernames exist.

## Code Examples

### Password hashing: vulnerable → fixed (Python)

```python
# VULNERABLE — three escalating versions of wrong, all seen in the wild:
users["ada"] = {"password": "hunter2"}                       # plaintext: leak = game over
users["ada"] = {"password": hashlib.md5(b"hunter2").hexdigest()}   # fast + unsalted:
# cracked in seconds by rainbow tables; every "hunter2" user has the SAME hash.
```

```python
# FIXED — bcrypt via the `bcrypt` package (pip install bcrypt)
import bcrypt

def register(username, password):
    # gensalt() generates a random salt; the salt is stored INSIDE the hash
    # string, so one column stores everything needed for verification.
    pw_hash = bcrypt.hashpw(password.encode("utf-8"), bcrypt.gensalt())
    save_user(username, pw_hash)

def login(username, password):
    user = load_user(username)
    if user is None:
        # Still do a dummy check so response time doesn't reveal
        # whether the username exists (timing side-channel).
        bcrypt.checkpw(password.encode("utf-8"), _DUMMY_HASH)
        return None
    if bcrypt.checkpw(password.encode("utf-8"), user.pw_hash):
        return user
    return None          # same generic failure either way
```

```javascript
// FIXED — Node equivalent (npm install bcrypt)
import bcrypt from "bcrypt";

const ROUNDS = 12;                      // cost factor: bigger = slower = stronger

export async function register(username, password) {
  const hash = await bcrypt.hash(password, ROUNDS);   // salt handled internally
  await saveUser(username, hash);
}

export async function login(username, password) {
  const user = await loadUser(username);
  if (!user) return null;
  const ok = await bcrypt.compare(password, user.passwordHash);
  return ok ? user : null;
}
```

### Secrets: vulnerable → fixed

```python
# VULNERABLE — config.py, committed to git. All three are now burned
# the moment this is pushed anywhere:
API_KEY = "sk_live_<your-real-key-here>"
DATABASE_URL = "postgres://admin:Sup3rS3cret@db.example.com/prod"
SECRET_KEY = "dev"        # also terrible: guessable = forgeable sessions
```

```bash
# FIXED — step 1: .env file at the project root (NEVER committed)
API_KEY=sk_live_<your-real-key-here>
DATABASE_URL=postgres://app_rw:GENERATED_PW@db.example.com/prod
SECRET_KEY=6f1c9d8e2b...64+ random hex chars...
```

```gitignore
# FIXED — step 2: .gitignore (this file IS committed — ideally in your
# very first commit, before any .env exists)
.env
*.env
.venv/
__pycache__/
node_modules/
```

```python
# FIXED — step 3: code reads the environment; fails fast if unset.
# pip install python-dotenv
import os
from dotenv import load_dotenv

load_dotenv()                                    # reads .env in development

API_KEY = os.environ["API_KEY"]                  # KeyError at startup if missing
SECRET_KEY = os.environ["SECRET_KEY"]            # -> fail fast beats limping on
# In production, the hosting platform injects these as real env vars;
# same code, no .env file on the server.
```

```javascript
// FIXED — Node version. Node 20+: `node --env-file=.env app.js`
// (or: npm install dotenv; import "dotenv/config" at the top of the entry file)
const apiKey = process.env.API_KEY;
if (!apiKey) throw new Error("API_KEY is not set — check your .env");
```

```bash
# FIXED — step 4: .env.example (committed) documents WHAT to set, not the values
API_KEY=your-stripe-key-here
DATABASE_URL=postgres://user:password@localhost/dev
SECRET_KEY=generate-64-random-hex-chars
```

Generate strong keys properly: `python -c "import secrets; print(secrets.token_hex(32))"`.

### Auditing dependencies (Windows PowerShell)

```powershell
# JavaScript projects
npm audit                    # report known vulnerabilities in the dependency tree
npm audit fix                # apply safe (semver-compatible) upgrades
npm outdated                 # what's behind, generally

# Python projects (inside your venv)
pip install pip-audit
pip-audit                    # checks installed packages against vuln databases
pip list --outdated
```

Reading an audit report: each finding names the package, the severity, the vulnerable version range, and the fixed version. Triage, don't panic: a critical hole in a package you actually ship matters today; a low-severity issue in a dev-only tool goes on the list. What you must *not* do is let `npm audit` become wallpaper you scroll past — that is "Security Logging & Monitoring Failures" happening to you personally.

### Session cookie hardening recap (Flask)

```python
app.config.update(
    SECRET_KEY=os.environ["SECRET_KEY"],   # signs the session cookie
    SESSION_COOKIE_HTTPONLY=True,          # JS can't read it (limits XSS damage)
    SESSION_COOKIE_SAMESITE="Lax",         # CSRF layer (Chapter 11)
    SESSION_COOKIE_SECURE=True,            # HTTPS only — set False only for local dev
)
```

## Common Pitfalls

- **"I deleted the key and committed again — safe now."** Git history retains it; bots scan history. Correction: rotate the credential immediately; add `.env` to `.gitignore` *before* the first commit of every future project.
- **Hashing with SHA-256 "because it's cryptographic."** Cryptographic ≠ suitable for passwords; it's a *speed* problem. Correction: bcrypt/scrypt/argon2 only, via a maintained library — and never write your own crypto, for anything, ever.
- **Committing `.env.example` with real values in it** (copy-paste accident). Correction: placeholders only; eyeball the diff of anything named `*.env*` before committing — and consider a pre-commit secret scanner (e.g., gitleaks) once you're comfortable.
- **Different validation for "trusted" internal tools.** The admin panel gets no rate limit, default password `admin`. Attackers look there *first* (Security Misconfiguration). Correction: internal tools get the same rigor — often more, since they're more privileged.
- **Revealing which part of a login failed** ("no such user" vs "wrong password") — lets attackers enumerate valid usernames. Correction: one generic message, same status code, similar response time for both cases.
- **Running audits only at project start.** New CVEs are published against *old* versions continuously; your unchanged app gets more vulnerable over time. Correction: run `npm audit`/`pip-audit` on a schedule (and before every deploy); treat new criticals like failing tests.
- **A weak or default `SECRET_KEY`** (`"dev"`, `"changeme"`). Anyone who guesses it can forge session cookies and become any user. Correction: 32+ random bytes from a CSPRNG, from the environment, different per environment.

## Practice Exercises

1. Retrofit secrets management onto one of your past projects: move every hardcoded key/URL to `.env` + `os.environ`/`process.env`, add `.gitignore` and `.env.example`, and make the app fail fast with a clear message when a required variable is missing. Then check `git log -p` for previously committed secrets — if you find any, practice the response: rotate (or, for toy keys, note that you *would*).
2. Build a minimal register/login flow (Flask or Express) using bcrypt: registration hashes, login verifies, failures are generic. Write tests asserting (a) the stored value is not the password, (b) two users with the same password have different stored hashes, (c) login fails identically for wrong-password and no-such-user.
3. Run `npm audit` and `pip-audit` on your two most recent projects. For each finding: package, severity, is it a runtime or dev dependency, and your triage decision (fix now / schedule / accept with reason). Apply at least one fix and re-run to confirm.
4. Write your personal "new project security checklist" — 10 or fewer checkbox items covering this chapter plus Chapters 10–11 (e.g., `.env` ignored before first commit, cookies hardened, parameterized queries only, audits scheduled). You'll use it verbatim in Projects 5 and 6.
5. Inspect a real site's cookies (your own deployed app if you have one, or any site you use) via DevTools → Application → Cookies: which have `HttpOnly`, `Secure`, `SameSite`? For your own app, fix any gaps. Explain in two sentences why `HttpOnly` limits the damage of an XSS bug — connecting Chapter 11 to this one.
