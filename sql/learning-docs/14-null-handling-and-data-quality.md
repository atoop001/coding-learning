# Chapter 14: NULL Handling & Data Quality

> **Reading order note:** this track's README suggests studying this chapter right after Chapter 10, alongside Project 4, rather than strictly in file order — that's where NULL handling and data-quality auditing are needed most. It's safe to read any time after Chapter 10; nothing here depends on Chapters 11–13.

## Overview

NULL has ambushed you in nearly every chapter: comparisons that silently match nothing, counts that differ, concatenations that vanish, `NOT IN` returning zero rows. This chapter finally gives NULL the full treatment — **three-valued logic**, every NULL-aware operator, and the query patterns that behave correctly — then widens out to practical **data quality**: finding and fixing duplicates, orphans, inconsistent formats, and out-of-range values in real tables. Real-world data is dirty; employers prize people who can audit and clean it with SQL.

## Definitions & Explanations

### What NULL actually means

NULL is **not** zero, not an empty string, not false. It means *"no value here"* — unknown, missing, or not applicable. Three different real-world situations collapse into one marker (unknown phone vs. has-no-phone vs. not-asked-yet), which is why Chapter 5 urged you to allow NULL only deliberately.

### Three-valued logic (3VL)

SQL conditions evaluate to **TRUE, FALSE, or UNKNOWN**. Any comparison with NULL yields UNKNOWN — including `NULL = NULL` (are two unknown values equal? Unknown!).

The rules cascade:

| Expression | Result |
|---|---|
| `NULL = 5`, `NULL <> 5`, `NULL > 5` | UNKNOWN |
| `NULL = NULL` | UNKNOWN |
| `NOT UNKNOWN` | UNKNOWN |
| `TRUE AND UNKNOWN` | UNKNOWN |
| `FALSE AND UNKNOWN` | FALSE (false wins an AND) |
| `TRUE OR UNKNOWN` | TRUE (true wins an OR) |
| `FALSE OR UNKNOWN` | UNKNOWN |

**WHERE keeps only TRUE rows.** UNKNOWN rows are dropped — silently. This single fact explains most NULL bugs: `WHERE phone <> '555-0100'` excludes not only that number but also *everyone with a NULL phone*.

### The NULL-aware toolkit

- `IS NULL` / `IS NOT NULL` — the *only* correct null tests.
- `COALESCE(a, b, ...)` — first non-NULL argument; the universal defaulter.
- `NULLIF(a, b)` — NULL if `a = b`; converts sentinel values (`''`, `0`, `'N/A'`) into honest NULLs and guards division.
- `IS NOT DISTINCT FROM` (PostgreSQL) / `IS` on values (SQLite quirk) — null-safe equality where `NULL` "equals" `NULL`. Portable version: `(a = b OR (a IS NULL AND b IS NULL))`.

### NULL behavior you've met, consolidated

- **Aggregates skip NULLs**: `COUNT(col)` < `COUNT(*)` when NULLs exist; `AVG` divides by non-NULL count only; `SUM` of no rows is NULL (wrap in COALESCE).
- **GROUP BY treats NULLs as one group** (they cluster together) — inconsistent with `=`, but useful.
- **UNIQUE constraints permit multiple NULLs** (in SQLite/PostgreSQL default) — two users with no email don't collide.
- **ORDER BY**: SQLite sorts NULLs first ascending; use `ORDER BY col IS NULL, col` (or `NULLS LAST` in SQLite 3.30+/PostgreSQL) to control placement.
- **`NOT IN` + NULL = empty result** (Chapter 8): prefer `NOT EXISTS`.
- **String/math with NULL yields NULL**: `NULL + 1`, `'a' || NULL`.
- **CASE without matching WHEN/ELSE yields NULL**; NULL fails every `WHEN x < 5`-style test into the ELSE bucket (test `IS NULL` first).

### Data quality: the audit mindset

Beyond NULLs, dirty data means: duplicates, orphaned references, format drift (`'CO'` vs `'Colorado'`), sentinel junk (`'N/A'`, `-1`, `''`), out-of-range values, and broken invariants (end before start). The professional workflow:

1. **Profile** — counts, distinct values, min/max, NULL rates per column.
2. **Audit** — targeted queries for each suspected problem class.
3. **Fix** — set-based UPDATE/DELETE, inside transactions, previewing first (Chapter 4 discipline).
4. **Prevent** — add the constraint that would have blocked the problem (Chapter 5), so the class of bug dies permanently.

## Sample Schema for This Chapter

A deliberately dirty contacts import:

