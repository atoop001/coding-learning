# Chapter 4: Changing Data — INSERT, UPDATE, DELETE

## Overview

Reading data is half the job; the other half is writing it. This chapter covers the three **DML** (Data Manipulation Language) statements that change table contents:

- `INSERT` — add new rows
- `UPDATE` — modify existing rows
- `DELETE` — remove rows

These statements are where beginners cause real damage — an `UPDATE` without a `WHERE` clause rewrites *every row in the table*. You'll learn the safe habits professionals use (test with SELECT first, count what you're about to touch, use transactions once you reach Chapter 12).

## Sample Schema for This Chapter

```sql
CREATE TABLE tasks (
    id        INTEGER PRIMARY KEY,
    title     TEXT NOT NULL,
    priority  INTEGER NOT NULL DEFAULT 3,   -- 1 = urgent ... 5 = someday
    status    TEXT NOT NULL DEFAULT 'todo', -- 'todo', 'doing', 'done'
    due_date  TEXT,                          -- nullable: not every task has one
    created   TEXT NOT NULL DEFAULT (DATE('now'))
);
```

## Definitions & Explanations

### INSERT — adding rows

**Full form** (recommended): name the columns, then provide matching values.

```sql
INSERT INTO tasks (title, priority, due_date)
VALUES ('Write chapter 4', 1, '2026-07-20');
```

Columns you don't list get their **DEFAULT** (here: `status` becomes `'todo'`, `created` becomes today) or NULL if no default exists. The `INTEGER PRIMARY KEY` column auto-assigns the next id in SQLite when omitted.

**Multi-row insert** — one statement, many rows, much faster than repeating INSERT:

```sql
INSERT INTO tasks (title, priority) VALUES
    ('Buy groceries', 4),
    ('Renew passport', 2),
    ('Practice SQL', 3);
```

**Column-less form** — `INSERT INTO tasks VALUES (...)` requires a value for *every* column in table order. Avoid it: it breaks the moment the table gains a column.

**INSERT ... SELECT** — copy rows from a query:

```sql
INSERT INTO archived_tasks (title, priority, status)
SELECT title, priority, status FROM tasks WHERE status = 'done';
```

### UPDATE — modifying rows

```sql
UPDATE tasks
SET status = 'doing'
WHERE id = 2;
```

- `SET` lists one or more `column = expression` pairs, comma-separated.
- `WHERE` chooses **which rows** get changed. **No WHERE = every row changes.**
- The expression can reference the row's own current values: `SET priority = priority - 1` bumps urgency.

### DELETE — removing rows

```sql
DELETE FROM tasks WHERE status = 'done';
```

- Removes whole rows (you can't delete a single column value — that's `UPDATE ... SET col = NULL`).
- **No WHERE = table emptied.** The table itself survives (unlike `DROP TABLE`, which destroys the structure too).

### The golden safety habit: SELECT first

Before any UPDATE or DELETE, run the *same WHERE clause* as a SELECT to see exactly which rows you're about to touch:

```sql
-- Step 1: preview
SELECT * FROM tasks WHERE due_date < DATE('now') AND status = 'todo';
-- Step 2: only if the preview looks right, reuse the WHERE:
UPDATE tasks SET priority = 1
WHERE due_date < DATE('now') AND status = 'todo';
```

### How many rows did I change?

