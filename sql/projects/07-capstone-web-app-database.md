# Project 7: Capstone — The Complete Database for a Web App

## Description

Design, build, seed, optimize, and operate the full database layer for a realistic web application, end to end — exactly the artifact you'd hand a team (or show an interviewer) to prove you can own "the data side" of a product. Recommended domain: a **community recipe-sharing site** (users, recipes, ingredients, tags, ratings, comments, favorites, follows). An equally rich domain of your own choosing is welcome — it must include at least: authentication-shaped user data, two many-to-many relationships, one self-referential relationship, one tree or thread structure, and an activity/event stream.

There is no web framework in this project — you're building what the framework would sit on: the schema, the seed data, the query layer, the performance work, and the operational scripts. The deliverable is a repository someone else could pick up and build an app against.

## Difficulty

**Advanced (capstone)** — estimated effort: 15–25 hours, best spread over 2–3 weeks.

## Chapters Used

All of them. Explicitly: 5 & 10 (design), 6–9 (query layer), 11 (performance), 12 (transactional operations), 13 (views & triggers), 14 (data quality), 15 (Python access layer), 16 (portability notes).

## Requirements Checklist

### Phase 1 — Design (before any CREATE TABLE)
- [ ] A written design doc: entity list, ER diagram (text or image), every relationship's cardinality, and for each table: its purpose, key choices, and NULLability decisions — with at least three explicitly-considered alternatives you rejected and why (e.g. "tags as CSV column — rejected, 1NF")
- [ ] The recipe domain needs ≥ 10 tables; expect roughly: users, recipes, ingredients, recipe_ingredients (with quantity/unit — attributes on the junction), tags, recipe_tags, ratings, comments (threaded — self-FK), favorites, follows (user→user self-referential M:N)
- [ ] Schema in 3NF with every deliberate denormalization (if any) documented with its consistency mechanism

### Phase 2 — Build
- [ ] `schema.sql`: full DDL, FKs with justified ON DELETE per relationship, CHECKs (rating ranges, non-empty titles, no self-follows), UNIQUEs (username, email, one-rating-per-user-per-recipe, no duplicate favorites), all rebuildable from scratch with one command
- [ ] All FK columns and hot query paths indexed, each index commented with the query it serves
- [ ] Views (≥ 3) defining the app's core read shapes — e.g. recipe cards (title, author, avg rating, rating count, tag list), user profiles with counts, a trending feed
- [ ] Triggers where justified (≥ 2) — e.g. denormalized rating summary maintenance, audit/immutability rules, updated_at stamping — each documented in the schema file

### Phase 3 — Seed
- [ ] A Python seeding script (`seed.py`, Chapter 15 discipline throughout) generating a *plausible* dataset: 200+ users, 500+ recipes, thousands of ratings/favorites/comments — with realistic skew (a few prolific users, popular recipes with many ratings, long-tail everything, comment threads 3+ deep, orphanless data)
- [ ] Seeding runs in bulk transactions and completes in seconds, is repeatable (idempotent or drop-and-rebuild), and includes edge-case users (no recipes, no followers) and the injection-string regression card from Project 6's tradition
- [ ] A data-quality audit script (Chapter 14 patterns) run post-seed proving: no orphans, no constraint-skirting values, distributions roughly as intended (counts by table reported)

### Phase 4 — The query layer (the app's API, as SQL)
Write, save, and comment the queries a real app needs — each named like the endpoint it would serve:
- [ ] Recipe page: full recipe with author, ingredients+quantities, tags, avg + count of ratings, and its comment thread rendered depth-first with indentation levels (recursive CTE)
- [ ] Home feed for a given user: recent recipes from users they follow, with rating summaries — paginated (LIMIT/OFFSET), sorted by recency
- [ ] Search: recipes matching a term in title or tag, ranked by a scoring expression you design (CASE: title match beats tag match), paginated
- [ ] Profile page: a user's recipes, their favorites, follower/following counts, and their average received rating
- [ ] Discovery: top recipes this month (time-windowed aggregation), most-used tags, rising creators (more followers gained this month than last — set ops or conditional aggregation)
- [ ] "Users who favorited this also favorited" — the collaborative-filtering join (self-join through favorites, excluding the seed recipe, ranked by overlap)
- [ ] Transactional write operations as scripts: publish a recipe with its ingredients and tags atomically; rate-or-update-rating (UPSERT); follow/unfollow with self-follow prevention proven; delete a user demonstrating every cascade/restriction firing exactly as your design doc promised

### Phase 5 — Performance & operations
- [ ] `EXPLAIN QUERY PLAN` review of every Phase-4 read query at full seed volume: plans captured in comments, no unjustified SCAN on a large table, with before/after evidence for at least two indexes you added because of this review — and timing numbers
- [ ] One deliberately denormalized optimization measured end-to-end (e.g. trigger-maintained `rating_avg` on recipes vs computing live): both implementations, timings, and your verdict in writing
- [ ] Operational scripts: backup (file copy done safely / `.backup`), integrity check (`PRAGMA integrity_check` + your own invariant queries), and a sample migration (add a feature — e.g. recipe photos table — as a numbered migration script applied to the live database without data loss)
- [ ] `PORTING.md`: what would change moving this schema to PostgreSQL (Chapter 16) — types, identity columns, the pragmas, placeholder styles, and which two features you'd gain that this app could use

### Phase 6 — Handoff
- [ ] Repository layout: `schema.sql`, `migrations/`, `seed.py`, `queries/` (organized, commented), `ops/`, design doc, and a README that gets a stranger from zero to seeded-and-querying in under five minutes on Windows
- [ ] A self-review: the three best decisions in the project, the one you'd redo, and what you'd learn next because of it

## Hints

- Do Phase 1 properly. Every hour of design saves three of migration. Steal shapes shamelessly from earlier chapters: junction-with-attributes (Ch 6), event streams (Project 5's ledger), trees (Ch 8's categories), audit triggers (Ch 13).
- Build Phase 4 queries against a *tiny* hand-seeded dataset first (predictable answers), then scale up with `seed.py` and watch which ones fall over — that's Phase 5 finding itself.
- The comment thread and the also-favorited queries are the two hardest; each is one chapter's pattern (recursive CTE; self-join dedup) wearing a costume.
- When a feed query needs data from four one-to-many relationships, resist one mega-join — compose CTEs that each aggregate one relationship to one-row-per-recipe *before* joining. Fan-out (Ch 7's pitfall) is the capstone's favorite trap.
- Skew in seed data matters: uniform random data hides the performance problems (and the interesting query results) that skewed real data exposes. Give 5% of users 50% of the content.
- Keep a running `DECISIONS.md`. Half the employability value of this project is being able to *narrate* it.

## Stretch Goals

- [ ] Full-text recipe search with SQLite FTS5, compared honestly against your LIKE-based search
- [ ] Rewrite three feed/discovery queries with window functions and benchmark both versions
- [ ] Actually port it: stand up PostgreSQL (winget/Docker), apply your PORTING.md, migrate the seed, and run the query layer there — the final proof of transferable skill
- [ ] Put a minimal web layer on top (Flask/FastAPI, a few endpoints serving your Phase-4 queries as JSON) — the moment the whole track clicks together
- [ ] A `pytest` suite asserting every invariant and transactional guarantee in the design doc
