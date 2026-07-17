# Chapter 9: Comprehensions

## Overview

You now know the build-a-list loop by heart: create an empty list, loop, `append`. Python offers a compact, expressive shorthand for exactly that pattern: the **comprehension**. One line replaces four, and — once your eyes adjust — it *reads better*, because it says what the result *is* rather than how to assemble it.

Comprehensions exist for lists, dicts, sets, and (as a preview of Chapter 15) generators. They are everywhere in professional Python code and in code you'll read online, so fluency matters even if you write the loop form at first.

## Definitions & Explanations

**List comprehension** — the general shape:

```python
[expression for item in iterable]                 # transform every item
[expression for item in iterable if condition]    # transform, keeping only some
```

Read it aloud as: "a list of *expression*, for each *item* in *iterable* (if *condition*)". It is exactly equivalent to:

```python
result = []
for item in iterable:
    if condition:
        result.append(expression)
```

**The three roles:**

1. **expression** — what each output element looks like (`n * 2`, `name.upper()`, `(x, y)`).
2. **for clause** — where items come from.
3. **optional `if` filter** — which items survive.

**Filtering vs. transforming** — `[x for x in nums if x > 0]` filters (expression is just `x`); `[x * 2 for x in nums]` transforms; `[x * 2 for x in nums if x > 0]` does both.

**Conditional expression inside** — to transform *differently* per item (rather than drop items), the ternary goes in the *expression* slot, before the `for`:

```python
["even" if n % 2 == 0 else "odd" for n in nums]
```

Rule of thumb: `if` **after** the `for` = filter (may shrink the list); `if/else` **before** the `for` = per-item choice (same length).

**Dict comprehension** — braces with a `key: value` expression:

```python
{name: len(name) for name in names}
```

**Set comprehension** — braces without a colon: `{n % 10 for n in nums}` (unique last digits).

**Nested comprehensions** — a second `for` clause flattens or combines: `[x * y for x in rows for y in cols]` behaves like nested loops (left clause is the *outer* loop). Legal, but readability drops fast — two `for`s is the sensible maximum.

**Generator expression (preview)** — same syntax with parentheses, `(x * x for x in nums)`, produces values lazily one at a time instead of building a list. When feeding directly into a function, the parentheses merge: `sum(x * x for x in nums)`. Full story in Chapter 15.

**When NOT to use a comprehension** — when the body needs multiple statements, complex error handling, or side effects (like printing); when it wouldn't fit readably on one or two lines; or when you're calling a function just for its effect. Comprehensions are for *building collections*, not for general looping.

## Code Examples

### From loop to comprehension

```python
# The loop you already know:
squares = []
for n in range(1, 11):
    squares.append(n * n)

# The comprehension — identical result:
squares = [n * n for n in range(1, 11)]
print(squares)      # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

### Transforming

```python
names = ["ada", "grace", "linus"]

capitalized = [name.title() for name in names]
print(capitalized)                       # ['Ada', 'Grace', 'Linus']

lengths = [len(name) for name in names]
print(lengths)                           # [3, 5, 5]

# Parsing a CSV-ish line into floats — very common in file work
line = "12.5, 7.25, 40.0, 9.99"
amounts = [float(part) for part in line.split(",")]
print(sum(amounts))                      # 69.74
```

### Filtering

```python
temps = [18, 25, 31, 22, 35, 15, 28]

hot_days = [t for t in temps if t >= 30]
print(hot_days)                          # [31, 35]

words = ["apple", "sky", "banana", "arc", "avocado"]
a_words = [w for w in words if w.startswith("a")]
print(a_words)                           # ['apple', 'arc', 'avocado']

# Filter AND transform together
loud_a_words = [w.upper() for w in words if w.startswith("a")]
print(loud_a_words)                      # ['APPLE', 'ARC', 'AVOCADO']
```

### If/else in the expression

```python
nums = [4, 7, 10, 13]

labels = ["even" if n % 2 == 0 else "odd" for n in nums]
print(labels)                            # ['even', 'odd', 'even', 'odd']

# Cap values at 10 — same length in, same length out
capped = [n if n <= 10 else 10 for n in nums]
print(capped)                            # [4, 7, 10, 10]
```

### Dict and set comprehensions

```python
fruits = ["apple", "banana", "cherry"]

name_lengths = {fruit: len(fruit) for fruit in fruits}
print(name_lengths)                      # {'apple': 5, 'banana': 6, 'cherry': 6}

