# Chapter 13: Views & Stored Logic — Named Queries, Triggers, and Where Logic Lives

## Overview

By now your useful queries are getting long — four joins, a CTE, careful NULL handling. Retyping them is error-prone, and every copy is a maintenance burden. **Views** solve this: they give a query a permanent name, so it can be used like a table. This chapter covers views in depth, plus a working introduction to **triggers** (SQL that runs automatically on data changes) and an honest overview of **stored procedures/functions** — which SQLite doesn't have, but PostgreSQL/MySQL do, and job postings mention.

The bigger theme: *where should logic live* — in the database or in application code? You'll leave with a practical rule of thumb.

## Definitions & Explanations

### Views — saved, named queries

```sql
CREATE VIEW view_name AS
SELECT ... ;         -- any SELECT, joins and all
```

After that, `SELECT * FROM view_name` runs the underlying query. Key facts:

- A view stores the **query, not the data**. Every read re-runs it, so views are always up to date — and cost whatever the query costs each time.
- Views can be filtered/joined/aggregated like tables: `SELECT ... FROM my_view WHERE ... JOIN ...`.
- `DROP VIEW view_name;` removes it; `CREATE VIEW IF NOT EXISTS` avoids duplicate errors. To change a view: drop and recreate (SQLite has no `CREATE OR REPLACE VIEW`; PostgreSQL does).
- Views can be built **on other views** — layering simple views into richer ones.

What views are for:
1. **Abstraction** — hide join complexity; the app selects from `active_customer_summary` instead of a 20-line query.
2. **Consistency** — "revenue" is defined *once*; every report using the view agrees.
3. **Security/scoping** (client-server databases) — grant users access to a view exposing only some columns/rows, not the raw table.
4. **Refactoring insurance** — restructure tables, rebuild the view to match, and code selecting from the view keeps working.

**Updatability:** simple single-table views are updatable in some engines, but as a rule treat views as **read-only** — write to base tables. (SQLite views are always read-only; `INSTEAD OF` triggers can simulate writes.)

### Materialized views (concept)

A **materialized view** stores the *results* physically and must be refreshed — trading freshness for speed on expensive aggregations. PostgreSQL: `CREATE MATERIALIZED VIEW ... / REFRESH MATERIALIZED VIEW ...`. SQLite doesn't have them; the manual equivalent is `CREATE TABLE summary AS SELECT ...` re-run on a schedule.

### Triggers — code that fires on data changes

```sql
CREATE TRIGGER trigger_name
AFTER INSERT ON table_name        -- or BEFORE/AFTER, INSERT/UPDATE/DELETE
FOR EACH ROW
[WHEN condition]
BEGIN
    -- one or more SQL statements; NEW.col / OLD.col reference the row
END;
```

- `NEW` is the row being inserted/updated; `OLD` is the row being updated/deleted.
- Classic uses: **audit logs** (record who changed what, automatically), **maintaining derived data** (keep a cached total in sync — the disciplined way to denormalize, Chapter 10), **enforcing rules** CHECK can't express (`RAISE(ABORT, 'message')` rejects the change).
- Triggers run inside the same transaction as the triggering statement — if the transaction rolls back, so do the trigger's effects. That's what keeps trigger-maintained data consistent.

### Stored procedures & functions (ecosystem overview)

Client-server databases let you store runnable code *in the database*: PostgreSQL functions (`CREATE FUNCTION ... LANGUAGE plpgsql`), MySQL stored procedures. Benefits: logic near the data, one round trip for multi-step operations, callable from any client language. Costs: harder to version-control/test/debug than app code, and dialect lock-in. **SQLite has no stored procedures** — its answer is that your application *is* next to the data, so app code (Chapter 15) plays that role.

### Where should logic live? A practical rule

