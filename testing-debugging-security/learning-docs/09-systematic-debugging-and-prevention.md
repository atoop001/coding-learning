# Chapter 9: Systematic Debugging & Prevention

## Overview

Chapters 7–8 taught you to find bugs. This chapter teaches the habits that make bugs *stay* fixed and show up less often: building minimal reproductions, turning every fix into a regression test, logging well enough that future bugs diagnose themselves, and using assertions to make code fail fast and loudly instead of limping along with corrupt state. These four habits are a large part of what separates professional codebases from hobby ones — not genius, just systems that don't rely on memory or luck.

## Definitions & Explanations

**Minimal reproduction (minimal repro)** — the smallest program/input that still exhibits the bug. Start from the failing situation and *delete everything you can* while the bug persists: remove functions, shrink the input file from 10,000 lines to the 3 that matter, replace the real API with a hardcoded response. Why bother? (1) A 10-line repro makes the cause nearly self-evident. (2) It's the required format for good bug reports — maintainers of libraries will ask for one. (3) The deletion process itself *is* debugging: the moment removing something makes the bug vanish, you've found a load-bearing suspect.

**Regression test** — a test written *from a bug*: it reproduces the bug, fails against the broken code, and passes once fixed. The iron rule: **no bug is "fixed" until a test exists that would have caught it.** The workflow is always: reproduce → write the failing test → fix → watch it pass → keep the test forever. Fixing without the test first is risky: you can't be certain your fix addressed the real cause rather than masking a symptom.

**Logging** — the permanent, structured version of print debugging: your program's diary. Unlike prints, log entries have **levels**, timestamps, and can be switched on/off or redirected without editing code. The standard levels:
- `DEBUG` — chatty detail for developers ("cache miss for key=42").
- `INFO` — normal notable events ("server started on port 5000", "user 17 logged in").
- `WARNING` — something odd but survivable ("retrying request, attempt 2/3").
- `ERROR` — an operation failed ("could not save order 991: disk full").
- `CRITICAL` — the program can't continue.

Good log lines answer *who/what/with which values/what happened*: `WARNING order=991 user=17 payment declined (code=51)` beats `WARNING payment problem`. In production you typically run at INFO and flip to DEBUG when investigating.

**Assertion (in production code)** — a statement of something that *must* be true at this point, or the program is already broken: `assert total >= 0`, "this list is sorted before we binary-search it." Assertions convert silent corruption into loud, located crashes near the *cause* — a bug caught by an assertion two lines after it happened is trivially debuggable; the same bug surfacing as a weird UI glitch three modules later can eat a day. Distinguish:
- **Assertions** guard against *programmer* errors (impossible states, violated invariants). They may be stripped in optimized runs (`python -O`), so:
- **Validation** guards against *user/input* errors (bad form data, malformed files) and must use real `if`/`raise` logic — never assertions — because user input being wrong is *expected*, not impossible.

**Defensive coding vs failing fast** — "defensive" code that quietly substitutes defaults (`total = total or 0`) hides bugs; failing fast surfaces them while the cause is fresh. Default to failing fast internally; be forgiving only at the true edges (user input, external APIs) where forgiveness is a spec decision.

**`git bisect`** — binary search over your commit history: mark a known-good old commit and the bad current one; git checks out midpoints and you answer good/bad until it names the exact commit that introduced the regression. With a test script, `git bisect run` automates the whole hunt.

**Reviewing code you didn't write** — the other half of prevention: most bugs that never happen are caught in review, before they reach main. A reviewer works in a fixed priority order: **correctness first** (does it do what it claims, is there a bug), then **clarity** (would a stranger — future you included — understand this in six months without asking), then **tests** (do they cover the new behavior, including the edge cases from Chapter 4), then style last, and often not worth blocking on at all. Separate every comment into **blocking** (must be addressed before merge — "this raises on an empty list, and that's a real input here") versus **nit** (optional polish, labeled as such so the author isn't guessing — "nit: `x` → `remaining_budget` would read clearer"). The difference between a useless review comment and a useful one is specificity: name the line, state what's wrong, and either suggest a fix or ask a pointed question. "This looks off" teaches nothing; "this doesn't handle `qty <= 0` — should it raise or clamp to zero?" does.

