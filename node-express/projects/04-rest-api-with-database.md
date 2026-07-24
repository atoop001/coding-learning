# Project 4: Bookmarks API with SQLite

## Description

Take the bookmarks API from Project 3 and make it real: bookmarks now survive a server restart because they live in a SQLite database (via `better-sqlite3`), and the code is reorganized into the layered structure from Chapter 13 — routes, controllers, a service layer, and a repository layer that is the *only* code that talks to the database.

You can either evolve your Project 3 codebase or rebuild from a fresh folder (rebuilding is genuinely good practice — the HTTP layer will pour out of you the second time). Either way, the external behavior should end up a superset of Project 3: same endpoints, same status codes, same error shape, plus pagination — but now `Ctrl+C` and restart, and your data is still there.

The two skills this project actually drills: **parameterized queries with zero exceptions** (every value the client influences goes in as a `?` parameter, never string-glued into SQL), and **layer discipline** (Express types like `req`/`res` never appear below the controller layer, and SQL never appears above the repository).

## Difficulty & Estimated Effort

Intermediate+ — 8–12 hours.

## Chapters Used

- Chapter 5: Express fundamentals
- Chapter 6: Middleware
- Chapter 7: REST API design
- Chapter 8: Validation & error handling
- Chapter 9: Databases from Node
- Chapter 13: Architecture & code organization

## Requirements

**Database foundation**

- [ ] `better-sqlite3` installed; a single module owns creating/opening the database connection and is imported by the repository layer only
- [ ] A `schema.sql` file (or a `migrate` script that applies it) defines a `bookmarks` table with columns for id (integer primary key), title, url, tags, and created_at — decide and document how you store tags in SQL (serialized in a column vs. a separate `tags` table; either is acceptable if you can defend it)
- [ ] An npm script (e.g., `npm run db:init`) creates the database from the schema so a fresh clone can be set up in one command
- [ ] The database file is listed in `.gitignore`; the schema file is committed

**Layered structure**

- [ ] Folder structure separates `routes/`, `controllers/`, `services/`, and `repositories/` (names can vary; the layers can't)
- [ ] Routes files contain *only* path-to-controller wiring — no logic
- [ ] Controllers translate HTTP (read params/body, choose status codes, send responses) and call the service; they contain no business rules and no SQL
- [ ] The service layer holds the rules (what makes a bookmark valid to save, what "not found" means) and knows nothing about Express — grep your services for `req` and `res` to prove it
- [ ] The repository layer is the only code containing SQL; it accepts and returns plain JS objects (e.g., `createdAt` as a proper value, tags as an array — the SQL storage format doesn't leak upward)

**Query safety**

- [ ] Every query with a client-influenced value uses parameter binding — audit the finished repository and confirm there is not a single template literal or string concatenation that inserts a value into SQL text
- [ ] Prepared statements are created once and reused where it's natural to do so
- [ ] Demonstrate to yourself that injection fails: create a bookmark titled `'; DROP TABLE bookmarks; --` and confirm it is stored and returned as an ordinary (weird) title

**Behavior (parity with Project 3, plus)**

- [ ] All Project 3 endpoints work identically against SQLite: full CRUD, tag filter, correct status codes, same error shape, centralized error handler
- [ ] `GET /api/bookmarks` supports `?limit=` and `?offset=` pagination with validated bounds (a cap on `limit`, non-negative integers, sane defaults) and the SQL itself does the limiting — not `.slice()` in JS
- [ ] The paginated response includes enough metadata for a client to page (e.g., total count)
- [ ] Updating or deleting a nonexistent ID returns `404` — determine this from what the database operation reports, not by a separate lookup race
- [ ] Data survives a full stop and restart of the server (prove it: create, stop, start, fetch)

**Robustness**

- [ ] Database errors (e.g., a violated constraint) are caught and surfaced through the centralized error handler in the standard shape — never as a raw stack trace or a 200
- [ ] A `NOT NULL` or `CHECK` constraint on the table backs up at least one validation rule, and you've tested what happens when it fires
- [ ] The README documents: setup steps (install, db:init, dev), the endpoint table, and your tags-storage decision with one sentence of rationale

## Hints

- Decide the tags representation *before* writing the schema. A serialized column is simpler and fine here; a join table is more relational and makes "filter by tag" a real SQL query instead of a scan. Chapter 9's migrations section plus the SQL track's JOIN material will inform you. Whichever you pick, the repository's job is to make the choice invisible to the service layer.
- `better-sqlite3` is synchronous — that's not a bug, and Chapter 9 explains why it's acceptable (and when it wouldn't be). Notice how it simplifies your repository compared to a promise-based driver.
- The cleanest path to layer discipline: write the repository first with plain functions (`findAll`, `findById`, `create`, ...), test them from a scratch script *without any server running*, then build upward. If you can exercise your whole data layer from `node scratch.js`, your layering is probably right.
- "How does the service say *not found* without knowing about HTTP 404?" is the central design question of this project. Chapter 13 discusses the options (return null, throw a domain error) — pick one and apply it consistently.
- For pagination metadata you'll need a `COUNT(*)` query alongside the page query. Think about whether the tag filter must apply to both.
- `INTEGER PRIMARY KEY` in SQLite auto-assigns row IDs; the info object returned by a run statement tells you the new ID and how many rows an update/delete touched — that second fact is your `404` signal.
- If you rebuild rather than evolve, keep Project 3 open in another window as your spec — the endpoint table you wrote there *is* the contract.

## Stretch Goals

- Implement the tags-as-join-table design (if you serialized) or benchmark filter-by-tag both ways (if you joined) — feel the tradeoff instead of reading about it.
- Add a `search` query parameter that matches against title, using SQL `LIKE` with a bound parameter — then read about what characters `%` and `_` mean in user input and handle them.
- Add sorting (`?sort=createdAt&order=desc`) with a strict allowlist of sortable columns — this is the one place parameter binding can't save you, so understand why the allowlist is the defense.
- Write a small seed script (`npm run db:seed`) that fills the database with realistic sample bookmarks for development.
- Add a simple numbered migrations folder (`001-init.sql`, `002-add-notes-column.sql`) and a script that applies only the ones not yet applied, tracked in a `migrations` table.
