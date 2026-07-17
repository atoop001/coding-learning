# Chapter 12: Error Handling & Exceptions

## Overview

Things go wrong: users type "abc" where a number belongs, files are missing, networks drop, and division hits zero. Unhandled, each of these **crashes** your program with a traceback. Python's exception system lets you *anticipate* failures and respond gracefully — retry, substitute a default, show a friendly message — instead of dying.

You've seen `try/except` briefly in earlier chapters; now we cover it thoroughly: the exception hierarchy, `try/except/else/finally`, raising your own exceptions, custom exception classes, and — just as important — the judgment of *when* to catch and when to let a crash happen.

## Definitions & Explanations

**Exception** — an object representing an error, "raised" when something goes wrong. If nothing catches it, Python prints a **traceback** (the chain of calls that led to the failure — read it *bottom-up*: the last line names the exception and message, the lines above show where) and exits.

**Common built-in exceptions you will actually meet:**

| Exception | Raised when... |
|---|---|
| `ValueError` | right type, bad value: `int("abc")` |
| `TypeError` | wrong type entirely: `"a" + 1` |
| `KeyError` | missing dict key: `d["nope"]` |
| `IndexError` | sequence index out of range |
| `FileNotFoundError` | opening a nonexistent file for reading |
| `ZeroDivisionError` | `x / 0` |
| `AttributeError` | `thing.method` that doesn't exist |
| `NameError` | using an undefined variable |
| `KeyboardInterrupt` | user pressed Ctrl+C |

Exceptions form a **hierarchy**: e.g. `FileNotFoundError` is a subclass of `OSError`; almost everything descends from `Exception`. Catching a parent class catches all its children.

**The full statement:**

```python
try:
    # code that might fail
except SomeError:
    # runs ONLY if that error was raised in the try block
except (OtherError, ThirdError) as e:
    # catch multiple types; `e` is the exception object (str(e) = its message)
else:
    # runs ONLY if the try block raised nothing (optional, less common)
finally:
    # runs ALWAYS — error or not, even through a return (cleanup code)
```

Flow: Python runs the `try` block; at the first exception it jumps to the *first matching* `except` clause (top-down), skips the rest of `try`, and continues after the statement.

**`raise`** — throw an exception yourself: `raise ValueError("age cannot be negative")`. Use it to reject bad inputs at the boundary of your functions rather than letting garbage flow deeper. A bare `raise` inside an `except` block re-throws the current exception (log-and-rethrow pattern).

**Custom exceptions** — subclass `Exception` to give your program's failures precise names:

```python
class InsufficientFundsError(Exception):
    """Raised when a withdrawal exceeds the balance."""
```

Callers can then catch *your* specific error without accidentally swallowing unrelated bugs. (Classes are Chapter 13; the two-line pattern above is all you need.)

**EAFP vs LBYL** — two styles: *Look Before You Leap* (`if path.exists(): open(...)`) vs. *Easier to Ask Forgiveness than Permission* (`try: open(...) except FileNotFoundError:`). Python culture favors **EAFP**: it avoids race conditions (the file could vanish between the check and the open) and handles all failure modes in one place.

**The golden rules of catching:**

1. Catch the **most specific** exception you can — `except ValueError:`, not `except Exception:`.
2. Wrap the **smallest** block of code that can fail — not your whole program.
3. Never silently swallow: an empty `except: pass` hides bugs from you. At minimum, log or explain.
4. Only catch what you can meaningfully **handle**. An unexpected crash with a clear traceback beats a program that limps along with corrupt state.

## Code Examples

### The essential input-validation loop, done properly

```python
def ask_int(prompt):
    """Keep asking until the user enters a valid integer."""
    while True:
        raw = input(prompt).strip()
        try:
            return int(raw)                 # success -> return escapes the loop
        except ValueError:
            print(f"'{raw}' is not a whole number — try again.")

age = ask_int("Your age: ")
print(f"Next year you'll be {age + 1}")
```

This replaces the fragile `.isdigit()` checks from earlier chapters (which reject negatives, for instance).

### Multiple except clauses and the exception object

```python
def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        print("Cannot divide by zero — returning None")
        return None
    except TypeError as e:
        print(f"Bad input types: {e}")
        return None

print(safe_divide(10, 2))       # 5.0
print(safe_divide(10, 0))       # message, then None
print(safe_divide(10, "x"))     # message, then None
```

### try/except/else/finally with a file

```python
def load_settings(path):
    """Return settings lines, or [] on a fresh install."""
    try:
        f = open(path, encoding="utf-8")
    except FileNotFoundError:
        print("No settings file yet — using defaults.")
        return []
    else:
        # only runs if open() succeeded; keeps try minimal
        with f:
            return [line.strip() for line in f if line.strip()]

def process_with_cleanup(items):
    try:
        for item in items:
            print(100 / item)
    except ZeroDivisionError:
        print("hit a zero!")
    finally:
        print("cleanup runs no matter what")   # closing connections, etc.

process_with_cleanup([4, 5, 0, 2])
```

