# Chapter 5: Hash Tables (How Dicts & Objects Work Under the Hood)

## Overview

Python's `dict` and `set`, and JavaScript's objects and `Map`, all run on the same engine: the **hash table** — arguably the most important data structure in practical programming. It delivers what sounds impossible: insert, lookup, and delete by key in O(1) *average* time, regardless of how many entries it holds. This chapter explains the trick (hash functions), the catch (collisions), and the fine print (worst cases, resizing, what makes a key valid). Half of all "make this code faster" wins in interviews are "replace an O(n) scan with a hash lookup," so internalizing this chapter pays off immediately.

## Definitions & Explanations

### The core idea: compute the location instead of searching for it

Arrays give O(1) access *by index* (Chapter 2). Hash tables extend that to access *by arbitrary key* in three steps:

1. **Hash** the key: a hash function turns any key into a big integer, e.g. `hash("cat") = 4499914`.
2. **Compress** it into a slot index: `index = hash(key) % capacity`.
3. **Go there**: read/write the array slot directly.

```
             hash("cat") = 4499914
             4499914 % 8 = 2
                              |
        0      1      2      v      4      5      6      7
      +------+------+----------+------+------+------+------+------+
      |      |      |("cat", 9)|      |      |      |      |      |
      +------+------+----------+------+------+------+------+------+
```

No scanning, no comparisons against other keys — the key itself tells you where to look. That's the whole miracle.

A good hash function is **deterministic** (same key → same hash, always) and **spreads keys uniformly** so slots fill evenly.

### Collisions: the inevitable complication

With more possible keys than slots, two keys must sometimes land in the same slot (`hash("cat") % 8 == hash("dog") % 8`, say). This is a **collision**, and handling it is most of a hash table's design. Two main families:

**Separate chaining** — each slot holds a small list (chain) of entries; colliding entries share the slot:

```
      0      1        2                     3
    +------+------+--------------------- +------+
    |      |      | [("cat",9),("dog",4)]|      |   slot 2's chain has 2 entries
    +------+------+--------------------- +------+
```

Lookup = hash to the slot, then scan the (short) chain comparing actual keys.

**Open addressing** — entries live directly in slots; on collision, *probe* for the next free slot by some rule (simplest: linear probing — try `index+1, index+2, ...` wrapping around). CPython's dict uses open addressing with a smarter probe sequence.

### Load factor and resizing

**Load factor = entries / slots.** As it rises, chains lengthen (or probes multiply) and O(1) decays toward O(n). So hash tables resize: when load factor passes a threshold (commonly ~0.66 for open addressing, ~0.75–1.0 for chaining), allocate a bigger array and **rehash** every entry into it (indices change because `% capacity` changed!). Same amortized story as dynamic arrays: occasional O(n) rebuilds, O(1) average per operation.

### The real complexity table

| Operation | Average | Worst case |
|---|---|---|
| insert / lookup / delete | **O(1)** | O(n) — all keys collide |

The worst case is real: an adversary who knows your hash function can craft keys that all collide (a known DoS attack — Python randomizes string hashing per-process partly for this reason). In interviews, say "O(1) average, O(n) worst case" for hash operations.

### What makes a valid key: hashability

A key must be **hashable**: it has a stable hash for its lifetime, and equal keys have equal hashes (`a == b` ⇒ `hash(a) == hash(b)` — the contract that makes lookup correct). In Python, immutable built-ins (str, int, float, tuple-of-hashables, frozenset) are hashable; mutable ones (list, dict, set) are not — if a list could be a key and then you mutated it, its hash would change and the table could never find it again.

JavaScript: object keys are coerced to strings; `Map` accepts any value as a key using identity semantics for objects.

## Code Examples

A full hash map with separate chaining, from scratch:

```python
# hash_map.py — a dict built from first principles (separate chaining).

class HashMap:
    """Hash table with separate chaining and automatic resizing."""

    def __init__(self, capacity=8):
        self._capacity = capacity
        self._size = 0
        # Each slot ("bucket") is a list of [key, value] pairs.
        self._buckets = [[] for _ in range(capacity)]

    def _index(self, key):
        """Hash + compress: which bucket does this key belong to?
        Uses Python's built-in hash(); any deterministic spreader works."""
        return hash(key) % self._capacity

    def put(self, key, value):
        """Insert or update. Average O(1)."""
        bucket = self._buckets[self._index(key)]
        for pair in bucket:                    # scan the chain: is key present?
            if pair[0] == key:
                pair[1] = value                # update existing entry
                return
        bucket.append([key, value])            # new entry
        self._size += 1
        if self._size / self._capacity > 0.75:  # load factor check
            self._resize()

    def get(self, key, default=None):
        """Average O(1): hash to the bucket, scan its short chain."""
        bucket = self._buckets[self._index(key)]
        for k, v in bucket:
            if k == key:                       # compare REAL keys, not hashes
                return v
        return default

    def remove(self, key):
        """Average O(1). Returns True if the key existed."""
        bucket = self._buckets[self._index(key)]
        for i, (k, _) in enumerate(bucket):
            if k == key:
                bucket.pop(i)                  # chain is tiny; O(len(chain))
                self._size -= 1
                return True
        return False

    def _resize(self):
        """Double capacity and REHASH every entry — indices change
        because `% capacity` changed."""
        old_buckets = self._buckets
        self._capacity *= 2
        self._buckets = [[] for _ in range(self._capacity)]
        self._size = 0
        for bucket in old_buckets:
            for key, value in bucket:
                self.put(key, value)           # re-inserts at new index

    def __len__(self):
        return self._size

    def __contains__(self, key):
        sentinel = object()                    # distinguishes "missing" from None
        return self.get(key, sentinel) is not sentinel

    def items(self):
        for bucket in self._buckets:
            yield from ((k, v) for k, v in bucket)


if __name__ == "__main__":
    m = HashMap()
    for word in "the quick brown fox jumps over the lazy dog the end".split():
        m.put(word, m.get(word, 0) + 1)        # word frequency count
    print("'the' appears:", m.get("the"))      # 3
    print("'cat' in map?", "cat" in m)         # False
    m.remove("fox")
    print(sorted(m.items()))
```