## Code Examples

### The regression workflow, in full

```python
# BUG REPORT: "Total shows $0.00 when a coupon covers the whole order,
#              but the receipt then says 'Payment failed'."

# STEP 1 — minimal repro (checkout.py stripped from 400 lines to this):
def amount_due(subtotal, coupon_value):
    due = subtotal - coupon_value
    if not due:                # suspicious: what does `not 0.0` do?
        raise PaymentError("payment amount invalid")
    return due
```

```python
# STEP 2 — regression test, written BEFORE the fix. Run it: it must FAIL.
import pytest
from checkout import amount_due, PaymentError

def test_coupon_covering_full_order_means_zero_due_not_error():
    """Regression test for issue: full-coverage coupon raised PaymentError.

    Root cause: `if not due:` treated a legitimate 0.0 as 'no amount'.
    """
    assert amount_due(subtotal=25.00, coupon_value=25.00) == 0.0
```

```python
# STEP 3 — the fix. `not due` conflated "zero" with "invalid";
# what we actually wanted to reject was a NEGATIVE amount.
def amount_due(subtotal, coupon_value):
    due = subtotal - coupon_value
    if due < 0:
        raise PaymentError("coupon exceeds order value")
    return due

# STEP 4 — run pytest: the new test passes, all old tests still pass.
# STEP 5 — the test stays in the suite forever. This bug can never return
#          unnoticed. That's the whole point.
```

### Logging done properly (Python)

```python
# app.py
import logging

# Configure once, at program startup — not inside library functions.
logging.basicConfig(
    level=logging.INFO,                                   # show INFO and above
    format="%(asctime)s %(levelname)-8s %(name)s: %(message)s",
    handlers=[
        logging.StreamHandler(),                          # console
        logging.FileHandler("app.log", encoding="utf-8"), # and a file
    ],
)
log = logging.getLogger("shop")                           # named logger


def charge(order_id, amount, gateway):
    log.info("charging order=%s amount=%.2f", order_id, amount)
    try:
        gateway.charge(amount)
    except GatewayTimeout:
        # exception() logs at ERROR level AND includes the full traceback
        log.exception("gateway timeout for order=%s", order_id)
        raise
    log.debug("charge succeeded order=%s", order_id)      # hidden at INFO level
```

Key habits on display: values in every message (`order=%s`), lazy `%s` formatting (the string is only built if the level is enabled), `log.exception` inside `except` blocks to preserve tracebacks, and DEBUG lines that cost nothing until you need them — flip `level=logging.DEBUG` and yesterday's mystery narrates itself.

### Logging in JavaScript (small-project pattern)

```javascript
// logger.js — console with levels; swap the sink later without touching callers.
const LEVELS = { debug: 10, info: 20, warn: 30, error: 40 };
let threshold = LEVELS[process.env.LOG_LEVEL ?? "info"];

function emit(level, msg, data = {}) {
  if (LEVELS[level] < threshold) return;
  const line = { ts: new Date().toISOString(), level, msg, ...data };
  console[level === "debug" ? "log" : level](JSON.stringify(line));
}

export const log = {
  debug: (msg, data) => emit("debug", msg, data),
  info:  (msg, data) => emit("info", msg, data),
  warn:  (msg, data) => emit("warn", msg, data),
  error: (msg, data) => emit("error", msg, data),
};

// usage:
// log.info("charge attempted", { orderId: 991, amount: 25.0 });
// -> {"ts":"2026-07-18T09:14:03.120Z","level":"info","msg":"charge attempted","orderId":991,"amount":25}
```

Structured (JSON) lines look over-engineered until the first time you need to answer "show me every event for order 991" — then it's one `findstr 991 app.log` / grep away.

### Assertions that catch bugs at the source

```python
def apply_payments(ledger, payments):
    # Invariant: the ledger must be sorted by date for the merge below.
    # If some caller forgets, we want to know HERE — not via a subtly
    # wrong balance discovered next quarter.
    assert all(
        ledger[i].date <= ledger[i + 1].date for i in range(len(ledger) - 1)
    ), "ledger must be sorted by date"

    for p in payments:
        # VALIDATION, not assertion: `payments` comes from user uploads.
        if p.amount <= 0:
            raise ValueError(f"payment {p.id} has non-positive amount")
        _merge(ledger, p)

    # Postcondition: money is conserved.
    assert round(sum(e.amount for e in ledger), 2) == round(
        _initial_total + sum(p.amount for p in payments), 2
    ), "ledger total drifted — merge bug"
```

