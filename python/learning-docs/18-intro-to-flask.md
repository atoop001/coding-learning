# Chapter 18: Intro to Web Development with Flask

## Overview

This is the chapter the whole track has been building toward: putting your Python on the web. **Flask** is a "micro" web framework — small enough to understand completely, real enough to power production services. With everything you've learned (functions, dicts, decorators, JSON, error handling, packages/venv), Flask will feel like assembly of familiar parts: a route is a *decorated function*; a request carries *dicts* of data; an API response is *JSON*.

You'll build up from "hello, browser" to a small multi-page app with templates, forms, and a JSON endpoint — the exact skeleton of the capstone project.

## Definitions & Explanations

**Web framework** — a library handling the plumbing of web serving — parsing HTTP requests, routing URLs to your code, building responses — so you write only the interesting part. Flask's philosophy: minimal core, add what you need.

**Client / server recap** — the browser (client) sends an HTTP request (Chapter 16 — you were the client then); your Flask app (server) receives it, runs one of *your functions*, and returns a response (HTML for humans, JSON for programs). Same conversation as Chapter 16, opposite chair.

**The Flask app object** — `app = Flask(__name__)` creates the central application; everything registers onto it. `__name__` tells Flask where to find templates and static files relative to your module.

**Route** — a URL pattern bound to a function via decorator: `@app.route("/about")`. The function (called a **view function**) returns the response body. Chapter 14 paying off: `app.route("/about")` is a decorator *factory* that registers your function in the app's URL map.

**Dynamic routes** — angle brackets capture URL segments as arguments: `@app.route("/user/<username>")` → `def profile(username):`. Converters enforce types: `<int:post_id>`.

**HTTP methods on routes** — `@app.route("/add", methods=["GET", "POST"])`. Convention: **GET** shows a page; **POST** submits/changes data. Inside the view, `request.method` tells you which arrived.

**The `request` object** — `from flask import request` — a per-request bundle of incoming data, mostly dict-like:

- `request.args` — query parameters (`?q=python`) for GET
- `request.form` — submitted form fields for POST
- `request.get_json()` — a JSON body, parsed to Python

**Templates & Jinja2** — HTML files in a `templates/` folder, rendered with `render_template("page.html", name=value, ...)`. Jinja syntax inside the HTML:

- `{{ expression }}` — insert a value (auto-escaped against HTML injection)
- `{% if %} ... {% endif %}`, `{% for x in items %} ... {% endfor %}` — logic
- `{% extends "base.html" %}` + `{% block content %}` — shared layout inheritance, so navigation/styling live in one file

**Other essentials:**

- `redirect(url_for("view_name"))` — send the browser elsewhere; `url_for` builds URLs from *function names* so links survive route changes. The **Post/Redirect/Get** pattern: after a successful POST, always redirect (prevents refresh-resubmits).
- `jsonify(obj)` — build a JSON response with correct headers: your API endpoints.
- `@app.errorhandler(404)` — custom not-found pages.
- **Debug mode** — `app.run(debug=True)`: auto-reload on save + in-browser tracebacks. Development only — never in production.
- **Development server** — `python app.py` serves on `http://127.0.0.1:5000` (localhost, port 5000). It serves *your machine only*.

**Setup (per Chapter 10 discipline):**

```powershell
mkdir flask-hello && cd flask-hello
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install flask
```

## Code Examples

### Hello, browser

```python
# app.py
from flask import Flask

app = Flask(__name__)

@app.route("/")                      # the root URL
def home() -> str:
    return "<h1>Hello from Flask!</h1>"

@app.route("/about")
def about() -> str:
    return "<p>My first web app. Chapters 1-17 led here.</p>"

if __name__ == "__main__":
    app.run(debug=True)
```

Run `python app.py`, open `http://127.0.0.1:5000` — your function's return value, in a browser. Edit the string, save, refresh: debug mode reloaded automatically. Stop with Ctrl+C.

### Dynamic routes and query parameters

```python
from flask import Flask, request

app = Flask(__name__)

@app.route("/greet/<name>")                  # /greet/ada
def greet(name: str) -> str:
    return f"<h1>Hello, {name.title()}!</h1>"

@app.route("/square/<int:number>")           # /square/7 — int converter
def square(number: int) -> str:
    return f"<p>{number}² = {number ** 2}</p>"

@app.route("/search")                        # /search?q=python
def search() -> str:
    query = request.args.get("q", "")        # .get with default — it's dict-like!
    if not query:
        return "<p>Try /search?q=something</p>"
    return f"<p>You searched for: {query}</p>"

if __name__ == "__main__":
    app.run(debug=True)
```

### Templates — separating Python from HTML

Project layout:

```
flask-notes/
├── app.py
└── templates/
    ├── base.html
    └── notes.html
```

`templates/base.html`:

```html
<!doctype html>
<html>
<head><title>{% block title %}Notes{% endblock %}</title></head>
<body>
  <nav><a href="/">Home</a> | <a href="/notes">Notes</a></nav>
  <hr>
  {% block content %}{% endblock %}
</body>
</html>
```

`templates/notes.html`:

```html
{% extends "base.html" %}
{% block title %}All Notes{% endblock %}
{% block content %}
  <h1>Notes ({{ notes|length }})</h1>
  {% if notes %}
    <ul>
    {% for note in notes %}
      <li>{{ note }}</li>
    {% endfor %}
    </ul>
  {% else %}
    <p>No notes yet — add one below!</p>
  {% endif %}

  <form method="POST" action="/notes">
    <input name="text" placeholder="New note...">
    <button type="submit">Add</button>
  </form>
{% endblock %}
```

