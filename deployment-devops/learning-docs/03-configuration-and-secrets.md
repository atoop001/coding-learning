# Chapter 3: Configuration & Secrets in Production

## Overview

The same code has to behave differently in different environments: talk to a local database in dev, the real one in prod; use a test payment key on staging, the live key in production; log verbosely on your laptop, quietly on the server. The professional answer to "how?" is a strict rule: **code is identical everywhere; everything that varies lives in configuration, supplied by the environment**. This chapter teaches that contract — environment variables, `.env` files, how platforms inject config in production — and its high-stakes subset: **secrets**. Mishandled secrets are the classic junior-developer catastrophe: an API key pushed to a public repo gets scraped by bots in *minutes* and can turn into a five-figure cloud bill overnight. You'll learn the 12-factor config principle, how secret managers work conceptually, and the one procedure everyone eventually needs: what to actually do when a secret leaks.

## Definitions & Explanations

**Configuration (config)** — every value your app needs that can differ between environments: database URLs, API keys, port numbers, feature flags, log levels, allowed CORS origins. Litmus test from the 12-factor methodology: *could you open-source your codebase right now without exposing anything sensitive or environment-specific?* Whatever would stop you is config that's wrongly living in code.

**Environment variable (env var)** — a named string value attached to a process by whatever launched it. Every OS supports them; every language reads them (`process.env.DATABASE_URL` in Node, `os.environ["DATABASE_URL"]` in Python). They're the universal config channel precisely because they live *outside* your code and *outside* your repo — the environment hands them to the process at start time.

**The deployment contract** — a useful mental frame: your app publishes a list of env vars it requires ("I need `DATABASE_URL`, `SESSION_SECRET`, and optionally `PORT`"), and every environment that wants to run it must supply them. Dev supplies them via a `.env` file; production supplies them via the platform dashboard. The app neither knows nor cares which environment it's in — it just reads its variables. Documenting this contract (see `.env.example` below) is what makes your app deployable by someone other than you.

**`.env` file** — a plain-text file of `KEY=value` lines in your project root, loaded at startup by a library (`dotenv` in Node — or natively via `node --env-file=.env` in modern Node; `python-dotenv` in Python). It exists to make the env-var contract convenient *in development only*. Two iron rules: it is **always gitignored**, and it is **never used in production** — production platforms inject real env vars directly.

**`.env.example`** — the committed, safe sibling of `.env`: same keys, placeholder values, no secrets. It documents the contract so a new developer (or you, on a new machine) knows what to fill in. Every project with config should have one.

**Secret** — config whose exposure causes harm: database passwords, API keys, signing/session secrets, OAuth client secrets, tokens. All secrets are config; not all config is secret (`PORT=3000` is config, not a secret). Secrets get all the rules of config *plus*: minimal distribution, never logged, and rotatable.

**12-factor config (Factor III)** — from the influential twelve-factor app methodology (12factor.net): *store config in the environment*. Not in code, not in committed files, not in a `config.prod.js` with real credentials. The payoff is that one build artifact runs anywhere — the environment differentiates it. This single principle is why Docker images (Chapters 4–5) and PaaS platforms all standardize on env vars.

**Config injection** — how production gets its values: you enter them once in the platform's dashboard (Render's "Environment" tab, Railway's "Variables", GitHub Actions' "Secrets"), the platform stores them encrypted, and sets them as real env vars on your process at launch. No file involved.

**Secret manager** — dedicated infrastructure for secrets at organizational scale: AWS Secrets Manager, HashiCorp Vault, Azure Key Vault. They add centralized storage with encryption, per-service access control ("the billing service may read the payment key; nothing else may"), audit logs of every access, and automated rotation. Conceptually: a password manager for programs. You won't need one personally for a while — a PaaS's env-var store is a perfectly respectable starter secret store — but you should recognize the names and the why.

**Secret rotation** — replacing a secret with a new value and retiring the old one. Done on a schedule in mature orgs, and done *immediately* when a secret leaks. Rotation is the only real remedy for exposure — which is why systems are designed so secrets are *cheap to change*.

**Secret scanning** — automated services (GitHub's push protection among them) that recognize credential formats in commits and block or alert. A safety net with holes in it — it knows famous formats, not your homegrown ones. Never a substitute for the gitignore discipline.

**The leak playbook** — when a secret hits a public repo (or any untrusted place), in order: **(1) Rotate first.** Generate a new credential at the provider and revoke the old one — *this* is what ends the danger. **(2) Update** every environment that used it. **(3) Then clean up** the repo history if you like — but understand that once pushed publicly, scrapers have already seen it; history-scrubbing is hygiene, not remedy. **(4) Check for abuse** (provider usage dashboards, billing alerts). The classic mistake is doing (3) first and feeling safe.

**Config drift** — when environments accumulate undocumented differences: a variable set by hand in the prod dashboard months ago that no `.env.example` mentions, a staging-only flag nobody remembers. Drift is why "works in staging, fails in prod" investigations start with diffing the two environments' variable lists. Defense: `.env.example` is kept ruthlessly current, and every dashboard variable you add gets added there (with a placeholder) in the same sitting.

**Blast radius** — the useful security question to ask of any secret: *if this leaked, what exactly could an attacker do?* A read-only weather-API key has a tiny radius; `DATABASE_URL` is total. Radius determines how carefully a secret is stored and how fast you rotate — and it's why granting the narrowest possible permissions per key (least privilege, which returns in Chapter 9's IAM) is worth the setup friction.

