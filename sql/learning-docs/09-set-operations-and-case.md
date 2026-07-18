# Chapter 9: Set Operations & CASE Expressions

## Overview

Two more tools that dramatically expand what you can express:

- **Set operations** (`UNION`, `INTERSECT`, `EXCEPT`) combine the *results of whole queries* — stacking result sets vertically, finding overlaps, finding differences. Joins combine tables side-by-side (more columns); set operations combine results top-to-bottom (more rows).
- **CASE expressions** are SQL's if/else — they compute a value by testing conditions, and they work anywhere an expression works: SELECT, WHERE, ORDER BY, even inside aggregates (a hugely useful trick called conditional aggregation, the basis of pivot-style reports).

## Sample Schema for This Chapter

Two event sign-up sheets and a members table:

```sql
CREATE TABLE workshop_a (
    email TEXT NOT NULL,
    name  TEXT NOT NULL
);
CREATE TABLE workshop_b (
    email TEXT NOT NULL,
    name  TEXT NOT NULL
);
CREATE TABLE members (
    id          INTEGER PRIMARY KEY,
    name        TEXT NOT NULL,
    email       TEXT NOT NULL UNIQUE,
    plan        TEXT NOT NULL,             -- 'free', 'pro', 'team'
    signup_date TEXT NOT NULL,
    age         INTEGER
);

INSERT INTO workshop_a (email, name) VALUES
    ('ada@x.com','Ada'), ('ben@x.com','Ben'), ('carla@x.com','Carla');
INSERT INTO workshop_b (email, name) VALUES
    ('ben@x.com','Ben'), ('dev@x.com','Dev'), ('elena@x.com','Elena');
INSERT INTO members (name, email, plan, signup_date, age) VALUES
    ('Ada','ada@x.com','pro','2025-11-02',34),
    ('Ben','ben@x.com','free','2026-01-15',27),
    ('Carla','carla@x.com','team','2025-08-20',45),
    ('Dev','dev@x.com','free','2026-03-01',19),
    ('Elena','elena@x.com','pro','2026-02-14',52),
    ('Frank','frank@x.com','free','2026-04-04',NULL);
```

## Definitions & Explanations

### Set operations — combining query results

```sql
SELECT ... FROM ...
UNION            -- or UNION ALL / INTERSECT / EXCEPT
SELECT ... FROM ...;
```

