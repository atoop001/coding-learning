# Chapter 2: Arrays & Dynamic Arrays (How Lists Really Work)

## Overview

The array is the most fundamental data structure in computing: a block of elements sitting side by side in memory. Python's `list` and JavaScript's `Array` are *dynamic arrays* — arrays that grow themselves as needed. Understanding what they actually do under the hood explains why `append` is fast, why `insert(0, x)` is slow, and why "just use a list" is sometimes the wrong answer. Every later structure in this track is either built on arrays or built to fix an array's weaknesses, so this chapter is load-bearing.

## Definitions & Explanations

### Static arrays and memory

Memory is one long row of numbered bytes. A **static array** is a contiguous run of equally-sized slots:

```
address:   1000   1004   1008   1012   1016
          +------+------+------+------+------+
  values: |  17  |  42  |   8  |  99  |  23  |
          +------+------+------+------+------+
  index:     0      1      2      3      4
```

Because slots are equal-sized and contiguous, the address of `arr[i]` is just `start_address + i * slot_size` — one multiplication and one addition. That is why **indexing is O(1)**: the computer never scans; it computes the address directly.

The trade-off: the block has a *fixed capacity*. Growing means allocating a new, bigger block and copying everything over.

(Detail for Python: a `list` actually stores *pointers* to objects, all pointer-sized, which is how one list holds mixed types. The contiguous-slots picture still holds — the slots hold addresses.)

### Dynamic arrays: capacity vs length

A **dynamic array** wraps a static array with two numbers:

- **length** — how many elements you've stored.
- **capacity** — how many slots the underlying block has.

```
length = 3, capacity = 8
+----+----+----+----+----+----+----+----+
| 17 | 42 |  8 | .. | .. | .. | .. | .. |
+----+----+----+----+----+----+----+----+
              ^ unused reserved space
```

`append` while there's spare capacity: write into slot `length`, increment `length`. O(1).

`append` when full: allocate a new block (typically ~2× bigger), copy all `n` elements, then append. O(n) for that one operation.

### Why append is *amortized* O(1)

Doubling makes the expensive copies exponentially rare. Growing from capacity 1 to 1024 requires copies at sizes 1, 2, 4, ..., 512 — totaling 1023 copy operations across 1024 appends. Total work is ~2n for n appends, so the **average per append is O(1)**. That averaged guarantee is called *amortized* O(1) (Chapter 1). Individual appends can occasionally be slow; the sequence never is.

(CPython grows by ~1.125× rather than 2×, trading a bit more copying for less wasted memory. The amortized math still works for any multiplicative factor.)

### The cost table you should memorize

| Operation | Python | Complexity | Why |
|---|---|---|---|
| Index read/write | `a[i]` | O(1) | address arithmetic |
| Append at end | `a.append(x)` | O(1) amortized | spare capacity, rare resize |
| Pop from end | `a.pop()` | O(1) | just decrement length |
| Insert at front/middle | `a.insert(0, x)` | O(n) | shift everything right |
| Delete at front/middle | `a.pop(0)`, `del a[i]` | O(n) | shift everything left |
| Search unsorted | `x in a` | O(n) | must scan |
| Slice of length k | `a[i:j]` | O(k) | copies elements |
| Concatenate | `a + b` | O(len(a)+len(b)) | copies both |

The shifting cost is the key intuition:

```
insert 99 at index 1:            every later element moves right
before: | 17 | 42 |  8 | 23 |
         	  <- shift ->
after:  | 17 | 99 | 42 |  8 | 23 |
                 ^ n-1 elements had to move in the worst case
```

JavaScript's `Array` behaves the same way at the ends: `push`/`pop` are cheap, `shift`/`unshift` are O(n).

## Code Examples

Building a dynamic array from scratch — the single most instructive exercise in this track. We simulate the fixed-size block with a pre-sized Python list that we promise not to resize directly.