The SQLite shell can tell you with `SELECT changes();` right after a statement. Client libraries expose this too (Python's `cursor.rowcount`, Chapter 15). Checking affected-row counts is how real applications detect "the update matched nothing."

### RETURNING (modern nicety)

SQLite (3.35+), PostgreSQL, and others support `RETURNING` to see what a write did without a second query:

```sql
INSERT INTO tasks (title) VALUES ('Call dentist') RETURNING id, created;
DELETE FROM tasks WHERE status = 'done' RETURNING id, title;
```

## Code Examples

Work through these in order in a fresh database with the schema above.

```sql
-- 1. Insert with defaults doing the work
INSERT INTO tasks (title) VALUES ('Set up backups');
SELECT * FROM tasks;   -- note auto id, priority 3, status 'todo', created today

-- 2. Insert specifying more columns
INSERT INTO tasks (title, priority, status, due_date)
VALUES ('File taxes', 1, 'doing', '2026-04-15');

-- 3. Bulk insert
INSERT INTO tasks (title, priority, due_date) VALUES
    ('Buy groceries', 4, NULL),
    ('Renew passport', 2, '2026-09-01'),
    ('Practice SQL',  3, NULL),
    ('Clean garage',  5, NULL);

-- 4. Simple targeted update: finish a task by id
UPDATE tasks SET status = 'done' WHERE id = 1;

-- 5. Update multiple columns at once
UPDATE tasks
SET status = 'doing',
    priority = 2
WHERE title = 'Practice SQL';

-- 6. Update using the current value: everything low-priority gets bumped up one
--    (preview first!)
SELECT id, title, priority FROM tasks WHERE priority >= 4;
UPDATE tasks SET priority = priority - 1 WHERE priority >= 4;

-- 7. Set a column to NULL (that's how you "clear" a value)
UPDATE tasks SET due_date = NULL WHERE title = 'File taxes';

-- 8. Conditional text update with an expression
UPDATE tasks
SET title = TRIM(title)          -- clean whitespace everywhere it's needed
WHERE title <> TRIM(title);      -- ...but only touch rows that actually change

-- 9. Delete one row
DELETE FROM tasks WHERE id = 4;

-- 10. Delete by condition (preview shown first)
SELECT COUNT(*) FROM tasks WHERE status = 'done';
DELETE FROM tasks WHERE status = 'done';

-- 11. INSERT ... SELECT: build an archive table and move finished work into it
CREATE TABLE archived_tasks (
    id       INTEGER PRIMARY KEY,
    title    TEXT NOT NULL,
    archived TEXT NOT NULL DEFAULT (DATE('now'))
);
INSERT INTO tasks (title, status) VALUES ('Old finished thing', 'done');
INSERT INTO archived_tasks (title)
SELECT title FROM tasks WHERE status = 'done';
DELETE FROM tasks WHERE status = 'done';

-- 12. RETURNING: create and immediately learn the new id
INSERT INTO tasks (title, priority) VALUES ('Ship the feature', 1)
RETURNING id, title, created;
```

## Common Pitfalls

**1. The missing WHERE — the classic disaster.**

```sql
-- ❌ Marks EVERY task done:
UPDATE tasks SET status = 'done';

-- ✅ What was meant:
UPDATE tasks SET status = 'done' WHERE id = 7;
```

Habit: write the `WHERE` clause *first*, or write the SELECT preview and edit `SELECT *` into `UPDATE ... SET`. Once you know transactions (Chapter 12), wrap risky changes in `BEGIN;` ... check ... `COMMIT;`/`ROLLBACK;`.

**2. AND instead of comma in SET.**

```sql
-- ❌ Legal SQL, wrong meaning: sets status to the RESULT of (('doing') AND priority = 2),
--    i.e. a boolean 0/1 — not what you want:
UPDATE tasks SET status = 'doing' AND priority = 2 WHERE id = 3;

-- ✅ Comma-separate assignments:
UPDATE tasks SET status = 'doing', priority = 2 WHERE id = 3;
```

**3. Expecting DELETE to reset auto-increment ids.**
After deleting rows, new inserts continue from the highest id ever used, and that's *good* — ids should never be reused (old references would silently point at new rows). Don't fight it.

**4. Violating constraints and not reading the error.**
`INSERT INTO tasks (priority) VALUES (2);` fails with `NOT NULL constraint failed: tasks.title`. The error names the exact column — read it. Constraint errors are the database *protecting* you (Chapter 5 covers designing constraints).

**5. Deleting parents that have children.**
Once foreign keys are in play (Chapters 5–6), deleting a row that other tables reference either fails (good default) or cascades (deletes children too). Know which behavior your schema declares *before* you delete. In SQLite, foreign key enforcement must be enabled per connection: `PRAGMA foreign_keys = ON;`.

**6. String-building your SQL in application code.**
When you reach Chapter 15: never paste user input into an INSERT/UPDATE string — that's SQL injection. Parameterized queries only.

## Practice Exercises

Set up the `tasks` schema fresh, then:

1. Insert five tasks of your own in a **single** multi-row INSERT: at least one with only a title (letting all defaults apply), one urgent (priority 1) with a due date next week, and one with status `'doing'`.
2. You've finished the urgent task. Write the preview SELECT, then the UPDATE that marks exactly that task `'done'` — targeting it by id, not title.
3. Priorities have drifted: make every `'todo'` task that has **no due date** one step *less* urgent (numerically higher), but never beyond 5 (hint: `MIN(...)` from Chapter 3, or a WHERE guard).
4. Create a table `deleted_tasks (id, title, deleted_on)` and write the two-statement sequence that archives then removes all `'done'` tasks. Verify with counts before and after.
5. Deliberately run an UPDATE with no WHERE on a *throwaway copy* of the table (make one with `CREATE TABLE tasks_copy AS SELECT * FROM tasks;`), observe the damage, and write down the two habits you'll use to prevent this in real data.
