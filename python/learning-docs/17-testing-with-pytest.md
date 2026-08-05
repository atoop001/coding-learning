# Chapter 17: Testing with pytest

## Overview

How do you *know* your code works — and keeps working after every change? Professionals don't re-run programs by hand and eyeball the output; they write **automated tests**: small programs that exercise their code and verify the results, runnable in seconds, forever. Testing is a top-tier employability skill — "do you write tests?" is a standard interview topic, and most teams won't merge untested code.

**pytest** is Python's dominant testing framework: minimal ceremony (plain functions + plain `assert`), excellent failure output, and powerful features (parametrize, fixtures) when you need them. This chapter covers writing and running tests, testing error cases, structuring testable code, and the basics of fixtures and test-driven thinking.

## Definitions & Explanations

**Unit test** — a function that calls one small piece of your code with known inputs and asserts on the outputs. Fast, focused, independent of the outside world. (Integration and end-to-end tests check bigger assemblies; unit tests are your bread and butter.)

**`assert`** — Python's built-in check: `assert expression` raises `AssertionError` if the expression is falsy. In pytest, a failed assert *fails the test*, and pytest rewrites asserts to show you the actual values involved — that's why no special `assertEquals`-style methods are needed.

**Discovery conventions** — pytest finds tests automatically:

- files named `test_*.py` (or `*_test.py`)
- functions named `test_*`
- (optionally) classes named `Test*` containing `test_*` methods

**Running** — install into your venv (`pip install pytest`), then from the project root:

```powershell
pytest              # run everything it can discover
pytest -v           # verbose: one line per test
pytest test_cart.py                 # one file
pytest test_cart.py::test_total    # one test
pytest -k "discount"               # tests whose names match a keyword
pytest -x           # stop at first failure
```

Exit code 0 = all green; non-zero = failures (this is how CI systems know).

**Arrange–Act–Assert** — the universal test shape: *arrange* inputs/objects, *act* by calling the code under test, *assert* on the result. One logical behavior per test, named descriptively: `test_withdraw_rejects_overdraft` beats `test_3`.

**`pytest.raises`** — asserting that code *fails correctly*:

```python
with pytest.raises(ValueError):
    withdraw(balance=10, amount=999)
```

The test fails if the block *doesn't* raise that exception. Testing error paths is as important as testing happy paths.