## Code Examples

The contract, end to end, in Node:

```text
# File: .env            <- gitignored. Real local values.
DATABASE_URL=postgres://postgres:localdevpassword@localhost:5432/myapp_dev
SESSION_SECRET=example-not-a-real-secret
PORT=3000
```

```text
# File: .env.example    <- committed. Same keys, placeholders, no secrets.
DATABASE_URL=postgres://user:password@host:5432/dbname
SESSION_SECRET=<generate-a-long-random-string>
PORT=3000
```

```js
// server.js — the app just reads its environment. It has no idea which env it's in.
require("dotenv").config();   // loads .env into process.env IN DEV; harmlessly finds nothing in prod

const port = process.env.PORT || 3000;          // default for convenience
const dbUrl = process.env.DATABASE_URL;

if (!dbUrl) {                                   // fail fast, loudly, at startup —
  console.error("FATAL: DATABASE_URL is not set");  // not with a cryptic crash an hour later
  process.exit(1);
}
```

Same idea in Python:

```python
import os
from dotenv import load_dotenv
load_dotenv()                                   # dev convenience; no-op in prod

DATABASE_URL = os.environ["DATABASE_URL"]       # KeyError at startup if missing — good!
DEBUG = os.environ.get("DEBUG", "false") == "true"   # optional, with a safe default
```

Working with env vars in PowerShell (syntax differs from bash — worth internalizing):

```powershell
# Set for THIS shell session only (bash equivalent: export MY_KEY="hello")
$env:MY_KEY = "hello"

# Read it back (bash: echo $MY_KEY)
$env:MY_KEY

# One-off for a single command: PowerShell has no bash-style `VAR=x cmd` prefix; do it in two steps
$env:NODE_ENV = "production"; node server.js

# List everything your processes inherit:
Get-ChildItem env: | Sort-Object Name
```

Locking the door:

```powershell
# .gitignore must contain .env BEFORE the file ever exists. Verify:
Get-Content .gitignore | Select-String "^\.env"

# Paranoia check — is .env tracked anyway? (Empty output = good.)
git ls-files | Select-String "\.env$"

# If it IS tracked: untrack it (keeps the local file), commit, AND ROTATE EVERY VALUE IN IT —
# gitignore does not apply to already-tracked files, and history still holds the old contents.
git rm --cached .env
```

A Windows-specific caution while we're on env vars — `setx` exists and you'll see it in old tutorials:

```powershell
# setx writes a PERSISTENT user-level env var into the registry:
#   setx MY_KEY "value"        <- DON'T do this for project config
# Why not: it applies to every future shell and every project, invisibly —
# the opposite of the per-project .env contract, and a classic source of
# "works on my machine (because of a var I set in 2024 and forgot)".
# Project config belongs in the project's .env; $env:X = "..." for one-offs.
```

Generating a decent secret locally (for session secrets and the like):

```powershell
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

In production there is no file — you'd enter each key/value in the platform dashboard (Render: service → Environment; Railway: service → Variables), and the platform sets them on your process. Same `process.env.DATABASE_URL` line of code reads them. That symmetry is the whole design.

A startup-validation pattern worth copying into every project — the "fail fast" idea, generalized:

```js
// config.js — one module owns the contract; the rest of the app imports from here.
const required = ["DATABASE_URL", "SESSION_SECRET"];
const missing = required.filter((key) => !process.env[key]);

if (missing.length > 0) {
  // Name the keys, never the values. This message will appear in deploy logs.
  console.error(`FATAL: missing required env vars: ${missing.join(", ")}`);
  process.exit(1);
}

module.exports = {
  databaseUrl: process.env.DATABASE_URL,
  sessionSecret: process.env.SESSION_SECRET,
  port: Number(process.env.PORT || 3000),        // env vars are ALWAYS strings —
  debug: process.env.DEBUG === "true",           // convert types deliberately, once, here
};
```

Two subtleties baked into that example that bite people who skip them:

```text
1. Env vars are strings. "false" is a truthy string! PORT is "3000", not 3000.
   Convert at the boundary (the config module); the rest of the app deals in real types.

