# Chapter 15: Generators & Iterators

## Overview

Every `for` loop you've written runs on a hidden machine called the **iterator protocol** — the same machinery that lets you loop over lists, strings, dicts, files, and `range`. Understanding it explains long-standing mysteries (why can `range(10**12)` exist without eating your RAM? why can a file be looped line-by-line?), and unlocks **generators**: functions that produce values one at a time, on demand, using the `yield` keyword.

Generators are how Python handles data too big for memory, infinite sequences, and pipelines — and they're the foundation under comprehensions' generator-expression cousin from Chapter 9.

## Definitions & Explanations

**Iterable** — anything you can loop over: lists, tuples, strings, dicts, sets, files, ranges, generators. Technically: anything `iter()` accepts.

**Iterator** — the object that actually produces items, one per call to `next()`. It remembers its position. When exhausted, it raises `StopIteration`. Key facts:

- `iter(iterable)` gets an iterator; `next(iterator)` gets the next item.
- A `for` loop is sugar for exactly this: call `iter()`, call `next()` repeatedly, stop cleanly at `StopIteration`.
- Iterators are **one-shot**: once exhausted, they stay empty. (Lists are re-iterable because each loop asks for a *fresh* iterator; a generator *is* its own iterator and can't rewind.)

**Lazy evaluation** — producing values only when asked, instead of building everything up front. `range(1_000_000_000)` stores three numbers (start, stop, step), not a billion. Files yield one line at a time. Laziness = tiny memory footprint + ability to represent infinite sequences.

**Generator function** — a function containing `yield`. Calling it runs *none* of its body; it returns a **generator object** (an iterator). Each `next()` runs the body to the next `yield`, hands out that value, and *freezes* — local variables and position preserved — until the next request. When the function ends, `StopIteration` is raised automatically.

`yield` vs `return`: `return` ends a function delivering one value; `yield` pauses it, delivering one of possibly many.

**Generator expression** — comprehension syntax with parentheses: `(x * x for x in nums)`. Lazy, one-shot, memory-flat. Use instead of a list comprehension whenever you only iterate the result once (especially feeding `sum`, `max`, `any`, `all`, `sorted`, `" ".join`).

**Pipelines** — generators compose: one generator's output feeds the next, and *nothing happens* until something at the end actually pulls values. This is the shape of log processing, ETL, and data cleaning at scale.

**Standard tools that return lazy iterators** (not lists!) in Python 3: `map`, `filter`, `zip`, `enumerate`, `reversed`, `dict.keys/values/items`, and the `itertools` module (`count`, `islice`, `chain`, `cycle`, ...). Wrap in `list(...)` when you truly need a list.

**When to use what:**

- Need to index, re-iterate, know `len`, or sort? → **list**.
- Iterate once, possibly huge/infinite, or streaming from a source? → **generator**.

## Code Examples

### The protocol, exposed

```python
letters = ["a", "b", "c"]

it = iter(letters)          # get an iterator from the iterable
print(next(it))             # a
print(next(it))             # b
print(next(it))             # c
# next(it)                  # StopIteration!

# A for loop does precisely the above, catching StopIteration for you:
for ch in letters:
    print(ch)
```

### Your first generator function

```python
from typing import Iterator

def countdown(n: int) -> Iterator[int]:
    """Yield n, n-1, ..., 1."""
    print("(starting)")             # runs on FIRST next(), not on call!
    while n > 0:
        yield n                     # pause here, hand out n
        n -= 1
    print("(done)")                 # runs when exhausted

gen = countdown(3)                  # nothing printed yet — body hasn't started
print(type(gen))                    # <class 'generator'>

print(next(gen))                    # (starting)  then 3
print(next(gen))                    # 2
for remaining in gen:               # generators plug into for loops directly
    print(remaining)                # 1   then (done)
```

### Memory: list vs generator

```python
import sys

as_list = [x * x for x in range(1_000_000)]
as_gen = (x * x for x in range(1_000_000))

print(sys.getsizeof(as_list))       # ~8,000,000+ bytes
print(sys.getsizeof(as_gen))        # ~200 bytes — it computes on demand

print(sum(as_gen))                  # totals fine without ever storing the squares
```

### Infinite generators + islice

```python
import itertools
from typing import Iterator

def fibonacci() -> Iterator[int]:
    """Yield Fibonacci numbers forever."""
    a, b = 0, 1
    while True:                     # infinite is FINE — values come only when pulled
        yield a
        a, b = b, a + b

# Take just the first 10 (never loop an infinite generator without a limit!)
first_ten = list(itertools.islice(fibonacci(), 10))
print(first_ten)                    # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

# First Fibonacci over a million:
for f in fibonacci():
    if f > 1_000_000:
        print(f)                    # 1346269
        break
```

### A realistic pipeline: log processing

```python
from typing import Iterator

def read_lines(path: str) -> Iterator[str]:
    """Yield stripped, non-empty lines — one at a time, any file size."""
    with open(path, encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if line:
                yield line

def errors_only(lines):
    """Pass through only ERROR lines."""
    for line in lines:
        if "ERROR" in line:
            yield line

def with_line_numbers(lines):
    for i, line in enumerate(lines, 1):
        yield f"[{i}] {line}"

# Compose: nothing is read until the for loop pulls
if __name__ == "__main__":
    pipeline = with_line_numbers(errors_only(read_lines("app.log")))
    for entry in pipeline:
        print(entry)
```

Each line flows through all three stages before the next line is even read — constant memory whether the log is 1 KB or 100 GB.

### Generator expressions in daily use

```python
scores = [82, 95, 67, 88, 74]

print(sum(s for s in scores if s >= 80))      # 265 — no throwaway list
print(max(len(w) for w in "lazy is thrifty".split()))   # 7
print(any(s < 70 for s in scores))            # True — stops at the first hit

# Cleaning + joining in one pass
raw = ["  Ada ", "", " Grace", "  "]
print(", ".join(name.strip() for name in raw if name.strip()))   # Ada, Grace
```

### `yield from` — delegating

```python
def flatten(nested):
    """Yield every item from a list of lists."""
    for sublist in nested:
        yield from sublist          # shorthand for: for x in sublist: yield x

print(list(flatten([[1, 2], [3], [4, 5, 6]])))   # [1, 2, 3, 4, 5, 6]
```

## Common Pitfalls

**1. Exhaustion — the big one**

```python
gen = (x * x for x in range(5))
print(list(gen))        # [0, 1, 4, 9, 16]
print(list(gen))        # [] — spent! Generators are one-shot.

nums = (x for x in [3, 1, 2])
print(max(nums))        # 3
# print(min(nums))      # ValueError: min() arg is an empty sequence — already consumed!
```

Need multiple passes? Materialize once: `values = list(gen)`.

**2. Calling a generator function and expecting values**

```python
def gen():
    yield 1
print(gen())            # <generator object ...> — not 1!
print(list(gen()))      # [1]
print(next(gen()))      # 1 — but note: EACH gen() call makes a FRESH generator
```

**3. Mixing `return value` semantics** — in a generator, `return` just *stops* iteration (its value hides inside StopIteration; normal loops never see it). Don't expect `return` to deliver data to a `for` loop.

**4. No `len()`, no indexing, no slicing**

```python
squares = (x * x for x in range(10))
# len(squares)          # TypeError
# squares[3]            # TypeError
# use itertools.islice for slicing, or list() if you need sequence powers
```

**5. Looping an infinite generator without an exit** — `for f in fibonacci(): print(f)` never ends. Pair infinite generators with `break`, `islice`, or a condition — *always* know your stopping rule.

**6. Laziness delays errors**

```python
def read(path):
    f = open(path, encoding="utf-8")     # doesn't run at call time!
    yield from f

lines = read("missing.txt")     # no error HERE...
# next(lines)                   # FileNotFoundError arrives here, far from the cause
```

Remember: a generator's body runs on *first pull*, so failures surface where you iterate, not where you construct. Validate inputs eagerly before the first `yield` if that matters.

**7. Using a generator where you'll need a list anyway** — if the next line is `list(...)`, or you need sorting/indexing/multiple passes, just build the list. Laziness is a tool, not a religion.

## Practice Exercises

1. **Protocol by hand.** Create `it = iter("abc")` and drive it with `next()` until `StopIteration`, catching that exception yourself with try/except and printing "exhausted". Then explain (comment) how this relates to what `for` does.
2. **`evens_up_to(n)`.** Write it as a generator function. Prove its laziness: create one for `n=10**12`, pull just five values with `next()`, and note that it returns instantly. Then use `sum` with `islice` to total the first 1000 even numbers.
3. **Chunker.** Write a generator `chunks(iterable, size)` yielding lists of `size` items (last chunk may be short). Test on a range and on a string. No reading the whole input up front — it must work on a generator input too.
4. **Pipeline practice.** Build three composable generators for a CSV-ish stream of `"name,amount"` strings: `parse(lines)` → yields `(name, float)` tuples, skipping malformed lines; `big_spenders(pairs, threshold)` → filters; `format_report(pairs)` → yields display strings. Wire them together over a list of sample lines, including some bad ones.
5. **Exhaustion detective.** Write a script that (a) demonstrates the double-`list()` exhaustion bug, (b) demonstrates the `max` then `min` crash, and (c) fixes both by materializing — each with a one-line comment explaining the behavior. Predict every output before running.
