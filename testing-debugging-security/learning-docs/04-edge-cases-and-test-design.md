# Chapter 4: Edge Cases & Test Design

## Overview

Anyone can write a test proving that `add(2, 3)` returns 5. Professional testers earn their keep by asking: *what inputs might break this?* This chapter teaches deliberate test design: boundary analysis, equivalence classes, error-path testing, and parameterized tests that let you cover many cases without copy-pasting. The goal is to replace "I tested it with a couple of values" with a systematic method for choosing a small set of inputs that gives high confidence.

## Definitions & Explanations

**Edge case** — an input at the extreme or unusual end of what's possible: empty collections, zero, negative numbers, the largest allowed value, unicode text, `None`/`null`/`undefined`, duplicated items, already-sorted data. Most real-world bugs live at edges, because the "middle" cases are what the author was thinking about while writing.

**Boundary value analysis** — bugs cluster at boundaries, so test *at*, *just below*, and *just above* every boundary. If the rule is "free shipping over $50," the interesting prices are `49.99`, `50.00`, and `50.01` — not `20` and `80`. Off-by-one errors (`>` vs `>=`) are among the most common bugs in existence, and boundary tests catch exactly those.

**Equivalence class (equivalence partition)** — a group of inputs the code should treat identically. For a function validating ages 18–120: all values in 18–120 form one class ("valid"), values below 18 another ("too young"), values above 120 another ("too old"), and non-numbers a fourth. You need only ~one test per class plus the boundaries between classes — testing `25`, `37`, and `52` separately adds almost nothing, because if `25` works, its classmates almost certainly do.

**Happy path vs error path** — the happy path is what happens with good input; error paths are how the code responds to bad input. Error paths deserve tests *of their own*: does the function raise the documented exception? With a helpful message? Does it avoid corrupting state before failing? Untested error paths are where crashes and security holes hide.

**Parameterized test** — one test body run against a table of inputs/expected outputs. Instead of ten near-identical test functions, you write one function and ten table rows. Both pytest and Vitest support this natively, and each row reports pass/fail individually.

**Test design as a checklist.** For any function, walk this list:
1. One typical case per equivalence class.
2. Every boundary: at / just below / just above.
3. Empty / zero / minimal input (`""`, `[]`, `0`, single element).
4. Invalid types or malformed input — what *should* happen? Encode the decision.
5. "Weird but legal" input: whitespace, unicode, very long strings, duplicates, negative numbers if allowed.
6. For anything stateful: calling twice, calling in unusual orders.

## Code Examples

### Boundary + equivalence classes with pytest parameterization

```python
# pricing.py
def shipping_cost(order_total):
    """Free shipping at $50 or more; $5.99 below that; reject negatives."""
    if order_total < 0:
        raise ValueError("order total cannot be negative")
    if order_total >= 50:
        return 0.0
    return 5.99
```

```python
# test_pricing.py
import pytest
from pricing import shipping_cost

# One decorator, many cases. Each tuple is (input, expected).
# Note how the cases map to the design method:
#   equivalence classes: "below threshold", "at/above threshold"
#   boundaries: 49.99 / 50.00 / 50.01, and 0 as the low edge
@pytest.mark.parametrize(
    ("order_total", "expected"),
    [
        (0, 5.99),        # minimal legal input
        (10.00, 5.99),    # typical member of "below" class
        (49.99, 5.99),    # just below the boundary
        (50.00, 0.0),     # AT the boundary — catches > vs >= bugs
        (50.01, 0.0),     # just above
        (500.00, 0.0),    # typical member of "above" class
    ],
)
def test_shipping_cost(order_total, expected):
    assert shipping_cost(order_total) == expected


def test_negative_total_is_rejected():
    # Error path gets its own test — and we check the message is useful.
    with pytest.raises(ValueError, match="negative"):
        shipping_cost(-1)
```

Run `pytest -v` and note that each parameter row appears as its own test: `test_shipping_cost[50.0-0.0]` etc. If only the `50.00` row fails, you know instantly it's a `>` vs `>=` bug.

### The same discipline in Vitest with `test.each`

