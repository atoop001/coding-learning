# Chapter 15: Using SQL from Code — Python's sqlite3, Parameterized Queries & SQL Injection

## Overview

Databases exist to serve applications. This chapter connects your two skill sets: Python programs that open a SQLite database, run queries, and process results. You'll learn the standard `sqlite3` module (built into Python — nothing to install), the connection/cursor workflow, transactions from code, and the single most important security lesson in web development: **parameterized queries vs. SQL injection**. The same patterns (with different library names) apply to JavaScript (`better-sqlite3`), PostgreSQL (`psycopg`), and every ORM you'll ever meet.

Prerequisites: Python basics (functions, loops, `with` blocks), and the SQL from Chapters 1–12.

## Definitions & Explanations

### The cast of characters

- **Connection** (`sqlite3.connect(path)`) — an open session with a database file. Creates the file if absent (which also means a typo'd path silently creates an empty database — a classic head-scratcher).
- **Cursor** — the object that executes statements and iterates results. `conn.execute(...)` shortcuts creating one.
- **Row** — by default a plain tuple; setting `conn.row_factory = sqlite3.Row` upgrades rows to support access by column name (`row["title"]`) — do this always.

### The lifecycle

```python
import sqlite3

with sqlite3.connect("library.db") as conn:   # 'with' = auto commit/rollback (see below)
    conn.row_factory = sqlite3.Row
    conn.execute("PRAGMA foreign_keys = ON")  # per-connection, every time! (Chapter 5)
    ...
conn.close()                                   # 'with' does NOT close in sqlite3 — close yourself
```

Quirk worth memorizing: for `sqlite3`, `with conn:` manages the **transaction** (commit on success, rollback on exception), *not* the connection's open/closed state. Call `conn.close()` when done (or use `contextlib.closing`).

### Executing statements

- `cur = conn.execute(sql, params)` — run one statement.
- `conn.executemany(sql, seq_of_params)` — run the same statement for many parameter tuples (bulk insert).
- `conn.executescript(big_sql_string)` — run multiple `;`-separated statements (schema files); no parameters allowed.
- Fetching: `cur.fetchone()` (next row or `None`), `cur.fetchall()` (list of rows), or just `for row in cur:`.
- After writes: `cur.rowcount` (affected rows — Chapter 4's `changes()`), `cur.lastrowid` (the new row's id after INSERT).

### Transactions from Python

By default, the `sqlite3` module opens a transaction before your first data-modifying statement and holds it until you call `conn.commit()` (or roll it back). **If you never commit, your changes are discarded** when the connection closes — the #1 beginner bug in this chapter. Three workable disciplines:

1. Explicit: `conn.commit()` after each logical unit; `conn.rollback()` in `except` blocks.
2. `with conn:` blocks around each unit (commit/rollback handled).
3. `sqlite3.connect(path, autocommit=True)` (Python 3.12+) plus explicit `BEGIN`/`COMMIT` where you need atomicity.

This chapter's examples use `with conn:` — it maps exactly onto Chapter 12's BEGIN/COMMIT/ROLLBACK mental model.

### Parameterized queries — the only way to include values

Never build SQL by gluing strings together. Use **placeholders** and let the driver deliver values safely:

```python
conn.execute("SELECT * FROM books WHERE author = ?", (author,))          # qmark style
conn.execute("SELECT * FROM books WHERE author = :a", {"a": author})     # named style
```

The parameters travel *separately* from the SQL text; the database treats them purely as data, never as SQL. Benefits: security (below), correctness (quotes/Unicode handled), and speed (statement reuse).

**Placeholders stand for values only** — not table names, not column names, not `ASC`/`DESC`. Dynamic identifiers must come from an allowlist you control (see example 7).

### SQL injection — the attack parameterization prevents

If you build `f"SELECT * FROM users WHERE name = '{name}'"` and a user submits `name = "x' OR '1'='1"`, your query becomes:

```sql
SELECT * FROM users WHERE name = 'x' OR '1'='1'   -- returns EVERY user
```

Submitting `"x'; DROP TABLE users; --"` can be worse. Injection has powered many of history's largest breaches, is on every security top-10 list, and is asked about in most web-dev interviews. The complete defense is boring: **parameterize every value, always** — there is no "safe enough" string-escaping alternative worth learning.

## Code Examples

A complete miniature program — save as `library_app.py` and run with `python library_app.py`:

```python
import sqlite3
from contextlib import closing

DB_PATH = "library.db"

SCHEMA = """
PRAGMA foreign_keys = ON;

CREATE TABLE IF NOT EXISTS authors (
    id   INTEGER PRIMARY KEY,
    name TEXT NOT NULL UNIQUE
);
CREATE TABLE IF NOT EXISTS books (
    id        INTEGER PRIMARY KEY,
    author_id INTEGER NOT NULL REFERENCES authors(id),
    title     TEXT NOT NULL,
    year      INTEGER,
    rating    REAL CHECK (rating BETWEEN 0 AND 5)
);
"""

def get_connection():
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row            # rows addressable by column name
    conn.execute("PRAGMA foreign_keys = ON")  # every connection, every time
    return conn

def setup(conn):
    conn.executescript(SCHEMA)                # multiple statements at once

def add_author(conn, name):
    """Insert an author, returning the new id (or the existing one)."""
    with conn:                                # transaction: commit or rollback
        cur = conn.execute(
            "INSERT INTO authors (name) VALUES (?) "
            "ON CONFLICT(name) DO NOTHING", (name,))
    row = conn.execute("SELECT id FROM authors WHERE name = ?", (name,)).fetchone()
    return row["id"]

def add_book(conn, author_name, title, year=None, rating=None):
    author_id = add_author(conn, author_name)
    with conn:
        cur = conn.execute(
            "INSERT INTO books (author_id, title, year, rating) VALUES (?, ?, ?, ?)",
            (author_id, title, year, rating))   # None becomes NULL automatically
        return cur.lastrowid

def bulk_add_books(conn, rows):
    """rows: iterable of (author_id, title, year, rating). One transaction = fast."""
    with conn:
        conn.executemany(
            "INSERT INTO books (author_id, title, year, rating) VALUES (?, ?, ?, ?)",
            rows)

def search_books(conn, term):
    """Safe LIKE search: the wildcard lives in the PARAMETER, not the SQL."""
    pattern = f"%{term}%"
    cur = conn.execute("""
        SELECT b.title, a.name AS author, b.year, b.rating
        FROM books b
        JOIN authors a ON a.id = b.author_id
        WHERE b.title LIKE ?
        ORDER BY b.title
    """, (pattern,))
    return cur.fetchall()

def rate_book(conn, book_id, rating):
    with conn:
        cur = conn.execute(
            "UPDATE books SET rating = ? WHERE id = ?", (rating, book_id))
    if cur.rowcount == 0:                     # Chapter 4: check what you touched
        raise ValueError(f"no book with id {book_id}")

ALLOWED_SORTS = {"title": "b.title", "year": "b.year", "rating": "b.rating"}

def list_books(conn, sort_by="title"):
    """Dynamic ORDER BY done safely: identifier from an ALLOWLIST, never from input."""
    column = ALLOWED_SORTS.get(sort_by)
    if column is None:
        raise ValueError(f"cannot sort by {sort_by!r}")
    sql = f"""SELECT b.title, a.name AS author, b.year, b.rating
              FROM books b JOIN authors a ON a.id = b.author_id
              ORDER BY {column}"""            # safe: column came from OUR dict
    return conn.execute(sql).fetchall()

def main():
    with closing(get_connection()) as conn:
        setup(conn)
        add_book(conn, "Octavia Butler", "Kindred", 1979, 4.8)
        add_book(conn, "Frank Herbert", "Dune", 1965, 4.5)
        add_book(conn, "Octavia Butler", "Parable of the Sower", 1993)

        print("-- search 'a' --")
        for row in search_books(conn, "a"):
            # sqlite3.Row: index by name; NULL arrives as None
            rating = row["rating"] if row["rating"] is not None else "unrated"
            print(f'{row["title"]} by {row["author"]} ({row["year"]}) — {rating}')

        rate_book(conn, 3, 4.9)
        print("-- by rating --")
        for row in list_books(conn, "rating"):
            print(dict(row))

if __name__ == "__main__":
    main()
```

The injection demonstration — study it, never imitate the first half:

```python
# ❌ VULNERABLE — never do this:
name = input("author: ")           # attacker types: x' OR '1'='1
cur = conn.execute(f"SELECT * FROM authors WHERE name = '{name}'")
# The attacker's quote ends your string; the rest becomes live SQL.

# ✅ SAFE — identical intent, immune:
cur = conn.execute("SELECT * FROM authors WHERE name = ?", (name,))
# The weird string is now just... a weird author name that matches nothing.
```

Error handling with rollback, the Chapter 12 discipline in Python form:

```python
try:
    with conn:                                 # BEGIN
        conn.execute("UPDATE accounts SET balance_cents = balance_cents - ? WHERE id = ?",
                     (amount, src))
        conn.execute("UPDATE accounts SET balance_cents = balance_cents + ? WHERE id = ?",
                     (amount, dst))
        # any exception here → automatic ROLLBACK of both updates
except sqlite3.IntegrityError as e:            # e.g. CHECK (balance >= 0) fired
    print(f"transfer refused: {e}")
```

## Common Pitfalls

**1. F-strings / `%` / `+` to build SQL with values.**

```python
# ❌ Injection + quote bugs + no plan reuse:
conn.execute(f"INSERT INTO books (title) VALUES ('{title}')")
# ✅
conn.execute("INSERT INTO books (title) VALUES (?)", (title,))
```

Grep your own code for `execute(f"` — each hit is a bug until proven otherwise (allowlisted identifiers being the one proven case).

**2. Forgetting `conn.commit()`.** The program runs, prints success, exits — and the database is unchanged. If you're not using `with conn:` blocks, every write path needs an explicit commit.

**3. The single-element tuple.**

```python
# ❌ TypeError (or worse, iterating the string into characters):
conn.execute("SELECT * FROM books WHERE title = ?", (title))
# ✅ The comma makes it a tuple:
conn.execute("SELECT * FROM books WHERE title = ?", (title,))
```

**4. Forgetting `PRAGMA foreign_keys = ON` per connection.** Your carefully designed FKs silently stop being enforced in the app. Bake it into a `get_connection()` helper (as above) so it can't be forgotten.

**5. Putting `%` wildcards or quotes into the SQL around a placeholder.**

```python
# ❌ WHERE title LIKE '%?%'   → the ? inside quotes is literal text, param unused → error
# ❌ WHERE title = '?'        → same disease
# ✅ Build the pattern in Python, pass it as the parameter:
conn.execute("... WHERE title LIKE ?", (f"%{term}%",))
```

**6. Opening a new connection per query (or sharing one across threads carelessly).** Connections have setup cost and transaction state. Pattern for scripts and small apps: one connection per logical task, passed into functions, closed at the end. Threads each get their own connection.

## Practice Exercises

1. Write `movies_app.py`: create a `movies` table (title, director, year, watched flag), then implement `add_movie`, `mark_watched(movie_id)` (must raise on unknown id via `rowcount`), and `list_unwatched()` printed as clean lines. Every value goes through placeholders; verify the file with DB Browser afterward.
2. Extend it with `search(term)` supporting partial, case-insensitive title matches, and `stats()` returning a dict with total movies, watched count, and earliest year (mind NULL → `None`). Demonstrate both with at least six seeded movies via `executemany`.
3. Injection lab: add a deliberately vulnerable `search_unsafe(term)` using an f-string, then craft an input that makes it return every movie despite matching none by title. Show the same input harmlessly returning nothing through your safe `search`. Delete the unsafe function afterward, with a commit message-style comment explaining why it can never come back.
4. Build a transfer function against Chapter 12's `accounts` schema entirely from Python: parameterized updates, `with conn:` atomicity, an exception path proving that a failed second update rolls back the first, and a final assertion that total money is conserved.
5. Write an import script that reads a CSV (make one with 1,000+ generated rows — Python's `csv` and `random` modules) into a table two ways: one execute-per-row with autocommit habits, versus one `executemany` inside a single transaction. Time both (`time.perf_counter()`), report the speedup, and connect the result to Chapter 12's durability discussion in a comment.
