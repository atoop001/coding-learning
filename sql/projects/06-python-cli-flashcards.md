# Project 6: Python CLI App — SQLite-Backed Flashcards

## Description

Build a complete command-line flashcard/study application in Python with SQLite as its storage engine — a real program with a schema it creates itself, decks, cards, study sessions with spaced repetition-lite scheduling, and statistics. This is the bridge project: everything the database does is skills from Chapters 1–14, and everything around it is Chapter 15's craft — connections, parameterized queries everywhere, transactions from code, and a codebase where SQL and Python each do what they're best at.

(If flashcards bore you, substitute an equivalent domain — habit tracker, workout log, recipe box — keeping every requirement's *shape*: two-level hierarchy, event log, scheduling query, stats.)

## Difficulty

**Advanced** — estimated effort: 10–15 hours.

## Chapters Used

- Chapter 15 (Python + sqlite3 — the core)
- Chapter 12 (transactions from application code)
- Chapters 5–8 (schema, joins, aggregation, subqueries in service of features)
- Chapter 11 (indexes for the scheduling query)
- Chapter 14 (NULL discipline at the Python boundary)

## Requirements Checklist

### Architecture
- [ ] A `flashcards.py` (or small package) runnable as `python flashcards.py <command> [args]` using `argparse` — commands listed below
- [ ] One `get_connection()` helper: sets `row_factory = sqlite3.Row`, enables foreign keys, WAL mode — used everywhere, no stray `sqlite3.connect` calls
- [ ] Schema created automatically on first run (`CREATE TABLE IF NOT EXISTS` via `executescript`), with the database file location clearly defined
- [ ] **Zero** SQL built by string interpolation of values — every value travels as a parameter; the only dynamic SQL permitted is allowlisted identifiers (Chapter 15's pattern), if you need it at all
- [ ] All multi-statement writes wrapped in transactions (`with conn:`), with database errors caught and reported as friendly messages, never raw tracebacks

### Schema
- [ ] `decks` (unique name, created date), `cards` (deck FK with sensible ON DELETE, front, back, created date), `reviews` — the event log: card FK, timestamp, result ('again'/'hard'/'good'/'easy' — CHECKed), and the computed next-due date
- [ ] Indexes chosen for the app's hottest queries (due-card selection above all), each justified in a comment

### Commands
- [ ] `add-deck <name>` / `list-decks` — listing shows per-deck card counts and due-today counts (LEFT JOIN + aggregation; empty decks show 0)
- [ ] `add-card <deck> "<front>" "<back>"` — errors cleanly on unknown deck
- [ ] `import <deck> <file.csv>` — bulk-add cards from CSV via `executemany` in one transaction; reports how many imported and skips/reports malformed lines without dying
- [ ] `study <deck>` — the core loop: fetch due cards (never-reviewed cards are due immediately — mind the NULL logic), show front, wait for Enter, show back, accept a result key, record the review and its next-due date in one transaction per card; scheduling rule: 'again' → due now, 'hard' → +1 day, 'good' → +3 days, 'easy' → +7 days (or your own documented intervals)
- [ ] `stats [deck]` — total cards, reviews logged, due now, success rate (good/easy share), current streak of consecutive study days (dates trickery — a worthy subquery), and the 5 most-lapsed cards (most 'again' results)
- [ ] `remove-deck <name>` — with a typed confirmation, demonstrating what your ON DELETE choice does to cards and reviews

### Quality proofs
- [ ] A `--seed` developer command generating 3 decks / 100+ cards / 500+ reviews with randomized history, so stats and due-selection are testable at once
- [ ] `EXPLAIN QUERY PLAN` output for the due-cards query captured in a comment, showing index use
- [ ] An injection self-test: a card whose front text is `x'); DROP TABLE cards; --` — added, studied, and listed without incident, present in the repo's seed data as a permanent regression test
- [ ] Manual test checklist in comments or a short md: each command run happy-path and at least one failure path, with observed behavior noted

## Hints

- The due-cards query is the app's heart: a card's due-ness comes from its **latest** review only. "Latest per group" is a correlated subquery or a `NOT EXISTS (a later review)` — Chapter 8 patterns. Get it right in the sqlite3 shell before touching Python.
- Never-reviewed cards have no latest review — that's a LEFT JOIN whose NULL means "due now." Write down the three-valued logic before coding it.
- Compute next-due in SQL (`DATE('now', '+3 days')`) or in Python (`datetime`) — one place, not both. Pick and comment.
- Keep SQL in module-level constants or a dedicated section; functions take a connection as their first argument (as Chapter 15's example does). Testability follows for free.
- `argparse` subparsers give you `flashcards.py study spanish` structure in ~15 lines; don't hand-roll argv parsing.
- For the streak: distinct review dates, ordered descending, counted until the first gap. A recursive CTE can do it; so can a short Python loop over a simple query. Either is legitimate — comment your reasoning.

## Stretch Goals

- [ ] `export <deck> <file.csv>` — round-trips with `import`
- [ ] Proper SM-2-lite scheduling: store an ease factor per card, adjust it per result
- [ ] A `history` command rendering the last 30 days as a text sparkline of reviews per day (conditional aggregation meets string building)
- [ ] Package it: `pyproject.toml`, `pip install -e .`, and a `flashcards` console entry point
- [ ] A pytest suite running against a temporary database file — the professional pattern for testing database code
