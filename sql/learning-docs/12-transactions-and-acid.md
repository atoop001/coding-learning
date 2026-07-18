# Chapter 12: Transactions & ACID

## Overview

Move $100 between bank accounts: subtract from one row, add to another. If the program crashes between the two UPDATEs, money has vanished. **Transactions** solve this: they group multiple statements into one all-or-nothing unit. Either every change lands, or none do — no matter what crashes, fails, or conflicts along the way.

Transactions are the "D" in CRUD apps growing up: any real application whose data must stay *consistent* uses them constantly. Project 5 (bank ledger) is built on this chapter.

## Definitions & Explanations

### The core commands

```sql
BEGIN;        -- start a transaction (BEGIN TRANSACTION is the long form)
  ... any number of INSERT / UPDATE / DELETE / SELECT ...
COMMIT;       -- make ALL the changes permanent, atomically
-- or
ROLLBACK;     -- undo EVERYTHING since BEGIN, as if it never happened
```

Between BEGIN and COMMIT, your changes are visible **to you** (your own SELECTs see them) but not durable — and in SQLite's default mode, not visible to other connections until commit.

### Autocommit — what happens without BEGIN

By default every single statement is its own tiny transaction, committed immediately ("autocommit"). That's why you've survived 11 chapters without BEGIN. Transactions matter the moment **two or more changes must succeed or fail together** — or when you want an undo button while experimenting.

### ACID — the four guarantees

