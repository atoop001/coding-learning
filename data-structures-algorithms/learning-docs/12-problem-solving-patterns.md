# Chapter 12: Common Problem-Solving Patterns

## Overview

By now you have the structures. This chapter is about *moves* — a small set of reusable techniques that each collapse a whole category of O(n²) brute-force solutions into O(n) or O(n log n). Interviewers rarely invent new algorithms; they dress up these patterns in costumes. The skill being tested is recognition: reading a problem and thinking "sorted input, pair target → two pointers" or "contiguous subarray, max/min/count → sliding window." This chapter covers the three highest-yield patterns — **two pointers**, **sliding window**, **frequency counting** — plus two supporting moves, **prefix sums** and **sort-then-sweep**, and ends with a recognition table you should internalize.

## Definitions & Explanations

### Pattern 1: Two pointers

Two indices moving through data with *purpose*, replacing a nested loop. Three sub-flavors:

**Converging** (ends → middle): for pair-finding in **sorted** arrays and symmetry checks. The insight: in a sorted array, if `a[lo] + a[hi]` is too small, only moving `lo` right can help; too big, only moving `hi` left can. Each step permanently discards one candidate → O(n).

```
target 12 in sorted [1, 3, 4, 6, 8, 11]:
 lo=1, hi=11: 12 == 12  ✓ found
 (had it been 10: 1+11=12>10 -> hi--; 1+8=9<10 -> lo++; 3+8=11>10 -> hi--; 3+6=9... )
```

**Parallel / fast-slow**: same direction, different speeds — cycle detection and middle-finding from Chapter 3, and the *writer/reader* variant for in-place filtering (reader scans, writer marks the boundary of kept elements).

**Merging**: one pointer per sorted sequence, always advancing the smaller — the merge from Chapter 7.

### Pattern 2: Sliding window

For problems about **contiguous** runs (subarrays/substrings): max sum of a window, longest substring with some property. Maintain a window `[left, right]` and a running summary of its contents; extend `right` to grow, advance `left` to restore validity. Each pointer only moves forward → O(n) total, versus O(n²)+ for re-scanning every subarray.

- **Fixed-size window**: slide by adding the entering element and subtracting the leaving one — never recompute from scratch.
- **Variable-size window**: grow until a constraint breaks, then shrink from the left until it holds again ("caterpillar" motion).

```
longest substring without repeats: "abcabcbb"
 a b c a ...      window [a b c], right hits second 'a'
 ^     ^          -> shrink left past first 'a', window [b c a], continue
```

The window summary is usually a counter dict or set (Chapter 5) so membership/violation checks are O(1).

### Pattern 3: Frequency counting

Count occurrences into a hash map, then reason about the counts instead of the raw data. Anagram checks, majority elements, "first unique," top-k-frequent (counts + heap, Chapter 10), grouping by canonical key. Almost always turns O(n²) comparisons into O(n) counting + O(n) analysis. Python gives you `collections.Counter`; knowing it *is* a dict subclass with arithmetic (`c1 - c2`, `c.most_common(k)`) makes many problems one-liners you can still explain from first principles.

### Supporting move: Prefix sums

Precompute `prefix[i]` = sum of the first i elements (O(n) once); then any range sum is `prefix[j] - prefix[i]` in O(1). Combined with a hash map of "prefix values seen so far," it answers "how many subarrays sum to k?" in one pass — the same *seen-before* trick as two-sum (Chapter 5), applied to running totals.

### Supporting move: Sort, then sweep

When order isn't given, consider buying it for O(n log n): after sorting, duplicates are adjacent, intervals can be merged left-to-right, pairs succumb to converging pointers, and greedy choices become safe. The interval-merge exercise from Chapter 7 is this pattern.

### Recognition table