# Invert a dict (values must be unique for this to be safe)
codes = {"US": "dollar", "JP": "yen", "GB": "pound"}
by_currency = {currency: country for country, currency in codes.items()}
print(by_currency)                       # {'dollar': 'US', ...}

# Filter a dict
inventory = {"apples": 12, "bananas": 0, "cherries": 45, "dates": 0}
in_stock = {k: v for k, v in inventory.items() if v > 0}
print(in_stock)                          # {'apples': 12, 'cherries': 45}

# Set comprehension: unique word lengths
sentence = "the cat sat on the mat with a rat"
print({len(w) for w in sentence.split()})    # {1, 2, 3, 4}
```

### Generator expressions feeding functions

```python
nums = [3, -1, 4, -1, 5, -9, 2, 6]

total_of_squares = sum(n * n for n in nums)        # no intermediate list built
any_negative = any(n < 0 for n in nums)            # True — stops at the first hit
all_small = all(abs(n) < 10 for n in nums)         # True
print(total_of_squares, any_negative, all_small)
```

### Realistic combo: cleaning messy data

```python
raw_emails = ["  Ana@Example.com ", "", "BEN@site.ORG", "  ", "cleo@mail.com\n"]

# strip, lowercase, drop empties — one readable line
emails = [e.strip().lower() for e in raw_emails if e.strip()]
print(emails)     # ['ana@example.com', 'ben@site.org', 'cleo@mail.com']

# grid construction done right (fixes Chapter 7's [[0]*3]*3 trap)
grid = [[0 for _ in range(3)] for _ in range(3)]
grid[0][0] = 9
print(grid)       # [[9, 0, 0], [0, 0, 0], [0, 0, 0]] — rows are independent
```

(`_` is the conventional name for a loop variable you don't use.)

## Common Pitfalls

**1. Putting the filter `if` in the wrong place**

```python
# labels = [n for n in nums if n % 2 == 0 else "odd"]   # SyntaxError!
# filter-if goes AFTER for and has no else:
evens = [n for n in nums if n % 2 == 0]
# if/ELSE belongs BEFORE the for, in the expression:
labels = [n if n % 2 == 0 else "odd" for n in nums]
```

**2. Comprehensions with side effects**

```python
[print(x) for x in items]     # works, but builds a useless list of Nones
for x in items:               # RIGHT: a plain loop for actions
    print(x)
```

**3. Overstuffed one-liners**

```python
# Legal. Also unreadable:
result = [f(x) if g(x) else h(x) for x in data if x.ok and not x.stale for y in x.parts]
```

If you have to squint, write the loop. Readability beats cleverness — your future self and your interviewers agree.

**4. Reusing the loop variable name**

```python
n = 100
squares = [n * n for n in range(5)]
print(n)     # 100 — comprehension variables are scoped INSIDE (Python 3), so fine —
             # but shadowing still confuses readers; pick distinct names.
```

**5. Building a list only to immediately reduce it**

```python
total = sum([x * x for x in big_list])    # allocates a full throwaway list
total = sum(x * x for x in big_list)      # generator expression: same result, no list
```

**6. Dict comprehension with colliding keys** — later pairs silently overwrite earlier ones: `{len(w): w for w in words}` keeps only the *last* word of each length. If collisions matter, use the grouping pattern from Chapter 8 instead.

## Practice Exercises

1. **Rewrite drill.** Convert each of these loop snippets into a comprehension: (a) squares of 1–20; (b) only the multiples of 3 from 1–50; (c) the lengths of each word in a sentence; (d) the words of a sentence longer than 3 letters, uppercased.
2. **Reverse drill.** Take `[w[0] for w in words if w]` and `{c: ord(c) for c in "python"}` and expand each back into an explicit `for` loop with `append`/assignment. (Being able to translate both directions proves you understand them.)
3. **Price adjustments.** Given `prices = {"laptop": 900, "mouse": 25, "monitor": 220, "cable": 8}`, build (a) a new dict with 10% off everything over 100, other prices unchanged; (b) a list of names of items under 50; (c) a set of all price digits used (e.g. `{"9","0","2","5","8"}` as ints or strs — your choice).
4. **Matrix column.** Given a 3×3 nested list, use a comprehension to extract the middle column as a list, and another to produce a flattened single list of all 9 values.
5. **One-line stats.** For `data = ["17", "x", "42", "", "8", "abc", "23"]`, use comprehensions (plus `str.isdigit`) to compute the sum of the values that are valid integers, the count of invalid entries, and whether *any* value exceeds 40.
