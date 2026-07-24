# Chapter 9: Databases from Node

## Overview

Every API you've built so far keeps its data in a JavaScript array — restart the server and everything is gone. Real applications need **persistence**, and in backend work that almost always means a relational database. This chapter connects the SQL you learned in the SQL track to your Node/Express code: opening a database, running queries from route handlers, and — non-negotiably — using **parameterized queries** so user input can never be executed as SQL. We use **better-sqlite3** as the primary tool because SQLite is a real, production-grade database that lives in a single file, needs zero setup, and (in this library) has a synchronous API that keeps the learning surface small; we'll also map everything to **pg** (PostgreSQL) so you can see the async shape you'll use with a client-server database at work. Finally, you'll learn the structural habit that matters as much as any query: pulling SQL out of route handlers into a dedicated **data layer**, and managing schema changes with **migrations** instead of hand-edited databases.

## Definitions & Explanations

**SQLite** is a relational database engine that stores an entire database — tables, indexes, everything — in one ordinary file (e.g., `data/app.db`). There is no server to install, no ports, no passwords; your process opens the file directly. It is not a toy: SQLite ships inside every phone and browser, and it's a legitimate production choice for small-to-medium web apps. Its main limitation is concurrency: one writer at a time, which matters for high-traffic multi-server systems but not for anything in this track.

