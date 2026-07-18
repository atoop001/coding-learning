# Chapter 3: Operators, Expressions & Built-in Functions

## Overview

Chapter 2 filtered and sorted raw column values. Real queries constantly *transform* values: trimming whitespace, formatting names, doing math on prices, extracting the year from a date. SQL provides operators (`+`, `||`, ...) and a library of built-in functions (`UPPER`, `ROUND`, `LENGTH`, date functions...) that operate **per row** inside `SELECT`, `WHERE`, and `ORDER BY`.

These are called **scalar functions** — one input row in, one value out. (They're different from *aggregate* functions like `SUM`, which collapse many rows into one; those come in Chapter 7.)

This chapter uses SQLite's function set; most have direct equivalents everywhere, and the differences are flagged.

## Sample Schema for This Chapter

```sql
CREATE TABLE products (
    id        INTEGER PRIMARY KEY,
    name      TEXT NOT NULL,
    category  TEXT NOT NULL,
    price     REAL NOT NULL,       -- decimal number
    stock     INTEGER NOT NULL,
    added_on  TEXT NOT NULL        -- 'YYYY-MM-DD'
);

INSERT INTO products (name, category, price, stock, added_on) VALUES
    ('  Mechanical Keyboard', 'electronics', 89.99, 12, '2024-11-02'),
    ('USB-C Cable 2m',        'electronics', 12.50, 130, '2025-01-15'),
    ('Standing Desk',         'furniture',   349.00, 4, '2024-06-30'),
    ('Desk Lamp',             'furniture',   24.95, 27, '2025-03-08'),
    ('Notebook (A5)',         'stationery',  4.25, 300, '2025-02-20'),
    ('Gel Pens 10-pack',      'stationery',  7.80, 85, '2024-12-01');
```

(The keyboard's name has sloppy leading spaces on purpose — we'll clean it.)

## Definitions & Explanations

### Expressions
An **expression** is anything that produces a value: a column name (`price`), a literal (`42`, `'hello'`), or a combination using operators and functions (`ROUND(price * 1.08, 2)`). Expressions can appear:
- in the `SELECT` list (computed output columns),
- in `WHERE` (computed conditions),
- in `ORDER BY` (computed sort keys).

### Arithmetic operators
`+  -  *  /  %` (modulo). Beware integer division: `7 / 2` is `3` in SQLite when both sides are integers; `7 / 2.0` is `3.5`. Multiply or divide by a decimal literal (`1.0`) to force real math.

### String concatenation
Standard SQL (and SQLite, PostgreSQL) uses `||`:

```sql
SELECT name || ' — $' || price FROM products;
```

(MySQL uses the `CONCAT()` function instead; SQL Server uses `+`. `CONCAT()` also exists in modern SQLite.)

### Core text functions (SQLite)
| Function | What it does | Example → result |
|---|---|---|
| `UPPER(s)` / `LOWER(s)` | change case | `UPPER('abc')` → `'ABC'` |
| `LENGTH(s)` | character count | `LENGTH('desk')` → `4` |
| `TRIM(s)` / `LTRIM` / `RTRIM` | strip whitespace | `TRIM('  hi ')` → `'hi'` |
| `SUBSTR(s, start, len)` | slice (1-based!) | `SUBSTR('Desk Lamp', 1, 4)` → `'Desk'` |
| `REPLACE(s, from, to)` | substitute text | `REPLACE('a-b-c', '-', '/')` → `'a/b/c'` |
| `INSTR(s, sub)` | position of substring (0 if absent) | `INSTR('cable', 'ab')` → `2` |

**SQL strings are 1-indexed** — `SUBSTR(s, 1, 3)` is the first three characters. This trips up every Python/JavaScript programmer once.

### Core numeric functions
| Function | What it does |
|---|---|
| `ROUND(x, n)` | round to n decimal places |
| `ABS(x)` | absolute value |
| `MIN(a, b, ...)` / `MAX(a, b, ...)` | smallest/largest of the *arguments* (scalar form — with a single argument they become aggregates, Chapter 7) |
| `CAST(x AS TYPE)` | convert types, e.g. `CAST('42' AS INTEGER)` |

### Date & time functions (SQLite)
SQLite stores dates as text/numbers and manipulates them with functions:

- `DATE('now')` → today's date as `'YYYY-MM-DD'`; `DATETIME('now')` adds the time (UTC); add `'localtime'` for local: `DATETIME('now', 'localtime')`.
- **Modifiers** shift dates: `DATE('now', '-7 days')`, `DATE('2025-01-15', '+1 month')`, `DATE('now', 'start of month')`.
- `STRFTIME(format, date)` extracts/formats parts: `STRFTIME('%Y', added_on)` → `'2024'` (as text); `%m` month, `%d` day, `%w` weekday (0=Sunday), `%H` hour.
- `JULIANDAY(a) - JULIANDAY(b)` → difference in days (fractional).

PostgreSQL instead uses a real `DATE` type with `EXTRACT(YEAR FROM col)`, `NOW()`, and interval arithmetic — the *concepts* transfer, the spellings differ.

### COALESCE and NULL-aware helpers
- `COALESCE(a, b, c, ...)` returns the first non-NULL argument — the standard "default value" tool: `COALESCE(nickname, name, 'Unknown')`.
- `IFNULL(a, b)` is SQLite's two-argument shorthand.
- `NULLIF(a, b)` returns NULL if `a = b`, else `a` — handy to avoid division by zero: `x / NULLIF(y, 0)`.

Chapter 14 goes deep on NULL; for now, remember any expression touching NULL usually yields NULL (`NULL + 1` → NULL, `'a' || NULL` → NULL).

## Code Examples

```sql
-- 1. Price math: show price with 8% tax, rounded to cents
SELECT name,
       price,
       ROUND(price * 1.08, 2) AS price_with_tax
FROM products;

-- 2. Inventory value per product (price × stock)
SELECT name, price * stock AS inventory_value
FROM products
ORDER BY inventory_value DESC;

-- 3. Clean and normalize messy names: trim spaces, uppercase category
SELECT TRIM(name)        AS clean_name,
       UPPER(category)   AS category
FROM products;

-- 4. Build a display label with concatenation
SELECT TRIM(name) || ' (' || category || ') — $' || price AS label
FROM products;

-- 5. Filter on an expression: names longer than 12 characters after trimming
SELECT TRIM(name) AS name, LENGTH(TRIM(name)) AS name_len
FROM products
WHERE LENGTH(TRIM(name)) > 12;

-- 6. First word of each product name (find the first space, slice before it)
SELECT name,
       SUBSTR(TRIM(name), 1, INSTR(TRIM(name) || ' ', ' ') - 1) AS first_word
FROM products;

-- 7. Extract the year and month a product was added
SELECT name,
       STRFTIME('%Y', added_on) AS year_added,
       STRFTIME('%m', added_on) AS month_added
FROM products;

-- 8. Products added in the last 200 days (relative to today)
SELECT name, added_on
FROM products
WHERE added_on >= DATE('now', '-200 days');

-- 9. How many days has each product been in the catalog?
SELECT name,
       CAST(JULIANDAY('now') - JULIANDAY(added_on) AS INTEGER) AS days_listed
FROM products
ORDER BY days_listed DESC;

-- 10. Defensive division: percent of a (possibly zero) target
--     NULLIF turns 0 into NULL so the division yields NULL instead of an error
SELECT name,
       stock,
       ROUND(100.0 * stock / NULLIF(stock + 0, 0), 1) AS pct_demo
FROM products;

-- 11. CAST in action: turn the text year back into a number and do math
SELECT name,
       2026 - CAST(STRFTIME('%Y', added_on) AS INTEGER) AS years_old_approx
FROM products;

-- 12. Sort by an expression: cheapest per-unit stationery first
SELECT name, price
FROM products
WHERE category = 'stationery'
ORDER BY ROUND(price, 0), name;
```

## Common Pitfalls

**1. Integer division silently truncating.**

```sql
-- ❌ stock is INTEGER; 12 / 130 → 0, so this always shows 0:
SELECT name, stock / 130 AS fraction FROM products;

-- ✅ Force real division:
SELECT name, stock / 130.0 AS fraction FROM products;
```

**2. Off-by-one with SUBSTR.**

```sql
-- ❌ Python habit: index 0 — returns unexpected results
SELECT SUBSTR('Lamp', 0, 2);   -- 'L' (weird SQLite edge behavior)

-- ✅ SQL strings start at 1:
SELECT SUBSTR('Lamp', 1, 2);   -- 'La'
```

**3. Concatenating with NULL wipes the whole string.**

```sql
-- ❌ If middle_name is NULL, the whole label becomes NULL:
SELECT first || ' ' || middle_name || ' ' || last AS full_name FROM people;

-- ✅ Provide a default:
SELECT first || ' ' || COALESCE(middle_name || ' ', '') || last FROM people;
```

**4. Expecting `DATE('now')` to be local time.**
SQLite's `'now'` is **UTC**. Late at night, `DATE('now')` can be "tomorrow." Use `DATE('now', 'localtime')` when you mean the user's calendar date.

**5. Comparing numbers stored as text.**
`'9' > '10'` is TRUE as text (character by character). If numbers arrive as text (e.g., imported CSV), `CAST` before comparing:

```sql
-- ❌ WHERE price_text > '100'      -- text comparison, wrong ordering
-- ✅ WHERE CAST(price_text AS REAL) > 100
```

**6. Using functions your target database doesn't have.**
`STRFTIME`, `IFNULL` are SQLite-flavored. When you later move to PostgreSQL (Chapter 16), prefer portable choices (`COALESCE` over `IFNULL`) where they exist.

## Practice Exercises

Use the `products` table above.

1. Produce a price list showing each product's trimmed name in UPPERCASE and its price formatted as text like `$89.99` (hint: concatenation — don't worry about padding decimals).
2. The store applies a 15% discount to everything in `furniture`. Show name, original price, and discounted price rounded to 2 decimals, for furniture only.
3. Show each product with the number of characters in its (trimmed) name, sorted longest name first; break ties alphabetically.
4. Using date functions, list products added in 2025 (derive the year with `STRFTIME`, don't hardcode a date-range comparison this time), and show how many whole days ago each was added.
5. Write one query that outputs a single text column per product in the exact shape `KEYBOARD-ELECTRONICS-2024` — the first word of the trimmed name uppercased, the category uppercased, and the year added, joined by hyphens. (Combines `SUBSTR`/`INSTR`, `UPPER`, `STRFTIME`, and `||`.)
