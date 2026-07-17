# Chapter 6: Functions

## Overview

Functions are named, reusable blocks of code. They are *the* fundamental unit of organization in programming: instead of one long script, you build small, well-named pieces — `calculate_tax(...)`, `format_receipt(...)`, `is_valid_email(...)` — and compose them. Functions make code testable (Chapter 17), readable, and non-repetitive.

This chapter covers defining and calling functions, parameters vs. arguments, return values, default values, keyword arguments, `*args` / `**kwargs`, variable scope, and docstrings. Mastering this chapter is the biggest single step toward writing professional-looking Python.

## Definitions & Explanations

**Defining a function** — the `def` statement:

```python
def function_name(parameter1, parameter2):
    """Optional docstring describing what it does."""
    # body — indented
    return some_value      # optional
```

Defining a function does **not** run it. It runs when *called*: `function_name(arg1, arg2)`.

**Parameter vs. argument** — parameters are the names in the `def` line (placeholders); arguments are the actual values you pass when calling. In `def greet(name)` / `greet("Ada")`, `name` is the parameter, `"Ada"` is the argument.

**`return`** — immediately exits the function and hands a value back to the caller. A function with no `return` (or a bare `return`) gives back `None`. `return` and `print` are completely different: `return` delivers a value to *code*; `print` shows text to a *human*. You can return multiple values: `return total, count` (actually a tuple, unpacked as `t, c = f()`).

**Default parameter values** — `def greet(name, punctuation="!")` makes `punctuation` optional; callers who omit it get `"!"`.

**Positional vs. keyword arguments** — `move(2, 5)` matches arguments to parameters by position; `move(x=2, y=5)` matches by name. Keyword arguments can appear in any order and make calls self-documenting. Positional arguments must come before keyword ones in a call.

**`*args`** — collects any extra positional arguments into a tuple, letting a function accept a variable number: `def total(*numbers):`. The name `args` is convention; the `*` is the mechanism.

**`**kwargs`** — collects extra keyword arguments into a dictionary: `def configure(**options):`. Together, `def f(*args, **kwargs)` accepts anything — you'll see this signature constantly in libraries and in decorators (Chapter 14).

**Unpacking at the call site** — the same symbols in reverse: `f(*my_list)` spreads a list into separate positional arguments; `f(**my_dict)` spreads a dict into keyword arguments.

**Scope** — where a name is visible:

- Names assigned inside a function are **local** — they exist only during that call and vanish afterwards.
- Names assigned at the top level of a file are **global** (module-level).
- A function can *read* globals, but assigning to a name inside a function creates a new local (unless you declare `global name`, which you should almost never do — pass values in as arguments and hand results back with `return` instead).
- Lookup order is **LEGB**: Local → Enclosing (nested functions, Chapter 14) → Global → Built-in.

**Docstring** — a string literal as the first line of a function body, in triple quotes, describing purpose, parameters, and return value. Tools and editors display it; `help(my_function)` prints it.

**Type hints (preview)** — optional annotations like `def area(width: float, height: float) -> float:`. Python doesn't enforce them, but they document intent and power editor autocomplete. Use them once comfortable; they appear increasingly in this track.

## Code Examples

### Defining, calling, returning

```python
def greet(name):
    """Return a greeting string for the given name."""
    return f"Hello, {name}!"

message = greet("Ada")        # call it; capture the returned value
print(message)                # Hello, Ada!
print(greet("Grace"))         # functions are reusable — that's the point

def print_banner(text):
    """Print text framed by dashes. Returns None (it's a 'do something' function)."""
    line = "-" * (len(text) + 4)
    print(line)
    print(f"| {text} |")
    print(line)

print_banner("Menu")
```

### Multiple returns and early exit

```python
def describe_number(n):
    """Return a short description of an integer."""
    if n == 0:
        return "zero"            # early return: no elif/else gymnastics needed
    if n % 2 == 0:
        return "even"
    return "odd"

def divide(a, b):
    """Return (quotient, remainder) — two values at once."""
    return a // b, a % b

q, r = divide(17, 5)
print(f"17 / 5 = {q} remainder {r}")     # 17 / 5 = 3 remainder 2
```

### Defaults and keyword arguments

```python
def make_coffee(size="medium", milk=False, shots=1):
    """Build a coffee order description."""
    desc = f"{size} coffee, {shots} shot(s)"
    if milk:
        desc += ", with milk"
    return desc

print(make_coffee())                          # medium coffee, 1 shot(s)
print(make_coffee("large"))                   # positional for the first param
print(make_coffee(milk=True, size="small"))   # keywords: any order, very readable
print(make_coffee("large", shots=3))          # mix: positionals first, then keywords
```

### `*args` and `**kwargs`

