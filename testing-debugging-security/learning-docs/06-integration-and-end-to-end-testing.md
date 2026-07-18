# Chapter 6: Integration & End-to-End Testing

## Overview

Unit tests with doubles prove your logic is right *assuming your assumptions about the outside world are right*. Integration tests check those assumptions: does your SQL actually run? Does the file you write really parse back? End-to-end (E2E) tests go further and drive the entire app like a user. This chapter shows how to test against real files and a real (temporary) SQLite database, how to test a Flask app through its HTTP interface, and gives you a working introduction to **Playwright** for browser E2E testing. You'll also learn the cost model — why you want these tests, but fewer of them.

## Definitions & Explanations

**Integration test** — a test where two or more real components interact: your code + the real filesystem, your code + a real database engine, module A + module B without doubles between them. The point is to catch *interface* bugs: wrong SQL syntax, wrong file encoding, mismatched JSON shapes — things unit tests with stubs are structurally blind to.

**Test fixture** — the prepared environment a test needs: a temp directory, a seeded database, a running app. Frameworks help you build and *tear down* fixtures reliably. pytest's fixtures (`tmp_path`, ones you define with `@pytest.fixture`) are the canonical example.

**Temporary resources** — integration tests should create their world from scratch and destroy it afterward: temp dirs (`tmp_path` in pytest), in-memory or temp-file SQLite (`sqlite:///:memory:`), a fresh app instance per test. Never point tests at your real development database — a cleanup bug will delete your data.

**Test client** — a fake HTTP client that calls your web app *in-process*, without a real network socket. Flask's `app.test_client()` lets you `GET`/`POST` routes and inspect responses — this tests routing, serialization, and status codes realistically at unit-test speed.

**End-to-end (E2E) test** — the full stack, no substitutions: real browser, real server, real database. For web apps, **Playwright** is the modern standard: it launches Chromium/Firefox/WebKit, navigates, clicks, types, and asserts on what's visible.

**Flakiness** — a test that passes sometimes and fails sometimes without code changes, usually due to timing (page not loaded yet), shared state, or network. E2E tests are the most flake-prone; Playwright fights this with *auto-waiting* assertions that retry until a timeout.

**Cost model** — a unit test costs milliseconds and pinpoints a function. An E2E test costs seconds-to-minutes and a failure means "somewhere in the whole stack." That's why the pyramid (Chapter 1) puts few tests at the top: use E2E to verify a handful of *critical user journeys* (sign up, log in, core action, checkout), not every edge case — push edge cases down to unit tests where they're cheap.

## Code Examples

### Real files with pytest's `tmp_path`

```python
# journal.py — code that genuinely touches the filesystem
import json
from pathlib import Path

def save_entries(path, entries):
    Path(path).write_text(json.dumps(entries, indent=2), encoding="utf-8")

def load_entries(path):
    p = Path(path)
    if not p.exists():
        return []
    return json.loads(p.read_text(encoding="utf-8"))
```

```python
# test_journal.py — tmp_path is a built-in pytest fixture: a real, unique,
# empty directory created for THIS test and auto-deleted afterward.
from journal import save_entries, load_entries

def test_round_trip_preserves_entries(tmp_path):
    file = tmp_path / "journal.json"
    entries = [{"date": "2026-07-01", "text": "Läsning & café ☕"}]  # unicode!

    save_entries(file, entries)          # real write
    result = load_entries(file)          # real read

    assert result == entries             # catches encoding bugs stubs can't

def test_loading_a_missing_file_returns_empty_list(tmp_path):
    assert load_entries(tmp_path / "nope.json") == []
```

### A real database: SQLite in a fixture

```python
# store.py
import sqlite3

def make_db(path=":memory:"):
    conn = sqlite3.connect(path)
    conn.execute("CREATE TABLE IF NOT EXISTS notes (id INTEGER PRIMARY KEY, body TEXT NOT NULL)")
    return conn

def add_note(conn, body):
    cur = conn.execute("INSERT INTO notes (body) VALUES (?)", (body,))
    conn.commit()
    return cur.lastrowid

def find_notes(conn, keyword):
    rows = conn.execute(
        "SELECT body FROM notes WHERE body LIKE ?", (f"%{keyword}%",)
    ).fetchall()
    return [r[0] for r in rows]
```

```python
# test_store.py
import pytest
from store import make_db, add_note, find_notes

@pytest.fixture
def db():
    """A fresh, real, in-memory SQLite DB per test. Fast AND real SQL."""
    conn = make_db()          # :memory: — vanishes when closed
    yield conn                # the test runs here
    conn.close()              # teardown, even if the test failed

def test_inserted_note_is_findable(db):
    add_note(db, "buy milk")
    add_note(db, "write tests")

    assert find_notes(db, "milk") == ["buy milk"]

def test_search_with_no_matches_returns_empty(db):
    add_note(db, "buy milk")
    assert find_notes(db, "xyzzy") == []
```

