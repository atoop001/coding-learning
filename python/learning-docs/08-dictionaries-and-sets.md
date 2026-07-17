# Chapter 8: Dictionaries & Sets

## Overview

Lists answer "what's at position 3?" Dictionaries answer the far more common question: "what's the value **for this key**?" — the price *for* "latte", the user *for* this username, the count *for* this word. Dictionaries (`dict`) are Python's key→value mapping and arguably its most important data structure: JSON data (Chapter 16), function `**kwargs`, class attributes, and Flask request data are all dictionaries or dict-like.

Sets (`set`) are their minimalist cousin: just keys, no values — perfect for membership tests and de-duplication.

## Definitions & Explanations

**Dictionary (`dict`)** — a mutable collection of **key: value** pairs with curly braces:

```python
prices = {"espresso": 3.0, "latte": 4.5}
```

- **Keys** must be unique and *hashable* (immutable): strings, numbers, tuples — not lists or dicts. Assigning to an existing key overwrites its value.
- **Values** can be anything, including lists and other dicts.
- Lookup is by key: `prices["latte"]` → `4.5`. Missing key → `KeyError`.
- Dicts preserve **insertion order** (guaranteed since Python 3.7).

**Core dict operations:**

| Operation | Effect |
|---|---|
| `d[key]` | get value (KeyError if missing) |
| `d[key] = value` | add or overwrite |
| `d.get(key)` / `d.get(key, default)` | get, but return `None`/default instead of erroring |
| `key in d` | membership test on **keys** |
| `del d[key]` / `d.pop(key)` | remove (pop returns the value; `d.pop(key, default)` avoids KeyError) |
| `d.keys()` / `d.values()` / `d.items()` | views of keys / values / `(key, value)` pairs |
| `d.update(other)` | merge another dict in (overwriting clashes) |
| `d.setdefault(key, default)` | get value; if absent, insert default first |
| `len(d)` | number of pairs |

**Iterating** — `for key in d:` loops over keys; `for key, value in d.items():` is the everyday pattern for both.

**Nesting** — dict values are often lists or other dicts; `data["users"][0]["email"]` chains lookups. This shape mirrors JSON exactly.

**The counting pattern** — tally occurrences with `counts[item] = counts.get(item, 0) + 1`. So common that the standard library ships `collections.Counter`, but learn the manual version first.

**The grouping pattern** — collect items under a key: `groups.setdefault(category, []).append(item)`.

**Set** — an unordered collection of unique, hashable items: `{1, 2, 3}`. No duplicates, no positions, no indexing. Blazingly fast `in` checks. Create an *empty* set with `set()` — `{}` is an empty **dict**!

**Set operations:**

- `s.add(x)`, `s.discard(x)` (no error if absent), `s.remove(x)` (KeyError if absent)
- `a | b` union, `a & b` intersection, `a - b` difference, `a ^ b` symmetric difference
- `a <= b` subset test
- `set(some_list)` — instant de-duplication (order lost)

**When to use which container:**

- Ordered sequence, positions matter → **list**
- Fixed-shape record, immutable → **tuple**
- Look things up by name/id → **dict**
- Only care about membership/uniqueness → **set**

## Code Examples

### Dict basics

```python
student = {
    "name": "Maya",
    "age": 21,
    "courses": ["math", "python"],     # a list as a value — totally normal
}

print(student["name"])                  # Maya
student["age"] = 22                     # overwrite
student["email"] = "maya@example.com"   # add a new pair
del student["courses"]

print("email" in student)               # True (checks KEYS)
print(len(student))                     # 3
```

### `.get()` vs `[]` — defensive lookups

```python
config = {"theme": "dark"}

# print(config["font"])                 # KeyError: 'font'
print(config.get("font"))               # None — no crash
print(config.get("font", "monospace"))  # 'monospace' — with fallback

# Typical real-world use: optional settings
font = config.get("font", "monospace")
print(f"Using theme={config['theme']}, font={font}")
```

### Iterating

```python
inventory = {"apples": 12, "bananas": 0, "cherries": 45}

for fruit in inventory:                       # keys
    print(fruit)

for fruit, qty in inventory.items():          # the everyday pattern
    status = "IN STOCK" if qty > 0 else "OUT"
    print(f"{fruit:<10} {qty:>3}  {status}")

print(sum(inventory.values()))                # 57
print(max(inventory, key=inventory.get))      # 'cherries' — key with biggest value
```

### The counting pattern

