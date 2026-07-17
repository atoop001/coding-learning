# Chapter 7: Lists & Tuples

## Overview

Real data comes in groups: the lines of a file, the items in a cart, the scores of a class. Python's **list** is the workhorse collection — ordered, changeable, and able to hold anything. Its sibling the **tuple** is the same idea but frozen: once created, it can't change, which turns out to be a feature.

You've already brushed against lists in the loops chapter; now we cover them properly: creating, indexing, slicing, mutating, sorting, copying (and the copying trap that bites everyone), plus when to reach for a tuple instead.

## Definitions & Explanations

**List** — an ordered, mutable sequence, written with square brackets: `[1, 2, 3]`, `["a", "b"]`, `[]` (empty), or mixed `[1, "two", 3.0]` (legal, usually unwise). "Ordered" means items keep their positions; "mutable" means you can add, remove, and replace items in place.

**Indexing and slicing** — exactly like strings (Chapter 3): `items[0]`, `items[-1]`, `items[1:4]`, `items[::-1]`. Slicing a list returns a *new* list.

**Core list operations:**

| Operation | Effect |
|---|---|
| `items.append(x)` | add `x` at the end (most common) |
| `items.insert(i, x)` | insert `x` at index `i` |
| `items.extend(other)` | append all elements of another iterable |
| `items.remove(x)` | delete the *first* occurrence of value `x` (ValueError if absent) |
| `items.pop()` / `items.pop(i)` | remove & return last item / item at `i` |
| `del items[i]` | delete by index without returning |
| `items.index(x)` | position of first `x` (ValueError if absent) |
| `items.count(x)` | how many times `x` appears |
| `items.sort()` | sort **in place** (returns `None`!) |
| `sorted(items)` | return a **new** sorted list, original untouched |
| `items.reverse()` / `reversed(items)` | same in-place vs. new distinction |
| `items.clear()` | empty the list |

**Useful built-ins on sequences** — `len(items)`, `sum(numbers)`, `min(...)`, `max(...)`, `x in items` (membership), `list(...)` (convert any iterable to a list).

**References and aliasing** — a variable holds a *reference* to a list, not a copy. `b = a` makes both names point at the **same** list; changing one "changes the other" (really: there is only one list). To copy: `b = a.copy()` or `b = a[:]` or `b = list(a)`. These are *shallow* copies — nested lists inside are still shared (`copy.deepcopy` exists for the rare deep case).

**Tuple** — an ordered, **immutable** sequence, written with parentheses: `(3, 4)`, `("Ada", 1815)`. Once built, no appending, removing, or replacing. Use tuples for:

- Fixed-shape records: a coordinate `(x, y)`, an RGB color, a `(name, age)` pair.
- Multiple return values from functions (Chapter 6 — that *was* a tuple).
- Data that must not be accidentally modified, or that needs to be a dict key (Chapter 8).

Gotcha: a one-element tuple needs a trailing comma — `(5,)`; plain `(5)` is just the number 5.

**Unpacking** — assigning a sequence's items to multiple names at once: `x, y = (3, 4)`; `first, *rest = [1, 2, 3, 4]` puts `1` in `first` and `[2, 3, 4]` in `rest`. Works in `for` loops too: `for name, score in pairs:`.

**Sorting with `key`** — both `sort()` and `sorted()` accept `key=`, a function applied to each item to decide order: `sorted(words, key=len)`, `sorted(words, key=str.lower)`, and `reverse=True` for descending.

**Nested lists** — lists of lists model grids/tables: `grid[row][col]`.

## Code Examples

### Building and modifying

```python
shopping = ["milk", "bread"]

shopping.append("eggs")            # ['milk', 'bread', 'eggs']
shopping.insert(0, "coffee")       # ['coffee', 'milk', 'bread', 'eggs']
shopping.extend(["jam", "tea"])    # ... 'jam', 'tea']
shopping.remove("bread")           # removes by VALUE

last = shopping.pop()              # 'tea' — removed and returned
print(last, shopping)

shopping[0] = "espresso"           # replace by index — lists are mutable
print(len(shopping), shopping)
print("milk" in shopping)          # True
```

### Slicing and copying

```python
nums = [10, 20, 30, 40, 50]

print(nums[1:4])      # [20, 30, 40]
print(nums[-2:])      # [40, 50]

# THE ALIASING TRAP — read this twice
a = [1, 2, 3]
b = a                 # NOT a copy — same list, two names
b.append(4)
print(a)              # [1, 2, 3, 4]  ← "a changed too"

c = a.copy()          # a real (shallow) copy
c.append(99)
print(a)              # [1, 2, 3, 4] — unaffected
```

### Sorting

