# Chapter 1: Why Testing & the Testing Mindset

## Overview

Before you write a single test, it helps to understand *why* professionals test at all. Testing is not busywork or an academic ritual — it is the cheapest insurance you can buy against shipping broken software. This chapter explains what tests actually buy you, the different kinds of tests, and how they fit together in the "testing pyramid." By the end you should be able to explain, in plain language, what a unit test is, why a project with tests is easier to change than one without, and where testing effort is best spent.

You already know some JavaScript and Python. Everything in this track builds on code like the code you have already written — small functions, scripts, and web apps.

## Definitions & Explanations

**Test** — a piece of code that runs *your* code with known inputs and checks that the output (or behavior) matches what you expect. If the check fails, the test fails, and you know something is wrong *before* a user does.

**Manual testing** — running your program yourself and eyeballing the result. You already do this every time you run a script to "see if it works." It works, but it does not scale: you forget cases, you get bored, and you can't re-run 200 manual checks after every change.

**Automated testing** — writing those checks as code so a machine re-runs all of them in seconds, every time, without forgetting anything.

**Regression** — a bug that reappears in something that used to work, usually because a change elsewhere broke it. Automated tests are the primary defense against regressions: once a test exists, that behavior is guarded forever.

### What tests buy you

1. **Confidence to change code.** The scariest thing about an untested codebase is that any edit might silently break something. With tests, you edit, run the suite, and know within seconds whether you broke anything the tests cover.
2. **Executable documentation.** A test named `test_empty_cart_total_is_zero` tells a reader exactly how the cart is supposed to behave — and unlike a comment, it can't quietly go stale, because it fails if the behavior changes.
3. **Better design.** Code that is hard to test is usually badly structured (we'll see why in Chapter 3). Writing tests pushes you toward small, focused, loosely-coupled functions.
4. **Faster debugging.** When a test fails, it points at a small area of code. Compare that to "the website is broken somewhere."
5. **A safety net for refactoring.** Refactoring means improving code structure without changing behavior. Without tests, you can't know behavior didn't change.

### Kinds of tests

**Unit tests** test one small piece — usually a single function or class — in isolation. They are fast (milliseconds), precise (a failure points at one function), and cheap to write. Example: "given the list `[3, 1, 2]`, does `sort_numbers` return `[1, 2, 3]`?"

**Integration tests** test that several pieces work *together*: your code plus a real file, a real database, or two of your own modules talking to each other. They catch problems unit tests can't — wrong SQL, mismatched data formats between modules, file-encoding surprises. They are slower and a failure points at a wider area.

**End-to-end (E2E) tests** drive the whole application the way a user would — for a web app, that means a real browser clicking real buttons. They are the most realistic and the most expensive: slow to run, fiddly to maintain, and a failure could be anywhere in the stack.

### The testing pyramid

The classic guidance is to have **many unit tests, fewer integration tests, and a small number of E2E tests** — drawn as a pyramid with unit tests as the wide base.

```
        /  E2E  \        few — slow, broad, realistic
       / integr. \       some — medium speed and scope
      /   unit    \      many — fast, precise, cheap
```

Why this shape? Because feedback speed matters. A unit suite that runs in 2 seconds gets run after every change; an E2E suite that takes 20 minutes gets run once a day, and bugs hide in the gap. The pyramid is a guideline, not a law — a script that mostly glues APIs together may legitimately need proportionally more integration tests.

### The testing mindset

Testing is fundamentally a shift in attitude:

- **Assume your code has bugs.** Everyone's does. Tests are how you find yours first.
- **Think in examples.** "What should this function do with an empty list? A negative number? A string with emoji?" Concrete examples are the raw material of tests.
- **Distrust "it works on my machine."** It worked *for the inputs you tried*. Tests make the set of tried inputs explicit and permanent.
- **Treat a failing test as good news.** A red test is a bug caught in your editor instead of in production.

## Code Examples

You don't need a framework to grasp the idea. Here is the world's simplest test, in plain Python:

```python
# calc.py — the code under test
def add(a, b):
    return a + b

# naive_test.py — a hand-rolled test, no framework
from calc import add

# Arrange: pick known inputs. Act: call the code. Assert: check the result.
result = add(2, 3)
assert result == 5, f"expected 5, got {result}"

result = add(-1, 1)
assert result == 0, f"expected 0, got {result}"

print("all checks passed")  # only prints if no assert failed
```

Run it with `python naive_test.py`. If someone later "improves" `add` and breaks it, this script screams immediately.

The same idea in plain JavaScript:

```javascript
// calc.js
export function add(a, b) {
  return a + b;
}

// naive-test.js — run with: node naive-test.js
import { add } from "./calc.js";
import assert from "node:assert";

assert.strictEqual(add(2, 3), 5);   // throws if not equal
assert.strictEqual(add(-1, 1), 0);

console.log("all checks passed");
```

Frameworks like **pytest** and **Vitest** (next chapter) do exactly this, plus: they find all your test files automatically, run every test even if one fails, and print a readable report of what failed and why.

Here is a taste of what a real bug being caught looks like. Suppose you wrote a discount function:

```python
def apply_discount(price, percent):
    # BUG: subtracts the percent number itself, not a percentage of price
    return price - percent

# A test built from a concrete example would catch it instantly:
# apply_discount(200, 10) should be 180 — but this returns 190.
assert apply_discount(200, 10) == 180  # FAILS -> bug found before shipping
```

The manual tester who only tried `apply_discount(100, 10)` would see `90` and think it works. The test with a *second* example exposes the bug. This is the core skill: choosing examples that distinguish correct code from almost-correct code.

## Common Pitfalls

- **"I'll add tests later."** Later never comes, and untested code grows structures that are hard to test (Chapter 3). Correction: write at least a few tests alongside the code, while the expected behavior is fresh in your mind.
- **Testing only the happy path.** `add(2, 3)` passing tells you little about `add("2", 3)`. Correction: for every function, test at least one normal case, one boundary case, and one error case (Chapter 4 goes deep on this).
- **Confusing "no test failures" with "no bugs."** Tests can only catch bugs in behaviors they check. Correction: treat green tests as "nothing I *thought to check* is broken," and keep adding checks when new bugs teach you what you missed.
- **Writing tests that test the language, not your code.** `assert len([1, 2]) == 2` verifies Python, not your logic. Correction: every test should be able to fail if *your* code is wrong.
- **Aiming for 100% coverage as the goal.** Coverage measures which lines ran, not whether anything meaningful was asserted. Correction: aim for covering *behaviors and risks*, and let the coverage number be a byproduct.

## Practice Exercises

1. Take any function you wrote in a previous track (Python or JS). Without a framework, write a plain script with five `assert` statements testing it — include at least one input you suspect might break it. Run it.
2. In writing, classify each of these as unit, integration, or E2E, and justify: (a) checking that `slugify("Hello World")` returns `"hello-world"`; (b) checking that your Flask route `/users` returns JSON from a real SQLite file; (c) a script that opens Chrome, logs into your app, and verifies the dashboard renders.
3. Find a bug you personally hit in a past project (check your memory or old commits). Write down the single `assert` statement that, had it existed, would have caught that bug.
4. Sketch a testing pyramid for a to-do list web app: list 5 unit-test ideas, 2 integration-test ideas, and 1 E2E-test idea.
5. Deliberately break the `add` function above in a subtle way (e.g., `return a + b + 0.0000001` or swap to `a - b` only when `b` is negative), and see which of your asserts catch it. What does this teach you about choosing examples?
