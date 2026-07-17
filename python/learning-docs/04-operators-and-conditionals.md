# Chapter 4: Operators & Conditionals

## Overview

So far your programs have run straight from top to bottom. Real programs make **decisions**: charge tax or don't, accept the password or reject it, show the admin menu or the regular one. This chapter covers Python's operators (arithmetic, comparison, logical) and the `if` / `elif` / `else` statements that use them to branch your program's flow.

This is also where Python's signature feature appears: **indentation is syntax**. The visual structure of your code *is* its logical structure.

## Definitions & Explanations

### Arithmetic operators (recap and completion)

| Operator | Meaning | Example |
|---|---|---|
| `+` `-` `*` | add, subtract, multiply | `7 * 6` → `42` |
| `/` | true division — always a float | `7 / 2` → `3.5` |
| `//` | floor division — round down to whole | `7 // 2` → `3`, `-7 // 2` → `-4` |
| `%` | modulo — the remainder | `7 % 2` → `1` |
| `**` | exponent | `2 ** 10` → `1024` |

Precedence follows math class: `**` first, then `* / // %`, then `+ -`. Use parentheses whenever there's any doubt — they cost nothing and prevent bugs.

### Comparison operators

Each returns a `bool` (`True` or `False`):

`==` equal, `!=` not equal, `<` `<=` `>` `>=` as in math. Comparisons can chain: `0 <= x < 10` means "x is between 0 and 9" and reads exactly like math.

Strings compare alphabetically (really: by character code): `"apple" < "banana"` is `True`; beware `"Zebra" < "apple"` is also `True` because uppercase letters sort before lowercase.

### Logical operators

- `and` — `True` only if **both** sides are true
- `or` — `True` if **at least one** side is true
- `not` — flips a boolean

`and`/`or` **short-circuit**: `a and b` doesn't evaluate `b` if `a` is already false; `a or b` doesn't evaluate `b` if `a` is already true.

### Truthiness

Any value can stand in for a boolean. **Falsy**: `False`, `None`, `0`, `0.0`, `""`, `[]`, `{}`, `()`, `set()`. Everything else is **truthy**. So `if name:` means "if name is not empty" — a common, idiomatic pattern.

### The `if` statement

```python
if condition:
    # runs when condition is truthy
elif other_condition:
    # checked only if the first was falsy; you may have many elifs
else:
    # runs when nothing above matched
```

Rules:

- The colon `:` at the end of each condition line is mandatory.
- The **body must be indented** — 4 spaces is the universal convention. Everything at the same indentation belongs to the same block.
- `elif` and `else` are optional. Conditions are checked top-down; **only the first truthy branch runs**.

### Related tools

- **Conditional expression (ternary)**: `value_if_true if condition else value_if_false` — an `if` squeezed into an expression, e.g. `label = "even" if n % 2 == 0 else "odd"`.
- **`in` / `not in`**: membership tests — `"@" in email`, `choice in ("a", "b", "c")`.
- **`is` / `is not`**: identity — "are these the *same object*?" Use only for `None`: `if x is None:`. Use `==` for value equality everywhere else.
- **`match` statement** (Python 3.10+): structural switch-like branching; mentioned here for recognition, `if/elif` covers all beginner needs.

## Code Examples

### Basic branching

```python
# ticket.py
age = int(input("Age: "))

if age < 5:
    price = 0
elif age < 18:            # only reached if age >= 5
    price = 8
elif age < 65:
    price = 15
else:
    price = 10

print(f"Ticket price: ${price}")
```

Note how each `elif` implicitly carries the failure of all previous checks — no need to write `age >= 5 and age < 18`.

### Combining conditions

```python
username = input("Username: ").strip()
password = input("Password: ")

if username == "admin" and password == "hunter2":
    print("Welcome, administrator.")
elif username == "admin" or username == "root":
    print("Wrong password for a privileged account!")
else:
    print("Hello, regular user.")

# Membership test version of a multi-way or:
if username in ("admin", "root", "superuser"):
    print("(privileged account name)")
```

### Truthiness in real code

```python
name = input("Name (optional): ").strip()

if name:                      # truthy = non-empty
    greeting = f"Hello, {name}!"
else:
    greeting = "Hello, mysterious stranger!"
print(greeting)

# The ternary form of the same logic:
greeting = f"Hello, {name}!" if name else "Hello, mysterious stranger!"
```

### Nested conditions vs. flat conditions