```python
# dynamic_array.py — a from-scratch dynamic array.

class DynamicArray:
    """A growable array built on a fixed-capacity block.
    We use a Python list of a FIXED size as our 'raw memory block'
    and only ever assign to existing slots, never append to it."""

    def __init__(self):
        self._capacity = 4                       # slots allocated
        self._length = 0                         # slots used
        self._data = self._make_block(self._capacity)

    def _make_block(self, capacity):
        """Simulate allocating a raw block of `capacity` slots."""
        return [None] * capacity

    def __len__(self):
        return self._length

    def get(self, index):
        """O(1) — direct slot access after a bounds check."""
        if not 0 <= index < self._length:
            raise IndexError(f"index {index} out of range")
        return self._data[index]

    def set(self, index, value):
        """O(1)."""
        if not 0 <= index < self._length:
            raise IndexError(f"index {index} out of range")
        self._data[index] = value

    def append(self, value):
        """Amortized O(1): usually a single write, occasionally a resize."""
        if self._length == self._capacity:
            self._resize(self._capacity * 2)     # the rare O(n) case
        self._data[self._length] = value
        self._length += 1

    def _resize(self, new_capacity):
        """O(n): allocate a bigger block and copy everything across."""
        new_block = self._make_block(new_capacity)
        for i in range(self._length):            # copy each element
            new_block[i] = self._data[i]
        self._data = new_block
        self._capacity = new_capacity

    def insert(self, index, value):
        """O(n): shift elements right to open a gap."""
        if not 0 <= index <= self._length:
            raise IndexError(f"index {index} out of range")
        if self._length == self._capacity:
            self._resize(self._capacity * 2)
        # Shift right, starting from the END so we don't overwrite.
        for i in range(self._length, index, -1):
            self._data[i] = self._data[i - 1]
        self._data[index] = value
        self._length += 1

    def pop(self):
        """O(1): remove and return the last element."""
        if self._length == 0:
            raise IndexError("pop from empty array")
        self._length -= 1
        value = self._data[self._length]
        self._data[self._length] = None          # let it be garbage-collected
        return value

    def delete(self, index):
        """O(n): shift elements left to close the gap."""
        if not 0 <= index < self._length:
            raise IndexError(f"index {index} out of range")
        for i in range(index, self._length - 1):
            self._data[i] = self._data[i + 1]
        self._length -= 1
        self._data[self._length] = None

    def __repr__(self):
        shown = ", ".join(repr(self._data[i]) for i in range(self._length))
        return f"DynamicArray([{shown}], capacity={self._capacity})"


if __name__ == "__main__":
    arr = DynamicArray()
    for x in [10, 20, 30, 40, 50]:               # forces one resize (4 -> 8)
        arr.append(x)
        print(arr)
    arr.insert(1, 15)
    print("after insert:", arr)
    arr.delete(0)
    print("after delete:", arr)
    print("pop ->", arr.pop(), "|", arr)
```

Watching amortization empirically with the real `list`:

```python
# capacity_watch.py — see CPython over-allocate in real time.
import sys

lst = []
last_size = sys.getsizeof(lst)
for i in range(64):
    lst.append(i)
    size = sys.getsizeof(lst)
    if size != last_size:
        # Size jumped: a resize (reallocation + copy) just happened.
        print(f"resize at length {len(lst)}: {last_size} -> {size} bytes")
        last_size = size
# Resizes get further apart as the list grows — that's amortization.
```

JavaScript comparison:

```javascript
// Same physics, different names:
const a = [1, 2, 3];
a.push(4);      // amortized O(1), like Python's append
a.pop();        // O(1)
a.unshift(0);   // O(n) — shifts everything right, like insert(0, x)
a.shift();      // O(n) — like pop(0)
```

## Common Pitfalls

**1. Popping/inserting at the front in a loop — accidental O(n²).**

```python
# O(n^2): each pop(0) shifts the entire remaining list left.
while tasks:
    task = tasks.pop(0)
    process(task)

# Corrected — O(n) total with a deque (Chapter 4):
from collections import deque
tasks = deque(tasks)
while tasks:
    task = tasks.popleft()      # O(1)
    process(task)
```

**2. Removing items from a list while iterating over it.** Deleting shifts elements left, so the loop index skips the shifted neighbor.

```python
# Bug: removes only every other odd number.
nums = [1, 3, 5, 2, 7, 9]
for x in nums:
    if x % 2 == 1:
        nums.remove(x)

# Corrected — build a new list (O(n), and clearer):
nums = [x for x in nums if x % 2 == 0]
```

**3. `[[0] * 3] * 4` — one inner list, four references.**

```python
grid = [[0] * 3] * 4
grid[0][0] = 9
# Every row shows the 9! All four rows are the SAME list object.

# Corrected:
grid = [[0] * 3 for _ in range(4)]
```

**4. Assuming `x in my_list` is cheap.** It's an O(n) scan. Inside a loop that's O(n²). If you're doing repeated membership checks, use a `set` (Chapter 5).

**5. Slicing without realizing it copies.** `total = sum(a[1:])` copies n−1 elements first. Fine once; expensive inside a loop or a recursion (a classic hidden cost in recursive solutions, revisited in Chapter 6).

## Practice Exercises

1. Extend `DynamicArray` with a `_shrink` policy: when `pop`/`delete` leaves the array at ≤ 1/4 capacity, resize to half capacity (never below 4). Explain in a comment why the shrink threshold is 1/4 rather than 1/2. (Hint: think about an append/pop sequence right at the boundary.)
2. Add `__getitem__`, `__setitem__`, and negative-index support to `DynamicArray` so `arr[-1]` returns the last element, matching Python's behavior.
3. Predict, then measure: build a list of 100,000 items with `append`, then again with `insert(0, ...)`. Time both. How much slower is the second, and why does that match the cost table?
4. Write `rotate_left(items, k)` that rotates a list left by `k` positions in O(n) time and O(1) extra space (e.g., `[1,2,3,4,5]` rotated by 2 → `[3,4,5,1,2]`). The three-reversal trick is one route.
5. Suppose a dynamic array grew by adding a *fixed* 10 slots each resize instead of doubling. Show (with a short argument or a measurement) that n appends now cost O(n²) total. This is why growth must be multiplicative.

---

**Next:** Chapter 3 introduces the linked list — the structure that makes front-insertion O(1) by giving up O(1) indexing. Every structure is a trade.
