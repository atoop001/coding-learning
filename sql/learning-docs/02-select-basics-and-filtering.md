# Chapter 2: SELECT Basics & Filtering — WHERE, ORDER BY, LIMIT

## Overview

`SELECT` is the workhorse of SQL. In real jobs, developers and analysts spend far more time *reading* data than writing it, and nearly every read starts with `SELECT`. This chapter covers the core query shape you will use thousands of times: choosing columns, filtering rows with `WHERE`, sorting with `ORDER BY`, and limiting output with `LIMIT`.

Everything here runs in SQLite exactly as shown, and works nearly identically in every other SQL database.

## Sample Schema for This Chapter

Run this once in a fresh database (e.g., `sqlite3 ch02.db`) so every example works:

```sql
CREATE TABLE employees (
    id         INTEGER PRIMARY KEY,
    name       TEXT NOT NULL,
    department TEXT NOT NULL,
    salary     INTEGER NOT NULL,   -- yearly, in dollars
    hire_date  TEXT NOT NULL,      -- ISO format: 'YYYY-MM-DD'
    remote     INTEGER NOT NULL    -- 0 = office, 1 = remote (SQLite has no BOOLEAN)
);

INSERT INTO employees (name, department, salary, hire_date, remote) VALUES
    ('Ada Chen',      'Engineering', 95000, '2019-03-14', 1),
    ('Ben Ortiz',     'Engineering', 87000, '2021-07-01', 0),
    ('Carla Diaz',    'Marketing',   64000, '2020-01-20', 1),
    ('Dev Patel',     'Engineering', 102000,'2018-11-05', 1),
    ('Elena Rossi',   'Sales',       58000, '2022-05-30', 0),
    ('Frank Mwangi',  'Sales',       61000, '2021-02-15', 0),
    ('Grace Kim',     'Marketing',   71000, '2019-09-09', 1),
    ('Hugo Silva',    'Support',     48000, '2023-01-10', 0);
```

## Definitions & Explanations

### The basic query shape

```sql
SELECT column1, column2   -- which columns you want
FROM table_name           -- which table to read
WHERE condition           -- (optional) which rows to keep
ORDER BY column1          -- (optional) how to sort the result
LIMIT n;                  -- (optional) at most n rows
```

The clauses must appear **in this order**. You can omit any optional clause, but you can't rearrange them.

### SELECT — choosing columns
- `SELECT *` returns every column. Great for exploring; avoid it in application code (you get fragile, wasteful queries).
- `SELECT name, salary` returns only those columns, in the order you list them.
- You can compute things: `SELECT name, salary / 12` returns a calculated monthly figure.
- **Aliases** rename output columns: `SELECT salary / 12 AS monthly_salary`. `AS` is optional but keep it for readability.
- `SELECT DISTINCT department` removes duplicate rows from the result — useful for "what values exist in this column?"

### FROM — which table
Names the table to read. Later chapters extend this with joins; for now it's one table.

### WHERE — filtering rows
`WHERE` keeps only rows for which the condition is **true**. Comparison operators:

| Operator | Meaning | Example |
|---|---|---|
| `=` | equal | `department = 'Sales'` |
| `<>` or `!=` | not equal | `department <> 'Sales'` |
| `<`, `<=`, `>`, `>=` | comparisons | `salary >= 60000` |
| `BETWEEN a AND b` | inclusive range | `salary BETWEEN 50000 AND 70000` |
| `IN (a, b, c)` | matches any listed value | `department IN ('Sales', 'Support')` |
| `LIKE` | pattern match | `name LIKE 'A%'` |
| `IS NULL` / `IS NOT NULL` | missing / present | `hire_date IS NOT NULL` |

Conditions combine with `AND`, `OR`, and `NOT`. `AND` binds tighter than `OR`, so use parentheses whenever you mix them (see Pitfalls).

**LIKE patterns:** `%` matches any run of characters (including none); `_` matches exactly one character. `'A%'` = starts with A. `'%son'` = ends with "son". `'%an%'` = contains "an". In SQLite, `LIKE` is case-insensitive for ASCII letters by default (this differs from PostgreSQL — noted again in Chapter 16).

### ORDER BY — sorting
- `ORDER BY salary` sorts ascending (smallest first) — `ASC` is the default.
- `ORDER BY salary DESC` sorts descending.
- Multiple keys: `ORDER BY department ASC, salary DESC` sorts by department, and within each department by salary high-to-low.
- **Important:** without `ORDER BY`, row order is *not guaranteed*. It may look stable, but the database is free to return rows in any order. If order matters, say so.

### LIMIT and OFFSET
- `LIMIT 5` returns at most 5 rows.
- `LIMIT 5 OFFSET 10` skips 10 rows then returns up to 5 — the classic recipe for pagination (page 3 of size 5 = `LIMIT 5 OFFSET 10`).
- `LIMIT` without `ORDER BY` gives you *an arbitrary* 5 rows — almost always pair them.

## Code Examples