- **In the database**: integrity rules (constraints first, triggers when constraints can't), definitions reused across many consumers (views), audit trails (triggers).
- **In application code**: business workflows, anything needing external services, logic that changes often, anything you want unit-tested easily.
- When in doubt: constraints > views > app code > triggers > stored procedures, in order of how eagerly you should reach for them. Hidden logic in triggers surprises maintainers — keep triggers few, small, and documented.

## Code Examples

Building on the store schema (customers/orders from Chapter 7 — recreate it, or adapt):

```sql
PRAGMA foreign_keys = ON;
CREATE TABLE customers (
    id INTEGER PRIMARY KEY, name TEXT NOT NULL, city TEXT NOT NULL,
    is_active INTEGER NOT NULL DEFAULT 1
);
CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(id),
    order_date TEXT NOT NULL, amount_cents INTEGER NOT NULL
);
INSERT INTO customers (name, city, is_active) VALUES
    ('Ada','Denver',1), ('Ben','Austin',1), ('Carla','Denver',0);
INSERT INTO orders (customer_id, order_date, amount_cents) VALUES
    (1,'2026-01-05',4999),(1,'2026-02-11',1250),(2,'2026-01-20',15000),(3,'2026-02-01',799);
```

```sql
-- 1. A basic view: hide the join, standardize the shape
CREATE VIEW order_details AS
SELECT o.id AS order_id,
       c.name AS customer,
       c.city,
       o.order_date,
       o.amount_cents / 100.0 AS amount_dollars
FROM orders o
JOIN customers c ON c.id = o.customer_id;

SELECT * FROM order_details WHERE city = 'Denver' ORDER BY order_date;

-- 2. An aggregating view: THE definition of customer value, written once
CREATE VIEW customer_summary AS
SELECT c.id AS customer_id,
       c.name,
       c.is_active,
       COUNT(o.id)                              AS num_orders,
       COALESCE(SUM(o.amount_cents), 0) / 100.0 AS lifetime_dollars,
       MAX(o.order_date)                        AS last_order_date
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.id
GROUP BY c.id, c.name, c.is_active;

-- 3. Views compose: build on the view, not the tables
CREATE VIEW vip_customers AS
SELECT * FROM customer_summary
WHERE lifetime_dollars >= 50 AND is_active = 1;

SELECT name, lifetime_dollars FROM vip_customers;

-- 4. Views are live: new data appears without any refresh
INSERT INTO orders (customer_id, order_date, amount_cents) VALUES (2,'2026-03-01',9000);
SELECT * FROM vip_customers;    -- Ben's numbers already updated

-- 5. Changing a view = drop + recreate (SQLite)
DROP VIEW vip_customers;
CREATE VIEW vip_customers AS
SELECT * FROM customer_summary
WHERE lifetime_dollars >= 100 AND is_active = 1;   -- threshold raised

-- 6. Trigger: automatic audit log of balance-relevant changes
CREATE TABLE order_audit (
    id INTEGER PRIMARY KEY,
    order_id INTEGER NOT NULL,
    action TEXT NOT NULL,                 -- 'INSERT' / 'UPDATE' / 'DELETE'
    old_amount INTEGER,
    new_amount INTEGER,
    logged_at TEXT NOT NULL DEFAULT (DATETIME('now'))
);

CREATE TRIGGER trg_orders_insert
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    INSERT INTO order_audit (order_id, action, new_amount)
    VALUES (NEW.id, 'INSERT', NEW.amount_cents);
END;

CREATE TRIGGER trg_orders_update
AFTER UPDATE OF amount_cents ON orders
FOR EACH ROW
BEGIN
    INSERT INTO order_audit (order_id, action, old_amount, new_amount)
    VALUES (NEW.id, 'UPDATE', OLD.amount_cents, NEW.amount_cents);
END;

-- Watch it work:
INSERT INTO orders (customer_id, order_date, amount_cents) VALUES (1,'2026-03-10',2000);
UPDATE orders SET amount_cents = 2500 WHERE id = 6;
SELECT * FROM order_audit;

-- 7. Trigger enforcing a rule constraints can't: no orders for inactive customers
CREATE TRIGGER trg_no_inactive_orders
BEFORE INSERT ON orders
FOR EACH ROW
WHEN (SELECT is_active FROM customers WHERE id = NEW.customer_id) = 0
BEGIN
    SELECT RAISE(ABORT, 'customer is inactive');
END;

INSERT INTO orders (customer_id, order_date, amount_cents) VALUES (3,'2026-03-11',100);
-- Error: customer is inactive

-- 8. Inspecting what exists
SELECT name, type FROM sqlite_master WHERE type IN ('view','trigger');

-- 9. Manual "materialized view" pattern for an expensive report
CREATE TABLE monthly_revenue_cache AS
SELECT STRFTIME('%Y-%m', order_date) AS month, SUM(amount_cents) AS revenue
FROM orders GROUP BY month;
-- Refresh = DELETE FROM ... + INSERT INTO ... SELECT (schedule it in app code)
```

## Common Pitfalls

**1. Expecting a view to be a snapshot.**
A view re-runs its query; if you wanted "the data as of now," you wanted `CREATE TABLE snap AS SELECT ...`. Conversely, if your "cache table" is stale, that's because it *is* a snapshot — know which you created.

**2. Treating views as free performance.**
`SELECT * FROM customer_summary WHERE name = 'Ada'` still aggregates as much as the underlying query does (engines can often push filters in, but not always). Views organize logic; **indexes** (Chapter 11) provide speed.

**3. Deep view-on-view towers.**
Four layers of views each adding a join produce queries nobody can debug. Two layers is a sensible ceiling; if a view tower gets slow, EXPLAIN the final query and consider flattening.

**4. INSERTing into a view.**
SQLite: `cannot modify order_details because it is a view`. Write to base tables. (If an engine does allow it, be even more careful — the rules for which views are updatable are subtle.)

**5. Trigger recursion and surprise cascades.**
A trigger on `orders` that inserts into `orders` recurses; a web of triggers firing each other becomes unfollowable. Keep triggers leaf-like: they write to *log/summary* tables, not to tables that themselves carry triggers.

**6. Hiding business rules in triggers nobody knows about.**
Six months later, "why does this insert fail?!" — because of `trg_no_inactive_orders`. If a trigger rejects data, its RAISE message must say why, and the trigger belongs in your schema file under version control, commented.

## Practice Exercises

1. Create a view `city_report` showing, per city: active customer count, total orders, and revenue in dollars (zero-order cities included). Query it three ways: sorted by revenue, filtered to one city, and joined back against `customers` to list VIPs per city.
2. Demonstrate view liveness and its cost: add an order, show `city_report` reflects it instantly, then articulate (comment or note) what work the database re-did to achieve that, and which index would reduce it.
3. Build an audit system for `customers`: a trigger-maintained log capturing INSERTs and any UPDATE that changes `is_active`, storing old and new values with timestamps. Prove a ROLLBACK of a customer change also rolls back the audit rows.
4. Use a trigger to maintain a deliberately denormalized column: add `order_count INTEGER NOT NULL DEFAULT 0` to `customers` and write the INSERT and DELETE triggers on `orders` that keep it accurate. Verify against `customer_summary` after mixed inserts/deletes inside and outside transactions.
5. Decide-and-defend: for each of these rules, state where it should live (constraint, view, trigger, or application code) and why, in 1–2 sentences each: (a) emails must be unique; (b) "churn risk" = active customer with no order in 90 days; (c) every deletion of an order must be recorded with who/when; (d) customers get a 10% discount email after their 5th order.
