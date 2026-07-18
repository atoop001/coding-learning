# Chapter 8: Subqueries & CTEs — Queries Inside Queries

## Overview

Some questions can't be answered in one pass: "Which employees earn more than the *average* salary?" requires computing the average first, then comparing against it. **Subqueries** (a query nested inside another) and **CTEs** (Common Table Expressions — named subqueries introduced with `WITH`) let you build multi-step logic in a single statement. CTEs in particular are how professionals keep complex analytics readable, and they unlock recursion for tree-shaped data.

## Sample Schema for This Chapter

Reusing a compact version of the store from Chapter 7, plus a categories tree:

```sql
PRAGMA foreign_keys = ON;

CREATE TABLE customers (
    id   INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    city TEXT NOT NULL
);
CREATE TABLE orders (
    id           INTEGER PRIMARY KEY,
    customer_id  INTEGER NOT NULL REFERENCES customers(id),
    order_date   TEXT NOT NULL,
    amount_cents INTEGER NOT NULL
);
CREATE TABLE categories (
    id        INTEGER PRIMARY KEY,
    name      TEXT NOT NULL,
    parent_id INTEGER REFERENCES categories(id)   -- NULL = top level
);

INSERT INTO customers (name, city) VALUES
    ('Ada','Denver'), ('Ben','Austin'), ('Carla','Denver'), ('Dev','Austin'), ('Elena','Boise');
INSERT INTO orders (customer_id, order_date, amount_cents) VALUES
    (1,'2026-01-05',4999), (1,'2026-02-11',1250), (1,'2026-03-02',8900),
    (2,'2026-01-20',15000), (2,'2026-03-18',2200), (3,'2026-02-01',799),
    (4,'2026-01-09',4500), (4,'2026-02-27',4500), (4,'2026-03-30',12000);
INSERT INTO categories (name, parent_id) VALUES
    ('Electronics', NULL),      -- 1
    ('Computers',   1),         -- 2
    ('Laptops',     2),         -- 3
    ('Accessories', 1),         -- 4
    ('Home',        NULL),      -- 5
    ('Kitchen',     5);         -- 6
```

## Definitions & Explanations

### The three shapes of subquery

**1. Scalar subquery** — returns exactly one value; usable anywhere a value goes:

```sql
SELECT name FROM ... WHERE amount > (SELECT AVG(amount_cents) FROM orders)
```

If it returns more than one row, you get an error (or in some engines, surprises). Design it to return one.

**2. Row-set subquery with IN / EXISTS** — returns a column of values for membership tests:

- `WHERE customer_id IN (SELECT id FROM customers WHERE city = 'Denver')`
- `WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id)` — true if the inner query finds *any* row. `NOT EXISTS` is the standard "has none" tool.

**3. Derived table** — a subquery in the FROM clause, treated as a temporary table (must be aliased):

```sql
SELECT ... FROM (SELECT customer_id, SUM(...) AS total FROM orders GROUP BY customer_id) t
```

### Correlated vs uncorrelated

An **uncorrelated** subquery runs once, independently (`SELECT AVG(...) FROM orders`). A **correlated** subquery references the outer row and conceptually re-runs per outer row:

```sql
SELECT c.name,
       (SELECT MAX(o.amount_cents) FROM orders o
        WHERE o.customer_id = c.id) AS biggest    -- c.id makes it correlated
FROM customers c;
```

Correlated subqueries are expressive but can be slow on big data (row-by-row work). Often a join+GROUP BY or a CTE does the same job set-at-a-time.

### CTEs — WITH ... AS

A **CTE** names a subquery up front, then the main query (or later CTEs) use it like a table:

```sql
WITH customer_totals AS (
    SELECT customer_id, SUM(amount_cents) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT c.name, ct.total
FROM customer_totals ct
JOIN customers c ON c.id = ct.customer_id
WHERE ct.total > 10000;
```

Why CTEs beat nested derived tables:
- **Readability** — steps read top-to-bottom like a recipe.
- **Reuse** — reference the same CTE twice in one statement.
- **Chaining** — `WITH a AS (...), b AS (SELECT ... FROM a ...)` builds pipelines.