**`pytest.approx`** — float-safe comparison: `assert 0.1 + 0.2 == pytest.approx(0.3)` (recall Chapter 2's float surprise).

**`@pytest.mark.parametrize`** — run one test body over many input/expected pairs, each reported separately. Kills copy-pasted near-identical tests.

**Fixture** — a function providing a prepared object/state to tests that name it as a parameter. Decorated with `@pytest.fixture`; pytest wires it up by matching names. Built-in fixture `tmp_path` hands you a fresh temporary directory (a `Path`) — the right way to test file I/O without littering your project.

**Testable code** — testing rewards good design from Chapter 6: small functions that *take parameters and return values*. Code that prints, calls `input()`, hits the network, or reads global state mid-logic is hard to test — separate calculation from I/O, and keep the `main()`/input/print shell thin.

## Code Examples

### The code under test

`cart.py`:

```python
"""A tiny shopping-cart module — deliberately test-friendly."""

def item_total(price: float, quantity: int) -> float:
    if price < 0 or quantity < 0:
        raise ValueError("price and quantity must be non-negative")
    return price * quantity

def cart_total(items: list[tuple[float, int]]) -> float:
    """items: list of (price, quantity) tuples."""
    return sum(item_total(p, q) for p, q in items)

def apply_discount(total: float, percent: float) -> float:
    """Return total after an N% discount. Percent must be 0-100."""
    if not 0 <= percent <= 100:
        raise ValueError(f"percent must be 0-100, got {percent}")
    return round(total * (1 - percent / 100), 2)
```

### First tests

`test_cart.py` (same folder):

```python
import pytest
from cart import item_total, cart_total, apply_discount

def test_item_total_multiplies():
    assert item_total(2.50, 4) == 10.0

def test_item_total_zero_quantity():
    assert item_total(9.99, 0) == 0

def test_cart_total_sums_items():
    items = [(2.0, 3), (5.0, 1), (0.5, 10)]     # arrange
    total = cart_total(items)                    # act
    assert total == 16.0                         # assert

def test_cart_total_empty_cart():
    assert cart_total([]) == 0
```

Run `pytest -v`. A deliberate failure shows pytest's superpower — introspected values:

```
    def test_cart_total_sums_items():
        items = [(2.0, 3), (5.0, 1), (0.5, 10)]
>       assert cart_total(items) == 17.0
E       assert 16.0 == 17.0
```

You see *both* sides of the comparison with zero effort.

### Testing the error paths

```python
def test_item_total_rejects_negative_price():
    with pytest.raises(ValueError):
        item_total(-1.0, 2)

def test_apply_discount_rejects_over_100():
    with pytest.raises(ValueError, match="0-100"):   # match: message must contain this
        apply_discount(50.0, 150)
```

### Floats and parametrize

```python
def test_discount_float_math():
    # 10% off 29.99 — float-safe comparison
    assert apply_discount(29.99, 10) == pytest.approx(26.99)

@pytest.mark.parametrize("total, percent, expected", [
    (100.0, 0, 100.0),       # no discount
    (100.0, 25, 75.0),       # simple quarter off
    (100.0, 100, 0.0),       # everything free
    (19.99, 50, 10.0),       # rounding: 9.995 -> 10.0
])
def test_apply_discount_cases(total, percent, expected):
    assert apply_discount(total, percent) == pytest.approx(expected)
```

`pytest -v` reports four separate tests, e.g. `test_apply_discount_cases[100.0-25-75.0]` — a failing case pinpoints itself.

### Fixtures — shared setup without repetition

```python
import pytest
from cart import cart_total

@pytest.fixture
def typical_cart():
    """A realistic cart used by several tests."""
    return [(12.50, 2), (3.99, 5), (0.99, 1)]

def test_typical_cart_total(typical_cart):        # named parameter = injected fixture
    assert cart_total(typical_cart) == pytest.approx(45.94)

def test_typical_cart_with_extra_item(typical_cart):
    typical_cart.append((10.0, 1))                # each test gets a FRESH copy
    assert cart_total(typical_cart) == pytest.approx(55.94)
```

### Testing file I/O with `tmp_path`

```python
import json

def save_report(path, data):        # imagine this lives in your module
    path.write_text(json.dumps(data, indent=2), encoding="utf-8")

def load_report(path):
    return json.loads(path.read_text(encoding="utf-8"))

def test_report_round_trip(tmp_path):             # built-in fixture: a temp directory
    target = tmp_path / "report.json"
    data = {"total": 45.94, "items": 3}

    save_report(target, data)

    assert target.exists()
    assert load_report(target) == data
    # tmp_path is auto-cleaned — nothing left on your disk
```

### Designing for testability — the refactor pattern

```python
# HARD to test: logic tangled with input/print
def main_bad():
    total = float(input("Total: "))
    if total > 100:
        print("Free shipping!")

# EASY to test: pure logic extracted; thin I/O shell around it
def shipping_cost(total):
    return 0.0 if total >= 100 else 7.99

def main():
    total = float(input("Total: "))
    print(f"Shipping: ${shipping_cost(total):.2f}")

# test file:
# def test_free_shipping_at_threshold():  assert shipping_cost(100) == 0.0
# def test_paid_shipping_below():         assert shipping_cost(99.99) == 7.99
```

## Common Pitfalls

**1. Tests that don't run** — file named `cart_tests.py` or function named `check_total()`: pytest silently finds nothing ("collected 0 items"). Stick to `test_*.py` / `test_*` religiously.

**2. Import errors from layout** — run `pytest` from the project root, keep tests next to (or in a `tests/` folder beside) the code, and remember your module must be importable (`from cart import ...` requires `cart.py` reachable from where you run).

**3. Testing by printing** — a test that `print()`s results and asserts nothing passes no matter what. Every test needs at least one `assert` (or `pytest.raises`).

**4. Exact float equality** — `assert apply_discount(29.99, 10) == 26.991` fails mysteriously. Reach for `pytest.approx` whenever floats are involved.

**5. Interdependent tests** — tests that rely on another test running first (shared globals, leftover files) break when run alone or reordered. Each test builds its own world (fixtures help); each cleans up (tmp_path does it for you).

**6. Only testing the happy path** — the bugs live at the edges: empty lists, zero, negatives, hugely long strings, missing files, bad user input. For every function ask: what are its boundaries, and what should it do when abused? Then *assert that it does*.

**7. One giant test** — a 40-line test asserting twelve behaviors reports only its first failure and hides the rest. Split by behavior; small tests localize breakage instantly.

**8. Chasing 100% coverage over meaning** — a test that merely calls code without meaningful asserts inflates coverage and catches nothing. Test *behaviors*, not lines.

## Practice Exercises

1. **First suite.** Take your Chapter 6 `is_strong(password)` function (or rewrite it) and build `test_passwords.py` with at least six tests: valid password, too short, no digit, no uppercase, no lowercase, and empty string. Run with `-v`; deliberately break the function to watch a failure report, then fix it.
2. **Parametrize FizzBuzz.** Test your `fizzbuzz_value(n)` from Chapter 6 with a single parametrized test covering: 1, 3, 5, 15, 30, 98. Then add a `pytest.raises` test defining what happens for `n=0` or negative n — deciding that behavior *is* part of the exercise.
3. **Bank account under test.** Write tests for Chapter 13's `BankAccount`: deposits increase balance, withdrawals decrease it, overdrafts raise, negative deposits raise, and a fresh account starts at its given balance. Use a fixture providing an account with balance 100.
4. **Temp-file round trip.** Using `tmp_path`, test the `load_contacts`/`save_contacts` pair from Chapter 16: saving then loading returns equal data; loading a nonexistent file returns `[]`; loading a file containing `not json{` returns `[]` without raising. (You may need to add a `path` parameter to those functions — that refactor is the lesson.)
5. **Bug hunt via tests.** This function is subtly wrong: `def median(nums): return sorted(nums)[len(nums) // 2]`. Write tests for odd-length lists, even-length lists, single elements, and empty input. Let the failures tell you exactly what's broken, then fix the function until the suite is green.
