# Chapter 14: Decorators & Closures

## Overview

You've already *used* a decorator: `@property` in the OOP chapter. Flask (Chapter 18) is practically built on them: `@app.route("/")`. Pytest uses them for fixtures and parametrization. Decorators look magical, but they rest on two ideas you're ready for: **functions are values**, and **functions can be defined inside other functions and remember their surroundings** (closures).

This chapter demystifies the `@` syntax so that when you see `@app.route`, `@login_required`, or `@functools.lru_cache`, you know exactly what's happening — and can write your own.

## Definitions & Explanations

**Functions are first-class values** — a function is an object like any other: you can assign it to a variable, put it in a list or dict, pass it as an argument (you did this with `key=len` in sorting!), and return it from another function. `shout` (no parentheses) is the function itself; `shout()` is the result of calling it.

**Higher-order function** — a function that takes a function as an argument and/or returns one. `sorted(..., key=...)`, `map`, and every decorator are higher-order.

**Nested function** — a `def` inside a `def`. The inner function is local to the outer one, created fresh on each call.

**Closure** — when an inner function *uses variables from the enclosing function* and outlives it, those variables travel along, captured inside the inner function. The inner function "closes over" them:

```python
def make_multiplier(factor):
    def multiply(x):
        return x * factor      # `factor` captured from the enclosing scope
    return multiply

double = make_multiplier(2)    # the 2 lives on inside `double`
```

This is the **E** in the LEGB scope rule from Chapter 6 (Local → **Enclosing** → Global → Built-in). To *rebind* (not just read) an enclosing variable from inside, declare `nonlocal name`.

**Decorator** — a function that takes a function and returns a (usually enhanced) replacement. The `@` syntax is pure shorthand:

```python
@timer
def work(): ...
# is EXACTLY equivalent to:
def work(): ...
work = timer(work)
```

The standard decorator recipe:

```python
import functools

def my_decorator(func):
    @functools.wraps(func)               # preserve func's name & docstring
    def wrapper(*args, **kwargs):        # accept anything, pass it through
        # ... do something BEFORE ...
        result = func(*args, **kwargs)   # call the original
        # ... do something AFTER ...
        return result                    # pass the return value through
    return wrapper
```

Memorize this shape. Every logging, timing, caching, authentication, and retry decorator is a variation on it.

**`functools.wraps`** — a small decorator applied to the wrapper that copies the original function's `__name__`, docstring, etc. onto it. Without it, every decorated function reports its name as `wrapper`, which wrecks debugging and tooling. Always include it.

**Decorators with arguments** — `@retry(times=3)` requires one more layer: a *decorator factory* — a function that takes the arguments and returns the actual decorator. Three levels of nesting; see the example below.

**Useful ready-made decorators:**

- `@functools.lru_cache` — memoizes a function: repeated calls with the same arguments return instantly from cache.
- `@property`, `@staticmethod`, `@classmethod` — the class-related trio from Chapter 13.
- `@app.route(...)` — Flask registering your function as the handler for a URL.

## Code Examples

### Functions as values (warm-up)

```python
def shout(text: str) -> str:
    return text.upper() + "!"

def whisper(text: str) -> str:
    return text.lower() + "..."

speak = shout                      # assign the function itself — no ()
print(speak("hello"))              # HELLO!

styles = {"loud": shout, "soft": whisper}      # functions in a dict
choice = "soft"
print(styles[choice]("Hello There"))           # hello there...

def apply_twice(func, value):      # a higher-order function
    return func(func(value))

print(apply_twice(shout, "hi"))    # HI!!
```

### Closures

```python
def make_counter():
    count = 0
    def increment():
        nonlocal count             # rebind the enclosing variable
        count += 1
        return count
    return increment

clicks = make_counter()
print(clicks(), clicks(), clicks())    # 1 2 3

other = make_counter()                 # a fresh, independent closure
print(other())                         # 1 — separate `count`

def make_greeter(greeting: str):
    def greet(name: str) -> str:
        return f"{greeting}, {name}!"
    return greet

hello = make_greeter("Hello")
hola = make_greeter("Hola")
print(hello("Ada"), hola("Ada"))       # Hello, Ada! Hola, Ada!
```

### Your first decorator, step by step

```python
import functools

def announce(func):
    """Decorator: print before and after the wrapped call."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"-> calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"<- {func.__name__} returned {result!r}")
        return result
    return wrapper

@announce
def add(a, b):
    """Add two numbers."""
    return a + b

total = add(2, 3)
# -> calling add
# <- add returned 5
print(total)                # 5
print(add.__name__)         # 'add' — thanks to functools.wraps
```

