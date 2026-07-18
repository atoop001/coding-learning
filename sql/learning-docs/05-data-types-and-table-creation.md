# Chapter 5: Data Types, CREATE TABLE & Constraints

## Overview

So far you've used tables handed to you. Now you learn to design them: choosing data types, declaring **constraints** (rules the database enforces so bad data physically cannot get in), and wiring tables together with **primary and foreign keys**. Constraints are one of the biggest practical advantages of databases over "just files" — the database becomes an active guardian of data quality instead of a passive bucket.

This chapter is SQLite-focused but constantly notes what changes in PostgreSQL/MySQL, because table design is where dialects differ most.

## Definitions & Explanations

### SQLite's type system (and how it's unusual)

SQLite has five **storage classes**:

| Storage class | Holds | Typical declared types |
|---|---|---|
| `INTEGER` | whole numbers (up to 8 bytes) | `INTEGER`, `INT` |
| `REAL` | floating-point numbers | `REAL`, `DOUBLE`, `FLOAT` |
| `TEXT` | strings (UTF-8) | `TEXT`, `VARCHAR(n)`, `CHAR(n)` |
| `BLOB` | raw bytes (images, files) | `BLOB` |
| `NULL` | the absence of a value | — |

Key quirks:

- **Type affinity, not strict typing.** Classic SQLite lets you store `'hello'` in an INTEGER column; the declared type is a *preference* (affinity), not a law. Modern SQLite (3.37+) offers `CREATE TABLE ... STRICT` tables that enforce types — use STRICT while learning if your version supports it. PostgreSQL/MySQL always enforce types.
- **No BOOLEAN type.** Convention: `INTEGER` with 0/1. (SQLite accepts the literals `TRUE`/`FALSE` as 1/0.)
- **No DATE/TIME types.** Convention: `TEXT` in ISO-8601 (`'2026-07-18'`, `'2026-07-18T14:30:00'`). ISO text sorts chronologically, and Chapter 3's date functions understand it.
- **No DECIMAL type.** For money, store **integer cents** (`1999` = $19.99) to avoid floating-point rounding errors, or accept REAL for casual use. PostgreSQL has `NUMERIC` for exact decimals.
- `VARCHAR(50)` is accepted but the length limit is **not enforced** by SQLite (it is elsewhere).

### CREATE TABLE anatomy

```sql
CREATE TABLE table_name (
    column_name TYPE [column-constraints...],
    ...,
    [table-constraints...]
);
```

### Column constraints

- `NOT NULL` — value required. Default is nullable; make columns NOT NULL unless "unknown" is genuinely meaningful.
- `DEFAULT expr` — value used when INSERT omits the column. Can be a literal or (in parentheses) an expression: `DEFAULT (DATE('now'))`.
- `UNIQUE` — no two rows may share this value (NULLs are exempt — multiple NULLs allowed).
- `CHECK (condition)` — every row must satisfy the condition: `CHECK (price >= 0)`, `CHECK (status IN ('todo','doing','done'))`.
- `PRIMARY KEY` — see below.
- `REFERENCES other_table(col)` — foreign key, see below.

### Primary keys

The **primary key (PK)** uniquely identifies each row; it's automatically UNIQUE and NOT NULL, and other tables point at it.

- In SQLite, `id INTEGER PRIMARY KEY` is special: it becomes an alias for the internal rowid and **auto-increments** — you rarely need the `AUTOINCREMENT` keyword (which is stricter and slower; skip it unless you must guarantee ids are never reused).
- A **natural key** is a real-world unique value (email, ISBN). A **surrogate key** is a meaningless auto-number. Best practice for beginners: use a surrogate `id INTEGER PRIMARY KEY` and add `UNIQUE` on natural candidates like email — real-world values change and repeat more often than you'd think.
- A **composite primary key** spans multiple columns, declared as a table constraint: `PRIMARY KEY (student_id, course_id)`. Common in junction tables (Chapter 6).
- PostgreSQL equivalents: `id SERIAL PRIMARY KEY` or the modern `id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY`.

### Foreign keys

A **foreign key (FK)** declares "values in this column must exist as a primary key in that table," making dangling references impossible:

```sql
owner_id INTEGER NOT NULL REFERENCES owners(id)
```

- **SQLite gotcha:** enforcement is OFF by default per connection. Run `PRAGMA foreign_keys = ON;` at the start of every session/connection (DB Browser has a setting; in Python you execute the pragma after connecting).
- **ON DELETE behavior** — what happens when the referenced parent row is deleted:
  - default (`NO ACTION`/`RESTRICT`): the delete **fails** if children exist — safe default.
  - `ON DELETE CASCADE`: children are deleted automatically — right for "owned" data (delete a post → delete its comments).
  - `ON DELETE SET NULL`: child's FK becomes NULL — right for optional links (FK column must be nullable).
- `ON UPDATE CASCADE` similarly follows PK changes (rare with surrogate keys).

### Changing and removing tables