### Raising your own errors

```python
def set_price(product, price):
    """Reject nonsense at the boundary."""
    if not isinstance(price, (int, float)):
        raise TypeError(f"price must be a number, got {type(price).__name__}")
    if price < 0:
        raise ValueError(f"price cannot be negative, got {price}")
    product["price"] = float(price)

item = {"name": "widget"}
set_price(item, 9.99)         # fine
try:
    set_price(item, -5)
except ValueError as e:
    print(f"Rejected: {e}")   # Rejected: price cannot be negative, got -5
```

### Custom exceptions in a mini-domain

```python
class BankError(Exception):
    """Base class for our banking failures."""

class InsufficientFundsError(BankError):
    """Withdrawal larger than the balance."""

def withdraw(balance, amount):
    if amount <= 0:
        raise ValueError("amount must be positive")
    if amount > balance:
        raise InsufficientFundsError(
            f"tried to withdraw {amount}, balance is only {balance}"
        )
    return balance - amount

balance = 100
for attempt in (30, 500, -3):
    try:
        balance = withdraw(balance, attempt)
        print(f"OK — new balance {balance}")
    except InsufficientFundsError as e:
        print(f"Declined: {e}")
    except ValueError as e:
        print(f"Bad request: {e}")
```

### Reading the traceback — a worked example

```
Traceback (most recent call last):
  File "app.py", line 12, in <module>
    total = checkout(cart)
  File "app.py", line 7, in checkout
    return sum(item["price"] for item in cart)
  File "app.py", line 7, in <genexpr>
    return sum(item["price"] for item in cart)
KeyError: 'price'
```

Read bottom-up: a `KeyError: 'price'` — some cart item lacks a `"price"` key — occurring inside `checkout`, which was called from line 12. The traceback is a map straight to the bug; never skip reading it.

## Common Pitfalls

**1. The bare / blanket except**

```python
try:
    result = compute(data)
except:                    # catches EVERYTHING — including typos in compute(),
    result = 0             # Ctrl+C, and bugs you desperately need to see
```

```python
try:
    result = compute(data)
except ZeroDivisionError:  # RIGHT: name the failure you expect
    result = 0
```

If you truly must catch broadly (e.g. a server loop that must not die), catch `Exception`, log `e`, and keep going — never `pass` silently.

**2. Wrapping too much in one try**

```python
try:
    name = input("Name: ")
    age = int(input("Age: "))
    profile = build_profile(name, age)
    save(profile)
except ValueError:
    print("Bad age")       # but a ValueError from build_profile or save is ALSO
                           # swallowed and mislabeled — wrap only int(...)
```

**3. Using exceptions for ordinary flow control** — don't `raise` to exit loops or signal normal outcomes ("not found" from a search is often better as a `None` return). Exceptions are for *exceptional* situations.

**4. Catching and losing the evidence**

```python
except FileNotFoundError:
    print("error")                       # WHICH file? WHY?
except FileNotFoundError as e:
    print(f"Could not open {e.filename}: {e}")   # actionable
```

**5. Forgetting exceptions skip the rest of `try`** — after the failing line, no further lines of the `try` block run. Variables assigned below the failure point don't exist in the `except` block.

**6. `finally` overriding returns** — a `return` inside `finally` silently discards both the `try`'s return value *and* any in-flight exception. Don't return from `finally`.

**7. Validating with the wrong tool** — `.isdigit()` fails for `"-5"` and `"3.14"`; `try: float(raw)` is the robust check. EAFP wins for parsing.

## Practice Exercises

1. **Bulletproof calculator.** Ask for two numbers and an operator (`+ - * /`) and print the result. It must survive: non-numeric input (re-ask), unknown operators (re-ask), and division by zero (friendly message) — crash-free through any input you can throw at it.
2. **Safe config reader.** Write `read_config(path)` returning a dict from lines like `key=value` in a file. Handle: missing file (return `{}`), lines without `=` (skip with a warning that includes the line number), duplicate keys (last wins). Create test files for each case.
3. **Retry wrapper.** Write `ask_float_with_retries(prompt, max_tries=3)` that returns the parsed float or raises a `ValueError("too many bad attempts")` after the limit. Have calling code catch that and exit politely.
4. **Custom exception design.** Build `parse_age(text)` that raises `AgeFormatError` (custom) for non-numbers and `AgeRangeError` (custom) for values outside 0–130, both subclassing a shared `AgeError`. Write a loop demonstrating catching each specifically, then a version catching the shared base.
5. **Traceback forensics.** Deliberately write three tiny buggy scripts producing a `KeyError`, an `IndexError`, and an `AttributeError` at least two function calls deep. For each, run it, and add a comment at the top of the file explaining — from the traceback alone — the file, line, and cause. Then fix them.