**better-sqlite3** is the standard Node library for SQLite. Its unusual trait: the API is **synchronous** — `stmt.get()` returns the row directly, no `await`. Normally synchronous I/O in Node is a sin (Chapter 3), but SQLite queries are local file reads measured in microseconds, and better-sqlite3 is deliberately built this way (it's actually faster than the async alternatives for SQLite). Just understand *why* it's acceptable here and not a pattern to generalize.

**PostgreSQL ("Postgres")** is the client-server relational database you're most likely to meet at work: a separate server process your app connects to over the network. The **pg** package is its Node driver. Because queries cross a network, every pg call is asynchronous (`await pool.query(...)`). A **connection pool** is pg's way of reusing a handful of open connections across many requests instead of paying connection setup cost per query — you create one pool at startup and share it.

A **prepared statement** is a query compiled once with **placeholders** where values go: `SELECT * FROM bookmarks WHERE id = ?`. When you execute it, you pass the values separately, and the database engine treats them purely as *data* — they can never change the query's structure. This is also called a **parameterized query**. better-sqlite3 uses `?` (or `@name`) placeholders; pg uses `$1, $2, ...`.

**SQL injection** is the attack that parameterized queries make impossible: if you build SQL by string concatenation with user input — `` `SELECT * FROM users WHERE name = '${name}'` `` — then a user whose "name" is `' OR '1'='1` rewrites your query, and a nastier payload reads or destroys your data. You saw this from the SQL side in the SQL track; this chapter is where you, the application developer, become the person responsible for preventing it. The rule has no exceptions: **user input never gets concatenated into SQL. Ever.**

A **data layer** (the file(s) often called a **repository**) is a module that owns all SQL for a resource and exposes plain functions like `findById(id)` and `create(data)`. Route handlers call these functions and never see SQL. The payoffs: handlers stay readable, queries are reusable and testable, and swapping SQLite for Postgres later touches one folder instead of every route.

A **transaction** groups multiple statements into an all-or-nothing unit: either every statement commits, or none do. Transfer money out of one account and into another — if the second statement fails, the first must roll back. better-sqlite3 wraps this as `db.transaction(fn)`.

A **migration** is a versioned, ordered script that changes the database schema (`001-create-bookmarks.sql`, `002-add-tags.sql`, ...). Instead of anyone hand-editing the live database, schema changes are files in git, applied in order, so every environment — your laptop, a teammate's, production — can be rebuilt to the identical schema. Full migration tooling (up/down scripts, migration tables) comes with frameworks and jobs; in this track you'll do the honest minimal version: a `schema.sql` applied at startup, then numbered migration files once the schema starts evolving.

## Code Examples

**Setup** (SQLite is a native module; on Windows the prebuilt binaries almost always install cleanly):

```powershell
npm install better-sqlite3
mkdir data
```

**Opening the database and creating the schema** — one shared connection module:

```js
// src/db.js
import Database from 'better-sqlite3';

export const db = new Database('data/app.db');
db.pragma('journal_mode = WAL');        // better concurrent-read behavior; standard practice
db.pragma('foreign_keys = ON');         // SQLite ships with FK enforcement OFF — turn it on

// Minimal "migration": idempotent schema, applied at startup
db.exec(`
  CREATE TABLE IF NOT EXISTS bookmarks (
    id         INTEGER PRIMARY KEY AUTOINCREMENT,
    url        TEXT NOT NULL,
    title      TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now'))
  );
`);
```

**The injection lesson — vulnerable vs. fixed.** Burn this contrast in:

```js
// ❌ VULNERABLE — user input concatenated into SQL.
// GET /api/v1/bookmarks/search?q=foo
const rows = db.prepare(
  `SELECT * FROM bookmarks WHERE title LIKE '%${req.query.q}%'`   // NEVER do this
).all();
// q = "%' OR 1=1 --"        -> returns every row, filter bypassed
// With string-concatenated statements elsewhere, payloads can read other
// tables (users, sessions...) or destroy data. The string IS the program.

// ✅ FIXED — parameterized. The value can only ever be a value.
const stmt = db.prepare('SELECT * FROM bookmarks WHERE title LIKE ?');
const rows2 = stmt.all(`%${req.query.q}%`);
// Building the *pattern string* in JS is fine — it's passed as one bound
// parameter. What's forbidden is building the *SQL text* from input.
```

One nuance: placeholders bind **values** only. You cannot parameterize a table or column name (`ORDER BY ?` won't do what you want). For dynamic sort columns, validate against an allowlist:

```js
const SORTABLE = { title: 'title', created: 'created_at' };
const sortCol = SORTABLE[req.query.sort] ?? 'created_at';   // input picks from OUR list,
const rows = db.prepare(`SELECT * FROM bookmarks ORDER BY ${sortCol}`).all(); // never enters SQL raw
```

**CRUD, better-sqlite3 style.** `get` returns one row (or `undefined`), `all` returns an array, `run` executes writes and reports what happened:

```js
const insert = db.prepare('INSERT INTO bookmarks (url, title) VALUES (@url, @title)');
const info = insert.run({ url: 'https://nodejs.org', title: 'Node.js' }); // named params
console.log(info.lastInsertRowid); // id of the new row
console.log(info.changes);         // number of rows affected

const byId = db.prepare('SELECT * FROM bookmarks WHERE id = ?');
const bookmark = byId.get(42);     // a row object, or undefined

const del = db.prepare('DELETE FROM bookmarks WHERE id = ?');
const existed = del.run(42).changes > 0; // .changes tells you if anything was deleted
```

**Extracting the data layer.** Naive: SQL inline in every handler. Better: a repository module, so handlers read like intentions:

```js
// src/repositories/bookmarks.repo.js — ALL bookmark SQL lives here and only here
import { db } from '../db.js';

const stmts = {
  list:   db.prepare('SELECT * FROM bookmarks ORDER BY created_at DESC LIMIT ? OFFSET ?'),
  byId:   db.prepare('SELECT * FROM bookmarks WHERE id = ?'),
  insert: db.prepare('INSERT INTO bookmarks (url, title) VALUES (@url, @title)'),
  remove: db.prepare('DELETE FROM bookmarks WHERE id = ?'),
};

export function listBookmarks({ page = 1, pageSize = 20 } = {}) {
  return stmts.list.all(pageSize, (page - 1) * pageSize);
}
export function findBookmark(id) {
  return stmts.byId.get(id); // undefined when missing — caller decides that's a 404
}
export function createBookmark({ url, title }) {
  const info = stmts.insert.run({ url, title: title ?? null });
  return findBookmark(info.lastInsertRowid);
}
export function deleteBookmark(id) {
  return stmts.remove.run(id).changes > 0;
}
```

```js
// src/routes/bookmarks.routes.js — no SQL in sight
import { Router } from 'express';
import * as repo from '../repositories/bookmarks.repo.js';
import { HttpError } from '../errors.js';

export const bookmarksRouter = Router();

bookmarksRouter.get('/:id', (req, res) => {
  const bookmark = repo.findBookmark(Number(req.params.id)); // validate id per Chapter 8
  if (!bookmark) throw new HttpError(404, 'Bookmark not found');
  res.json(bookmark);
});

bookmarksRouter.post('/', (req, res) => {
  const created = repo.createBookmark(req.body); // validated input per Chapter 8
  res.status(201).json(created);
});
```

**Transactions** — all-or-nothing writes:

```js
const insertTag = db.prepare('INSERT INTO tags (bookmark_id, name) VALUES (?, ?)');

// db.transaction returns a function; calling it runs everything atomically.
const createWithTags = db.transaction((bookmark, tags) => {
  const created = createBookmark(bookmark);
  for (const name of tags) insertTag.run(created.id, name);
  return created; // any throw anywhere above rolls back EVERYTHING
});

const bookmark = createWithTags({ url: 'https://expressjs.com' }, ['docs', 'express']);
```

**The same ideas in pg** — so the Postgres shape isn't foreign when you meet it. Async everywhere, `$1`-style placeholders, one shared pool:

```js
// Postgres version of the repository, for contrast (npm install pg)
import pg from 'pg';
const pool = new pg.Pool({ connectionString: process.env.DATABASE_URL }); // config: Chapter 11

export async function findBookmark(id) {
  const result = await pool.query('SELECT * FROM bookmarks WHERE id = $1', [id]);
  return result.rows[0]; // undefined when missing — same contract as before
}
```

Because the repository hid the SQL, switching drivers changes this file and (since pg is async) adds `await` at call sites — the routes' *logic* survives intact. That's the layering paying off.

## Common Pitfalls

1. **String-building SQL "just this once."** Template literals make concatenation feel natural, and the vulnerable version *works perfectly* in every normal test — injection only shows up when someone hostile arrives. Correction: placeholders for every value, every time, no exceptions; if you typed user input inside backticks that produce SQL, stop.

2. **Opening a new database connection per request.** `new Database(...)` (or a new pg client) inside a handler leaks file handles and destroys performance. Correction: one module (`db.js`) creates the connection/pool once at startup; everything imports it.

3. **Forgetting that SQLite foreign keys are off by default.** Your `FOREIGN KEY` constraints silently don't enforce anything, and orphaned rows accumulate until some join breaks. Correction: `db.pragma('foreign_keys = ON')` immediately after opening — and a test that proves deleting a parent behaves as designed.

4. **Treating `undefined` from `.get()` as an error condition to swallow.** A missing row is a *normal outcome* that should become a 404, not a `TypeError: Cannot read properties of undefined` three lines later. Correction: check the return of `.get()` explicitly, and let the repository's contract be "returns the row or `undefined`" — the route decides what that means in HTTP.

5. **Committing the database file to git.** `data/app.db` changes on every request, bloats the repo, and merges catastrophically. Correction: add `data/*.db*` to `.gitignore` (WAL mode creates `-wal`/`-shm` sidecar files too); what belongs in git is the *schema and migrations*, which can rebuild the database anywhere.

6. **Editing the live schema by hand.** You add a column with a DB browser tool on your machine; your code assumes it; nobody else's database has it. Correction: every schema change is a numbered SQL file in the repo, applied in order — the database is *derived from* the migrations, never the source of truth itself.

7. **Doing pagination in JavaScript instead of SQL.** `db.prepare('SELECT * FROM bookmarks').all()` then `.slice(...)` fetches the entire table to return 20 rows — fine at 100 rows, catastrophic at a million. Correction: push filtering, sorting, and paging into the query (`WHERE ... ORDER BY ... LIMIT ? OFFSET ?`); databases are extremely good at this, and it's what your SQL track trained you for.

## Practice Exercises

1. **Injection lab (on your own database only).** Build a tiny two-route app: `/search-bad` using string concatenation and `/search-good` using a placeholder, both querying a throwaway table you seed with a few rows. Craft an input that makes `/search-bad` return every row, and confirm the identical input returns nothing suspicious from `/search-good`. Write three sentences explaining *mechanically* why the placeholder version is immune (what does the engine do differently?).

2. **Persist Project 3.** Take your in-memory bookmarks API and replace the array with better-sqlite3: schema in `db.js`, all SQL in a `bookmarks.repo.js`, handlers untouched except for calling the repo. Restart the server and prove the data survives. Requirement: `git grep -i "prepare\|SELECT"` outside the repo/db files should find nothing.

3. **Pagination and filtering in SQL.** Extend the repository so `listBookmarks` supports `page`, `pageSize` (capped), a `tag` filter, and a `sort` allowlist — all in the query, not in JS. Return `{ data, page, pageSize, total }`; getting `total` correct under filtering requires a second `COUNT(*)` query with the *same* WHERE clause. Verify with 30+ seeded rows.

4. **Transaction proof.** Add a `tags` table with a foreign key to bookmarks. Write `createWithTags(bookmark, tags)` inside `db.transaction`, then make it fail on purpose halfway (e.g., one tag violates a `NOT NULL`). Prove — with SELECTs before and after — that the bookmark row rolled back too. Then remove the transaction wrapper, run the same failure, and document the corrupted half-written state you get.

5. **Write your first real migration.** Your live database has bookmarks; product wants a `favorite` boolean. Write `migrations/002-add-favorite.sql` using `ALTER TABLE`, apply it *without losing existing rows*, and update repository + validation to expose the field. Then write a short prose note: what would have gone wrong if you'd instead changed `CREATE TABLE` in the original schema file and deleted the database?

6. **Driver-swap thought experiment (prose, no code).** List every file in your exercise-2 app that would change if you moved from better-sqlite3 to pg, and what kind of change each needs (placeholder syntax, async/await, pool setup). Which layers change not at all? What does that tell you about where the layering boundary earns its keep?