| Signal in the problem | Reach for |
|---|---|
| Sorted array + find pair/triple with target | Converging two pointers |
| In-place remove/compact/partition | Writer/reader two pointers |
| "Contiguous subarray/substring" + max/min/longest/count | Sliding window |
| Anagrams, "appears k times," top-k frequent | Frequency counting (+ heap for top-k) |
| Many range-sum queries / "subarray summing to k" | Prefix sums (+ hash map) |
| Intervals, meetings, overlaps | Sort then sweep |
| "Seen before?" anywhere | Hash set |
| Unsorted but only need k extremes | Heap (Chapter 10) |

## Code Examples

```python
# patterns.py — one worked example per pattern, brute force vs pattern.
from collections import Counter, defaultdict

# ---- Two pointers (converging) ------------------------------------------
def pair_sum_brute(a, target):
    """O(n^2): every pair."""
    for i in range(len(a)):
        for j in range(i + 1, len(a)):
            if a[i] + a[j] == target:
                return (a[i], a[j])
    return None

def pair_sum_sorted(a, target):
    """O(n) on SORTED input: each comparison discards one end forever."""
    lo, hi = 0, len(a) - 1
    while lo < hi:
        s = a[lo] + a[hi]
        if s == target:
            return (a[lo], a[hi])
        elif s < target:
            lo += 1                    # smallest element can't be in any answer
        else:
            hi -= 1                    # largest element can't be in any answer
    return None

# ---- Two pointers (writer/reader) ---------------------------------------
def remove_value_in_place(a, val):
    """Compact `a`, dropping val, in O(n)/O(1). Returns new length.
    write = boundary of the kept region; read scans everything."""
    write = 0
    for read in range(len(a)):
        if a[read] != val:
            a[write] = a[read]
            write += 1
    return write                       # a[:write] is the answer

# ---- Sliding window (fixed size) ----------------------------------------
def max_window_sum(a, k):
    """Best sum of k consecutive elements. O(n): add entering, drop leaving."""
    if len(a) < k:
        return None
    window = sum(a[:k])                # first window: computed once
    best = window
    for right in range(k, len(a)):
        window += a[right] - a[right - k]      # slide in O(1)
        best = max(best, window)
    return best

# ---- Sliding window (variable size) -------------------------------------
def longest_unique_substring(s):
    """Longest run of distinct characters. O(n): both pointers only advance."""
    last_seen = {}                     # char -> most recent index
    left = best = 0
    for right, ch in enumerate(s):
        if ch in last_seen and last_seen[ch] >= left:
            left = last_seen[ch] + 1   # jump past the previous occurrence
        last_seen[ch] = right
        best = max(best, right - left + 1)
    return best

# ---- Frequency counting --------------------------------------------------
def is_anagram(s, t):
    """O(n) via counts. (sorted(s) == sorted(t) works too, at O(n log n).)"""
    return Counter(s) == Counter(t)

def top_k_frequent(items, k):
    """Counter + most_common: counting O(n), selection O(m log m) for m
    distinct values (heap-based; Chapter 10's top-k under the hood)."""
    return [x for x, _ in Counter(items).most_common(k)]

# ---- Prefix sums + hash map ---------------------------------------------
def count_subarrays_summing_to(a, k):
    """How many contiguous subarrays sum to k — O(n), handles negatives.
    running - k seen before  <=>  some subarray ending here sums to k."""
    seen = defaultdict(int)
    seen[0] = 1                        # empty prefix: subarray starting at 0
    running = count = 0
    for x in a:
        running += x
        count += seen[running - k]     # every earlier matching prefix = 1 subarray
        seen[running] += 1
    return count

# ---- Sort then sweep ------------------------------------------------------
def min_meeting_rooms(intervals):
    """Fewest rooms for all meetings: sweep sorted start/end events. O(n log n)."""
    starts = sorted(s for s, _ in intervals)
    ends = sorted(e for _, e in intervals)
    rooms = best = 0
    i = j = 0
    while i < len(starts):
        if starts[i] < ends[j]:        # a meeting starts before earliest end
            rooms += 1; i += 1
            best = max(best, rooms)
        else:                          # a meeting ended: free a room
            rooms -= 1; j += 1
    return best


if __name__ == "__main__":
    a = [1, 3, 4, 6, 8, 11]
    assert pair_sum_sorted(a, 12) == pair_sum_brute(a, 12) == (1, 11)
    nums = [3, 1, 3, 5, 3]
    n = remove_value_in_place(nums, 3)
    print(nums[:n])                                    # [1, 5]
    print(max_window_sum([2, 1, 5, 1, 3, 2], 3))       # 9
    print(longest_unique_substring("abcabcbb"))        # 3
    print(is_anagram("listen", "silent"))              # True
    print(top_k_frequent("aabbbcc", 2))                # ['b', then a or c]
    print(count_subarrays_summing_to([1, 2, 3, -3, 3], 3))   # 4
    print(min_meeting_rooms([(0, 30), (5, 10), (15, 20)]))   # 2
```