### Forms and the Post/Redirect/Get pattern

`app.py`:

```python
from flask import Flask, render_template, request, redirect, url_for

app = Flask(__name__)

notes = []                # in-memory "database" — resets on restart (see pitfalls)

@app.route("/")
def home():
    return render_template("base.html")

@app.route("/notes", methods=["GET", "POST"])
def notes_page():
    if request.method == "POST":
        text = request.form.get("text", "").strip()   # form fields: dict-like
        if text:
            notes.append(text)
        return redirect(url_for("notes_page"))        # PRG: redirect after POST
    return render_template("notes.html", notes=notes) # GET: render the page

if __name__ == "__main__":
    app.run(debug=True)
```

Walk the loop in your head: browser POSTs the form → we mutate state → redirect → browser GETs the same URL → template renders the updated list. Refreshing now re-runs only the harmless GET.

### A JSON API endpoint (Chapter 16, other chair)

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

books = [
    {"id": 1, "title": "Automate the Boring Stuff", "read": True},
    {"id": 2, "title": "Fluent Python", "read": False},
]

@app.route("/api/books")                       # GET a collection
def list_books():
    return jsonify(books)

@app.route("/api/books/<int:book_id>")         # GET one resource
def get_book(book_id):
    for book in books:
        if book["id"] == book_id:
            return jsonify(book)
    return jsonify({"error": "book not found"}), 404    # (body, status_code) tuple

@app.route("/api/books", methods=["POST"])     # CREATE via JSON body
def add_book():
    data = request.get_json(silent=True)
    if not data or "title" not in data:
        return jsonify({"error": "a 'title' field is required"}), 400
    new = {"id": max((b["id"] for b in books), default=0) + 1,
           "title": data["title"], "read": False}
    books.append(new)
    return jsonify(new), 201

if __name__ == "__main__":
    app.run(debug=True)
```

Test it with your Chapter 16 skills — `requests.get("http://127.0.0.1:5000/api/books").json()` — or just open the URL in a browser. You have now written both halves of a web API conversation.

### Custom error page

```python
@app.errorhandler(404)
def not_found(error):
    return render_template("404.html"), 404     # or a jsonify(...) for APIs
```

## Common Pitfalls

**1. Nothing at `/` — "Not Found"** — you defined `/hello` but browsed to `/`. Flask serves exactly the routes you declared; check the startup log and your `@app.route` paths. Trailing-slash mismatch (`/about` vs `/about/`) can also 404.

**2. Templates not found** — `jinja2.exceptions.TemplateNotFound: notes.html` means the file isn't in a folder literally named `templates/` next to `app.py`, or you ran Python from a different directory. Also: `render_template("notes.html")` takes the *filename*, no `templates/` prefix.

**3. Forgetting to return** — a view function that only `print()`s returns `None` → "The view function did not return a valid response." Views *return* their response; `print` goes to your terminal, not the browser.

**4. Variables not passed to the template** — `{{ notes }}` renders empty (or errors) unless you pass it: `render_template("notes.html", notes=notes)`. Every name used in the template must be handed over as a keyword argument.

**5. Expecting in-memory data to persist** — the `notes` list dies with every restart — and debug mode restarts on *every save*. That's fine for learning; real persistence means writing JSON/CSV (Chapters 11 & 16) or a database. The capstone project makes you fix exactly this.

**6. Form field name mismatches** — `request.form["text"]` raises a 400 error if the HTML input was `name="note_text"`. The `name=` attribute in HTML is the key in `request.form`. Use `.get()` while developing to avoid hard crashes.

**7. Port already in use** — `OSError: [WinError 10048]`... an old server is still running in another terminal. Ctrl+C it, or run on another port: `app.run(debug=True, port=5001)`.

**8. Hand-building URLs in templates** — hardcoded `<a href="/notes">` breaks when routes change. Prefer `{{ url_for('notes_page') }}`. Same for redirects in Python.

**9. Debug mode in the wrong place** — `debug=True` exposes an interactive debugger to anyone who can reach the server. On your laptop, great; on anything public, never.

## Practice Exercises

1. **Personal site.** Build a three-page Flask app (`/`, `/about`, `/projects`) using a shared `base.html` with a nav bar. The projects page renders a list of your track projects from a Python list of dicts using a `{% for %}` loop.
2. **Dynamic profile.** Add `/hello/<name>` that greets the visitor, and `/hello/<name>/<int:age>` that also tells them their age next year. Then add `/temp?c=25` which reads a query parameter and shows the Fahrenheit conversion — with a helpful message when `c` is missing or non-numeric.
3. **Guestbook.** A `/guestbook` page with a form (name + message) that POSTs, appends to an in-memory list, and redirects (PRG pattern). Display entries newest-first with a count. Reject empty submissions with a message in the page.
4. **Quotes API.** Build `/api/quotes` (GET returns all, POST adds one from a JSON body with validation and proper status codes) and `/api/quotes/<int:id>` (GET one, 404 JSON error when absent). Exercise every path using the `requests` library from a separate script and print the status code of each call.
5. **Persist the guestbook.** Combine Chapters 11 & 16 with this one: load the guestbook entries from `guestbook.json` at startup and save after each addition, so entries survive a restart. Confirm by restarting the server mid-session.