```javascript
// validate.js
export function validateUsername(name) {
  // Rules: 3-20 chars, letters/digits/underscore only, must start with a letter.
  if (typeof name !== "string") return { ok: false, reason: "not a string" };
  if (name.length < 3) return { ok: false, reason: "too short" };
  if (name.length > 20) return { ok: false, reason: "too long" };
  if (!/^[a-zA-Z][a-zA-Z0-9_]*$/.test(name)) {
    return { ok: false, reason: "invalid characters" };
  }
  return { ok: true };
}
```

```javascript
// validate.test.js
import { describe, it, expect, test } from "vitest";
import { validateUsername } from "./validate.js";

describe("validateUsername", () => {
  // Table-driven happy/edge cases. %s interpolates the input into the name.
  test.each([
    ["abc", true],            // exactly at min length boundary
    ["ab", false],            // just below min
    ["a".repeat(20), true],   // exactly at max
    ["a".repeat(21), false],  // just above max
    ["alice_42", true],       // typical valid
    ["1alice", false],        // starts with digit — rule boundary
    ["alice bob", false],     // space — invalid character class
    ["émile", false],         // unicode letter: legal-looking but outside rules
  ])("validateUsername(%j) -> ok: %j", (input, expectedOk) => {
    expect(validateUsername(input).ok).toBe(expectedOk);
  });

  it("explains WHY a name was rejected", () => {
    // Error paths should be informative, and that is testable too.
    expect(validateUsername("ab").reason).toBe("too short");
  });

  it("handles non-string input without crashing", () => {
    expect(validateUsername(null).ok).toBe(false);
    expect(validateUsername(42).ok).toBe(false);
  });
});
```

### Testing stateful edge cases

```python
# test_cart_state_edges.py — edges aren't only about values; order matters too.
from cart import Cart

def test_total_after_adding_then_reading_twice_is_stable():
    cart = Cart()
    cart.add("book", 10)
    assert cart.total() == 10
    assert cart.total() == 10     # calling total() must not consume/mutate

def test_two_carts_do_not_share_items():
    a, b = Cart(), Cart()
    a.add("book", 10)
    assert b.total() == 0         # catches the classic mutable-default bug
```

That last test catches a real, famous Python trap: `def __init__(self, items=[])` shares one list across all instances. A test with two objects finds it; a test with one never will.

## Common Pitfalls

- **Testing three values from the same equivalence class and calling it thorough.** `add(2,3)`, `add(4,5)`, `add(10,20)` are effectively one test. Correction: spend those tests on different classes and boundaries.
- **Skipping the "at the boundary" case.** Testing 49.99 and 50.01 but not 50.00 misses the exact input where `>` vs `>=` differ. Correction: always test the boundary value itself.
- **Assuming errors "obviously" work.** Untested error paths often crash with the *wrong* error (e.g., `TypeError` instead of your `ValueError`), leaking confusing messages to users. Correction: assert the exception type and, where the message matters, the message.
- **Parameterizing tests that need different assertions.** If half the rows check a value and half check an exception, the single body fills with `if`s and becomes unreadable. Correction: two parameterized tests — one for values, one for errors.
- **Writing edge-case tests after the bug ships.** Fine — but then *keep* them. Correction: every production bug becomes a permanent test row (this is the regression habit, formalized in Chapter 9).
- **Forgetting `null`/`None`/`undefined` in dynamically-typed code.** In JS especially, `undefined` sneaks in everywhere. Correction: for every public function, decide and test the behavior for missing/absent input.

## Practice Exercises

1. For a function `is_leap_year(year)`, list the equivalence classes and boundary values *before writing any code* (remember: divisible by 4, except centuries, except centuries divisible by 400). Then implement it and write a parameterized test covering your list. Include 1900 and 2000.
2. Write `paginate(items, page, page_size)` returning the requested slice. Design tests for: empty list, page beyond the end, page_size of 0 (decide: error or empty?), exactly-full last page, and partially-full last page.
3. Take the `validateUsername` rules above and find an input that the provided implementation and tests both miss (hint: think about what `.length` counts vs what a "character" is, or what happens with leading/trailing whitespace). Write the test that exposes your case.
4. Pick any function from a past project with at least one `if` in it. Enumerate its boundaries, write at/below/above tests for each, and report whether any fail.
5. Write a password-strength checker spec (min length, must contain classes of characters) as a table of 12+ parameterized cases *first*, then implement the function until all rows pass. Notice how the table becomes the specification.