CTEs exist only for the duration of one statement — they are *not* saved objects (that's a VIEW, Chapter 13).

### Recursive CTEs — walking trees

`WITH RECURSIVE` lets a CTE reference itself: start from **anchor** rows, repeatedly apply the **recursive step** to find more rows, `UNION ALL` the results, stop when nothing new appears. Perfect for hierarchies: org charts, category trees, threaded comments.

```sql
WITH RECURSIVE subtree AS (
    SELECT id, name, 0 AS depth
    FROM categories WHERE id = 1              -- anchor: Electronics
    UNION ALL
    SELECT c.id, c.name, s.depth + 1          -- step: children of what we have
    FROM categories c
    JOIN subtree s ON c.parent_id = s.id
)
SELECT * FROM subtree;
```

## Code Examples

```sql
-- 1. Scalar subquery: orders above the average order size
SELECT id, customer_id, amount_cents
FROM orders
WHERE amount_cents > (SELECT AVG(amount_cents) FROM orders);

-- 2. Scalar subquery in SELECT: each order vs. the average, as a difference
SELECT id,
       amount_cents,
       amount_cents - (SELECT AVG(amount_cents) FROM orders) AS vs_avg
FROM orders;

-- 3. IN: customers who live where Ada lives (without hardcoding 'Denver')
SELECT name FROM customers
WHERE city IN (SELECT city FROM customers WHERE name = 'Ada')
  AND name <> 'Ada';

-- 4. NOT EXISTS: customers with no orders (compare with Chapter 6's LEFT JOIN trick)
SELECT c.name
FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);

-- 5. Correlated scalar: each customer's most recent order date
SELECT c.name,
       (SELECT MAX(o.order_date) FROM orders o WHERE o.customer_id = c.id) AS last_order
FROM customers c;

-- 6. Derived table: average of the per-customer totals
--    (an "average of sums" needs two aggregation levels — impossible in one pass)
SELECT ROUND(AVG(total) / 100.0, 2) AS avg_customer_spend_dollars
FROM (SELECT customer_id, SUM(amount_cents) AS total
      FROM orders GROUP BY customer_id) AS per_customer;

-- 7. Same thing as a CTE — notice the readability win
WITH per_customer AS (
    SELECT customer_id, SUM(amount_cents) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT ROUND(AVG(total) / 100.0, 2) AS avg_customer_spend_dollars
FROM per_customer;

-- 8. Chained CTEs: monthly revenue, then flag above-average months
WITH monthly AS (
    SELECT STRFTIME('%Y-%m', order_date) AS month,
           SUM(amount_cents) AS revenue
    FROM orders
    GROUP BY month
),
benchmark AS (
    SELECT AVG(revenue) AS avg_rev FROM monthly
)
SELECT m.month,
       m.revenue / 100.0 AS revenue_dollars,
       CASE WHEN m.revenue > b.avg_rev THEN 'above avg' ELSE 'below avg' END AS flag
FROM monthly m
CROSS JOIN benchmark b        -- benchmark is a single row; CROSS JOIN attaches it to all
ORDER BY m.month;

-- 9. CTE used twice: customers whose total spend beats the top city's average
WITH totals AS (
    SELECT c.id, c.name, c.city, SUM(o.amount_cents) AS total
    FROM customers c JOIN orders o ON o.customer_id = c.id
    GROUP BY c.id, c.name, c.city
)
SELECT name, city, total
FROM totals
WHERE total > (SELECT AVG(total) FROM totals);

-- 10. Recursive CTE: full category tree with depth and an indented label
WITH RECURSIVE tree AS (
    SELECT id, name, parent_id, 0 AS depth
    FROM categories
    WHERE parent_id IS NULL                       -- anchors: all roots
    UNION ALL
    SELECT c.id, c.name, c.parent_id, t.depth + 1
    FROM categories c
    JOIN tree t ON c.parent_id = t.id
)
SELECT PRINTF('%.*c%s', depth * 2 + 1, ' ', name) AS outline, depth
FROM tree
ORDER BY id;

-- 11. Recursive CTE generating data: the numbers 1..10 (handy for testing)
WITH RECURSIVE nums(n) AS (
    SELECT 1
    UNION ALL
    SELECT n + 1 FROM nums WHERE n < 10
)
SELECT n FROM nums;
```

## Common Pitfalls

**1. A scalar slot fed a multi-row subquery.**

```sql
-- ❌ The subquery returns many cities; '=' needs exactly one value:
SELECT name FROM customers WHERE city = (SELECT city FROM customers);

-- ✅ Use IN for sets, or constrain the subquery to one row:
SELECT name FROM customers WHERE city IN (SELECT city FROM customers WHERE name = 'Ada');
```

**2. NOT IN with NULLs — the silent empty result.**
If the subquery returns any NULL, `x NOT IN (...)` is never true (NULL logic, Chapter 14) and you get **zero rows** with no error:

```sql
-- ❌ Fragile if referrer_id can be NULL:
WHERE id NOT IN (SELECT referrer_id FROM signups);

-- ✅ NOT EXISTS is NULL-safe:
WHERE NOT EXISTS (SELECT 1 FROM signups s WHERE s.referrer_id = customers.id);
```

Prefer `NOT EXISTS` as your default "absence" tool.

**3. Forgetting the alias on a derived table.**
`SELECT * FROM (SELECT ...)` — most engines require a name: `... ) AS t`. SQLite forgives, PostgreSQL does not.

**4. Correlated subquery when a join was cheaper.**
A correlated lookup per row over a big table can be dramatically slower than one join + GROUP BY. If a query with a correlated subquery crawls, rewrite it set-at-a-time (and see Chapter 11 for measuring).

**5. Runaway recursion.**
A recursive CTE over data with a cycle (A's parent is B, B's parent is A) loops forever. Guard with a depth cap in the recursive step (`WHERE t.depth < 20`) while developing, and design schemas to prevent cycles.

**6. Expecting a CTE to persist.**
`WITH totals AS (...) SELECT ...;` then, in a *new* statement, `SELECT * FROM totals;` → "no such table". A CTE lives inside one statement only. Persistent named queries are views (Chapter 13); persistent data snapshots are `CREATE TABLE ... AS SELECT`.

## Practice Exercises

Use this chapter's schema.

1. Using a scalar subquery, list all orders whose amount is above the average **for that customer's own orders** (correlated). Show order id, customer name, amount, and that customer's average.
2. Rewrite Chapter 6's "customers with no orders" three ways: LEFT JOIN + IS NULL, NOT IN, and NOT EXISTS. Then insert an order row with a NULL customer_id removed — instead, explain (in a comment) which of the three versions would break if the subquery column could contain NULL, and why.
3. With a CTE pipeline: compute each city's total revenue, then return only cities whose revenue exceeds the average city revenue, with a column showing how many dollars above average they are.
4. Find the customer(s) with the single largest order, *without* using LIMIT — use a subquery comparing against `MAX`. Then find the second-largest order amount using a subquery (hint: the max of amounts *below* the max).
5. Add these categories: `Blenders` under `Kitchen`, and `Gaming Laptops` under `Laptops`. Write a recursive CTE that, given any category name (say 'Electronics'), returns all of its descendants with a `depth` column and a `path` column like `Electronics > Computers > Laptops` (hint: build the path string in the recursive step).
