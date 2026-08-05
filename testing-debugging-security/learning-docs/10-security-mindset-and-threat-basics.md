# Chapter 10: Security Mindset & Threat Basics

## Overview

The last third of this track is about writing code that stays safe when the world is *not* friendly. Everything you've built so far assumed users who type sensible things into forms. Security starts with dropping that assumption. This chapter builds the defensive mindset: what attackers actually want from your app (yes, even your small hobby app), where trust boundaries lie, why "never trust input" is the first commandment, and how defense in depth keeps one mistake from becoming a catastrophe. This is a *defensive* track: the goal is to find and fix weaknesses in your own code, not to attack anyone else's — and that framing matters legally and ethically as well as practically.

## Definitions & Explanations

**Security** — protecting three properties, often abbreviated **CIA**:
- **Confidentiality** — secrets stay secret (passwords, emails, personal data).
- **Integrity** — data can't be tampered with (nobody edits someone else's grades).
- **Availability** — the service keeps working (nobody can crash or flood it).
Every vulnerability you'll study breaks at least one of these.

**Threat model** — a structured answer to four questions: *What am I protecting? From whom? What can go wrong? What am I doing about it?* You don't need formal methodology at this stage — a paragraph per app is transformative. A to-do app's threat model differs wildly from a tutoring-payments app, and the defenses should differ accordingly.

**What attackers actually want.** Beginners assume "nobody would bother attacking my little app." Wrong, for two reasons. First, most attacks are *automated*: bots scan every public IP and every deployed site for known weaknesses, no human deciding you're "worth it." Second, your app is valuable even if its data isn't: attackers want your server's CPU (crypto mining, spam relay), your users' *reused passwords* (tried against their email and bank accounts), your domain's reputation (phishing host), and any personal data at all (sold in bulk). "Not worth attacking" is not a defense; being unattractive to *automated* attacks — by not having the common holes — is.