```sql
-- 1. Everything (exploration only)
SELECT * FROM employees;

-- 2. Just names and salaries
SELECT name, salary FROM employees;

-- 3. Computed column with an alias: monthly pay, rounded
SELECT name,
       ROUND(salary / 12.0, 2) AS monthly_pay   -- 12.0 forces decimal division
FROM employees;

-- 4. Which departments exist? (deduplicated)
SELECT DISTINCT department FROM employees;

-- 5. Engineers only
SELECT name, salary
FROM employees
WHERE department = 'Engineering';

-- 6. Well-paid remote workers: combine conditions with AND
SELECT name, department, salary
FROM employees
WHERE remote = 1
  AND salary > 70000;

-- 7. Sales OR Support, using IN (cleaner than chained ORs)
SELECT name, department
FROM employees
WHERE department IN ('Sales', 'Support');

-- 8. Mid-range salaries, inclusive on both ends
SELECT name, salary
FROM employees
WHERE salary BETWEEN 58000 AND 71000;

-- 9. Names containing an 'a' (case-insensitive in SQLite)
SELECT name FROM employees WHERE name LIKE '%a%';

-- 10. Hired in 2021 (ISO date strings compare correctly as text!)
SELECT name, hire_date
FROM employees
WHERE hire_date >= '2021-01-01' AND hire_date < '2022-01-01';

-- 11. Highest earners first
SELECT name, salary
FROM employees
ORDER BY salary DESC;

-- 12. Multi-key sort: by department A→Z, then salary high→low within each
SELECT department, name, salary
FROM employees
ORDER BY department ASC, salary DESC;

-- 13. Top 3 earners — filter, sort, then limit
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;

-- 14. "Page 2" of employees alphabetically, 3 per page
SELECT name
FROM employees
ORDER BY name
LIMIT 3 OFFSET 3;

-- 15. Realistic combined query: remote engineers hired before 2020,
--     best paid first, top 2 only
SELECT name, salary, hire_date
FROM employees
WHERE department = 'Engineering'
  AND remote = 1
  AND hire_date < '2020-01-01'
ORDER BY salary DESC
LIMIT 2;
```

A note on example 10: storing dates as ISO-8601 text (`'2021-07-01'`) means alphabetical order **is** chronological order, so `>=` and `<` work naturally. This is the standard SQLite convention (Chapter 5 covers types in depth).

## Common Pitfalls

**1. Mixing AND/OR without parentheses.**

```sql
-- ❌ Intended "remote workers in Sales or Marketing", actually returns
--    ALL Sales people (remote or not) plus remote Marketing people,
--    because AND binds tighter than OR:
SELECT name FROM employees
WHERE department = 'Sales' OR department = 'Marketing' AND remote = 1;

-- ✅ Correct:
SELECT name FROM employees
WHERE (department = 'Sales' OR department = 'Marketing')
  AND remote = 1;
```

**2. Using `=` with NULL.**
`WHERE manager = NULL` never matches anything, because NULL isn't equal to anything — not even NULL. Use `IS NULL` / `IS NOT NULL`. (Chapter 14 is entirely about NULL.)

**3. Assuming rows come back in insertion order.**

```sql
-- ❌ Fragile: "works on my machine" ordering
SELECT name FROM employees LIMIT 3;

-- ✅ Deterministic:
SELECT name FROM employees ORDER BY name LIMIT 3;
```

**4. Quoting numbers or not quoting text.**

```sql
-- ❌ Comparing a number column to text (may "work" in SQLite, breaks elsewhere):
WHERE salary = '95000'
-- ✅
WHERE salary = 95000

-- ❌ Unquoted text is treated as a column name → "no such column: Sales" error:
WHERE department = Sales
-- ✅
WHERE department = 'Sales'
```

**5. Referring to a column alias in WHERE.**

```sql
-- ❌ Fails in most databases: WHERE runs before SELECT aliases exist
SELECT salary / 12.0 AS monthly FROM employees WHERE monthly > 6000;

-- ✅ Repeat the expression (or use a subquery, Chapter 8):
SELECT salary / 12.0 AS monthly FROM employees WHERE salary / 12.0 > 6000;
```

(You *can* use aliases in `ORDER BY` — sorting happens after selection.)

**6. Integer division surprises.**
`salary / 12` with an integer column does integer division in many databases (SQLite included: `95000 / 12` → `7916`). Write `salary / 12.0` to get decimals.

## Practice Exercises

Use the `employees` table above.

1. List the name and hire date of every employee, sorted from most recently hired to least recently hired.
2. Show the name, department, and salary of everyone who is **not** in Engineering and earns at least $60,000, ordered by salary descending.
3. Find all employees whose name starts with a letter from A through D (hint: think about what `LIKE` patterns or comparison operators can do with text), and display only their names in alphabetical order.
4. Write a query that returns the *second and third* highest-paid employees — not the first. Use `ORDER BY`, `LIMIT`, and `OFFSET`.
5. Management wants a list of remote employees hired in 2019 or 2020, in either Marketing or Engineering, showing name, department, and hire date, sorted by department then by hire date. Write it with correctly parenthesized conditions, then deliberately remove the parentheses and observe how the results change.
