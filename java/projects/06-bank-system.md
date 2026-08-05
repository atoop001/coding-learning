# Project 6: Bank Account System

## Description

A banking backend in miniature: multiple account types with different rules (checking with overdraft, savings with withdrawal limits), money movement between accounts, a transaction history, and — the real subject of this project — **failure handling done professionally**. Every rule violation raises a specific exception; no operation ever leaves an account in a corrupted state; the console layer catches everything and turns it into human-readable messages. Interfaces define the contracts; custom checked and unchecked exceptions carry structured failure data. Money is stored as **long cents** (or `BigDecimal`, your call) from the start — per Chapter 2, `double` is never an acceptable representation for currency, and this project treats that as a hard requirement rather than an afterthought.

## Difficulty

**Intermediate-plus** — estimated effort: 8–12 hours.

## Chapters used

- 09 (Inheritance), 10 (Interfaces & Abstract Classes) — the account type hierarchy
- 12 (Collections) — accounts registry, transaction lists
- 14 (Exceptions) — custom exceptions, checked vs unchecked, catch-or-declare, finally
- (Supporting: 11 packages, 13 generics where natural)

## Requirements checklist

- [ ] Abstract `Account` (or interface + abstract base): account number, owner, balance (private!) stored as **long cents** or `BigDecimal` — never `double` — with `deposit`, `withdraw`, transaction history
- [ ] `CheckingAccount` allows overdraft down to a per-account limit (e.g., −500.00, i.e. −50000 cents)
- [ ] `SavingsAccount` forbids overdraft entirely and allows at most 3 withdrawals per "month" (a counter with a reset operation is fine)
- [ ] Custom exceptions, each carrying relevant data as fields with getters:
  - [ ] `InsufficientFundsException` (checked) — includes requested amount and shortfall
  - [ ] `AccountNotFoundException` (checked) — includes the offending account number
  - [ ] `WithdrawalLimitException` (checked) — includes the limit
  - [ ] Invalid amounts (zero or negative cents) throw `IllegalArgumentException` (unchecked) — a comment explains *why* this one is unchecked while the others are checked
- [ ] Validation happens **before** any state changes: a failed operation provably leaves the balance untouched (write a demonstration in `main`)
- [ ] `Bank` class managing accounts in a `Map<String, Account>`, with `open`, `find` (throws `AccountNotFoundException`), `transfer`
- [ ] `transfer(from, to, amount)` is atomic: if the deposit side cannot proceed, the already-withdrawn money is restored (document your approach in a comment)
- [ ] Every mutation appends a `Transaction` record (timestamp via `java.time`, type, amount, resulting balance) to the account's history; history printout formatted as a table
- [ ] The console layer catches each custom exception **specifically** and prints distinct, useful messages (using the exception's data fields, not just `getMessage()`)
- [ ] No empty catch blocks anywhere; no `catch (Exception e)` except (optionally) one top-level guard around the menu loop that keeps the app alive
- [ ] Wrong catch order (subclass after superclass) tried once, compiler message recorded in a comment, then fixed

## Hints

- Build exceptions first — they define your failure vocabulary and force you to think through what can go wrong before writing the happy path.
- The rule "validate, then mutate" makes atomicity almost free for single accounts. For `transfer`, the tidy sequence is: find both accounts (may throw), *validate the withdrawal without performing it* if you can, then withdraw, then deposit inside a try that restores on failure.
- Give `Account` a `protected` helper like `record(String type, double amount)` so subclasses log transactions without exposing the history list publicly.
- Account numbers: a static counter formatted like `"ACC-%04d"` is plenty.
- Test the nasty paths deliberately: transfer from a missing account; withdraw exactly the overdraft limit (should succeed) and one cent past it (should fail); the 4th savings withdrawal.
- A `record Transaction(...)` is ideal here — immutable history entries.
- Working in cents means every amount is a whole `long` — no rounding, no `0.1 + 0.2` surprises. Do the cents-vs-dollars conversion (parsing user input like `"42.50"`, formatting output back to `"$42.50"`) at the console boundary only; the domain layer (`Account`, `Bank`) never touches a fractional dollar amount.

## Stretch goals

- Interest: a `InterestBearing` interface (`applyInterest()`); Bank applies it to all qualifying accounts in one polymorphic pass.
- Fraud guard: single withdrawals over 10,000 throw a `SuspiciousActivityException` that must be overridden by a "confirm" flow.
- Statement export: write an account's history to a text file (peek at Chapter 16), handling `IOException` — your first two-exception-type method.
- A tiny audit log: every exception thrown anywhere is also appended (type + message + timestamp) to a `List<String>` in `Bank`, printable via a hidden menu option.
