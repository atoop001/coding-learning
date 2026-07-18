# Chapter 7: Sorting Algorithms

## Overview

Sorting is the drosophila of algorithms: simple to state, endlessly instructive, and the foundation for everything from binary search (Chapter 8) to database indexes. This chapter builds four sorts by hand — bubble and insertion for intuition, merge and quick for real power — then explains stability, the O(n log n) barrier, and what `sorted()` actually runs. You will almost never hand-roll a sort in production (the built-in is better), but interviews expect you to implement merge/quick sort cold and, more importantly, to *reason* with sorting: "if I sort first, this O(n²) problem becomes O(n log n)."

## Definitions & Explanations

### The scoreboard

| Algorithm | Best | Average | Worst | Space | Stable? |
|---|---|---|---|---|---|
| Bubble sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Insertion sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick sort | O(n log n) | O(n log n) | **O(n²)** | O(log n) avg | No (typical) |
| Timsort (built-in) | O(n) | O(n log n) | O(n log n) | O(n) | Yes |

### Stability

A sort is **stable** if elements that compare equal keep their original relative order. Sort people by first name, then stable-sort by last name → people sharing a last name stay ordered by first name. This is why Python lets you do multi-key sorts as successive single-key sorts. Instability is quicksort's quiet drawback.

### Bubble sort — the intuition pump

Repeatedly sweep the list, swapping adjacent out-of-order pairs. After sweep k, the k largest values have "bubbled" to the end. Never used in practice; perfect for understanding "compare and swap."

### Insertion sort — the card player's sort

Grow a sorted prefix; take each next element and walk it left into position, like sorting a hand of cards. Quadratic in general, but **O(n) on nearly-sorted data** and genuinely fast for tiny arrays — which is why real libraries use it as the small-input base case inside fancier sorts.

### Merge sort — divide and conquer

Split in half, recursively sort each half, then **merge** two sorted lists in O(n) (the Chapter 3 exercise). The recursion tree has O(log n) levels (halving) and O(n) merge work per level → **O(n log n) guaranteed**, worst case included.

```
                [38, 27, 43, 3]
                 /           \
           [38, 27]         [43, 3]        split: log n levels
            /    \           /    \
         [38]   [27]      [43]    [3]
            \    /           \    /
           [27, 38]         [3, 43]        merge: O(n) work per level
                 \           /
                [3, 27, 38, 43]
```

Cost: O(n) auxiliary space for the merge buffers.

### Quick sort — partition and conquer

Pick a **pivot**; partition the array so smaller elements land left of it, larger right (the pivot is now in final position); recurse on both sides. Average O(n log n) with small constants and in-place partitioning — usually the fastest comparison sort in practice. But a consistently bad pivot (e.g., always the smallest element — which "first element as pivot" produces on *already-sorted input*) gives one-sided splits and **O(n²)**. Defense: random or median-of-three pivots.

### Why O(n log n) is the floor

Any sort that works by *comparing* elements must distinguish among n! possible orderings; a binary comparison tree needs depth ≥ log₂(n!) ≈ n log n. So no comparison sort beats O(n log n) in the worst case. Non-comparison sorts (counting sort, radix sort) beat it for restricted key types by never comparing at all.

### What the built-ins do

Python's `sorted()`/`list.sort()` use **Timsort**: merge sort restructured to exploit existing sorted "runs" in real data, with insertion sort for small pieces. Stable, O(n log n) worst case, O(n) on sorted-ish input. JavaScript's `Array.prototype.sort` is stable per the spec (V8 also uses Timsort) — but mind the default *string* comparison: `[10, 9, 1].sort()` gives `[1, 10, 9]`; you must pass `(a, b) => a - b`.

## Code Examples

```python
# sorts.py — four sorts from scratch. Each returns/produces ascending order.
import random

def bubble_sort(a):
    """O(n^2). Early-exit flag gives O(n) on already-sorted input."""
    n = len(a)
    for sweep in range(n - 1):
        swapped = False
        # After each sweep, the last `sweep` items are final — skip them.
        for i in range(n - 1 - sweep):
            if a[i] > a[i + 1]:
                a[i], a[i + 1] = a[i + 1], a[i]     # swap adjacent pair
                swapped = True
        if not swapped:            # a full sweep with no swaps = sorted
            break
    return a


def insertion_sort(a):
    """O(n^2) general, O(n) nearly-sorted. Sorts in place, stable."""
    for i in range(1, len(a)):
        key = a[i]                 # the card we're inserting
        j = i - 1
        # Shift larger elements right until key's spot opens up.
        # `>` (not >=) preserves stability: equal keys don't jump over.
        while j >= 0 and a[j] > key:
            a[j + 1] = a[j]
            j -= 1
        a[j + 1] = key
    return a


def merge_sort(a):
    """O(n log n) always, O(n) space, stable. Returns a NEW list."""
    if len(a) <= 1:                          # base case: trivially sorted
        return a[:]
    mid = len(a) // 2
    left = merge_sort(a[:mid])               # sort each half...
    right = merge_sort(a[mid:])
    return _merge(left, right)               # ...then zip them together

def _merge(left, right):
    """Merge two sorted lists in O(len(left) + len(right))."""
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:              # <= keeps it stable
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    result.extend(left[i:])                  # one of these is empty
    result.extend(right[j:])
    return result


def quick_sort(a, lo=0, hi=None):
    """Average O(n log n), in place. Random pivot avoids the sorted-input trap."""
    if hi is None:
        hi = len(a) - 1
    if lo >= hi:                             # 0 or 1 elements: done
        return a
    p = _partition(a, lo, hi)
    quick_sort(a, lo, p - 1)                 # left of pivot
    quick_sort(a, p + 1, hi)                 # right of pivot
    return a

def _partition(a, lo, hi):
    """Lomuto partition: put pivot in its final spot, return that spot."""
    r = random.randint(lo, hi)               # random pivot -> expected balance
    a[r], a[hi] = a[hi], a[r]                # stash pivot at the end
    pivot = a[hi]
    store = lo                               # boundary of the "< pivot" zone
    for i in range(lo, hi):
        if a[i] < pivot:
            a[i], a[store] = a[store], a[i]
            store += 1
    a[store], a[hi] = a[hi], a[store]        # pivot into final position
    return store


if __name__ == "__main__":
    data = [38, 27, 43, 3, 9, 82, 10, 3]
    print(bubble_sort(data[:]))
    print(insertion_sort(data[:]))
    print(merge_sort(data))
    print(quick_sort(data[:]))
    # Sanity: agree with the built-in on random data
    for _ in range(100):
        d = [random.randint(0, 99) for _ in range(50)]
        assert merge_sort(d) == sorted(d) == quick_sort(d[:])
    print("all sorts agree with sorted()")
```