```python
def average(*numbers):
    """Average any quantity of numbers."""
    if not numbers:                 # numbers is a tuple; empty tuple is falsy
        return 0.0
    return sum(numbers) / len(numbers)

print(average(3, 4, 5))             # 4.0
print(average(10))                  # 10.0

scores = [88, 92, 79]
print(average(*scores))             # unpack the list into 3 arguments

def build_profile(name, **details):
    """Combine a required name with arbitrary extra fields."""
    profile = {"name": name}
    profile.update(details)         # details is a dict of the extra keywords
    return profile

print(build_profile("Ada", role="engineer", city="London"))
# {'name': 'Ada', 'role': 'engineer', 'city': 'London'}
```

### Scope in action

```python
counter = 0                 # global

def increment_broken():
    # counter += 1          # UnboundLocalError — assignment makes `counter` local,
    pass                    # but you'd be reading it before assigning

def increment(value):
    """The right way: take input, return output."""
    return value + 1

counter = increment(counter)
print(counter)              # 1

def show():
    label = "inside"        # local — born and dies with this call
    print(label)

show()
# print(label)              # NameError: label doesn't exist out here
```

### Putting it together — small functions composing

```python
# tip_calculator.py

def get_positive_float(prompt):
    """Keep asking until the user enters a positive number."""
    while True:
        raw = input(prompt).strip()
        try:                                  # formally in Chapter 12
            value = float(raw)
            if value > 0:
                return value
        except ValueError:
            pass
        print("Please enter a positive number.")

def calculate_tip(bill, percent=18.0):
    """Return the tip amount for a bill."""
    return round(bill * percent / 100, 2)

def format_summary(bill, tip):
    """Return a printable multi-line summary."""
    return (f"Bill:  ${bill:>8.2f}\n"
            f"Tip:   ${tip:>8.2f}\n"
            f"Total: ${bill + tip:>8.2f}")

def main():
    bill = get_positive_float("Bill amount: $")
    tip = calculate_tip(bill)
    print(format_summary(bill, tip))

if __name__ == "__main__":     # explained fully in Chapter 10; means "run only as a script"
    main()
```

## Common Pitfalls

**1. Printing instead of returning**

```python
def add(a, b):
    print(a + b)          # shows the number, returns None

result = add(2, 3)        # prints 5...
print(result * 10)        # TypeError: None * 10

def add(a, b):
    return a + b          # FIXED: now the caller can use the value
```

**2. Forgetting the parentheses when calling**

```python
def roll():
    return 4

value = roll        # no parens: `value` is the function itself, not 4!
print(value)        # <function roll at 0x...>
value = roll()      # correct — calls it
```

**3. Mutable default arguments** — the classic Python gotcha

```python
def add_item(item, items=[]):      # WRONG: the [] is created ONCE, shared by all calls
    items.append(item)
    return items

print(add_item("a"))    # ['a']
print(add_item("b"))    # ['a', 'b']  — surprise! the list persisted

def add_item(item, items=None):    # RIGHT: the standard pattern
    if items is None:
        items = []
    items.append(item)
    return items
```

**4. Relying on globals instead of parameters**

```python
# Fragile — works only if a global `rate` happens to exist:
def tax(amount):
    return amount * rate

# Robust — everything the function needs comes in the front door:
def tax(amount, rate):
    return amount * rate
```

**5. Code after `return` never runs**

```python
def f(x):
    return x * 2
    print("done")      # dead code — unreachable
```

**6. Doing too much in one function**

A function named `process()` that reads input, validates it, computes, and prints is four functions wearing a trench coat. Aim for one clear job per function; if you struggle to name it, it's probably doing too much.

## Practice Exercises

1. **Temperature toolkit.** Write `c_to_f(celsius)` and `f_to_c(fahrenheit)`, each returning (not printing) the conversion. Then write a few lines that call both to verify `f_to_c(c_to_f(25))` gives back `25.0`.
2. **Password checker.** Write `is_strong(password)` returning `True` only if the password is at least 10 characters, contains a digit, and contains both upper- and lowercase letters. Use it in a loop that asks until a strong password is entered.
3. **Flexible max.** Write `largest(*numbers)` that returns the biggest of any number of arguments without using the built-in `max()`. Decide what to return when called with no arguments, and write a docstring documenting that choice.
4. **Order builder.** Write `create_order(item, quantity=1, **extras)` returning a dict describing the order, with any extras (like `gift_wrap=True`, `note="Happy birthday"`) merged in. Print a couple of differently shaped calls.
5. **Refactor drill.** Take your FizzBuzz from Chapter 5 and refactor it into `fizzbuzz_value(n)` (returns the string/number for one `n`) plus a loop that calls it. Then write a second loop that only prints the values for `n` where the result is exactly `"FizzBuzz"`.
