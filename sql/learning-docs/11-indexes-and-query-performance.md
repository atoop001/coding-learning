# Chapter 11: Indexes & Query Performance

## Overview

A correct query that takes 30 seconds is a broken feature. As tables grow from hundreds of rows to millions, the difference between a query that scans everything and one that uses an **index** is the difference between milliseconds and minutes. This chapter explains what indexes are, when the database uses them, how to read a **query plan** (`EXPLAIN QUERY PLAN`), and the cost side of the ledger — because indexes are not free.

Everything here is hands-on measurable in SQLite, and the mental model transfers directly to PostgreSQL/MySQL.

## Definitions & Explanations

### What an index is

An **index** is a separate, sorted data structure (a **B-tree** in virtually every SQL database) that maps values of one or more columns to the rows containing them — exactly like a book's index maps terms to page numbers. Looking up `email = 'ada@x.com'`:

- **Without an index**: the database reads *every row* and checks — a **full table scan**, O(n).
- **With an index on email**: it descends the sorted tree — a handful of steps even for millions of rows, O(log n).

Because a B-tree is *sorted*, an index accelerates three things: equality lookups (`=`, `IN`), **range scans** (`<`, `BETWEEN`, `LIKE 'abc%'` — prefix only), and **ordering** (`ORDER BY` on the indexed column can skip sorting entirely).

### What you get automatically