- **Atomicity** — all-or-nothing. A transaction never half-applies. Crash mid-transaction → on restart, the database rolls back the incomplete work automatically (SQLite does this via its journal/WAL file).
- **Consistency** — every transaction moves the database from one *valid* state to another: all constraints (NOT NULL, CHECK, FK, UNIQUE) hold at commit. If a statement violates one, it fails and you can roll back.
- **Isolation** — concurrent transactions don't see each other's half-done work. Each behaves as if it were alone. (Engines offer *isolation levels* trading strictness for concurrency; SQLite's answer is simpler — see below.)
- **Durability** — once COMMIT returns, the data survives power loss. The engine forces the changes (or a log of them) to actual disk before reporting success.

### Errors inside a transaction

A failed statement inside a transaction does **not** automatically roll back the transaction in SQLite (unlike PostgreSQL, which poisons the transaction until rollback). Your application decides: catch the error, then either fix-and-continue or `ROLLBACK`. The discipline that always works everywhere: **on any error, ROLLBACK the whole transaction and retry or report.**

### Savepoints — partial undo

```sql
SAVEPOINT step2;
...
ROLLBACK TO step2;   -- undo back to the savepoint, transaction still open
RELEASE step2;       -- discard the savepoint, keep the work
```

Useful for "try this optional part; if it fails, keep the rest."

### Concurrency in SQLite specifically

SQLite locks at the **database level**: one writer at a time (readers can proceed, especially in WAL mode — `PRAGMA journal_mode = WAL;` is the recommended setting for apps). A second writer gets `SQLITE_BUSY`; well-behaved apps set a busy timeout and retry. Client/server databases (PostgreSQL) instead lock at row level and support many concurrent writers — one of the key reasons to graduate to them (Chapter 16). The *transaction discipline you learn here is identical* in both.

Two classic concurrent hazards transactions prevent:
- **Lost update**: two sessions read balance 100, both add 10, both write 110 — one update lost. Fix: do the math *in the database* (`SET balance = balance + 10`) inside a transaction, rather than read-modify-write in app code.
- **Inconsistent read**: a report sums all accounts while a transfer is mid-flight and counts the money twice or zero times. Isolation prevents the report from seeing the half-done transfer.

### Transactions as a speed tool

Committing has a fixed disk-sync cost. 10,000 single-statement autocommits = 10,000 syncs. One transaction around all 10,000 inserts = one sync — often **100× faster**. Always wrap bulk loads in a transaction.

## Code Examples

A minimal bank:

```sql
PRAGMA foreign_keys = ON;

CREATE TABLE accounts (
    id            INTEGER PRIMARY KEY,
    owner         TEXT NOT NULL,
    balance_cents INTEGER NOT NULL CHECK (balance_cents >= 0)  -- no overdrafts
);

CREATE TABLE transfers (
    id           INTEGER PRIMARY KEY,
    from_account INTEGER NOT NULL REFERENCES accounts(id),
    to_account   INTEGER NOT NULL REFERENCES accounts(id),
    amount_cents INTEGER NOT NULL CHECK (amount_cents > 0),
    made_at      TEXT NOT NULL DEFAULT (DATETIME('now'))
);

INSERT INTO accounts (owner, balance_cents) VALUES
    ('Ada', 50000),    -- $500.00
    ('Ben', 12000);    -- $120.00
```

```sql
-- 1. The canonical transfer: three statements, one atomic unit
BEGIN;
UPDATE accounts SET balance_cents = balance_cents - 10000 WHERE id = 1;
UPDATE accounts SET balance_cents = balance_cents + 10000 WHERE id = 2;
INSERT INTO transfers (from_account, to_account, amount_cents) VALUES (1, 2, 10000);
COMMIT;

SELECT * FROM accounts;   -- Ada 40000, Ben 22000 — and a matching audit row

-- 2. ROLLBACK: change your mind
BEGIN;
UPDATE accounts SET balance_cents = 0 WHERE id = 1;
SELECT balance_cents FROM accounts WHERE id = 1;   -- 0 (visible to YOU, uncommitted)
ROLLBACK;
SELECT balance_cents FROM accounts WHERE id = 1;   -- 40000 — as if nothing happened

-- 3. A constraint saving you mid-transaction: overdraft attempt
BEGIN;
UPDATE accounts SET balance_cents = balance_cents - 999999 WHERE id = 2;
-- Error: CHECK constraint failed: balance_cents >= 0
-- The statement failed; the transaction is still open. Correct move:
ROLLBACK;
SELECT * FROM accounts;   -- untouched

-- 4. Atomicity + the audit trail: if ANY piece fails, no piece survives.
--    (Try a transfer where the INSERT references a nonexistent account —
--    after ROLLBACK, the balance changes are gone too. Money and records
--    can never disagree.)
BEGIN;
UPDATE accounts SET balance_cents = balance_cents - 500  WHERE id = 1;
UPDATE accounts SET balance_cents = balance_cents + 500  WHERE id = 2;
INSERT INTO transfers (from_account, to_account, amount_cents) VALUES (1, 99, 500); -- FK fails!
ROLLBACK;

-- 5. Savepoints: optional sub-step
BEGIN;
INSERT INTO accounts (owner, balance_cents) VALUES ('Carla', 0);
SAVEPOINT bonus;
UPDATE accounts SET balance_cents = balance_cents + 2500 WHERE owner = 'Carla';
-- Business rule check fails? Undo just the bonus:
ROLLBACK TO bonus;
COMMIT;                    -- Carla exists with balance 0; the bonus never happened

-- 6. The lost-update fix: compute in SQL, don't read-modify-write in app code
-- ❌ App pattern: SELECT balance; add in Python; UPDATE ... SET balance = 41000
--    (two apps doing this concurrently lose one update)
-- ✅ Atomic in-database arithmetic:
BEGIN;
UPDATE accounts SET balance_cents = balance_cents + 1000 WHERE id = 1;
COMMIT;

-- 7. Bulk-load speed: compare these two (see Chapter 11's timing setup)
-- Slow: 10,000 autocommitted inserts (conceptually — don't actually type 10,000!)
-- Fast:
CREATE TABLE load_test (id INTEGER PRIMARY KEY, v INTEGER);
BEGIN;
WITH RECURSIVE n(i) AS (SELECT 1 UNION ALL SELECT i+1 FROM n WHERE i < 10000)
INSERT INTO load_test (v) SELECT i FROM n;
COMMIT;

-- 8. WAL mode for applications (persists in the database file)
PRAGMA journal_mode = WAL;
```

To *see* isolation: open the same database in **two** sqlite3 shells. In shell 1, `BEGIN; UPDATE accounts SET balance_cents = balance_cents + 1 WHERE id = 1;` (no commit). In shell 2, `SELECT` the balance — you'll see the old value (WAL mode) or a busy error on write attempts. COMMIT in shell 1 and shell 2 sees the new value.

## Common Pitfalls

**1. Forgetting to COMMIT.**
Changes sit invisible to other connections, and the shell holding the open transaction blocks other writers. If another tool says "database is locked," look for an open transaction someone forgot to close. (In Python, Chapter 15, forgetting `conn.commit()` is *the* classic bug — changes silently vanish when the program exits.)

**2. Half-transaction thinking.**

```sql
-- ❌ Two independent autocommitted statements — crash between them loses money:
UPDATE accounts SET balance_cents = balance_cents - 10000 WHERE id = 1;
UPDATE accounts SET balance_cents = balance_cents + 10000 WHERE id = 2;

-- ✅ One atomic unit:
BEGIN;
UPDATE ...; UPDATE ...;
COMMIT;
```

Rule: if you'd be upset seeing only *some* of a group of changes applied, that group is a transaction.

**3. Assuming an error auto-rolled-back (SQLite).**
After a failed statement your transaction is still open; blindly COMMITting keeps the statements that *did* succeed — possibly a half-transfer. On error: ROLLBACK, always.

**4. Read-modify-write across the app boundary.**
Fetching a value into Python, computing, and writing it back invites lost updates. Push arithmetic into the UPDATE (`SET x = x + ?`) or use constraints/`WHERE` guards (`WHERE balance_cents >= ?` and check the affected-row count).

**5. Long-lived transactions.**
A transaction held open during user think-time (or a big report) blocks writers (SQLite) or bloats/locks (other engines). Keep transactions short: gather inputs first, then BEGIN–work–COMMIT quickly.

**6. Nesting BEGIN.**
`BEGIN` inside an open transaction errors in SQLite. Use SAVEPOINTs for nesting-like behavior.

## Practice Exercises

Use the bank schema above.

1. Write the complete, safe transfer of $37.50 from Ben to Ada — balance updates plus audit row — then verify total money in the system is unchanged before and after with a SUM query. Then repeat the exercise but ROLLBACK instead of COMMIT and re-verify.
2. Demonstrate atomicity: construct a three-statement transaction where the *last* statement is guaranteed to fail (violate any constraint you like), handle it correctly, and show that the first two statements left no trace.
3. Build the "overdraft-safe withdrawal" pattern *without* relying on the CHECK constraint: an UPDATE whose WHERE clause only matches when funds suffice, followed by inspecting `SELECT changes();` to decide between COMMIT (success) and ROLLBACK (insufficient funds). Test both paths.
4. Using two simultaneous sqlite3 shells on the same database file (WAL mode), reproduce: (a) shell 2 not seeing shell 1's uncommitted update, and (b) a busy/locked error from concurrent writes. Write down the exact sequence of commands and what each shell displayed.
5. Time the bulk-load difference yourself: insert 5,000 rows one-per-autocommit via a script or generated statements versus the same rows in one transaction (`.timer on`). Report the two times, then explain in two sentences which ACID property makes the slow version slow.