```python
scores = [88, 42, 95, 67]

print(sorted(scores))              # [42, 67, 88, 95] — new list
print(scores)                      # original order preserved

scores.sort(reverse=True)          # in place, descending
print(scores)                      # [95, 88, 67, 42]

words = ["banana", "Fig", "apple", "date"]
print(sorted(words))                    # ['Fig', 'apple', ...] — capitals sort first!
print(sorted(words, key=str.lower))     # ['apple', 'banana', 'date', 'Fig']
print(sorted(words, key=len))           # shortest first

# result = scores.sort()   # None! sort() mutates and returns nothing — classic trap
```

### Tuples and unpacking

```python
point = (3, 7)
x, y = point                       # unpack
print(f"x={x}, y={y}")

# point[0] = 99                    # TypeError: tuples are immutable

def min_max(numbers):
    """Return the smallest and largest as a tuple."""
    return min(numbers), max(numbers)

low, high = min_max([4, 9, 1, 7])
print(low, high)                   # 1 9

# Star-unpacking
first, *middle, last = [1, 2, 3, 4, 5]
print(first, middle, last)         # 1 [2, 3, 4] 5

# Looping over pairs
menu = [("espresso", 3.0), ("latte", 4.5), ("mocha", 5.0)]
for drink, price in menu:
    print(f"{drink:<10} ${price:.2f}")
```

### Lists of tuples: a mini leaderboard

```python
# leaderboard.py
players = [
    ("Nia", 3120),
    ("Omar", 4470),
    ("Pia", 2890),
    ("Quinn", 4470),
]

# Sort by score descending; tuple comparison breaks ties by name if we build the key
ranked = sorted(players, key=lambda p: (-p[1], p[0]))   # lambda: tiny inline function

print("RANK  NAME    SCORE")
for rank, (name, score) in enumerate(ranked, 1):
    print(f"{rank:<5} {name:<7} {score}")
```

(`lambda p: ...` is a one-expression anonymous function — a compact way to write a `key`. You could equally define a small named `def` and pass it.)

### Nested lists — a grid

```python
grid = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]

print(grid[1][2])          # row 1, column 2 → 6

for row in grid:
    for cell in row:
        print(cell, end=" ")
    print()
```

## Common Pitfalls

**1. `b = a` is not a copy.** See above — this causes "my list changed by itself!" bugs. When in doubt: `b = a.copy()`.

**2. Capturing the return of an in-place method**

```python
nums = [3, 1, 2]
nums = nums.sort()        # nums is now None!
print(nums)               # None

nums = [3, 1, 2]
nums.sort()               # RIGHT: mutate, don't reassign
# or: nums = sorted(nums)
```

Same applies to `.append()`, `.reverse()`, `.extend()` — they all return `None`.

**3. IndexError from off-the-end access**

```python
items = ["a", "b", "c"]
# print(items[3])         # IndexError — valid indexes are 0..2 (and -1..-3)
if len(items) > 3:
    print(items[3])
```

Note: *slices* forgive out-of-range bounds (`items[1:99]` → `['b', 'c']`), but *indexes* don't.

**4. `append` vs. `extend`**

```python
a = [1, 2]
a.append([3, 4])      # [1, 2, [3, 4]] — nested list! probably not what you meant
a = [1, 2]
a.extend([3, 4])      # [1, 2, 3, 4]
```

**5. Removing items while iterating** — covered in Chapter 5; the fix is to build a new list (or iterate over a copy: `for x in items[:]:`).

**6. Multiplying nested lists**

```python
grid = [[0] * 3] * 3      # three references to the SAME inner list
grid[0][0] = 9
print(grid)               # [[9, 0, 0], [9, 0, 0], [9, 0, 0]] — !!!

grid = [[0] * 3 for _ in range(3)]   # RIGHT (comprehensions: Chapter 9)
```

**7. Forgetting the comma in a single-item tuple**

```python
t = ("solo")      # just a string
t = ("solo",)     # a 1-tuple — the comma makes the tuple, not the parens
```

## Practice Exercises

1. **List gym.** Start with `nums = [5, 3, 8, 1, 9, 2]`. Without retyping the list: append 7; insert 0 at the front; remove the value 8; sort ascending; print the largest, smallest, sum, and average; finally print the list reversed *without* permanently reversing it.
2. **De-duplicate, order preserved.** Given `emails` with duplicates, build a new list containing each email once, in the order first seen, using only a loop, `in`, and `append`. (Chapter 8 will show a faster way.)
3. **Tuple records.** Create a list of `(city, temperature)` tuples for five cities. Print the hottest and coldest city, then all cities sorted alphabetically, then all cities above the average temperature.
4. **Alias detective.** Write a short script demonstrating the difference between `b = a` and `b = a.copy()` for lists: mutate `b` and print `a` in both scenarios, with comments explaining each output *before* you run it.
5. **Tic-tac-toe board.** Build a 3×3 nested list filled with `" "`. Write code that sets the center to `"X"` and a corner to `"O"`, then prints the board with `|` and `-` separators so it looks like an actual tic-tac-toe grid.
