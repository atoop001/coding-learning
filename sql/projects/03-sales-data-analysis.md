# Project 3: Sales Data Analysis

## Description

Play the data analyst. Given a small e-commerce schema (customers, products, orders, order line items) that you'll create and seed with a few hundred generated rows, produce the analytical reports a business owner would actually ask for: revenue over time, best sellers, customer segmentation, category performance, and above/below-average comparisons. Every deliverable is a query — this project is pure GROUP BY, HAVING, subqueries, CTEs, CASE, and set operations, exercised until they're reflexes.

## Difficulty

**Intermediate** — estimated effort: 6–9 hours.

## Chapters Used

- Chapter 7 (aggregation — the heart of the project)
- Chapter 8 (subqueries & CTEs)
- Chapter 9 (CASE expressions, conditional aggregation, set operations)
- Chapter 6 (joins underneath everything)
- Chapter 3 (date functions for time bucketing)

## Requirements Checklist

### Setup
- [ ] Schema: `customers` (id, name, city, signup_date), `products` (id, name, category, price in cents), `orders` (id, customer_id, order_date), `order_items` (order_id, product_id, quantity, unit_price_cents — the price *at time of sale*)
- [ ] Proper PKs, FKs, and a composite key or uniqueness rule preventing duplicate product lines within one order
- [ ] Seed: at least 15 products across 4+ categories, 20+ customers across several cities, 100+ orders over at least 12 months, 200+ order line items — use Chapter 8's recursive-CTE generation trick with `RANDOM()`, or write a small Python generator (Chapter 15 preview); hand-typing is not required
- [ ] Include edge cases on purpose: customers with zero orders, at least one product never sold, order sizes from 1 to many items

### Core reports (one saved query each, commented with the business question)
- [ ] Total revenue, total orders, and average order value — one summary row (remember: revenue lives in order_items as quantity × unit price)
- [ ] Monthly revenue trend: month, order count, revenue, sorted chronologically
- [ ] Top 10 products by revenue, with units sold — and separately, the products that have **never** sold
- [ ] Revenue by category, highest first, including each category's share of total revenue as a percentage (subquery or CTE for the grand total)
- [ ] Top customers: name, order count, lifetime spend, average order value — including zero-order customers showing zeros, not NULLs
- [ ] Customers segmented with CASE into 'VIP' / 'regular' / 'one-time' / 'inactive' tiers by your own defined thresholds, with a count per tier in a second query
- [ ] Months with revenue above the average month (CTE computing monthly revenue, then compared against its own average)
- [ ] A pivot-style table: one row per category, columns for each quarter's revenue (conditional aggregation)
- [ ] Repeat-purchase analysis: for each product category, how many *distinct* customers bought from it, and which categories does the average customer buy from more than once (HAVING on the right grouping)
- [ ] One report using a set operation meaningfully (e.g., customers who bought in Q1 but not in Q4 — EXCEPT on customer ids)

### Analysis quality
- [ ] Every money figure displayed in dollars with sensible rounding, never raw cents
- [ ] Every report handles NULL/zero groups correctly (no silently vanishing customers or categories — check each one against the LEFT JOIN question)
- [ ] A short `findings.md` (10–15 lines): three genuine observations from your data with the query that supports each

## Hints

- Build a `revenue_per_order` CTE early (order_id → total); half the reports compose on top of it.
- When a SUM looks implausibly large, you've fanned out a join — count your rows before aggregating.
- `COUNT(DISTINCT ...)` and `COUNT(...)` answer different questions in almost every report here; pause on each COUNT and pick deliberately.
- For quarter columns: `CASE WHEN STRFTIME('%m', order_date) IN ('01','02','03') THEN ... END` inside SUM — get one quarter working, then copy.
- Percentages: multiply by `100.0` (not `100`) before dividing, and guard denominators with NULLIF.
- Cross-check totals: the sum of your category revenues must equal total revenue. Build these self-checks into the script.

## Stretch Goals

- [ ] Cohort-flavored report: for each signup month, how many customers ordered at least once in their first 90 days
- [ ] "Basket analysis" lite: the top 5 pairs of products appearing in the same order (self-join on order_items — Chapter 6's classmate-pairs pattern)
- [ ] Rewrite two reports using window functions (`RANK() OVER ...`, running-total revenue) after the Chapter 16 taster, and compare readability
- [ ] After Chapter 13: turn your three most-reused CTEs into views and rebuild the reports on top of them