The three hash-powered patterns you'll use constantly:

```python
# hash_patterns.py — the everyday superpowers of O(1) lookup.
from collections import Counter, defaultdict

def two_sum(nums, target):
    """Find indices of two numbers summing to target — THE classic.
    Brute force is O(n^2); a hash of 'seen values' makes it O(n)."""
    seen = {}                                  # value -> index
    for i, x in enumerate(nums):
        if target - x in seen:                 # O(1) average lookup
            return [seen[target - x], i]
        seen[x] = i
    return None

def first_duplicate(items):
    """O(n) duplicate detection with a set (a hash table of keys only)."""
    seen = set()
    for x in items:
        if x in seen:
            return x
        seen.add(x)
    return None

def group_anagrams(words):
    """Group words that are anagrams: hash on a canonical form.
    defaultdict(list) auto-creates the bucket on first touch."""
    groups = defaultdict(list)
    for w in words:
        key = "".join(sorted(w))               # 'eat','tea' -> 'aet'
        groups[key].append(w)
    return list(groups.values())

if __name__ == "__main__":
    print(two_sum([2, 7, 11, 15], 9))                       # [0, 1]
    print(first_duplicate([3, 1, 4, 1, 5]))                 # 1
    print(group_anagrams(["eat", "tea", "tan", "ate"]))     # [['eat','tea','ate'], ['tan']]
    print(Counter("mississippi").most_common(2))            # [('i',4),('s',4)] or ('s',4),('i',4)
```

JavaScript comparison:

```javascript
const counts = new Map();                     // prefer Map over {} for data
for (const w of words) counts.set(w, (counts.get(w) ?? 0) + 1);
const seen = new Set();  seen.add(x);  seen.has(x);   // O(1) average
// Plain objects {} hash string keys only; Map keys can be anything.
```

## Common Pitfalls

**1. Comparing hashes instead of keys.** Equal hashes do NOT imply equal keys (collisions!). The chain scan in `get` must compare `k == key`, never `hash(k) == hash(key)`. Skipping the real comparison returns wrong values exactly when collisions occur — rare, silent, and maddening.

**2. Using a mutable object as a key.**

```python
d = {}
point = [3, 4]
d[point] = "home"          # TypeError: unhashable type: 'list'

# Corrected — use an immutable equivalent:
d[(3, 4)] = "home"         # tuples are hashable (if their contents are)
```

**3. Assuming order — or assuming no order.** Since Python 3.7, dicts preserve *insertion* order (an implementation nicety, now guaranteed). They are still not *sorted*; if you need sorted keys, sort them explicitly or use a tree (Chapter 9).

**4. Mutating a dict while iterating over it.**

```python
# RuntimeError: dictionary changed size during iteration
for k in counts:
    if counts[k] < 2:
        del counts[k]

# Corrected — iterate over a snapshot:
for k in list(counts):
    if counts[k] < 2:
        del counts[k]
```

**5. Using `dict.get` with a `None` default when `None` is a legal value.** `m.get(k)` returning `None` is ambiguous: missing, or stored `None`? Use a sentinel (`object()`, as `__contains__` above does) or `in` when the distinction matters.

**6. Reaching for a hash table when you need ordering or ranges.** "Give me all keys between 10 and 20" or "the smallest key" are O(n) on a hash table. Those workloads belong to trees (Chapter 9) and heaps (Chapter 10).

## Practice Exercises

1. Instrument `HashMap` with a `stats()` method reporting load factor, the longest chain, and the number of empty buckets. Insert 1,000 random strings and check: does the longest chain stay small? Then force `_index` to `return 0` and watch performance degrade — measure lookup time before and after.
2. Convert `HashMap` from separate chaining to **open addressing with linear probing**. You'll need a special `DELETED` marker (a "tombstone") — write a comment explaining why plain slot-clearing on delete breaks subsequent lookups.
3. Using a dict, write `first_unique_char(s)` returning the index of the first character that appears exactly once in the string, in O(n). Then answer: why can't this be done in one pass with no extra structure?
4. Implement `TwoSum` as a class with `add(number)` and `find(target)` methods (find reports whether any pair of added numbers sums to target). State the time complexity of each method for your design, and describe an alternative design that trades their costs.
5. Build a tiny spell-checker: load a word list into a set, then for an input word, generate all single-edit variants (deletions, swaps, substitutions, insertions) and return those present in the set. Estimate the complexity for a word of length k against a dictionary of n words — and note why the set is the only reason this is feasible.

---

**Next:** Chapter 6 — recursion: the call stack as a data structure, base cases, and when self-reference beats loops.