**Trust boundary** — any line in your system where data crosses from a less-trusted zone into a more-trusted one. The big ones for a web developer:
- Browser → server: **everything** arriving in a request — form fields, URL parameters, headers, cookies, JSON bodies — was composed on a machine you don't control. Client-side validation is a *convenience for honest users*; an attacker talks to your server directly (with `curl`) and never runs your JavaScript. Juniors are also expected to recognize the GUI/CLI tools that do the same job as `curl` with a friendlier interface — Postman, Insomnia, HTTPie — for building and replaying requests during normal API poking, not just attacks.
- Server → database: data heading into queries (Chapter 11's SQL injection lives here).
- Server → browser: data heading into HTML (Chapter 11's XSS lives here).
- Your code → third-party services and packages: they can be compromised too (Chapter 12).
Rule: **every crossing gets validation, and output crossings get encoding.**

**Never trust input** — the operational meaning: at each boundary, treat incoming data as *hostile until proven otherwise*. Two strategies:
- **Allowlist (preferred)** — define exactly what's acceptable and reject everything else ("username: 3–20 chars of `[a-z0-9_]`").
- **Blocklist (weak)** — enumerate known-bad patterns and strip them ("remove `<script>`"). Blocklists lose forever, because attackers know more encodings than you do (`<ScRiPt>`, `%3Cscript%3E`, event handlers, nested tags…).

**Defense in depth** — multiple independent layers, so one failure isn't fatal: validate input *and* use parameterized queries *and* encode output *and* run with least privilege *and* keep backups. Each layer assumes the previous one has already failed. Its sibling principle, **least privilege**, says every component gets the minimum access it needs — your web app's DB account doesn't need `DROP TABLE` rights, so it shouldn't have them; then even a successful injection can do less harm.

**Fail closed** — when a security check errors out (the auth service is down, the token doesn't parse), deny access. Failing *open* ("couldn't verify, let them in") converts every outage into a breach.

**Responsible practice** — you test security *only* on systems you own or have explicit written permission to test. Your own local apps: always fine, and that's exactly what this track uses. Deliberately vulnerable practice environments you host yourself (e.g., OWASP Juice Shop on localhost): fine. Anyone else's site, even "just to check": illegal in most jurisdictions and out of scope here. When you find a vulnerability in software you use, report it privately to the maintainer (responsible disclosure) rather than publishing it.

## Code Examples

### The trust boundary made concrete

```python
# A Flask route where beginners trust three things they shouldn't.
from flask import Flask, request

app = Flask(__name__)

# VULNERABLE THINKING: "the form only has a quantity dropdown of 1-5,
# so quantity is always 1-5." The FORM constrains honest browsers.
# curl constrains nothing:
#   curl -X POST http://localhost:5000/order -d "item=book&quantity=-100"
@app.post("/order")
def order_vulnerable():
    item = request.form["item"]
    quantity = int(request.form["quantity"])   # crashes on "abc"; accepts -100
    price = PRICES[item]                       # KeyError on unknown item
    return {"total": price * quantity}         # negative total = free money?
```

```python
# FIXED: validate at the boundary, allowlist-style, fail with clear 400s.
@app.post("/order")
def order_safe():
    item = request.form.get("item", "")
    if item not in PRICES:                       # allowlist: known items only
        return {"error": "unknown item"}, 400

    raw_qty = request.form.get("quantity", "")
    if not raw_qty.isdigit():                    # digits only -> also bans "-100"
        return {"error": "quantity must be a positive integer"}, 400
    quantity = int(raw_qty)
    if not (1 <= quantity <= 5):                 # enforce the business rule HERE,
        return {"error": "quantity must be 1-5"}, 400   # not just in the dropdown

    return {"total": PRICES[item] * quantity}
```

The lesson generalizes: *the server re-checks everything the client claims* — quantities, prices (never accept a price from a form!), user IDs, roles.

### Client-side checks are UX, not security (JavaScript)

```javascript
// register.html snippet — this is FINE as user experience:
form.addEventListener("submit", (e) => {
  if (password.value.length < 12) {
    e.preventDefault();
    showError("Password must be at least 12 characters");
  }
});
// ...but the SERVER must enforce the same rule, because this code runs on
// the attacker's machine and can be deleted in DevTools in five seconds.
// Rule of thumb: client-side validation is a mirror of server-side
// validation, never a replacement.
```

### Defense in depth, sketched for one feature

```text
Feature: "user downloads their invoice PDF"    GET /invoices/<invoice_id>

Layer 1 (validate):    invoice_id must match [0-9]+  -> else 400
Layer 2 (authenticate): is anyone logged in?          -> else 401
Layer 3 (authorize):   does THIS user own THIS invoice? (query by
                       invoice_id AND user_id)        -> else 404
Layer 4 (least priv):  the file-serving code can read only the invoices
                       folder, not the whole disk
Layer 5 (audit log):   log.info("invoice download", user, invoice_id)

Layer 3 is where beginners slip: checking "logged in" but not "owns it"
is the bug class called IDOR (Insecure Direct Object Reference) —
changing /invoices/17 to /invoices/18 in the URL bar and getting
someone else's invoice. Authorization must bind the RESOURCE to the USER.
```

### A five-minute threat model (template you'll reuse in Project 5)

```text
App: TNT Tutoring session booker (Flask + SQLite)

Assets (what's worth protecting?)
  - Student names, emails, session notes  (confidentiality)
  - Booking records                       (integrity — nobody edits others')
  - The booking service itself            (availability during sign-up week)

Entry points (where does untrusted data enter?)
  - Login form, booking form, profile-photo upload, URL parameters,
    the session cookie

Attackers & motives (realistic, not Hollywood)
  - Automated scanners probing every form for SQLi/XSS       (constant)
  - A curious student changing IDs in URLs to see others' notes
  - Password-stuffing bots trying leaked email/password pairs

Top risks -> planned defenses
  1. SQLi via booking form      -> parameterized queries      (Ch. 11)
  2. IDOR on /notes/<id>        -> ownership check in query   (this ch.)
  3. Stored XSS in session notes-> escape output              (Ch. 11)
  4. Leaked passwords           -> hash with bcrypt           (Ch. 12)
```

## Common Pitfalls

- **"Security by obscurity"** — hoping nobody finds the admin page at `/secret-admin-do-not-visit`. Scanners enumerate common paths in seconds. Correction: obscurity can be a *bonus* layer, never a load-bearing one; the admin page needs real authentication.
- **Validating on the client only.** The dropdown/maxlength/regex in HTML disappears the moment someone uses curl. Correction: server-side validation is the real one; client-side is a courtesy copy.
- **Authenticating without authorizing.** "User is logged in" does not mean "user may see record 18." Correction: every query for user-owned data includes the owner in the lookup (`WHERE id=? AND user_id=?`).
- **Blocklist input filtering.** Stripping `<script>` misses forty other vectors. Correction: allowlist what's valid; encode on output (Chapter 11 shows how).
- **Trusting hidden form fields, cookies, and headers** because "users can't see them." Every byte of the request is editable. Correction: hidden fields carry only non-authoritative data; anything security-relevant is stored or verified server-side.
- **Fail-open error handling.** `try: check_permission() except: pass` turns every exception into an open door. Correction: on any failure in a security check, deny and log.
- **All-powerful database accounts.** The app connects as an admin that can drop tables. Correction: least privilege — grant SELECT/INSERT/UPDATE on the specific tables the app needs.

## Practice Exercises

1. Write a five-minute threat model (using the template above) for one of your real past projects: assets, entry points, three realistic attackers, top four risks with planned defenses. Keep it — Project 5 builds on it.
2. Take any form-handling route in one of your Flask/Express apps and list every field it receives. For each: what does the server currently assume, and what should the allowlist rule be? Implement the server-side validation with proper 400 responses.
3. Demonstrate to yourself that client-side validation is decoration: build (or reuse) a page with a JS-validated form, then bypass it — submit directly with `curl` or by deleting the handler in DevTools — against your own local app. Then add the server-side check that makes the bypass harmless.
4. Hunt for IDOR in your own code: find every route that fetches a record by ID from a URL. For each, answer: "if a logged-in user changes the number, whose data do they get?" Fix any route where the answer isn't "only their own," and write the test proving it (request another user's ID, expect 403/404).
5. For the invoice-download feature sketch, write the actual defense-in-depth table for a feature in *your* app (e.g., "delete my account," "view session notes"): five layers, one line each, and mark which layers currently exist versus which are missing.