- `ALTER TABLE t ADD COLUMN col TYPE ...;` — SQLite supports adding columns (with limitations: the new column can't be `UNIQUE`, and NOT NULL requires a default). SQLite 3.25+ also supports `RENAME COLUMN` and `RENAME TO`. Dropping columns works from 3.35+. Anything fancier: create a new table, copy data, drop the old one.
- `DROP TABLE t;` — destroys table and data. `DROP TABLE IF EXISTS t;` avoids an error when it's already gone (standard at the top of setup scripts).

## Code Examples

A small but realistic two-table design — pet owners and their pets:

```sql
PRAGMA foreign_keys = ON;             -- ALWAYS in SQLite sessions using FKs

DROP TABLE IF EXISTS pets;            -- children first (FK order matters)
DROP TABLE IF EXISTS owners;

CREATE TABLE owners (
    id        INTEGER PRIMARY KEY,                 -- surrogate, auto-numbered
    name      TEXT NOT NULL,
    email     TEXT NOT NULL UNIQUE,                -- natural candidate key
    city      TEXT NOT NULL DEFAULT 'Unknown',
    joined_on TEXT NOT NULL DEFAULT (DATE('now'))
);

CREATE TABLE pets (
    id        INTEGER PRIMARY KEY,
    owner_id  INTEGER NOT NULL
              REFERENCES owners(id) ON DELETE CASCADE,  -- pet belongs to owner
    name      TEXT NOT NULL,
    species   TEXT NOT NULL CHECK (species IN ('dog','cat','bird','other')),
    birth_year INTEGER CHECK (birth_year BETWEEN 1990 AND 2100),
    weight_kg REAL CHECK (weight_kg > 0)
);

-- Seed data
INSERT INTO owners (name, email, city) VALUES
    ('Ada Chen',  'ada@example.com',  'Denver'),
    ('Ben Ortiz', 'ben@example.com',  'Austin');

INSERT INTO pets (owner_id, name, species, birth_year, weight_kg) VALUES
    (1, 'Biscuit', 'dog',  2019, 24.5),
    (1, 'Mochi',   'cat',  2021, 4.2),
    (2, 'Kiwi',    'bird', 2022, 0.09);
```

Now watch the constraints do their job — each of these **fails on purpose**:

```sql
-- UNIQUE violation: email already used
INSERT INTO owners (name, email) VALUES ('Fake Ada', 'ada@example.com');
-- Error: UNIQUE constraint failed: owners.email

-- CHECK violation: species not in the allowed list
INSERT INTO pets (owner_id, name, species) VALUES (1, 'Rex', 'dinosaur');
-- Error: CHECK constraint failed

-- NOT NULL violation
INSERT INTO pets (owner_id, species) VALUES (1, 'cat');
-- Error: NOT NULL constraint failed: pets.name

-- FK violation: owner 99 does not exist
INSERT INTO pets (owner_id, name, species) VALUES (99, 'Ghost', 'cat');
-- Error: FOREIGN KEY constraint failed
```

And the cascade in action:

```sql
DELETE FROM owners WHERE id = 1;   -- Ada leaves...
SELECT * FROM pets;                -- ...Biscuit and Mochi are gone too (CASCADE)
```

Evolving a table:

```sql
ALTER TABLE pets ADD COLUMN microchipped INTEGER NOT NULL DEFAULT 0;
ALTER TABLE pets RENAME COLUMN weight_kg TO weight;
```

A STRICT table (if your SQLite is 3.37+):

```sql
CREATE TABLE readings (
    id    INTEGER PRIMARY KEY,
    value REAL NOT NULL
) STRICT;

INSERT INTO readings (value) VALUES ('not a number');
-- Error in STRICT mode; silently stored as text in a normal table!
```

## Common Pitfalls

**1. Forgetting `PRAGMA foreign_keys = ON;` in SQLite.**
Without it, FK declarations are decorative — bad references insert silently. Put the pragma at the top of every script and every application connection.

```sql
-- ❌ (new connection, no pragma) — succeeds even though owner 999 doesn't exist:
INSERT INTO pets (owner_id, name, species) VALUES (999, 'Oops', 'cat');
-- ✅ PRAGMA foreign_keys = ON;  -- then the same insert correctly fails
```

**2. Storing money as REAL and being surprised.**
`0.1 + 0.2` is not exactly `0.3` in floating point. For a ledger (Project 5), store integer cents: `amount_cents INTEGER NOT NULL`.

**3. Using a changeable real-world value as the primary key.**

```sql
-- ❌ Emails change; every referencing row would need updating:
CREATE TABLE users (email TEXT PRIMARY KEY, ...);

-- ✅ Surrogate PK + unique natural key:
CREATE TABLE users (id INTEGER PRIMARY KEY, email TEXT NOT NULL UNIQUE, ...);
```

**4. Dropping/creating tables in the wrong order.**
You can't drop a parent while children reference it (with FKs on), and you can't create a child before its parent exists. Order scripts: **create parents first, drop children first.**

**5. Relying on `VARCHAR(50)` to limit length in SQLite.**
It won't. If a length cap matters, enforce it: `CHECK (LENGTH(username) <= 50)`.

**6. Nullable-by-default sloppiness.**
Every column you don't mark NOT NULL can be NULL, and Chapter 14 shows how NULLs complicate everything. Decide *deliberately* for each column: is "unknown" a valid state?

## Practice Exercises

1. Design and create a `students` table for a tutoring business: surrogate primary key, required name, unique optional email, grade level restricted by CHECK to 1–12, hourly rate stored safely for money math, and a signup date defaulting to today. Insert three students, then prove each constraint works by attempting one violating insert per constraint.
2. Add a `subjects` table (id, unique name) and a `sessions` table where each session references one student and one subject, has a date, and a duration in minutes that must be positive. Choose and justify (in a comment) the ON DELETE behavior for each foreign key.
3. Turn foreign keys ON, insert valid sessions, then try to (a) insert a session for a nonexistent student and (b) delete a student who has sessions. Record what happens for each, and explain why your chosen ON DELETE behavior produced it.
4. You need to add a `notes` column and rename `rate` to `hourly_rate_cents` on your live `students` table without losing data. Write the ALTER TABLE statements, and note which (if either) would have required the copy-to-new-table workaround on an older SQLite.
5. Recreate your `students` table as STRICT (if supported) or write CHECK constraints that simulate strictness (e.g., `CHECK (typeof(grade_level) = 'integer')`). Demonstrate one insert that a sloppy table would accept but your strict version rejects.