### A genuinely useful decorator: timing

```python
import functools
import time

def timer(func):
    """Report how long the wrapped function took."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@timer
def slow_sum(n):
    return sum(i * i for i in range(n))

slow_sum(1_000_000)         # slow_sum took 0.06...s
```

### A decorator with arguments (three layers)

```python
import functools

def repeat(times):                      # layer 1: takes the ARGUMENT, returns a decorator
    def decorator(func):                # layer 2: takes the FUNCTION, returns the wrapper
        @functools.wraps(func)
        def wrapper(*args, **kwargs):   # layer 3: the actual replacement
            last = None
            for _ in range(times):
                last = func(*args, **kwargs)
            return last
        return wrapper
    return decorator

@repeat(times=3)
def beep():
    print("beep!")

beep()      # beep! beep! beep!
# Desugared: beep = repeat(times=3)(beep)
```

### Caching with the standard library

```python
import functools

@functools.lru_cache(maxsize=None)
def fib(n):
    """Naive recursion — instant with caching, glacial without."""
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(80))              # 23416728348467685 — immediate
print(fib.cache_info())     # hits/misses statistics
```

### Stacking decorators

```python
@timer
@announce
def greet(name):
    return f"Hi, {name}"

greet("Sam")
# Applied bottom-up: greet = timer(announce(greet))
# so timer wraps announce's wrapper, which wraps greet.
```

## Common Pitfalls

**1. Decorating with a call — or forgetting one**

```python
@timer        # correct for a plain decorator (no parentheses)
@repeat(3)    # correct for a decorator FACTORY (parentheses required)

@timer()      # WRONG — timer isn't a factory; TypeError
@repeat       # WRONG — repeat expects `times`, gets your function instead; breaks weirdly
```

Rule: match the decorator's design. Factories are called; plain decorators are not.

**2. Losing the return value**

```python
def wrapper(*args, **kwargs):
    func(*args, **kwargs)          # computed... and dropped!
    # forgot: return
```

Every decorated function silently starts returning `None`. Always `return func(...)`'s result from the wrapper.

**3. Omitting `*args, **kwargs`** — a wrapper defined as `def wrapper():` only fits zero-argument functions; the decorator breaks on anything else. The pass-through signature makes it universal.

**4. Skipping `functools.wraps`** — decorated functions all claim to be named `wrapper`, docstrings vanish, and debugging tools mislead you. One line prevents it.

**5. `nonlocal` vs. missing it**

```python
def make_counter():
    count = 0
    def increment():
        count += 1        # UnboundLocalError — assignment makes count local
        return count
    return increment
```

Reading an enclosing variable is automatic; *rebinding* one requires `nonlocal`. (Mutating a captured *list or dict* needs no `nonlocal` — you're not rebinding the name.)

**6. Late binding in loops**

```python
makers = [lambda x: x * i for i in range(3)]
print([m(10) for m in makers])       # [20, 20, 20] — all captured the SAME i (final value)

makers = [lambda x, i=i: x * i for i in range(3)]   # default arg freezes each i
print([m(10) for m in makers])       # [0, 10, 20]
```

**7. Decorator soup** — decorators shine for *cross-cutting* concerns (logging, timing, auth, caching). Business logic hidden inside decorators is hard to follow and test. When in doubt, a plain function call is clearer.

## Practice Exercises

1. **Closure bank.** Write `make_account(initial_balance)` returning two closures, `deposit(amount)` and `get_balance()`, sharing hidden state — no classes allowed. Demonstrate two independent accounts.
2. **`@debug` decorator.** Write a decorator that prints the function's name, its positional/keyword arguments, and its return value each call — with `functools.wraps` and full pass-through. Apply it to three differently-shaped functions to prove it's universal.
3. **`@validate_positive`.** Write a decorator that raises `ValueError` if *any* positional argument is a negative number, before calling the function. Apply it to `area(w, h)` and `withdraw(balance, amount)`.
4. **`@retry(times, delay)`.** Build a decorator factory that re-calls the function when it raises an exception, up to `times` attempts with `time.sleep(delay)` between, re-raising the final failure. Test it on a function that fails randomly (`random.random() < 0.7`).
5. **Trace the sugar.** For a stacked pair `@d1` over `@d2` on function `f`, write out the desugared assignment form, then verify your understanding by writing two trivial printing decorators and predicting the exact output order of a call *before* running it.
