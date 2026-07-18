# Chapter 10: Database Design & Normalization (1NF–3NF, ER Modeling)

## Overview

Up to now you've written queries against schemas someone else designed. This chapter teaches you to design schemas yourself — the difference between a database that stays clean for years and one that rots into contradictions. You'll learn **ER modeling** (turning a real-world domain into entities and relationships) and **normalization** (the rules — First, Second, and Third Normal Form — that eliminate redundant, contradiction-prone data).

This is the most conceptual chapter in the track and the one interviewers love. Project 4 is a hands-on normalization exercise.

## Definitions & Explanations

### Why bad design hurts: anomalies

Imagine a tutoring business tracking everything in one big table:

```
sessions_flat(student_name, student_phone, subject, tutor_name, tutor_rate, session_date)
```

Three failure modes, called **anomalies**:

- **Update anomaly**: Ada's phone number is stored on every one of her 40 session rows. Change it in 39 places and miss one → the database now *contradicts itself*, and no query can tell which number is right.
- **Insert anomaly**: you can't record a new tutor until they've taught a session — there's no row to put them on.
- **Delete anomaly**: delete Ben's only session and you've also deleted the only record that Ben exists and what his phone number was.

Normalization removes these by ensuring **every fact is stored exactly once**.

### ER modeling: entities, attributes, relationships

Before writing CREATE TABLE, model the domain:

1. **Entities** — the nouns your system must track: Student, Tutor, Subject, Session. Each becomes a table; each occurrence a row.
2. **Attributes** — properties of one entity: a student's name, phone. Each becomes a column *on that entity's table*.
3. **Relationships** — how entities connect, with **cardinality**:
   - **One-to-many (1:N)** — a tutor teaches many sessions; a session has one tutor. → FK on the *many* side (`sessions.tutor_id`).
   - **Many-to-many (M:N)** — a student studies many subjects; a subject has many students. → **junction table** (Chapter 6).
   - **One-to-one (1:1)** — a user has one profile. → FK with UNIQUE, or merge into one table.

A quick pencil sketch — boxes for entities, lines for relationships, crow's feet on the "many" ends — catches most design errors before any SQL exists. Ask of every line: "Can one X have many Y? Can one Y have many X?" The two answers determine the shape.

### Functional dependencies (the idea under the forms)

Column B is **functionally dependent** on column A if knowing A tells you B: `student_id → student_phone`. Normal forms are rules about *which* dependencies are allowed to live in the same table. The slogan worth memorizing: every non-key column must depend on **"the key, the whole key, and nothing but the key."**

### First Normal Form (1NF) — atomic values, no repeating groups

Each cell holds **one** value; no arrays-in-a-cell, no numbered column families.

Violations:
- `subjects = 'Math, Physics, Chemistry'` in one cell → can't query, index, or constrain individual subjects (`LIKE '%Math%'` also wrongly matches "Aftermath").
- `phone1, phone2, phone3` columns → what about the 4th phone? And "find anyone with phone X" needs an OR across every column.

Fix: one row per value, in a child table (`student_phones(student_id, phone)`), or a junction table for M:N.

### Second Normal Form (2NF) — no partial dependencies

Applies to tables with **composite keys**. Every non-key column must depend on the *whole* key, not just part.

Violation: `enrollments(student_id, course_id, grade, student_name)` — key is (student_id, course_id); `grade` depends on both (fine); `student_name` depends on student_id **alone** (partial dependency). Ada's name repeats per enrollment → update anomaly.

Fix: move `student_name` to `students`, keyed by student_id. Tables with single-column surrogate keys are automatically 2NF — one reason surrogate keys are pleasant.

### Third Normal Form (3NF) — no transitive dependencies

Non-key columns must not depend on **other non-key columns**.

Violation: `students(id, name, zip_code, city)` where zip determines city: `id → zip → city`. City is stored redundantly per student sharing a zip; a typo makes zip 80202 map to two different cities.

Fix: `zip_codes(zip, city)` table; students keep only `zip`.

**Test each non-key column:** "Do I know this because of the key, or because of some other column?" If the latter → move it out.

### How far to go — and when to stop

3NF is the practical target for transactional (web app) databases. Higher forms (BCNF, 4NF...) exist and matter in edge cases; you can learn them when a real case bites you.

**Deliberate denormalization** — storing derived or duplicated data on purpose (e.g., caching `order_total` on orders instead of summing line items every time) — is a legitimate *performance* decision made **after** designing a clean 3NF schema and measuring, never a starting point. Every denormalized copy is a consistency obligation you must maintain in code.

One nuance that surprises people: an order's line item should copy `unit_price` from products at purchase time — not because of denormalization sloppiness, but because *the price at time of sale* is a genuinely different fact from *the product's current price*. Historical facts deserve their own storage.

## Code Examples

The messy flat table, and its normalization:

```sql
-- ❌ THE MESS: everything in one table (run it to experiment)
CREATE TABLE sessions_flat (
    student_name  TEXT,
    student_phone TEXT,
    subjects      TEXT,      -- 'Math, Physics'  ← 1NF violation
    tutor_name    TEXT,
    tutor_rate    REAL,      -- depends on tutor, not on the session ← 3NF violation
    session_date  TEXT,
    session_subject TEXT
);
INSERT INTO sessions_flat VALUES
    ('Ada Chen','555-0101','Math, Physics','Ms. Rivera',60,'2026-07-01','Math'),
    ('Ada Chen','555-0101','Math, Physics','Mr. Okafor',55,'2026-07-03','Physics'),
    ('Ben Ortiz','555-0102','Math','Ms. Rivera',60,'2026-07-02','Math');
-- Note Ada's phone stored twice, Rivera's rate stored twice → anomalies waiting.
```