```sql
CREATE TABLE contacts (
    id         INTEGER PRIMARY KEY,
    name       TEXT,
    email      TEXT,
    phone      TEXT,
    state      TEXT,
    birth_year INTEGER,
    referred_by INTEGER            -- should reference contacts.id, but no FK — imported!
);

INSERT INTO contacts (name, email, phone, state, birth_year, referred_by) VALUES
    ('Ada Chen',  'ada@x.com',   '555-0101', 'CO',       1992, NULL),
    ('Ben Ortiz', 'BEN@X.COM',   NULL,       'Colorado', 1999, 1),
    ('ben ortiz', 'ben@x.com',   '555-0102', 'CO',       1999, 1),      -- duplicate?
    ('Carla D',   '',            '555-0103', 'TX',       2085, 1),      -- '' email, future birth year
    ('Dev P',     'dev@x.com',   'N/A',      'tx',       NULL, 42),     -- sentinel phone, orphan referrer
    (NULL,        'ghost@x.com', NULL,       NULL,       NULL, NULL);   -- who is this?
```

## Code Examples

### NULL logic, demonstrated

```sql
-- 1. The classic trap: neither query finds everyone
SELECT COUNT(*) FROM contacts WHERE phone = '555-0101';    -- 1
SELECT COUNT(*) FROM contacts WHERE phone <> '555-0101';   -- 2  ← NULL phones excluded!
SELECT COUNT(*) FROM contacts;                             -- 6  (1 + 2 ≠ 6)

-- ✅ "Not this phone, including unknowns":
SELECT COUNT(*) FROM contacts
WHERE phone <> '555-0101' OR phone IS NULL;                -- 4 (excludes 'N/A' issue aside)

-- 2. COUNT(*) vs COUNT(col): instant NULL-rate profiling
SELECT COUNT(*)                          AS total_rows,
       COUNT(email)                      AS has_email,      -- skips NULL, counts ''!
       COUNT(*) - COUNT(phone)           AS missing_phone,
       ROUND(100.0 * COUNT(birth_year) / COUNT(*), 1) AS pct_birth_year_known
FROM contacts;

-- 3. COALESCE for display defaults; NULLIF to demote sentinels first
SELECT id,
       COALESCE(name, '(no name)')                       AS display_name,
       COALESCE(NULLIF(email, ''), '(no email)')         AS display_email,   -- '' → NULL → default
       COALESCE(NULLIF(phone, 'N/A'), '(no phone)')      AS display_phone
FROM contacts;

-- 4. NULL-safe comparison: which contacts share a birth year (both-unknown ≠ match)
SELECT a.id, b.id
FROM contacts a JOIN contacts b ON a.id < b.id
WHERE a.birth_year = b.birth_year;         -- NULL pairs correctly NOT matched

-- 5. Controlling NULL sort position
SELECT name, birth_year FROM contacts
ORDER BY birth_year IS NULL, birth_year;   -- known years first, NULLs last
```

### The data-quality audit

```sql
-- 6. Profile a column's value landscape
SELECT state, COUNT(*) FROM contacts GROUP BY state ORDER BY COUNT(*) DESC;
-- Reveals: 'CO', 'Colorado', 'TX', 'tx', NULL → format drift

-- 7. Find likely duplicates (same email, case-insensitive, ignoring blanks)
SELECT LOWER(NULLIF(email, '')) AS norm_email, COUNT(*) AS copies,
       GROUP_CONCAT(id) AS ids
FROM contacts
WHERE NULLIF(email, '') IS NOT NULL
GROUP BY norm_email
HAVING COUNT(*) > 1;

-- 8. Find orphaned references (referred_by pointing nowhere)
SELECT c.id, c.name, c.referred_by
FROM contacts c
WHERE c.referred_by IS NOT NULL
  AND NOT EXISTS (SELECT 1 FROM contacts r WHERE r.id = c.referred_by);

-- 9. Range/invariant violations
SELECT id, name, birth_year FROM contacts
WHERE birth_year IS NOT NULL
  AND (birth_year < 1900 OR birth_year > STRFTIME('%Y', 'now'));

-- 10. Rows failing basic completeness (define YOUR minimum viable record)
SELECT * FROM contacts
WHERE name IS NULL
   OR NULLIF(email, '') IS NULL AND NULLIF(phone, 'N/A') IS NULL;
```

### The cleanup (transactional, preview-first)