Rules that apply to all of them:
- Both queries must return the **same number of columns**, in compatible types, in the same order.
- Column *names* in the final result come from the **first** query.
- A single `ORDER BY` at the very end sorts the combined result (you can't ORDER BY each half separately).

The four operations:

| Operation | Result | Duplicates |
|---|---|---|
| `UNION` | rows appearing in either query | **removed** |
| `UNION ALL` | rows from both queries stacked | **kept** (faster — no dedup pass) |
| `INTERSECT` | rows appearing in *both* queries | removed |
| `EXCEPT` | rows in the first query but *not* the second | removed |

`UNION ALL` vs `UNION` matters: dedup costs time and may hide real duplicates you wanted to see. Default to `UNION ALL` unless you specifically need de-duplication. (`EXCEPT` is called `MINUS` in Oracle; MySQL gained INTERSECT/EXCEPT only in 8.0.31.)

### CASE — SQL's conditional expression

Two forms:

**Searched CASE** (the general one — learn this first):

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE fallback
END
```

Conditions are checked **top to bottom; the first true WHEN wins**. Without `ELSE`, non-matching rows get NULL.

**Simple CASE** (compact equality matching on one expression):

```sql
CASE plan
    WHEN 'free' THEN 0
    WHEN 'pro'  THEN 12
    WHEN 'team' THEN 30
END
```

CASE is an *expression*, not a statement — it produces a value inside a query. It never changes data by itself.

### Conditional aggregation — the killer combo

Putting CASE inside an aggregate counts/sums *only rows matching a condition*, letting one query produce a columns-per-category summary (a mini pivot table):

```sql
SELECT
    COUNT(CASE WHEN plan = 'free' THEN 1 END) AS free_count,   -- non-matches → NULL → not counted
    COUNT(CASE WHEN plan = 'pro'  THEN 1 END) AS pro_count
FROM members;
```

## Code Examples

```sql
-- 1. UNION: one combined attendee list, dedup'd (Ben appears once)
SELECT email, name FROM workshop_a
UNION
SELECT email, name FROM workshop_b
ORDER BY name;

-- 2. UNION ALL: raw stacked list (Ben appears twice) — plus a source tag
SELECT email, name, 'A' AS workshop FROM workshop_a
UNION ALL
SELECT email, name, 'B' AS workshop FROM workshop_b
ORDER BY name;

-- 3. INTERSECT: who attended both workshops?
SELECT email FROM workshop_a
INTERSECT
SELECT email FROM workshop_b;

-- 4. EXCEPT: who attended A but not B?
SELECT email FROM workshop_a
EXCEPT
SELECT email FROM workshop_b;

-- 5. EXCEPT for data auditing: members who attended neither workshop
SELECT email FROM members
EXCEPT
(SELECT email FROM workshop_a UNION SELECT email FROM workshop_b);

-- 6. Searched CASE: label members by age bracket (note NULL handling!)
SELECT name, age,
       CASE
           WHEN age IS NULL THEN 'unknown'
           WHEN age < 25    THEN 'under 25'
           WHEN age < 45    THEN '25-44'
           ELSE                  '45+'
       END AS age_bracket
FROM members;

-- 7. Simple CASE: plan → monthly price
SELECT name, plan,
       CASE plan
           WHEN 'free' THEN 0
           WHEN 'pro'  THEN 12
           WHEN 'team' THEN 30
       END AS monthly_price
FROM members;

-- 8. CASE in ORDER BY: custom sort order (team first, then pro, then free)
SELECT name, plan
FROM members
ORDER BY CASE plan
             WHEN 'team' THEN 1
             WHEN 'pro'  THEN 2
             ELSE 3
         END,
         name;

-- 9. CASE in an UPDATE: one statement, different changes per row
UPDATE members
SET plan = CASE
               WHEN signup_date < '2026-01-01' AND plan = 'free' THEN 'pro'  -- loyalty upgrade
               ELSE plan                                                     -- unchanged
           END;

-- 10. Conditional aggregation: plan counts as COLUMNS, one row total
SELECT
    COUNT(*)                                        AS total_members,
    COUNT(CASE WHEN plan = 'free' THEN 1 END)       AS free_members,
    COUNT(CASE WHEN plan = 'pro'  THEN 1 END)       AS pro_members,
    COUNT(CASE WHEN plan = 'team' THEN 1 END)       AS team_members
FROM members;

-- 11. Pivot-style report: signups per quarter, split by plan
SELECT STRFTIME('%Y', signup_date) AS year,
       SUM(CASE WHEN plan = 'free' THEN 1 ELSE 0 END) AS free_signups,
       SUM(CASE WHEN plan <> 'free' THEN 1 ELSE 0 END) AS paid_signups
FROM members
GROUP BY STRFTIME('%Y', signup_date)
ORDER BY year;

-- 12. Percentage with conditional aggregation
SELECT ROUND(100.0 * COUNT(CASE WHEN plan <> 'free' THEN 1 END) / COUNT(*), 1)
           AS pct_paying
FROM members;
```

## Common Pitfalls

**1. Mismatched columns across a UNION.**

```sql
-- ❌ 2 columns vs 1 column — error:
SELECT email, name FROM workshop_a
UNION
SELECT email FROM workshop_b;

-- ✅ Same column count and order; pad with NULL/constants if needed:
SELECT email, name FROM workshop_a
UNION
SELECT email, NULL FROM workshop_b;
```

Also watch *order*: `SELECT name, email` UNIONed with `SELECT email, name` runs fine and silently scrambles your data — the columns match positionally, not by name.

**2. UNION when you meant UNION ALL (or vice versa).**
Combining January sales with February sales via `UNION` silently drops any row that happens to be identical across months — a real data-loss bug. Stacking data: `UNION ALL`. Building a distinct set: `UNION`.

**3. CASE branch order shadowing later branches.**

```sql
-- ❌ 'under 45' catches EVERYONE under 45, so 'under 25' never fires:
CASE WHEN age < 45 THEN 'under 45'
     WHEN age < 25 THEN 'under 25'   -- unreachable!
     ELSE '45+' END

-- ✅ Most specific first:
CASE WHEN age < 25 THEN 'under 25'
     WHEN age < 45 THEN '25-44'
     ELSE '45+' END
```

**4. Forgetting that missing ELSE yields NULL.**
`CASE WHEN plan = 'pro' THEN 12 END` gives NULL for free/team members, and NULL then poisons any math it touches (`price * 12` → NULL). Add an explicit `ELSE` unless NULL is truly what you want.

**5. NULL falling through CASE comparisons.**
`CASE WHEN age < 25 ...` — when age is NULL the comparison is neither true nor false, so NULL rows fall to ELSE and get mislabeled '45+'. Test `WHEN age IS NULL` **first** (as example 6 does).

**6. `COUNT(CASE ... ELSE 0 END)` counting everything.**
`COUNT` counts *non-NULL* values, and `0` is not NULL:

```sql
-- ❌ Counts every row — 0 is a value:
COUNT(CASE WHEN plan = 'pro' THEN 1 ELSE 0 END)
-- ✅ Let non-matches be NULL (omit ELSE), or use SUM with ELSE 0:
COUNT(CASE WHEN plan = 'pro' THEN 1 END)
SUM(CASE WHEN plan = 'pro' THEN 1 ELSE 0 END)
```

## Practice Exercises

Use this chapter's schema.

1. Build a de-duplicated master contact list (email + name) of everyone who is either a member or attended any workshop, sorted by name. Then produce the count of people who attended a workshop but are **not** members.
2. Using set operations only (no joins), answer: which emails attended workshop B but are on a paid plan (`pro` or `team`)? Think about which operation chains you need.
3. Write one SELECT that labels every member with an `engagement` column: `'both workshops'`, `'one workshop'`, or `'no workshops'` (hint: `IN (SELECT email ...)` conditions inside a CASE).
4. Produce a one-row summary with these columns: total members, members under 30, members 30+, members with unknown age, and the percentage of members on any paid plan (one decimal place). Every figure must come from conditional aggregation — no WHERE clause.
5. The business wants a "priority support queue" ordering: team plan first, then pro, then free; within each plan, oldest signup first — except members with unknown age go last within their plan regardless of signup date. Write the query using CASE expressions in ORDER BY.