```sql
-- ✅ THE 3NF DESIGN
PRAGMA foreign_keys = ON;

CREATE TABLE students (
    id    INTEGER PRIMARY KEY,
    name  TEXT NOT NULL,
    phone TEXT                    -- stored ONCE per student
);

CREATE TABLE tutors (
    id                INTEGER PRIMARY KEY,
    name              TEXT NOT NULL,
    hourly_rate_cents INTEGER NOT NULL   -- stored ONCE per tutor
);

CREATE TABLE subjects (
    id   INTEGER PRIMARY KEY,
    name TEXT NOT NULL UNIQUE
);

-- M:N: which subjects each student studies (replaces the comma list)
CREATE TABLE student_subjects (
    student_id INTEGER NOT NULL REFERENCES students(id) ON DELETE CASCADE,
    subject_id INTEGER NOT NULL REFERENCES subjects(id),
    PRIMARY KEY (student_id, subject_id)
);

-- The event entity: each session is one student+tutor+subject at a time
CREATE TABLE sessions (
    id           INTEGER PRIMARY KEY,
    student_id   INTEGER NOT NULL REFERENCES students(id),
    tutor_id     INTEGER NOT NULL REFERENCES tutors(id),
    subject_id   INTEGER NOT NULL REFERENCES subjects(id),
    session_date TEXT NOT NULL,
    -- rate CHARGED for this session — a historical fact, deliberately copied:
    rate_charged_cents INTEGER NOT NULL
);

-- Migrate the seed data
INSERT INTO students (name, phone) VALUES ('Ada Chen','555-0101'), ('Ben Ortiz','555-0102');
INSERT INTO tutors (name, hourly_rate_cents) VALUES ('Ms. Rivera',6000), ('Mr. Okafor',5500);
INSERT INTO subjects (name) VALUES ('Math'), ('Physics');
INSERT INTO student_subjects VALUES (1,1),(1,2),(2,1);
INSERT INTO sessions (student_id, tutor_id, subject_id, session_date, rate_charged_cents) VALUES
    (1,1,1,'2026-07-01',6000),
    (1,2,2,'2026-07-03',5500),
    (2,1,1,'2026-07-02',6000);

-- The flat view is now a QUERY, not a storage format:
SELECT st.name AS student, st.phone, su.name AS subject,
       t.name AS tutor, s.session_date, s.rate_charged_cents / 100.0 AS rate
FROM sessions s
JOIN students st ON st.id = s.student_id
JOIN tutors   t  ON t.id  = s.tutor_id
JOIN subjects su ON su.id = s.subject_id;
```

Now the payoffs: change Ada's phone in exactly one place (`UPDATE students SET phone = ... WHERE id = 1;`); add a tutor before they teach; delete a session without erasing a person.

A migration pattern you'll use in Project 4 — populating clean tables *from* a mess with `INSERT ... SELECT DISTINCT`:

```sql
INSERT INTO students (name, phone)
SELECT DISTINCT student_name, student_phone FROM sessions_flat;
```

## Common Pitfalls

**1. Comma-separated lists in a column.** The classic 1NF sin. If you ever write `WHERE tags LIKE '%urgent%'`, stop and build the child/junction table.

**2. Numbered columns (`phone1, phone2, ...`).** Same sin in disguise. Rows are cheap; columns are rigid.

```sql
-- ❌ CREATE TABLE students (..., phone1 TEXT, phone2 TEXT);
-- ✅ CREATE TABLE student_phones (student_id INTEGER REFERENCES students(id),
--                                phone TEXT NOT NULL);
```

**3. Storing computed values you could derive.** `age` goes stale every birthday — store `birth_date`, compute age in queries. Same for `order_total` (sum the line items) *until* measured performance says otherwise, and then keep it in sync deliberately.

**4. One mega-table "to avoid joins."** Joins are not the enemy; contradictory data is. Databases are built to join. Fear of joins produces exactly the flat mess this chapter dismantles.

**5. Over-normalizing genuinely atomic data.** A `colors(id, name)` lookup table for a column with three fixed values adds joins for zero integrity benefit — a `CHECK (color IN (...))` does the job. Normalize *facts that repeat*, not every string in sight.

**6. Confusing "same value" with "same fact."** Two customers both living in 'Denver' is not redundancy — those are two independent facts that happen to match. Redundancy is the *same* fact (Ada's phone) written in multiple places. Only true redundancy needs normalizing.

## Practice Exercises

1. For each table, name the normal form it violates and the specific column(s) at fault, then sketch the fix: (a) `orders(id, customer, items TEXT /* 'apple x2; bread x1' */)`; (b) `grades(student_id, course_id, score, course_title)` with PK (student_id, course_id); (c) `employees(id, name, dept_code, dept_name, dept_floor)`.
2. Draw (paper or text) the ER model for a lending library: members borrow books; a book title can have multiple physical copies; each loan has checkout and due dates; members can place holds on titles. Identify every entity, every relationship's cardinality, and where each FK or junction table goes.
3. Convert your library ER model into CREATE TABLE statements with full constraints (PKs, FKs with sensible ON DELETE, NOT NULL, UNIQUE, CHECK where fitting). Seed 2–3 rows per table and write one join query proving a member's current loans can be listed with book titles.
4. Take this 1NF-violating table and normalize it end-to-end with SQL — create the clean tables, migrate with `INSERT ... SELECT DISTINCT`, verify row counts: `playlists_flat(owner_email, playlist_name, song_titles TEXT /* comma-separated */)`. (You may hand-enter the song splits; automating string-splitting comes later.)
5. Identify one deliberate denormalization you would defend in a real ticketing system (events, ticket sales, refunds) — say precisely which derived value you'd store, what could now become inconsistent, and what code/constraint would keep it honest.
