# Chapter 8: Searching & Binary Search

## Overview

Searching is the most common thing programs do, and there are exactly two speeds: O(n) when the data has no exploitable structure, and O(log n) when it's sorted. Binary search — repeatedly halving the search space — is deceptively simple to describe and famously easy to get wrong: a published study found bugs in a majority of professional implementations, and Java's own library binary search carried an overflow bug for nine years. This chapter makes you one of the people who writes it correctly, and — the bigger prize — teaches the *generalized* form ("binary search on the answer") that solves whole families of interview problems that don't look like searching at all.

## Definitions & Explanations

### Linear search — the baseline

Scan until found: O(n), works on anything, unbeatable on unsorted data (you can't know an element is absent without looking everywhere). `x in my_list` is exactly this.

### Binary search — the halving game

Precondition: **the data is sorted**. Compare the target to the middle element; that one comparison eliminates half the remaining candidates. Repeat.

```
target = 33 in [8, 13, 21, 33, 42, 57, 91]

 lo                 mid                  hi
 [8,  13,  21,  33,  42,  57,  91]   33 < 42 -> discard right half
 lo   mid        hi
 [8,  13,  21,  33]                  33 > 13 -> discard left half
           lo/mid  hi
          [21, 33]                   33 > 21 -> discard left
               [33]                  found at index 3
```

7 elements → 3 comparisons. A billion elements → 30. That's log₂(n).

### The invariant mindset

Correct binary search is about maintaining an **invariant** — a statement that stays true every iteration. For the classic version: *"if the target exists, its index is within [lo, hi]."* Every line either preserves that invariant or shrinks the window. When `lo > hi`, the window is empty → not present. Bugs come from breaking the invariant (wrong boundary update) or failing to shrink (infinite loop).

### Finding boundaries, not just membership

Real problems usually need more than "is it there?":

- **Leftmost position** (`bisect_left`): first index where the target could be inserted keeping order — also "index of the first element ≥ target."
- **Rightmost position** (`bisect_right`): last such insertion point — "index of the first element > target."

With duplicates `[5, 7, 7, 7, 9]` and target 7: `bisect_left` → 1, `bisect_right` → 4, and `right - left` counts the 7s in O(log n).

### Binary search on the answer

The generalization: you don't need a sorted *array* — you need any **monotonic yes/no predicate** over an ordered range of candidates. If `can(x)` is false-false-false-true-true-true as x grows, binary search finds the boundary in O(log range) predicate calls.

Examples: smallest capacity that ships packages in D days; integer square root ("is m² ≤ n?"); first bad version in a release history. The skill is *spotting the monotone predicate*.

## Code Examples

```python
# searching.py — linear, binary (classic + boundary forms), and search-on-answer.

def linear_search(items, target):
    """O(n). The only option when data is unsorted."""
    for i, x in enumerate(items):
        if x == target:
            return i
    return -1


def binary_search(a, target):
    """Classic membership search on a SORTED list. O(log n).
    Invariant: if target is in `a`, its index is in [lo, hi]."""
    lo, hi = 0, len(a) - 1
    while lo <= hi:                        # window [lo, hi] still non-empty
        mid = (lo + hi) // 2               # Python ints don't overflow;
        if a[mid] == target:               # in C/Java use lo + (hi-lo)//2
            return mid
        elif a[mid] < target:
            lo = mid + 1                   # mid checked -> exclude it
        else:
            hi = mid - 1                   # mid checked -> exclude it
    return -1                              # window empty: not present


def bisect_left(a, target):
    """Index of the FIRST element >= target (insertion point, left).
    Different invariant: answer is in [lo, hi], hi starts at len(a).
    Never tests equality — it locates a boundary, not a value."""
    lo, hi = 0, len(a)
    while lo < hi:
        mid = (lo + hi) // 2
        if a[mid] < target:
            lo = mid + 1                   # a[mid] too small: answer is right of mid
        else:
            hi = mid                       # a[mid] >= target: mid could BE the answer
    return lo                              # lo == hi == the boundary


def bisect_right(a, target):
    """Index of the first element STRICTLY > target."""
    lo, hi = 0, len(a)
    while lo < hi:
        mid = (lo + hi) // 2
        if a[mid] <= target:               # only difference: <= vs <
            lo = mid + 1
        else:
            hi = mid
    return lo


def count_occurrences(a, target):
    """O(log n) count in a sorted list with duplicates."""
    return bisect_right(a, target) - bisect_left(a, target)


def isqrt(n):
    """Integer square root via binary search ON THE ANSWER.
    Predicate 'mid*mid <= n' is monotone: True...True False...False."""
    lo, hi = 0, n
    while lo < hi:
        mid = (lo + hi + 1) // 2           # bias UP when keeping lo=mid (see pitfalls)
        if mid * mid <= n:
            lo = mid                       # mid works; try bigger
        else:
            hi = mid - 1                   # mid too big
    return lo


def min_ship_capacity(weights, days):
    """Least capacity to ship all weights, in order, within `days` days.
    Classic 'binary search the answer': feasibility is monotone in capacity."""
    def can_ship(cap):
        used_days, load = 1, 0
        for w in weights:
            if load + w > cap:             # start a new day
                used_days += 1
                load = 0
            load += w
        return used_days <= days

    lo, hi = max(weights), sum(weights)    # answer must lie in this range
    while lo < hi:
        mid = (lo + hi) // 2
        if can_ship(mid):
            hi = mid                       # feasible: try smaller
        else:
            lo = mid + 1                   # infeasible: need bigger
    return lo


if __name__ == "__main__":
    a = [5, 7, 7, 7, 9, 12]
    print(binary_search(a, 9))             # 4
    print(bisect_left(a, 7), bisect_right(a, 7))    # 1 4
    print(count_occurrences(a, 7))         # 3
    print(isqrt(17))                       # 4
    print(min_ship_capacity([1,2,3,4,5,6,7,8,9,10], 5))   # 15
```

The standard library does this for you — know it exists, but be able to write it:

```python
import bisect
a = [5, 7, 7, 7, 9, 12]
bisect.bisect_left(a, 7)    # 1
bisect.bisect_right(a, 7)   # 4
bisect.insort(a, 8)         # O(log n) find + O(n) insert — insert cost still linear!
```

JavaScript has no built-in binary search — you write `bisect_left` yourself with the exact logic above (`Math.floor((lo + hi) / 2)`).

## Common Pitfalls

**1. Binary searching unsorted data.** No error is raised; you just get confidently wrong answers. The precondition is on you. If the data changes often, keeping it sorted for search has its own cost (see `insort` above: the *insert* is still O(n) — Chapter 9's trees fix this).

**2. The infinite loop: `lo = mid` with a floor midpoint.**

```python
# Bug: when hi == lo+1, mid == lo; if the branch keeps lo = mid,
# the window never shrinks. Loops forever.
while lo < hi:
    mid = (lo + hi) // 2
    if condition(mid):
        lo = mid          # <- window can stall
    else:
        hi = mid - 1

# Corrected — bias the midpoint UP whenever a branch assigns lo = mid:
    mid = (lo + hi + 1) // 2
```

Rule of thumb: `lo = mid` needs a round-up midpoint; `hi = mid` needs a round-down midpoint. `isqrt` above shows the round-up form in action.

**3. Off-by-one in boundary updates.** In the classic version, `mid` has been checked, so exclude it: `lo = mid + 1` / `hi = mid - 1`. In the boundary version, `hi = mid` (not `mid - 1`) because mid *could be the answer*. Mixing the two styles in one function is the most common way to lose a wedding-cake of test cases. Pick one template per problem and follow it entirely.

**4. Wrong initial `hi`.** Classic search: `hi = len(a) - 1` (inclusive window). Bisect style: `hi = len(a)` (exclusive; the answer can be "insert at the end"). Copying one style's initialization into the other's loop yields subtle misses at the array's edges.

**5. Overflow midpoint (for your JS/C/Java life).** `(lo + hi) / 2` overflows fixed-width integers when both are huge; write `lo + Math.floor((hi - lo) / 2)`. Python's arbitrary-precision ints make this a non-issue in Python — but interviewers ask about it.

**6. Using equality checks in boundary searches.** `bisect_left` never tests `a[mid] == target`, and adding an early-return equality "optimization" breaks its guarantee of returning the *leftmost* position. Boundary searches locate a frontier between False and True; equality is irrelevant to them.

## Practice Exercises

1. Implement `search_range(a, target)` returning the first and last index of `target` in a sorted list (or `(-1, -1)`), in O(log n), using your own `bisect_left`/`bisect_right`. Test on: absent target, single occurrence, all-duplicates array, empty array.
2. A sorted array was rotated at an unknown pivot: `[6, 7, 9, 1, 2, 4]`. Write O(log n) `find_min(a)` returning the smallest element. Invariant hint: compare `a[mid]` with `a[hi]` and reason about which half must contain the minimum.
3. Write `first_true(lo, hi, pred)` — a reusable "binary search the answer" helper returning the smallest x in [lo, hi] where the monotone predicate `pred(x)` is True (or None). Re-implement `isqrt` and `min_ship_capacity` on top of it.
4. `koko_speed(piles, hours)`: Koko eats bananas at speed k (one pile per hour, at most k bananas from it). Find the minimum integer k to finish all piles within `hours`. Identify the monotone predicate before writing any code, and state the overall complexity in terms of the pile sizes.
5. You have a `guess(x)` oracle answering "too low / too high / correct" but calls are expensive and n is unknown (unbounded above). Describe and implement the two-phase strategy — exponential search to bracket the answer, then binary search inside the bracket — and give its complexity in terms of the true answer's magnitude.

---

**Next:** Chapter 9 — trees: what you get when you want sorted-order queries AND fast inserts at the same time.