This suite would catch a typo in the SQL — something no amount of mocking could.

### Testing a Flask app through HTTP (integration)

```python
# test_app.py — assumes app.py defines a Flask `app` with a /notes API
import pytest
from app import app

@pytest.fixture
def client():
    app.config["TESTING"] = True          # better error reporting
    with app.test_client() as client:     # in-process HTTP — no server needed
        yield client

def test_get_notes_returns_json_list(client):
    resp = client.get("/notes")

    assert resp.status_code == 200
    assert resp.is_json
    assert isinstance(resp.get_json(), list)

def test_posting_invalid_note_returns_400(client):
    resp = client.post("/notes", json={})          # missing required field

    assert resp.status_code == 400                 # error path, again!
```

### Intro to Playwright (browser E2E)

```powershell
# One-time setup in your JS project (Windows PowerShell)
npm init playwright@latest
# accept defaults; this installs browsers and creates playwright.config.js + tests/
```

```javascript
// tests/todo.spec.js — drives a real browser against your running app.
// Start your app first (e.g. `npm run dev` on http://localhost:5173),
// or set `webServer` in playwright.config.js so Playwright starts it for you.
import { test, expect } from "@playwright/test";

test("a user can add a todo and see it in the list", async ({ page }) => {
  // Arrange: open the app in a real browser page
  await page.goto("http://localhost:5173");

  // Act: interact like a user — prefer role/label selectors over CSS classes,
  // because they track what users see, not your styling.
  await page.getByLabel("New todo").fill("Learn Playwright");
  await page.getByRole("button", { name: "Add" }).click();

  // Assert: expect(...) auto-waits and retries until timeout — this is the
  // built-in flakiness defense. No sleep() calls needed.
  await expect(page.getByRole("listitem")).toContainText("Learn Playwright");
});

test("submitting an empty todo shows a validation message", async ({ page }) => {
  await page.goto("http://localhost:5173");

  await page.getByRole("button", { name: "Add" }).click();

  await expect(page.getByText("Todo cannot be empty")).toBeVisible();
});
```

```powershell
npx playwright test            # run headless
npx playwright test --headed   # watch the browser do it — great for learning
npx playwright show-report     # HTML report with screenshots of failures
```

## Common Pitfalls

- **Pointing tests at real/shared data.** A test that wipes a table will someday wipe the wrong database. Correction: tests create their own temp resources (`tmp_path`, `:memory:`), always.
- **Tests that depend on leftover state from previous runs.** "Works the first time, fails the second" means a fixture isn't cleaning up. Correction: create-and-destroy per test; put teardown after `yield` so it runs even on failure.
- **Using `sleep()` to wait for pages or servers.** Too short → flaky; too long → slow. Correction: use auto-waiting assertions (Playwright `expect`), or poll-with-timeout helpers.
- **Selecting elements by brittle CSS selectors** (`.btn.btn-primary:nth-child(2)`). A styling change breaks every test. Correction: select by role, label, or visible text — the things users actually perceive.
- **Recreating every unit-test edge case at the E2E level.** 40 browser tests for input validation take 10 minutes and tell you what a 40ms unit suite already did. Correction: E2E covers a few critical journeys; edges live in unit tests.
- **Ignoring flaky tests ("just re-run it").** A flaky suite trains you to ignore red, which is how real failures slip through. Correction: treat flakes as bugs — usually a missing wait or shared state — and fix or quarantine them immediately.

## Practice Exercises

1. Write a CSV-based expense logger (`add_expense`, `monthly_total`) that stores data in a file, then write integration tests using `tmp_path` covering: round-trip, empty file, file with a malformed row, and unicode in descriptions.
2. Extend the SQLite example with `delete_note(conn, note_id)` and write tests for: deleting an existing note, deleting a nonexistent id (decide the behavior first), and verifying other notes survive a delete.
3. Take any Flask app from your earlier tracks and write four `test_client` tests: one 200 happy path, one 404, one 400 for invalid input, and one asserting the response's JSON shape (keys and types).
4. Install Playwright and write an E2E test against any local app you've built (or a simple HTML form page you create): fill a form, submit, assert the visible result. Run it `--headed` once to watch, then headless.
5. Decide, in writing, for your most recent project: which THREE user journeys deserve E2E tests? Justify each in one sentence, and name two things you would deliberately *not* E2E-test and where those checks belong instead.
