# Chapter 7: Aggregation — GROUP BY, HAVING & Aggregate Functions

## Overview

Every dashboard number you've ever seen — total revenue, average rating, orders per customer — is an **aggregation**: collapsing many rows into summary values. This chapter covers the aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`), grouping rows with `GROUP BY`, and filtering *groups* with `HAVING`. Combined with joins, this is the core of analytics and reporting — and Project 3 is built entirely on it.

## Sample Schema for This Chapter

A small online store:

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
    amount_cents INTEGER NOT NULL     -- money as integer cents (Chapter 5)
);

INSERT INTO customers (name, city) VALUES
    ('Ada',   'Denver'),
    ('Ben',   'Austin'),
    ('Carla', 'Denver'),
    ('Dev',   'Austin'),
    ('Elena', 'Boise');              -- Elena never orders

INSERT INTO orders (customer_id, order_date, amount_cents) VALUES
    (1, '2026-01-05', 4999),
    (1, '2026-02-11', 1250),
    (1, '2026-03-02', 8900),
    (2, '2026-01-20', 15000),
    (2, '2026-03-18', 2200),
    (3, '2026-02-01',  799),
    (4, '2026-01-09', 4500),
    (4, '2026-02-27', 4500),
    (4, '2026-03-30', 12000);
```

## Definitions & Explanations

### Aggregate functions

An **aggregate function** takes a *set of rows* and returns one value:

| Function | Returns | Notes |
|---|---|---|
| `COUNT(*)` | number of rows | counts everything, NULLs included |
| `COUNT(col)` | number of rows where `col` is not NULL | the NULL-skipping matters! |
| `COUNT(DISTINCT col)` | number of different non-NULL values | |
| `SUM(col)` | total | NULLs skipped; returns NULL (not 0) on zero rows |
| `AVG(col)` | mean | NULLs excluded from both top and bottom of the fraction |
| `MIN(col)` / `MAX(col)` | smallest / largest | works on text and dates too |

Without GROUP BY, an aggregate collapses the *whole result* to one row: `SELECT COUNT(*) FROM orders;` → `9`.

### GROUP BY — one summary row per group

`GROUP BY expr` partitions rows into groups sharing the same value(s) of `expr`, then aggregates run **within each group**:

```sql
SELECT customer_id, COUNT(*), SUM(amount_cents)
FROM orders
GROUP BY customer_id;
```

One output row per customer_id.

**The golden rule:** every column in SELECT must be either (a) inside an aggregate function, or (b) listed in GROUP BY. A bare column that varies within a group has no single defensible value. SQLite is permissive and will pick an arbitrary row's value instead of erroring — PostgreSQL rejects the query. Follow the rule always.

You can group by multiple expressions (`GROUP BY city, STRFTIME('%m', order_date)`) — one row per unique combination — and by computed expressions, not just raw columns.

### HAVING — filtering groups

`WHERE` filters **rows before grouping**; `HAVING` filters **groups after aggregation**. You cannot put an aggregate in WHERE — the aggregates don't exist yet at that stage.

```sql
SELECT customer_id, SUM(amount_cents) AS total
FROM orders
GROUP BY customer_id
HAVING SUM(amount_cents) > 10000;   -- keep only big spenders
```

### Logical order of evaluation

Understanding the pipeline explains almost every aggregation error:

```
FROM (+ JOINs) → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

- WHERE runs before grouping → can't see aggregates.
- HAVING runs after → can use aggregates, can't cheaply replace WHERE (filter early when you can — it's clearer and faster).
- SELECT aliases exist by ORDER BY time → `ORDER BY total DESC` works; SQLite also allows aliases in HAVING, but that's nonstandard — repeat the aggregate for portability.

### Aggregating across joins

Join first, then group — typically grouping by the "one" side's primary key and counting/summing the "many" side. Use LEFT JOIN when zero-count groups must appear (Elena!), and `COUNT(child.id)` — not `COUNT(*)` — so those groups show 0 (see Pitfall 4).

## Code Examples