```python
# Nested — sometimes necessary, but harder to read:
if logged_in:
    if is_admin:
        print("Admin panel")
    else:
        print("User dashboard")
else:
    print("Please log in")

# Often you can flatten with and:
if logged_in and is_admin:
    print("Admin panel")
elif logged_in:
    print("User dashboard")
else:
    print("Please log in")
```

### A realistic example: input validation

```python
# bmi.py — with light validation
raw_weight = input("Weight in kg: ").strip()
raw_height = input("Height in m: ").strip()

if not raw_weight or not raw_height:
    print("Both values are required.")
elif not raw_weight.replace(".", "", 1).isdigit():
    print("Weight must be a number.")
else:
    weight = float(raw_weight)
    height = float(raw_height)
    if height <= 0:
        print("Height must be positive.")
    else:
        bmi = weight / height ** 2
        category = (
            "underweight" if bmi < 18.5
            else "normal" if bmi < 25
            else "overweight" if bmi < 30
            else "obese"
        )
        print(f"BMI: {bmi:.1f} ({category})")
```

### Even/odd and divisibility — the `%` idiom

```python
n = int(input("A number: "))

if n % 2 == 0:
    print(f"{n} is even")
else:
    print(f"{n} is odd")

# FizzBuzz-style multi-condition (order matters!)
if n % 15 == 0:
    print("FizzBuzz")       # must check 15 BEFORE 3 or 5
elif n % 3 == 0:
    print("Fizz")
elif n % 5 == 0:
    print("Buzz")
else:
    print(n)
```

## Common Pitfalls

**1. `=` instead of `==` in a condition**

```python
# if score = 100:        # SyntaxError — assignment isn't allowed here
if score == 100:          # correct
    ...
```

**2. Indentation errors**

```python
if x > 0:
print("positive")         # IndentationError: expected an indented block
```

Also dangerous: *inconsistent* indentation silently changes meaning:

```python
if x > 0:
    print("positive")
    print("definitely")    # inside the if
print("done")              # OUTSIDE the if — runs every time. Is that intended?
```

**3. `x == 1 or 2` doesn't mean what it reads like**

```python
if choice == 1 or 2:          # ALWAYS true! Parsed as (choice == 1) or (2), and 2 is truthy
    ...
if choice == 1 or choice == 2:  # correct
    ...
if choice in (1, 2):            # correct and shorter
    ...
```

This is probably the single most common beginner logic bug.

**4. Comparing numbers as strings**

```python
guess = input("Guess: ")     # "9"
if guess > "10":             # string comparison: "9" > "10" is True (alphabetical!)
    ...
guess = int(input("Guess: "))   # FIX: convert first, then compare numbers
```

**5. Order of `elif` branches**

Branches are checked top-down; the first match wins. Checking `bmi < 30` before `bmi < 18.5` would classify everyone under 30 as the first category. Put the most specific / narrowest condition first.

**6. Using `is` for value comparison**

```python
a = 1000
b = 1000
print(a is b)     # may be False! `is` asks "same object?", not "same value?"
print(a == b)     # True — this is what you almost always want
# Reserve `is` for:  x is None / x is not None
```

**7. Redundant boolean comparison**

```python
if is_ready == True:      # works, but noisy
if is_ready:              # idiomatic
if not is_ready:          # instead of  == False
```

## Practice Exercises

1. **Grade mapper.** Read a score 0–100 and print the letter grade: 90+ → A, 80+ → B, 70+ → C, 60+ → D, otherwise F. Then add input validation: if the number is below 0 or above 100, print an error instead.
2. **Leap year.** A year is a leap year if it's divisible by 4, except years divisible by 100 — unless also divisible by 400. Read a year and print whether it's a leap year. Test with 2024 (yes), 1900 (no), 2000 (yes).
3. **Login gate.** Store a correct username and password in constants. Ask the user for both. Print different messages for: both correct; correct username but wrong password; unknown username. Normalize the username with `.strip().lower()` before comparing.
4. **Shipping calculator.** Free shipping for orders of $50+, $5 shipping for orders of $20–49.99, $10 below that — but membership (`ask yes/no`) always gives free shipping. Print the shipping cost using one `if/elif/else` chain with `and`/`or` where helpful.
5. **Predict, then verify.** Without running them, decide `True` or `False` for each, then check in the REPL: `3 == 3.0`, `"3" == 3`, `bool("False")`, `0.1 + 0.2 == 0.3`, `not ""`, `"b" in "abc" and 5 > 2`, `None == False`.
