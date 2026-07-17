# Chapter 2: Variables & Data Types

## Overview

Programs work with data: user names, prices, scores, on/off flags. A **variable** is a name you attach to a piece of data so you can refer to it later, and every piece of data has a **type** that determines what you can do with it. Understanding types early prevents the most common class of beginner bugs — trying to do math on text, comparing things that can't sensibly be compared, or being surprised by how numbers behave.

This chapter covers Python's core scalar types (`int`, `float`, `str`, `bool`, `None`), how assignment works, naming conventions, and converting between types. Collections (lists, dicts, etc.) get their own chapters later.

## Definitions & Explanations

**Variable** — a name bound to a value. In Python you create one simply by assigning: `score = 10`. There is no separate declaration step and no type annotation required. The name is a label pointing at an object; it is *not* a box that contains the value (this distinction matters later with lists).

**Assignment (`=`)** — binds the name on the left to the value on the right. One `=` means "assign"; two (`==`) means "compare for equality" (Chapter 4).

**Dynamic typing** — the *value* has a type, not the variable. The same name may be rebound to values of different types over time (legal, but usually a bad idea for readability):

```python
x = 10        # x points to an int
x = "ten"     # now x points to a str — allowed, but confusing
```

**The core types:**

- `int` — whole numbers: `42`, `-7`, `0`. Python ints have *unlimited* size; `2 ** 1000` works fine.
- `float` — numbers with a decimal point: `3.14`, `-0.5`, `2.0`. Stored in binary floating point, which means tiny rounding quirks (see pitfalls).
- `str` — text (a *string* of characters): `"hello"`, `'a'`, `""` (empty). Strings get a full chapter next.
- `bool` — exactly two values: `True` and `False` (capitalized!). Used for decisions and conditions.
- `NoneType` — the single value `None`, meaning "no value / nothing here." Functions that don't explicitly return something return `None`.

**`type()`** — a built-in function that tells you the type of any value: `type(3.5)` → `<class 'float'>`. Great for REPL exploration.

**Type conversion (casting)** — creating a value of one type from another: `int("42")` → `42`, `str(3.14)` → `"3.14"`, `float(7)` → `7.0`. Conversion can fail: `int("hello")` raises a `ValueError`.

**Naming rules and conventions:**

