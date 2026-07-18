# Chapter 2: Writing Your First Unit Tests (pytest & Vitest)

## Overview

This chapter gets you from "I understand why tests matter" to "I have a real test suite running on my machine." You will install and run **pytest** (Python) and **Vitest** (JavaScript), learn the **Arrange–Act–Assert** structure that nearly every good test follows, and learn how to name tests so failures read like bug reports. Everything here runs on Windows in a normal terminal (PowerShell) — no special setup beyond Python and Node, which you have from earlier tracks.

## Definitions & Explanations

**Test framework** — a tool that discovers your test files, runs each test in isolation, catches assertion failures, and prints a report. You write functions; the framework does the orchestration.

**Test runner** — the command-line program (`pytest`, `vitest`) that executes the framework. In practice the terms blur together.

**Assertion** — the line in a test that states an expectation: `assert total == 42` (Python) or `expect(total).toBe(42)` (JS). If the expectation is false, the test fails.

**Arrange–Act–Assert (AAA)** — the three-part shape of a good unit test:

1. **Arrange** — set up inputs and any needed state ("given a cart with two items…").
2. **Act** — call the one thing you're testing ("…when I compute the total…").
3. **Assert** — check the outcome ("…then it equals the sum of prices").

Keeping the three phases visually distinct (blank lines between them) makes tests dramatically easier to read.

**Test discovery** — how the framework finds tests:
- *pytest*: files named `test_*.py` or `*_test.py`, containing functions named `test_*`.
- *Vitest*: files named `*.test.js` / `*.spec.js` (or `.ts`), containing `test(...)` / `it(...)` calls.

**Naming tests** — a test name should describe *behavior*, not implementation: `test_total_is_zero_for_empty_cart` beats `test_cart_1`. When it fails months later, the name alone tells you what broke. A common template: `test_<unit>_<scenario>_<expected result>` or, in Vitest's sentence style, `it("returns zero for an empty cart")`.

**Test isolation** — each test should stand alone: it must not depend on another test having run first, or on leftover state. Frameworks run tests in arbitrary order precisely to flush out hidden dependencies.

## Code Examples

### Setting up pytest (Windows / PowerShell)

```powershell
# Inside your project folder — use a virtual environment (good habit)
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install pytest
```

Project layout:

```
cart-kata/
├── cart.py
└── test_cart.py
```

```python
# cart.py — the code under test
class Cart:
    """A tiny shopping cart: items are (name, price) pairs."""

    def __init__(self):
        self._items = []

    def add(self, name, price):
        if price < 0:
            raise ValueError("price cannot be negative")
        self._items.append((name, price))

    def total(self):
        return sum(price for _, price in self._items)
```

```python
# test_cart.py — pytest discovers this automatically
import pytest
from cart import Cart


def test_total_is_zero_for_empty_cart():
    # Arrange
    cart = Cart()

    # Act
    result = cart.total()

    # Assert
    assert result == 0


def test_total_sums_all_item_prices():
    # Arrange
    cart = Cart()
    cart.add("book", 12.50)
    cart.add("pen", 1.50)

    # Act
    result = cart.total()

    # Assert
    assert result == 14.00


def test_add_rejects_negative_price():
    cart = Cart()

    # pytest.raises asserts that the code inside the block raises the error.
    # If no ValueError is raised, the test FAILS.
    with pytest.raises(ValueError):
        cart.add("weird", -5)
```

Run it:

```powershell
pytest            # run everything
pytest -v         # verbose: one line per test, names visible
pytest -k "empty" # run only tests whose name contains "empty"
```

A passing run prints dots; a failing run shows the assertion, both values, and the exact line — read that output carefully, it is doing half your debugging for you.

### Setting up Vitest

```powershell
# Inside your JS project folder
npm init -y
npm install --save-dev vitest
```

Add to `package.json` so `npm test` works:

```json
{
  "type": "module",
  "scripts": { "test": "vitest run", "test:watch": "vitest" }
}
```