JavaScript flavor of the variable window — the pattern is language-independent:

```javascript
function longestUniqueSubstring(s) {
  const lastSeen = new Map();
  let left = 0, best = 0;
  for (let right = 0; right < s.length; right++) {
    const ch = s[right];
    if (lastSeen.has(ch) && lastSeen.get(ch) >= left) left = lastSeen.get(ch) + 1;
    lastSeen.set(ch, right);
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

## Common Pitfalls

**1. Two pointers on unsorted data.** The converging argument ("too small → only lo helps") is a theorem *about sorted arrays*. On unsorted input it silently returns wrong answers. Either sort first (losing original indices — capture them if needed) or use the hash-map two-sum (Chapter 5).

**2. Shrinking the window without updating the summary.**

```python
# Bug: left advances but the counts dict still includes the evicted char.
while violates(counts):
    left += 1

# Corrected — evict THEN advance, keeping summary and window in sync:
while violates(counts):
    counts[s[left]] -= 1
    if counts[s[left]] == 0:
        del counts[s[left]]
    left += 1
```

The window summary must be a perfect mirror of `s[left:right+1]` at all times — most sliding-window bugs are a broken mirror.

**3. Recomputing the window from scratch each slide.** `sum(a[i:i+k])` inside the loop turns O(n) back into O(nk). The entire point is the incremental add-one-drop-one update.

**4. Sliding window on problems it can't solve.** The shrink step assumes *monotonicity*: growing the window moves the property in one direction. With negative numbers, "window sum too big → shrink" is invalid (shrinking might increase the sum). That's why `count_subarrays_summing_to` uses prefix sums + hash map instead of a window. Check monotonicity before reaching for the window.

**5. `left = last_seen[ch] + 1` without the `>= left` guard.** In `longest_unique_substring`, a stale entry from *before* the current window would drag `left` backward, breaking the both-pointers-only-advance invariant (and the O(n) bound). Stale state is the price of never cleaning the map; the guard is the payment.

**6. Missing `seen[0] = 1` in prefix-sum counting.** Without it, subarrays that start at index 0 are never counted — an off-by-one that passes casual tests and fails `[3], k=3`.

## Practice Exercises

1. `three_sum(a)`: all unique triples summing to zero. Sort, fix one element, converge two pointers on the rest — O(n²) with careful duplicate-skipping. List the three places duplicates must be skipped.
2. `min_subarray_len(a, target)`: length of the shortest contiguous subarray of *positive* integers with sum ≥ target (0 if none) — O(n) variable window. Then explain, with a concrete counterexample, why the same code is wrong if negatives are allowed.
3. `find_all_anagrams(s, p)`: every start index in `s` where an anagram of `p` begins — fixed window of size `len(p)` plus a frequency map, O(n). Make the per-slide update O(1), not a Counter rebuild.
4. `products_except_self(a)`: output[i] = product of all elements except a[i], O(n) time, no division. Hint: it's prefix sums with × — a prefix pass and a suffix pass.
5. `car_pool(trips, capacity)`: trips are `(passengers, start, end)`; can one vehicle serve them all? Solve two ways — sort-then-sweep over events, and a difference-array (prefix-sum inverse) over stops — and compare complexities as a function of trip count vs route length.

---

**Next:** Chapter 13 — dynamic programming: what to do when the subproblems overlap and brute-force recursion explodes.