```sql
-- 1. Whole-table aggregates: the store at a glance
SELECT COUNT(*)                          AS num_orders,
       SUM(amount_cents) / 100.0         AS revenue_dollars,
       ROUND(AVG(amount_cents) / 100.0, 2) AS avg_order_dollars,
       MIN(order_date)                   AS first_order,
       MAX(order_date)                   AS latest_order
FROM orders;

-- 2. How many distinct customers have ordered?
SELECT COUNT(DISTINCT customer_id) AS customers_with_orders FROM orders;

-- 3. Orders and spend per customer id
SELECT customer_id,
       COUNT(*)               AS num_orders,
       SUM(amount_cents)      AS total_cents
FROM orders
GROUP BY customer_id
ORDER BY total_cents DESC;

-- 4. Same, but with names (join then group by the customer)
SELECT c.name,
       COUNT(*)                        AS num_orders,
       SUM(o.amount_cents) / 100.0     AS total_dollars
FROM orders o
JOIN customers c ON c.id = o.customer_id
GROUP BY c.id, c.name           -- group by PK; include name to satisfy the rule
ORDER BY total_dollars DESC;

-- 5. INCLUDE customers with zero orders: LEFT JOIN + COUNT(child column)
SELECT c.name,
       COUNT(o.id)                             AS num_orders,   -- 0 for Elena
       COALESCE(SUM(o.amount_cents), 0) / 100.0 AS total_dollars
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
GROUP BY c.id, c.name
ORDER BY num_orders DESC, c.name;

-- 6. Revenue per city (grouping the joined result by a customer attribute)
SELECT c.city,
       SUM(o.amount_cents) / 100.0 AS city_revenue
FROM orders o
JOIN customers c ON c.id = o.customer_id
GROUP BY c.city;

-- 7. Monthly revenue: group by a computed expression
SELECT STRFTIME('%Y-%m', order_date) AS month,
       COUNT(*)                      AS orders,
       SUM(amount_cents) / 100.0     AS revenue
FROM orders
GROUP BY STRFTIME('%Y-%m', order_date)
ORDER BY month;

-- 8. WHERE + GROUP BY + HAVING together:
--    among 2026 Q1 orders, which customers placed 2 or more?
SELECT customer_id, COUNT(*) AS q1_orders
FROM orders
WHERE order_date BETWEEN '2026-01-01' AND '2026-03-31'   -- row filter first
GROUP BY customer_id
HAVING COUNT(*) >= 2;                                    -- then group filter

-- 9. Cities with more than one customer (aggregating a lookup table itself)
SELECT city, COUNT(*) AS residents
FROM customers
GROUP BY city
HAVING COUNT(*) > 1;

-- 10. Biggest single order per customer, and who beats $100 at least once
SELECT c.name, MAX(o.amount_cents) / 100.0 AS biggest_order
FROM orders o
JOIN customers c ON c.id = o.customer_id
GROUP BY c.id, c.name
HAVING MAX(o.amount_cents) > 10000;
```

## Common Pitfalls

**1. Bare columns not in GROUP BY.**

```sql
-- ❌ Which order_date? Each group has several. SQLite guesses; PostgreSQL errors:
SELECT customer_id, order_date, SUM(amount_cents)
FROM orders GROUP BY customer_id;

-- ✅ Either aggregate it, group by it, or drop it:
SELECT customer_id, MAX(order_date) AS latest, SUM(amount_cents)
FROM orders GROUP BY customer_id;
```

**2. Aggregate in WHERE.**

```sql
-- ❌ Error: misuse of aggregate — WHERE runs before grouping:
SELECT customer_id FROM orders WHERE SUM(amount_cents) > 10000 GROUP BY customer_id;

-- ✅ Group filters belong in HAVING:
SELECT customer_id FROM orders GROUP BY customer_id HAVING SUM(amount_cents) > 10000;
```

**3. HAVING doing WHERE's job.**

```sql
-- ❌ Works but wasteful/unclear — row-level condition applied after grouping:
... GROUP BY customer_id HAVING MIN(order_date) >= '2026-01-01' -- when you really
                                                                 -- meant to filter rows
-- ✅ Filter rows early:
... WHERE order_date >= '2026-01-01' GROUP BY customer_id
```

(They're only equivalent when the condition truly is per-row; think about which you mean.)

**4. COUNT(*) with LEFT JOIN counting the NULL row as 1.**

```sql
-- ❌ Elena shows num_orders = 1 (her single all-NULL joined row is still a row):
SELECT c.name, COUNT(*) FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id GROUP BY c.id, c.name;

-- ✅ COUNT a column from the right table — NULLs aren't counted:
SELECT c.name, COUNT(o.id) FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id GROUP BY c.id, c.name;
```

**5. Join fan-out inflating SUMs.**
If a customer joins to 3 orders *and* 2 addresses in the same query, each order row appears twice and `SUM(amount)` doubles. When sums look too big, check whether an extra one-to-many join is multiplying rows; aggregate in a subquery/CTE first (Chapter 8).

**6. Expecting SUM of no rows to be 0.**
`SELECT SUM(amount_cents) FROM orders WHERE customer_id = 5;` → NULL, not 0. Wrap it: `COALESCE(SUM(amount_cents), 0)`.

## Practice Exercises

Use the store schema above.

1. For each customer (including Elena), show name, number of orders, total spend in dollars, and their average order value rounded to 2 decimals — zero-order customers must show 0, not NULL.
2. Produce a per-city report: number of customers living there, number of orders placed by them, and total revenue — sorted by revenue descending. One city should appear with customers but zero revenue.
3. Which months had revenue above $100? Show month and revenue, using WHERE only if genuinely needed and HAVING for the revenue threshold.
4. Find customers whose *average* order (not total) exceeds $40, showing name and the average. Then modify it to consider only orders from February onward.
5. Write a query answering: "What is each customer's share of total store revenue, as a percentage?" (Hint: you'll want the overall total inside the query — a subquery like `(SELECT SUM(amount_cents) FROM orders)` works, and Chapter 8 explains why.) Verify your percentages sum to 100.