```javascript
// cart.js — the code under test
export class Cart {
  #items = [];

  add(name, price) {
    if (price < 0) throw new Error("price cannot be negative");
    this.#items.push({ name, price });
  }

  total() {
    return this.#items.reduce((sum, item) => sum + item.price, 0);
  }
}
```

```javascript
// cart.test.js
import { describe, it, expect } from "vitest";
import { Cart } from "./cart.js";

// describe() groups related tests; it() is one test.
describe("Cart.total", () => {
  it("returns zero for an empty cart", () => {
    // Arrange
    const cart = new Cart();

    // Act
    const result = cart.total();

    // Assert
    expect(result).toBe(0);
  });

  it("sums all item prices", () => {
    const cart = new Cart();
    cart.add("book", 12.5);
    cart.add("pen", 1.5);

    expect(cart.total()).toBe(14);
  });
});

describe("Cart.add", () => {
  it("throws on a negative price", () => {
    const cart = new Cart();

    // Note: pass a FUNCTION to expect(...).toThrow — if you call cart.add
    // directly, the error happens before expect can catch it.
    expect(() => cart.add("weird", -5)).toThrow("negative");
  });
});
```

Run it:

```powershell
npm test            # single run, good for scripts/CI
npm run test:watch  # re-runs affected tests every time you save — very fast loop
```

### Useful assertions cheat sheet

```python
# pytest — plain asserts do everything; pytest rewrites them to show details
assert x == y
assert x != y
assert item in collection
assert abs(a - b) < 1e-9          # floats: never use == on computed floats
assert result is None
with pytest.raises(KeyError):
    lookup("missing")
```

```javascript
// Vitest
expect(x).toBe(y);                 // strict equality (===) — primitives
expect(obj).toEqual({ a: 1 });     // deep equality — objects/arrays
expect(list).toContain("pen");
expect(value).toBeCloseTo(0.3);    // floats: 0.1 + 0.2 !== 0.3
expect(value).toBeNull();
expect(fn).toThrow();
```

## Common Pitfalls

- **Using `toBe` on objects/arrays in Vitest.** `expect([1]).toBe([1])` fails because they are different objects. Correction: use `toEqual` for structural comparison; reserve `toBe` for primitives.
- **Comparing floats with `==`.** `0.1 + 0.2 == 0.3` is `False` in both languages. Correction: `pytest.approx(0.3)` or `toBeCloseTo(0.3)`.
- **Multiple unrelated behaviors in one test.** If `test_cart_works` checks add, total, and errors, a failure tells you almost nothing. Correction: one behavior per test; three small tests beat one blob.
- **Tests that depend on each other** (test B assumes test A already added items to a shared cart). Frameworks may run them in any order. Correction: each test builds its own fresh objects in its Arrange step.
- **Forgetting the lambda/function wrapper for exception tests in JS.** `expect(cart.add("x", -5)).toThrow()` explodes before Vitest can intercept. Correction: `expect(() => cart.add("x", -5)).toThrow()`.
- **Print-debugging inside tests and leaving it there.** Noise drowns signal. Correction: rely on the failure output; pytest shows local values with `pytest -l`.

## Practice Exercises

1. Set up both toolchains from scratch: a Python folder with pytest and a JS folder with Vitest, each with the Cart example running green. Then break `total()` on purpose and study the failure output until you can explain every line of it.
2. Write a function `clamp(value, low, high)` (returns `value` limited to the range) in either language, then write five AAA-structured tests for it: inside range, below, above, exactly at each boundary.
3. Take three tests you wrote for exercise 2 and rewrite their names using the behavior-naming convention. Ask: "If only the name appeared in a failure report, would I know what broke?"
4. Write `parse_price("$12.50") -> 12.5` (or JS equivalent) and tests including: normal input, missing `$`, empty string, and non-numeric text. Decide — and encode in tests — whether bad input should raise/throw or return a sentinel like `None`/`null`.
5. In your Vitest project, run `npm run test:watch`, then deliberately introduce and fix three different bugs in `cart.js`, watching the tests catch each one. Time how long each catch took — this feedback speed is the whole point.
