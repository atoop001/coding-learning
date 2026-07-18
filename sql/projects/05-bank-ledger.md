# Project 5: Indexed, Transactional Bank Ledger

## Description

Build the database core of a small banking system: accounts, an append-only transaction ledger, atomic transfers, audit trails, and performance that holds up under a hundred thousand ledger rows. This project is about *correctness under pressure* — money must never appear or vanish, every change must be attributable, and the balance queries must stay fast. You'll prove your guarantees the way engineers do: by trying to break them, and by measuring.

## Difficulty

**Intermediate–Advanced** — estimated effort: 8–12 hours.

## Chapters Used

- Chapter 12 (transactions & ACID — the core)
- Chapter 11 (indexes & query plans — the other core)
- Chapter 5 (constraints as the last line of defense)
- Chapter 13 (triggers for the audit trail)
- Chapter 7–8 (balance and statement queries)

## Requirements Checklist

### Schema
- [ ] `customers`, `accounts` (owner FK, account type via CHECK, status active/frozen/closed, opened date), and `ledger_entries` — the append-only source of truth
- [ ] Design decision, documented in comments: **derived balances** — either (a) balance always computed by summing the ledger, or (b) a stored `balance_cents` column kept in sync. Pick one, state the trade-off, and enforce your choice's consistency
- [ ] Every ledger entry: account, amount in **integer cents** (positive = credit, negative = debit — or a signed/typed design you justify), an entry type ('deposit'/'withdrawal'/'transfer_in'/'transfer_out'/'fee' — CHECKed), a timestamp, and for transfers a shared `transfer_id` linking both legs
- [ ] Constraints making illegal states unrepresentable: no zero-amount entries, no entries on closed accounts (trigger territory), CHECK preventing negative *stored* balance if you chose design (b)
- [ ] `PRAGMA foreign_keys = ON;` and `PRAGMA journal_mode = WAL;` in your standard connection setup

### Transactional operations (each written as a reusable, commented SQL script block)
- [ ] Deposit and withdrawal: single-entry transactions with the overdraft-guard pattern — the WHERE-clause/`changes()` technique (or CHECK constraint) so an insufficient-funds withdrawal cleanly fails and rolls back
- [ ] Transfer: both ledger legs (+ stored-balance updates if design (b)) in **one transaction**, sharing a `transfer_id` — with a deliberate mid-transaction failure version proving nothing partial ever survives
- [ ] Account closure: only permitted at zero balance, enforced by your chosen mechanism
- [ ] Monthly fee sweep: one transaction applying a fee entry to every active account meeting a condition
- [ ] Every operation script ends with an invariant check: `SELECT` proving total money across the system changed by exactly the expected external amount (zero for transfers)

### Audit trail (triggers)
- [ ] An `audit_log` table populated automatically by triggers on ledger inserts and on any account status/owner change (old value, new value, timestamp)
- [ ] Proof that a rolled-back operation leaves **no** audit rows (triggers live inside the transaction — demonstrate it)
- [ ] A trigger that **forbids** UPDATE and DELETE on `ledger_entries` entirely (append-only means append-only) — with the blocked attempts kept, commented, in the script

### Performance (measured, not asserted)
- [ ] Generate 100,000+ ledger entries across 500+ accounts (recursive CTE with RANDOM(), inside one transaction)
- [ ] Statement query — one account's entries for one month, newest first — shown with `EXPLAIN QUERY PLAN` before and after adding the right (composite) index, with `.timer on` measurements recorded in comments
- [ ] Balance computation for one account and for all accounts: measured, then indexed appropriately; if you chose design (a), demonstrate the point where summing gets slow and discuss (comments) when design (b) or a snapshot table earns its complexity
- [ ] Every FK column indexed; a comment inventory of all indexes with one-line justifications, and at least one candidate index you considered and *rejected*, with reasoning
- [ ] The bulk-load speed comparison: your 100k generation with vs without a wrapping transaction, timed

### Reports
- [ ] A customer's full statement with running math left to the reader's choice (plain, or window-function stretch)
- [ ] Bank-wide daily totals: deposits, withdrawals, transfer volume per day (conditional aggregation)
- [ ] Suspicious-activity query: accounts with more than N withdrawals within any single day (grouping by account + day)

## Hints

- Write the invariant check *first* — "sum of all ledger amounts equals sum of all external deposits minus withdrawals" — and run it after every operation while developing. It catches half your bugs instantly.
- The two legs of a transfer are +X and −X: system-wide the ledger sums to the same total before and after. That's your atomicity test.
- For the overdraft guard with computed balances: the balance subquery goes inside the WHERE of an INSERT ... SELECT, or you check-then-insert inside one transaction — think about why the transaction makes check-then-insert safe here (single-writer SQLite) and why client-server engines need more care.
- Composite index order for statements: equality column (account) before range column (timestamp) — Chapter 11's rule.
- If a trigger seems not to fire, check you're not violating it *before* it can act (BEFORE vs AFTER matters).

## Stretch Goals

- [ ] Two-shell concurrency demo: reproduce a blocked writer and a clean retry with busy_timeout
- [ ] Interest accrual: a monthly transaction computing interest per account from its ledger and posting entries — idempotent (running it twice for the same month must not double-pay; design the guard)
- [ ] Snapshot/checkpoint table: monthly balance snapshots so statements only sum the current month against the last snapshot — the professional pattern for design (a) at scale
- [ ] After Chapter 15: wrap deposit/withdraw/transfer as Python functions with tests that assert the invariants, including a test that forces a failure between transfer legs
