# SQL & Databases Learning Track

A complete, self-paced path from "what is a database?" to designing, optimizing, and operating the full data layer of a web application — built for a learner with some Python/JavaScript basics, on Windows, aiming at web development and employability.

**Primary tools:** SQLite (zero setup, everything runs locally), DB Browser for SQLite, VS Code, Python's built-in `sqlite3`. The final chapter maps everything onto PostgreSQL and the wider ecosystem (ORMs, migrations, NoSQL) so the skills transfer directly to real jobs.

## How to use this track

1. **Read chapters in order** — each assumes only the ones before it. Type out and run every code example; the sample schemas at the top of each chapter make everything runnable as-is.
2. **Do the practice exercises** at the end of each chapter (no solutions provided — the struggle is the learning; verify your answers by inspecting results).
3. **Build the projects at the marked checkpoints.** Projects bundle several chapters and are where knowledge becomes skill. No solution code exists anywhere — the specs, hints, and your chapters are the support system.
4. Keep everything in SQL script files so any database can be rebuilt from scratch — a habit the track reinforces throughout.

## Chapters (`learning-docs/`)

| # | File | Topic |
|---|------|-------|
| 1 | `01-databases-relational-model-setup.md` | What databases are, the relational model, SQLite + DB Browser setup on Windows |
| 2 | `02-select-basics-and-filtering.md` | SELECT, WHERE, ORDER BY, LIMIT |
| 3 | `03-operators-expressions-functions.md` | Operators, expressions, text/number/date functions |
| 4 | `04-inserting-updating-deleting.md` | INSERT, UPDATE, DELETE — and the safety habits |
| 5 | `05-data-types-and-table-creation.md` | Data types, CREATE TABLE, constraints, primary & foreign keys |
| 6 | `06-joins.md` | INNER/LEFT/self joins, many-to-many with junction tables |
| 7 | `07-aggregation-group-by.md` | GROUP BY, HAVING, aggregate functions |
| 8 | `08-subqueries-and-ctes.md` | Subqueries, CTEs, recursive CTEs |
| 9 | `09-set-operations-and-case.md` | UNION/INTERSECT/EXCEPT, CASE, conditional aggregation |
| 10 | `10-database-design-and-normalization.md` | ER modeling, 1NF–3NF, design anomalies |
| 11 | `11-indexes-and-query-performance.md` | Indexes, EXPLAIN QUERY PLAN, performance |
| 12 | `12-transactions-and-acid.md` | Transactions, ACID, concurrency |
| 13 | `13-views-and-stored-logic.md` | Views, triggers, where logic should live |
| 14 | `14-null-handling-and-data-quality.md` | Three-valued logic, NULL patterns, auditing & cleaning dirty data |
| 15 | `15-sql-from-code-python.md` | Python `sqlite3`, parameterized queries, SQL injection |
| 16 | `16-postgresql-and-the-wider-ecosystem.md` | PostgreSQL, dialect differences, ORMs, migrations, NoSQL, what to learn next |

## Projects (`projects/`)

| # | File | Builds on chapters | What you build |
|---|------|--------------------|----------------|
| 1 | `01-personal-media-library.md` | 1–5 | Single-table library of your real media, queried every which way |
| 2 | `02-tutoring-gradebook.md` | 2–6 | Multi-table tutoring-business schema with join-heavy reports |
| 3 | `03-sales-data-analysis.md` | 3, 6–9 | Analyst-style reporting over generated e-commerce data |
| 4 | `04-normalize-the-spreadsheet.md` | 4, 5, 8, 10, 14 | Rescue a messy flat data dump into a clean 3NF schema |
| 5 | `05-bank-ledger.md` | 5, 7, 8, 11–13 | Append-only transactional ledger with audit triggers, indexed at 100k rows |
| 6 | `06-python-cli-flashcards.md` | 5–8, 11, 12, 14, 15 | Complete Python CLI app backed by SQLite |
| 7 | `07-capstone-web-app-database.md` | All | The entire database layer for a web app: designed, seeded, queried, optimized, documented |

## Suggested cadence (~14 weeks at 5–7 hrs/week)

| Week | Study | Build |
|------|-------|-------|
| 1 | Chapters 1–2 | — |
| 2 | Chapters 3–4 | — |
| 3 | Chapter 5 | **Project 1** |
| 4 | Chapter 6 | start **Project 2** |
| 5 | Chapter 7 | finish Project 2 |
| 6 | Chapters 8–9 | start **Project 3** |
| 7 | — | finish Project 3 |
| 8 | Chapter 10 | start **Project 4** |
| 9 | Chapter 14 (pairs naturally with 10) | finish Project 4 |
| 10 | Chapters 11–12 | start **Project 5** |
| 11 | Chapter 13 | finish Project 5 |
| 12 | Chapter 15 | start **Project 6** |
| 13 | Chapter 16, finish Project 6 | start **Project 7** |
| 14+ | — | **Project 7** (capstone; allow 2–3 weeks) |

Faster or slower is fine — the ordering is what matters. Note the deliberate resequencing: Chapter 14 (NULLs & data quality) is studied alongside Project 4, where it's needed most; chapters 11–13 cluster before the ledger project that exercises all three.

## Milestones of "I've got this"

- **After Project 2:** you can design small relational schemas and join across them without notes.
- **After Project 4:** you can look at any spreadsheet and see the 3NF schema hiding inside it — a genuinely interview-ready skill.
- **After Project 6:** you can build database-backed applications in Python with safe, parameterized SQL.
- **After Project 7:** you own the vocabulary and the artifacts (schema, migrations, seeded data, tuned queries) of a working backend developer. Next steps from there are in Chapter 16: PostgreSQL hands-on, window functions, and the ORM of your chosen web stack.