Using the built-in like a professional — keys, ordering, stability:

```python
# sorting_idioms.py
people = [("Ada", 36), ("Bob", 25), ("Cy", 36), ("Dee", 25)]

by_age = sorted(people, key=lambda p: p[1])            # sort by one field
by_age_desc = sorted(people, key=lambda p: p[1], reverse=True)

# Multi-key: age ascending, then name descending — two routes:
combo = sorted(people, key=lambda p: (p[1], [-ord(c) for c in p[0]]))
# Cleaner: exploit STABILITY — sort by secondary key first, primary second.
tmp = sorted(people, key=lambda p: p[0], reverse=True)  # name desc
combo2 = sorted(tmp, key=lambda p: p[1])                # then age asc (stable!)

print(by_age)
print(combo2)   # [('Dee',25), ('Bob',25), ('Cy',36), ('Ada',36)]
```

## Common Pitfalls

**1. Quicksort with first-element pivot on sorted input.** Partition splits n into (0, n−1) every level → O(n²) time and O(n) recursion depth — a `RecursionError` on a few thousand *already sorted* items. Corrected: random pivot (as above) or median-of-three.

**2. Off-by-one in merge: using `<` instead of `<=`.** With `<`, equal elements from the right list jump ahead of equal elements from the left — the sort still sorts, but it's no longer stable. Silent, and it only matters until the day it really matters (multi-key sorting).

**3. Recursing without a shrinking base case.** `merge_sort` on `len(a) <= 1` must return; recursing on a slice equal to the input (e.g., `mid = 0` from a wrong formula) loops forever. If your merge sort hangs, print `len(a)` at entry — it should strictly decrease.

**4. Mutating and returning inconsistently.** `sorted(a)` returns new and leaves `a` alone; `a.sort()` mutates and returns `None`. The classic bug:

```python
a = a.sort()      # a is now None!
# Corrected:
a.sort()          # in place, or:
a = sorted(a)     # new list
```

**5. Comparing incomparable types / relying on default JS sort.** Python 3 raises on `sorted([3, "1"])` — good. JavaScript silently string-sorts numbers: `[10, 9, 1].sort()` → `[1, 10, 9]`. Always pass a comparator in JS: `.sort((a, b) => a - b)`.

**6. Sorting when a scan would do.** Finding the max is O(n); `sorted(a)[-1]` is O(n log n) plus a copy. Sorting is a powerful hammer — check whether the problem needs total order or just an extreme (heaps, Chapter 10, handle "top-k" better).

## Practice Exercises

1. Add a counter of comparisons to bubble, insertion, and merge sort. Run each on random, sorted, and reverse-sorted inputs of size 1,000 and tabulate the counts. Which results match the best/worst-case table, and which surprised you?
2. Implement **selection sort** (repeatedly select the minimum of the unsorted region and swap it into place). Show with a concrete 4-element example that the swap makes it unstable, and state its best-case complexity (it's not O(n) — why?).
3. Write `merge_sort_inplace_hybrid(a)`: merge sort that switches to insertion sort for subarrays of length ≤ 16. Benchmark against your plain merge sort on 100,000 random integers. Explain why the hybrid wins despite identical Big-O.
4. Implement **counting sort** for integers in a known range 0..k: O(n + k), no comparisons. Then explain precisely why it doesn't contradict the O(n log n) comparison-sort lower bound, and why it's a poor choice when k ≫ n.
5. Given a list of intervals like `[(1,4), (2,6), (8,10)]`, merge all overlapping intervals in O(n log n). (Hint: what should you sort by, and what single pass follows?) This sort-then-sweep shape reappears constantly in interviews.

---

**Next:** Chapter 8 — the payoff for sorted data: binary search, the O(log n) workhorse, and its surprisingly sharp edges.