Note the split: impossible-by-design conditions → `assert`; expected-bad input → `raise`. Both are tested like any error path (Chapter 4).

### A diff to review — find the seeded problems before reading on

This is a real pull request diff, added to signup.py and its test file. Read it the way a reviewer would, in priority order, before checking the practice exercise below.

```diff
--- a/signup.py
+++ b/signup.py
@@ -0,0 +1,5 @@
+MIN_SIGNUP_AGE = 18
+
+def validate_age(age):
+    """Check whether a user meets the minimum signup age."""
+    return age > MIN_SIGNUP_AGE
```

```diff
--- a/test_signup.py
+++ b/test_signup.py
@@ -0,0 +1,6 @@
+from signup import validate_age
+
+def test_adult_can_register():
+    assert validate_age(25) is True
+
+def test_child_cannot_register():
+    assert validate_age(10) is False
```

This diff has three seeded problems typical of real reviews: a logic bug, a name that promises the wrong thing, and a missing edge case — see Practice Exercise 6.

## Common Pitfalls

- **Fixing the bug without writing the test.** Three months later, a refactor reintroduces it and nothing notices. Correction: failing test first, fix second — no exceptions, even for "trivial" bugs (especially for those: they recur the most).
- **Writing the regression test after the fix and never seeing it fail.** A test that never failed might be testing nothing — it could pass against the broken code too. Correction: stash or comment out the fix once, run the test, confirm red, restore.
- **Log messages without values.** `ERROR: save failed` — for whom? which record? why? Correction: every log line carries the identifiers and values you'd need at 2 a.m.
- **Logging at the wrong level** — everything at ERROR (alarm fatigue) or everything at DEBUG (invisible in production). Correction: use the level definitions above literally; if it wakes nobody up, it isn't ERROR.
- **Using assertions to validate user input.** `assert age >= 18` disappears under `python -O`, and even when present gives users an ugly crash instead of a message. Correction: assertions for invariants, `raise`/HTTP 400 for input.
- **Swallowing exceptions** (`except: pass`, empty `catch {}`) to "make errors stop." The bug is still there — now invisible, and the state is corrupt. Correction: catch only exceptions you can genuinely handle; otherwise log with `log.exception` and re-raise.
- **Minimal repros that aren't minimal.** Posting 300 lines to a forum, or "reproducing" with your whole app. Correction: keep deleting until removing anything else makes the bug disappear — that boundary is information.

## Practice Exercises

1. Take a real bug from any past project (or introduce one: an off-by-one in pagination). Execute the full five-step regression workflow, including the step where you verify the test fails against the broken code. Save the test into the project's suite.
2. Shrink a bug: write a 60+ line script containing one planted bug, then produce a minimal repro of 10 lines or fewer that still exhibits it. Record which deletions taught you something.
3. Retrofit logging into one of your Flask or Node apps: startup INFO line, one INFO per meaningful user action with identifiers, WARNING on retryable failures, `log.exception` in every catch. Then break something on purpose and diagnose it *using only the log file* — no debugger, no prints.
4. Add three assertions to an existing project: one precondition, one invariant, one postcondition. For each, write the comment explaining *what programmer mistake* it would catch. Then write one input-validation check and justify why it's `raise` rather than `assert`.
5. Create a small git repo, make 12 commits to a function where commit ~7 quietly introduces a bug, then use `git bisect` (with a test script if you can) to find the guilty commit. Note how many steps it took versus checking all 12.
6. Review the `signup.py` diff above. It has (at least) three problems: a logic bug, a misleadingly-named function, and a test suite missing the one input most likely to expose the bug. Write the actual review comments you'd leave — one per problem. For each: mark it **blocking** or **nit**, name the specific line, state what's wrong, and phrase it actionably per this chapter's guidance. Finish with an approve/request-changes decision and one sentence justifying it.