```python
sentence = "the quick brown fox jumps over the lazy dog the end"

counts = {}
for word in sentence.split():
    counts[word] = counts.get(word, 0) + 1

print(counts)                                  # {'the': 3, 'quick': 1, ...}

# Show the most frequent words first
for word, n in sorted(counts.items(), key=lambda kv: kv[1], reverse=True):
    print(f"{word}: {n}")
```

### The grouping pattern

```python
expenses = [
    ("food", 12.5), ("travel", 40.0), ("food", 7.25),
    ("fun", 15.0), ("travel", 9.99),
]

by_category = {}
for category, amount in expenses:
    by_category.setdefault(category, []).append(amount)

for category, amounts in by_category.items():
    print(f"{category:<8} total ${sum(amounts):.2f} across {len(amounts)} purchases")
```

### Nested dicts — records by id

```python
users = {
    "ada":   {"name": "Ada Lovelace", "logins": 4},
    "alan":  {"name": "Alan Turing",  "logins": 9},
}

print(users["alan"]["name"])          # Alan Turing
users["ada"]["logins"] += 1

# Add a user safely
username = "grace"
if username not in users:
    users[username] = {"name": "Grace Hopper", "logins": 0}
```

### Sets

```python
seen = set()
clicks = ["home", "shop", "home", "cart", "shop", "home"]

for page in clicks:
    seen.add(page)
print(seen)                            # {'home', 'shop', 'cart'} — order not guaranteed
print(len(set(clicks)))                # 3 unique pages, one-liner

python_devs = {"ana", "ben", "cleo"}
js_devs = {"ben", "dara", "cleo"}

print(python_devs & js_devs)           # {'ben', 'cleo'}   both
print(python_devs | js_devs)           # everyone
print(python_devs - js_devs)           # {'ana'}           Python-only
print("ana" in python_devs)            # True — very fast even for huge sets
```

## Common Pitfalls

**1. KeyError from assuming a key exists**

```python
scores = {"ana": 10}
# print(scores["ben"])          # KeyError
print(scores.get("ben", 0))     # 0 — choose .get() when absence is normal
```

Use `[]` when a missing key is a *bug you want to hear about*; use `.get()` when it's an expected case.

**2. `{}` is an empty dict, not a set**

```python
s = {}
print(type(s))     # <class 'dict'>
s = set()          # the only way to write an empty set
```

**3. Unhashable keys**

```python
# d = {[1, 2]: "point"}        # TypeError: unhashable type: 'list'
d = {(1, 2): "point"}          # tuples work as keys
```

**4. Mutating a dict while iterating it**

```python
for key in inventory:
    if inventory[key] == 0:
        del inventory[key]     # RuntimeError: dictionary changed size during iteration

for key in list(inventory):   # FIX: iterate over a snapshot of the keys
    if inventory[key] == 0:
        del inventory[key]
```

**5. Expecting order from a set** — sets have no positions; `my_set[0]` is a `TypeError`, and printed order can differ between runs. If you de-duplicate with `set()` but need original order, use the loop-and-check pattern from Chapter 7 or `dict.fromkeys(items)`.

**6. Confusing `in` semantics**

```python
d = {"a": 1}
print("a" in d)            # True  — keys
print(1 in d)              # False — values are NOT checked
print(1 in d.values())     # True  — explicit
```

**7. Shared nested defaults** — the mutable-default trap (Chapter 6) applies to dicts too: never use `def f(cache={})`.

## Practice Exercises

1. **Phone book.** Build a dict mapping three friends' names to phone numbers. Write a loop that asks for a name and prints the number, or "not found" without crashing, until the user enters nothing. Add a feature: entering `name=number` adds or updates an entry.
2. **Letter frequency.** Ask for a sentence and produce a dict of letter → count (letters only, case-insensitive, ignore spaces/punctuation). Print the counts sorted from most to least frequent.
3. **Gradebook.** Given `grades = {"ana": [88, 92, 79], "ben": [95, 70], "cleo": [100, 100, 90]}`, print each student's average, the class-wide average, and the name of the top student — using loops and dict methods, no manual repetition per student.
4. **Set logic.** Two lists contain the usernames of people who liked post A and post B (with duplicates). Using sets, print: how many unique people liked each; who liked both; who liked exactly one; whether all likers of A also liked B.
5. **Inventory merge.** Two warehouses report stock as dicts (`{"bolt": 50, "nut": 20}` etc.). Produce one combined dict summing quantities for shared keys — not just `update()`, which would overwrite. Then list items only present in one warehouse (sets can help).