```sql
BEGIN;

-- 11. Normalize sentinels into honest NULLs
UPDATE contacts SET email = NULL WHERE email = '';
UPDATE contacts SET phone = NULL WHERE phone = 'N/A';

-- 12. Standardize formats
UPDATE contacts SET email = LOWER(email) WHERE email <> LOWER(email);
UPDATE contacts SET state = 'CO' WHERE state IN ('Colorado', 'co');
UPDATE contacts SET state = 'TX' WHERE state IN ('Texas', 'tx');

-- 13. Null out impossible values (can't invent the truth — record its absence)
UPDATE contacts SET birth_year = NULL
WHERE birth_year > CAST(STRFTIME('%Y', 'now') AS INTEGER);

-- 14. Break orphan links
UPDATE contacts SET referred_by = NULL
WHERE referred_by IS NOT NULL
  AND NOT EXISTS (SELECT 1 FROM contacts r WHERE r.id = contacts.referred_by);

-- 15. Merge duplicates: keep the lowest id, repoint referrers, delete the rest
UPDATE contacts
SET referred_by = (SELECT MIN(c2.id) FROM contacts c2
                   WHERE LOWER(c2.email) = (SELECT LOWER(c3.email) FROM contacts c3
                                            WHERE c3.id = contacts.referred_by))
WHERE referred_by IN (SELECT id FROM contacts c
                      WHERE id > (SELECT MIN(c4.id) FROM contacts c4
                                  WHERE LOWER(c4.email) = LOWER(c.email)));
DELETE FROM contacts
WHERE email IS NOT NULL
  AND id > (SELECT MIN(c2.id) FROM contacts c2 WHERE LOWER(c2.email) = LOWER(contacts.email));

-- Verify before committing!
SELECT * FROM contacts;
COMMIT;   -- or ROLLBACK if anything looks wrong

-- 16. Prevention: the constraints this table should have had
-- CREATE TABLE contacts_clean (... email TEXT UNIQUE CHECK (email = LOWER(email)),
--   state TEXT CHECK (LENGTH(state) = 2), birth_year INTEGER CHECK (birth_year BETWEEN 1900 AND 2100),
--   referred_by INTEGER REFERENCES contacts_clean(id), ...);
```

## Common Pitfalls

**1. `= NULL` instead of `IS NULL`.** `WHERE phone = NULL` matches nothing, errors nowhere, and wastes an afternoon. Always `IS NULL`.

**2. Forgetting the NULL bucket in "not equal" filters.** Any `col <> value` filter silently discards NULL rows. Ask every time: *should unknowns be in this result?* If yes: `OR col IS NULL`.

**3. Treating `''` and NULL as interchangeable.** They're different values with different behavior (`COUNT` counts `''`; UNIQUE collides on `''`). Pick one representation for "missing" — NULL — and convert sentinels on the way in (`NULLIF`).

**4. "Fixing" unknowns by inventing values.** Setting missing birth years to `1900` or missing states to `'CO'` creates *plausible lies* that poison every later aggregate. Missing data should be NULL, and reports should say how much is missing (example 2's percentage pattern).

**5. Cleaning without a transaction or preview.** A dedup DELETE with a subtly wrong join condition can halve your table. BEGIN first, preview with SELECT, verify counts, then COMMIT — and back up the file before large cleanups (`copy contacts.db contacts.backup.db`).

**6. Auditing once instead of constraining forever.** If your audit found orphans, adding the FK (and NOT NULL, CHECK, UNIQUE as appropriate) is the actual fix; the cleanup query just treats today's symptoms.

## Practice Exercises

Use the dirty `contacts` table (re-insert the original rows if you've cleaned them).

1. Without running them first, predict the output of: `SELECT COUNT(*), COUNT(email), COUNT(DISTINCT email) FROM contacts;` and the row-counts of `WHERE state = 'CO'`, `WHERE state <> 'CO'`, and `WHERE state <> 'CO' OR state IS NULL`. Then run and reconcile every miss with a 3VL rule.
2. Write a single "data quality dashboard" query returning one row with: total contacts, % missing name, % missing usable email (blank counts as missing), duplicate-email groups, orphaned referrals, and impossible birth years. (Conditional aggregation from Chapter 9 is your friend.)
3. Produce a "best contact method" column for every contact: their email if usable, else phone if usable, else `'unreachable'` — with all sentinel values handled. Then count how many fall into each of the three buckets in the same query.
4. Perform the full cleanup as one careful transaction — sentinels, casing, states, impossible values, orphans, dedup — but *your own version*, previewing each step. After COMMIT, re-run your exercise-2 dashboard to show all metrics now clean.
5. Design `contacts_v2` with constraints that make every problem class from this chapter impossible or visible (FK for referrals, UNIQUE lowercase email, CHECKed state codes and birth years, deliberate NULLability choices per column). Migrate your cleaned data into it with INSERT...SELECT and document (comments) which rows, if any, the constraints rejected and why that's correct behavior.