2. Exactly one module reads process.env; everything else imports the config object.
   Scattering process.env.X across forty files makes the contract undiscoverable —
   the config module IS the documentation.
```

What GitHub's push protection looks like when it catches you — worth recognizing so it doesn't read as a random Git error:

```text
remote: - GITHUB PUSH PROTECTION
remote:   Push cannot contain secrets
remote:   —— Stripe API Key ————————————————————
remote:    locations:
remote:      - commit: a1b2c3...  path: server.js:14
```

The push is *blocked* — the secret never reached GitHub, which means (this one time) you don't have to rotate. Fix the commit (move the value to `.env`, amend or rebase it out of history), then push again. If you ever see the *bypass* option: it exists for false positives, not for "I'm in a hurry."

Auditing history for past sins — do this once per old project as you adopt these habits:

```powershell
# Search every version of every file ever committed for suspicious words:
git log -p --all | Select-String -Pattern "password|secret|api_key|token" | Select-Object -First 40

# Anything real that turns up: the playbook. Rotate first. History cleanup second, if at all.
```

## Common Pitfalls

1. **Committing `.env` "just this once" to get a deploy working.** Public or private repo, it's now in history forever, cloned to every machine, visible to every future collaborator. Correction: gitignore `.env` in your project templates *before writing any secrets*, and use the platform dashboard for production values. If it already happened: run the leak playbook — rotate first.

2. **Hardcoding a key in source "temporarily."** `const API_KEY = "..."` ships to Git the moment you commit. Temporary hardcodes have a way of becoming permanent. Correction: the very first time a secret enters a project, it enters via `process.env` and `.env`. Building the habit is the entire defense.

3. **Scrubbing Git history instead of rotating.** Force-pushing a cleaned history feels decisive but does nothing about the bots that scraped the key within minutes of the push. Correction: rotation is the remedy; scrubbing is cosmetics. Rotate, update environments, then tidy history if you want.

4. **Frontend "secrets."** Anything a React/Vite app reads at build time (`VITE_*`, `REACT_APP_*`, `NEXT_PUBLIC_*`) is bundled into JavaScript served to every visitor — inspectable in devtools by anyone. Correction: browsers can't keep secrets. Truly secret keys belong on a backend that the frontend calls; frontend env vars are for *non-secret* config (API base URLs, feature flags) only.

5. **Missing-variable failures that surface late and cryptically.** Without `SESSION_SECRET`, some apps boot fine and then fail weirdly on the first login attempt — in production, at night. Correction: validate required env vars at startup and exit immediately with a message naming the missing key ("fail fast"). Your deploy logs then tell you exactly what's wrong.

6. **Sharing secrets over chat/email to teammates (or your other laptop).** Every channel a secret transits is a place it can leak, and chat history is forever. Correction: use the platform's env store as the source of truth; for person-to-person, use a password manager's sharing feature. And notice each secret's blast radius: the fewer places it exists, the better.

7. **One secret reused everywhere.** Same password for local Postgres, staging, and prod means one leak compromises everything, and rotation becomes a multi-environment ordeal. Correction: distinct values per environment, always — this also makes it obvious *which* environment leaked when a credential shows up somewhere it shouldn't.

## Practice Exercises

1. **Retrofit the contract.** Take one of your existing Node or Python projects and move every environment-dependent or sensitive value into env vars: create `.env`, `.env.example`, gitignore verification, dotenv loading, and startup validation that exits with a clear message when a required key is missing. Prove the validation works by renaming `.env` and starting the app.

2. **The open-source test.** Audit that same project against the 12-factor litmus: could you make the repo public right now? Grep yourself honest — search the codebase for `password`, `secret`, `key`, `token`, and your own username, and list what you find and where it should live instead.

3. **PowerShell fluency drill.** Without notes: set an env var in your shell, read it, prove a child Node/Python process inherits it (`node -e "console.log(process.env.X)"`), then open a *new* PowerShell window and show it's gone. Write one sentence on why that scoping is exactly what you want from dev config.

4. **Leak drill (safe simulation).** Create a throwaway private repo. Commit a fake "secret" (`FAKE_KEY=example-not-a-real-secret-123`), push, then walk the full playbook as if it were real: write down the rotation step you *would* take at the provider, remove the file properly (`git rm --cached`, gitignore), and inspect `git log -p` to confirm the "secret" is still visible in history. State in your notes why that visibility means rotation had to come first.

5. **Read a real contract.** Find any popular open-source web app on GitHub and locate its `.env.example` (or equivalent config docs). For each variable, classify: secret or plain config? Required or optional? What would break without it?

6. **Frontend exposure proof.** In any built Vite/React project that uses a `VITE_*` variable, run the production build and find the variable's value inside the files in `dist/` (Select-String is your friend). Attach the one-line conclusion to your notes: *the browser bundle is a publication, not a vault.*
