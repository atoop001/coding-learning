# Project 1: Personal Media Library

## Description

Build and query your first real database from scratch: a personal library of the media you own or track — books, movies, games, albums, whatever you actually collect. You'll design a small schema, seed it with genuine data (at least 25 items — real data makes querying interesting), and then answer a series of questions about your collection using SELECT, filtering, sorting, expressions, and data modification.

This project cements the fundamentals: if you can create, populate, and interrogate this database fluently, the rest of the track has a solid floor to build on.

## Difficulty

**Beginner** — estimated effort: 3–5 hours.

## Chapters Used

- Chapter 1 (setup, creating a database)
- Chapter 2 (SELECT, WHERE, ORDER BY, LIMIT)
- Chapter 3 (expressions, text/date functions)
- Chapter 4 (INSERT, UPDATE, DELETE)
- Chapter 5 (CREATE TABLE, types, constraints — basic use)

## Requirements Checklist

### Schema & data
- [ ] Create a database file `media_library.db` with a single `items` table
- [ ] Columns must include at minimum: an auto-numbered primary key, title (required), media type (e.g. book/movie/game — required), creator (author/director/artist), release year, a personal rating (0–10, may be empty if unrated), date you acquired it (ISO text), and a "finished/completed" flag
- [ ] At least two constraints beyond the primary key: one NOT NULL (besides title) and one CHECK (e.g. valid rating range or allowed media types)
- [ ] Seed at least 25 real items using multi-row INSERT statements, covering at least 3 media types, including some unrated items (NULL rating)
- [ ] Keep all your SQL in a script file (`setup.sql` / `queries.sql`) so you can rebuild the database from scratch with one command

### Queries (write and save each one, with a comment stating the question)
- [ ] All items of one media type, alphabetized by title
- [ ] Your top 5 rated items overall (think about where unrated items should go)
- [ ] Items released before 2000 that you have not finished
- [ ] Items acquired in the last 12 months, newest first (use a date function, not a hardcoded date)
- [ ] A search: all items whose title contains a word of your choice, case handled sensibly
- [ ] A computed column: each item's age in years based on release year, sorted oldest first
- [ ] A display label built with concatenation, e.g. `Dune (1965) — book — 9/10`, with unrated items showing `unrated` instead of a number
- [ ] "Page 2" of your collection: items 11–20 alphabetically

### Data changes
- [ ] Update: you finished something — set its finished flag and give it a rating, targeting by id
- [ ] Update: a batch change affecting several rows with one statement (e.g. bump every rating of one creator, or fix a misspelled creator name everywhere)
- [ ] Delete: remove one item you no longer track — with the preview-SELECT-first habit shown in your script
- [ ] Demonstrate one intentionally failed INSERT per constraint you declared (keep them in the script, commented out, with the error message noted)

## Hints

- Decide your media types up front and enforce them with a CHECK — it will save you from `'Book'` vs `'book'` inconsistency later.
- For the "top 5 rated" query, remember what NULL does to sorting — try it and look carefully at where unrated items land.
- The label-building query combines `||`, `COALESCE`, and `CAST` — build it up one piece at a time.
- If a query result surprises you, run just its WHERE clause with `SELECT *` and check what rows survive.
- Rebuild from your script often: `sqlite3 media_library.db ".read setup.sql"` after `DROP TABLE IF EXISTS` at the top.

## Stretch Goals

- [ ] Add a `notes` column and backfill notes for a few items with UPDATEs
- [ ] Write a query producing a "shelf report": one line per media type with a count (a preview of Chapter 7 — try it with what you know, then revisit after aggregation)
- [ ] Add a second table `wishlist` with a similar shape, and use INSERT ... SELECT to "purchase" an item — moving it from wishlist to items in two statements
- [ ] Import your real data from a CSV export (DB Browser's File → Import can help) and clean up whatever mess arrives