- The **primary key** is always indexed (in SQLite, `INTEGER PRIMARY KEY` *is* the table's internal order).
- Every **UNIQUE** constraint creates an index (that's how uniqueness gets checked fast).
- **Foreign key columns are NOT auto-indexed** in SQLite or PostgreSQL — and FK columns are exactly what your joins use. *Indexing your FK columns is the single highest-value habit in this chapter.* (MySQL/InnoDB does auto-index FKs.)

### Creating and dropping

```sql
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE UNIQUE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_cust_date ON orders(customer_id, order_date);  -- composite
DROP INDEX idx_orders_customer;
```

Naming convention: `idx_<table>_<columns>` keeps schemas readable.

### Composite indexes and the leftmost-prefix rule

An index on `(customer_id, order_date)` is sorted by customer first, date second — like a phone book sorted by last name then first name. It serves:
- `WHERE customer_id = ?` ✅ (leftmost column alone)
- `WHERE customer_id = ? AND order_date > ?` ✅ (both)
- `WHERE order_date > ?` ❌ alone — the dates aren't contiguous in the index (you can't find "everyone named John" in a phone book sorted by last name).

Order the columns: **equality-tested columns first, range-tested columns last**. A composite index makes a separate index on its first column redundant.

### The price of indexes

Every INSERT/UPDATE/DELETE must also update **every index** on the table. Indexes also occupy disk space. Ten indexes on a hot table can noticeably slow writes. The discipline: index what your actual queries filter/join/sort on — not every column "just in case." Reading query plans tells you what's actually needed.

### EXPLAIN QUERY PLAN — seeing what the database will do

Prefix any query with `EXPLAIN QUERY PLAN` (SQLite; PostgreSQL uses `EXPLAIN` / `EXPLAIN ANALYZE`):

```
SCAN orders                              ← full table scan (fine for tiny tables, alarm for big)
SEARCH orders USING INDEX idx_... (customer_id=?)   ← index used 🎉
USING COVERING INDEX ...                 ← even better: query answered from index alone
```

Make a habit: for any query you care about, look at the plan.

### When the index does NOT get used

- **Functions/arithmetic on the indexed column**: `WHERE LOWER(email) = 'x'` or `WHERE salary / 12 > 5000` — the index stores `email`/`salary`, not the transformed value. Rewrite to compare the bare column (`WHERE salary > 60000`), or create an **expression index**: `CREATE INDEX idx ON users(LOWER(email));`.
- **Leading-wildcard LIKE**: `LIKE '%son'` can't use a B-tree (no known prefix). `LIKE 'son%'` can.
- **Tiny tables / low selectivity**: if a filter matches half the table, scanning is genuinely cheaper and the optimizer rightly skips the index. Indexes shine on *selective* conditions.
- **Type mismatches**: comparing an indexed TEXT column to a number can defeat the index.

### Other performance levers (quick tour)

- `SELECT *` drags every column through memory; naming needed columns can enable **covering indexes** (query satisfied entirely from the index).
- `LIMIT` + index-aligned `ORDER BY` stops early — pagination heaven.
- Batch inserts inside one transaction (Chapter 12) — thousands of separate auto-committed INSERTs are dominated by commit overhead.
- Run `ANALYZE;` occasionally so SQLite's optimizer has statistics to plan with.

## Code Examples

Build a big enough table to *feel* the difference (the recursive CTE trick from Chapter 8):

```sql
CREATE TABLE big_orders (
    id           INTEGER PRIMARY KEY,
    customer_id  INTEGER NOT NULL,
    order_date   TEXT NOT NULL,
    amount_cents INTEGER NOT NULL
);

-- 200,000 synthetic rows: ~1000 customers, dates through 2026, random amounts
WITH RECURSIVE n(i) AS (
    SELECT 1 UNION ALL SELECT i + 1 FROM n WHERE i < 200000
)
INSERT INTO big_orders (customer_id, order_date, amount_cents)
SELECT ABS(RANDOM()) % 1000 + 1,
       DATE('2024-01-01', '+' || (ABS(RANDOM()) % 900) || ' days'),
       ABS(RANDOM()) % 50000 + 100
FROM n;

-- Turn on the shell's timer to measure (sqlite3 shell only):
.timer on
```

Now experiment:

```sql
-- 1. Baseline: filter on an unindexed column → SCAN
EXPLAIN QUERY PLAN
SELECT COUNT(*) FROM big_orders WHERE customer_id = 42;
-- Output: SCAN big_orders

SELECT COUNT(*) FROM big_orders WHERE customer_id = 42;   -- note the time

-- 2. Add the index, look again
CREATE INDEX idx_big_orders_customer ON big_orders(customer_id);

EXPLAIN QUERY PLAN
SELECT COUNT(*) FROM big_orders WHERE customer_id = 42;
-- Output: SEARCH big_orders USING COVERING INDEX idx_big_orders_customer (customer_id=?)

SELECT COUNT(*) FROM big_orders WHERE customer_id = 42;   -- dramatically faster

-- 3. Composite index serving filter + sort together
CREATE INDEX idx_big_orders_cust_date ON big_orders(customer_id, order_date);

EXPLAIN QUERY PLAN
SELECT order_date, amount_cents
FROM big_orders
WHERE customer_id = 42
ORDER BY order_date DESC
LIMIT 10;
-- SEARCH ... USING INDEX idx_big_orders_cust_date — and no separate sort step

-- 4. Leftmost-prefix rule demonstrated: date alone ignores the composite index
EXPLAIN QUERY PLAN
SELECT COUNT(*) FROM big_orders WHERE order_date = '2025-06-15';
-- SCAN (the composite starts with customer_id) → needs its own index if hot:
CREATE INDEX idx_big_orders_date ON big_orders(order_date);

-- 5. Function on the column defeats the index...
EXPLAIN QUERY PLAN
SELECT COUNT(*) FROM big_orders WHERE STRFTIME('%Y', order_date) = '2025';
-- SCAN

-- ...rewritten as a range on the bare column, index-friendly:
EXPLAIN QUERY PLAN
SELECT COUNT(*) FROM big_orders
WHERE order_date >= '2025-01-01' AND order_date < '2026-01-01';
-- SEARCH USING INDEX idx_big_orders_date

-- 6. Join speed: FK indexes are what make joins fast
CREATE TABLE big_customers (id INTEGER PRIMARY KEY, name TEXT NOT NULL);
WITH RECURSIVE n(i) AS (SELECT 1 UNION ALL SELECT i+1 FROM n WHERE i < 1000)
INSERT INTO big_customers SELECT i, 'Customer ' || i FROM n;

EXPLAIN QUERY PLAN
SELECT c.name, SUM(o.amount_cents)
FROM big_customers c
JOIN big_orders o ON o.customer_id = c.id
GROUP BY c.id, c.name;
-- With idx on customer_id: SEARCH on the order side per customer. Try DROPping
-- the customer indexes and re-running the EXPLAIN + timing to feel the difference.

-- 7. The write-side cost, measured: time a bulk insert with all these indexes,
--    then drop them and repeat (wrap in a transaction either way):
BEGIN;
WITH RECURSIVE n(i) AS (SELECT 1 UNION ALL SELECT i+1 FROM n WHERE i < 50000)
INSERT INTO big_orders (customer_id, order_date, amount_cents)
SELECT ABS(RANDOM()) % 1000 + 1, '2026-01-01', 500 FROM n;
COMMIT;

-- 8. Statistics for the optimizer
ANALYZE;
```

## Common Pitfalls

**1. Unindexed foreign keys.** The most common real-world performance bug: joins and `ON DELETE CASCADE` checks crawl. After creating a schema, walk every `REFERENCES` and add an index on the referencing column.

```sql
-- ✅ Routine after schema creation:
CREATE INDEX idx_sessions_student ON sessions(student_id);
CREATE INDEX idx_sessions_tutor   ON sessions(tutor_id);
```

**2. Wrapping the indexed column in a function.**

```sql
-- ❌ SCAN: WHERE UPPER(name) = 'ADA'
-- ✅ Index the expression, or store normalized data:
CREATE INDEX idx_customers_name_upper ON customers(UPPER(name));
```

**3. Indexing everything.** A table with an index per column pays on every write and most of those indexes never serve a query. Add indexes *because a query plan showed a scan that matters*, not preemptively.

**4. Composite index in the wrong column order.** `(order_date, customer_id)` does not efficiently serve `WHERE customer_id = ? AND order_date > ?` the way `(customer_id, order_date)` does. Equality columns first.

**5. Trusting timing on toy data.** Everything is instant at 100 rows; plans, not stopwatch feelings, tell the truth — and generate realistic data volumes (as above) before concluding anything.

**6. Forgetting that plans can change.** Adding data, running ANALYZE, or upgrading the engine can change plans. Re-check EXPLAIN on queries that regress.

## Practice Exercises

1. Using the `big_orders` setup above, find the slowest of these three queries by timing and by plan, then fix it with exactly one index: (a) count orders for customer 500; (b) total revenue on '2025-03-14'; (c) the ten largest orders overall (`ORDER BY amount_cents DESC LIMIT 10`).
2. Predict — before running — which of these can use `idx_big_orders_cust_date (customer_id, order_date)`: (a) `WHERE customer_id = 7`; (b) `WHERE order_date > '2025-01-01'`; (c) `WHERE customer_id = 7 AND order_date > '2025-01-01'`; (d) `WHERE customer_id > 7 AND order_date = '2025-01-01'`. Verify each with EXPLAIN QUERY PLAN and explain any surprise.
3. Write a query that currently defeats an index by using a function on the column (e.g., finding customers by case-insensitive name in `big_customers`), demonstrate the SCAN, then fix it two ways: an expression index, and a query rewrite. Show both plans.
4. Measure the write cost of indexes: time inserting 50,000 rows into `big_orders` with all your indexes present, then `DROP` them, repeat the insert, and compare. Report the ratio and one sentence on what this implies for a logging-heavy table.
5. Take your Chapter 10 tutoring schema (or Project 2 if done): list every query the application would realistically run (at least five), decide the minimal set of indexes to serve them, create the indexes, and verify each query's plan shows SEARCH rather than SCAN where it matters. Justify any deliberate non-index.