- Names may contain letters, digits, and underscores, but cannot *start* with a digit: `player2` ✔, `2player` ✘.
- Names are case-sensitive: `total` and `Total` are different variables.
- Convention (PEP 8, Python's style guide): use `snake_case` for variables — lowercase words joined by underscores: `items_sold`, `max_retries`.
- `ALL_CAPS` is the convention for constants — values you don't intend to change: `MAX_PLAYERS = 4`. Python won't stop you from changing them; it's a signal to human readers.
- You cannot use Python's ~35 reserved keywords as names: `if`, `for`, `class`, `def`, `return`, `True`, `None`, etc.

**Truthiness** — every value can be interpreted as `True` or `False` in a condition. The "falsy" values are: `False`, `None`, `0`, `0.0`, `""` (empty string), and empty collections. Everything else is "truthy." This will matter a lot from Chapter 4 onward.

## Code Examples

### Basic assignment and printing

```python
# variables.py

age = 34                 # int
height_m = 1.75          # float
name = "Priya"           # str
is_member = True         # bool
middle_name = None       # "no value"

print(name, "is", age, "years old")
print(type(age))         # <class 'int'>
print(type(height_m))    # <class 'float'>
print(type(is_member))   # <class 'bool'>
```

### Reassignment and multiple assignment

```python
count = 0
count = count + 1        # read old value, add 1, rebind the name
count += 1               # shorthand for the same thing ("augmented assignment")
print(count)             # 2

# Assign several variables at once (tuple unpacking, more in Chapter 7)
x, y = 10, 20
print(x, y)              # 10 20

# Swap two variables — famously clean in Python
x, y = y, x
print(x, y)              # 20 10
```

### Numbers in practice

```python
# Ints are exact and unlimited
big = 2 ** 100
print(big)               # 1267650600228229401496703205376

# Floats appear whenever division or decimals are involved
print(10 / 4)            # 2.5   (/ ALWAYS gives a float, even 10 / 5 -> 2.0)
print(10 // 4)           # 2     (// is floor division: divide and round down)
print(10 % 4)            # 2     (% is the remainder / "modulo")

# Mixing int and float produces a float
subtotal = 19 + 0.99
print(subtotal)          # 19.99

# round() controls decimal places
price = 7.128
print(round(price, 2))   # 7.13
```

### Type conversion — the everyday pattern

```python
# input() always returns a str, so numeric input must be converted:
raw = input("How many tickets? ")     # user types: 3
tickets = int(raw)                    # "3" -> 3
total = tickets * 12.50
print("Total: $" + str(total))        # str(...) to glue onto text
# (f-strings, next chapter, make this last line much nicer)
```

### None and its proper use

```python
winner = None            # "we don't know yet"

# ... later, maybe:
winner = "Team A"

# Checking for None uses `is`, not ==  (convention + correctness)
if winner is None:
    print("No winner yet")
else:
    print("Winner:", winner)
```

### A realistic mini-example

```python
# receipt.py — compute a simple receipt

ITEM_NAME = "Notebook"       # constants in ALL_CAPS by convention
UNIT_PRICE = 4.25
TAX_RATE = 0.08

quantity = int(input("How many notebooks? "))

subtotal = quantity * UNIT_PRICE
tax = round(subtotal * TAX_RATE, 2)
total = round(subtotal + tax, 2)

print("Item:", ITEM_NAME)
print("Qty:", quantity)
print("Subtotal:", subtotal)
print("Tax:", tax)
print("Total:", total)
```

## Common Pitfalls

**1. Using a variable before assigning it**

```python
print(total)      # NameError: name 'total' is not defined
total = 100
```

Python runs top to bottom; a name must be assigned before it's used.

**2. Confusing `=` and `==`**

```python
# WRONG intent: this assigns, it doesn't compare
# if x = 5:            # SyntaxError

# RIGHT
if x == 5:
    print("x is five")
```

**3. Forgetting `input()` returns a string**

```python
age = input("Age? ")          # user types 20
print(age + 1)                # TypeError: can only concatenate str to str

# FIXED
age = int(input("Age? "))
print(age + 1)                # 21
```

**4. Float surprise**

```python
print(0.1 + 0.2)              # 0.30000000000000004  — not a bug, it's binary rounding
print(0.1 + 0.2 == 0.3)       # False!

# FIX for display: round it
print(round(0.1 + 0.2, 2))    # 0.3
# FIX for money in serious code: use the decimal module or work in cents (ints)
```

**5. Capitalizing booleans wrong or quoting them**

```python
# is_ready = true       # NameError — must be True
# is_ready = "True"     # this is a STRING, not a bool; it's truthy but wrong type
is_ready = True          # correct
```

**6. Shadowing built-in names**

```python
# Works, but now the built-in str() function is gone in this file:
str = "hello"
# str(42)              # TypeError: 'str' object is not callable
```

Avoid using `str`, `int`, `list`, `type`, `sum`, `max`, `min`, `input` as variable names.

## Practice Exercises

1. **Profile card.** Create variables for a person's name (str), age (int), height in meters (float), and whether they're a student (bool). Print each one along with its type using `type()`.
2. **Temperature converter.** Ask the user for a temperature in Fahrenheit with `input()`, convert it to a float, compute Celsius with `(f - 32) * 5 / 9`, and print the result rounded to 1 decimal place.
3. **Swap without a helper.** Set `a = "first"` and `b = "second"`, then swap them in a single line and print both to prove it worked.
4. **Minutes breakdown.** Given `total_minutes = 347`, use `//` and `%` to compute hours and leftover minutes, then print something like `347 minutes = 5 h 47 min`.
5. **Predict the types.** Without running them first, write down the type of each expression, then verify in the REPL: `10 / 2`, `10 // 3`, `"10" + "2"`, `int("7") * 2.0`, `5 == 5.0`, `None`.
