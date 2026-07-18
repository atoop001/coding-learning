# Chapter 6: Recursion (The Call Stack, Base Cases, and When to Use It)

## Overview

Recursion is a function calling itself to solve a smaller version of the same problem. It is not a party trick — it is the natural language for anything self-similar: trees (Chapter 9), divide-and-conquer sorting (Chapter 7), graph exploration (Chapter 11), and dynamic programming (Chapter 13) are all recursion wearing different hats. Learners often find recursion mystifying because they try to trace every call. The skill to build instead is the **recursive leap of faith**: assume the recursive call works for smaller inputs, and verify only that (a) you handle the smallest inputs directly and (b) you combine sub-results correctly. This chapter builds that skill and connects it to the call stack you met in Chapter 4.

## Definitions & Explanations

### The two mandatory parts

Every correct recursive function has:

1. **Base case(s)** — inputs so small the answer is returned directly, no recursion. This is the exit.
2. **Recursive case** — do a piece of work, then call yourself on a **strictly smaller** input that is guaranteed to reach a base case.

Miss the base case, or fail to shrink toward it, and you recurse forever — until Python stops you (`RecursionError`, default limit ~1000 frames).

### The call stack, concretely

Each call pushes a frame holding its own locals. `factorial(4)`:

```
call factorial(4)                     stack grows:      then unwinds:
  -> calls factorial(3)
    -> calls factorial(2)             |fact(1)| ret 1
      -> calls factorial(1)  BASE     |fact(2)|  <- 2*1 = 2
                                      |fact(3)|  <- 3*2 = 6
                                      |fact(4)|  <- 4*6 = 24
                                      +-------+
```

Two consequences:

- **Space cost.** A recursion `n` levels deep uses O(n) stack space *even if it allocates nothing else*. Depth is your hidden space complexity.
- **Work can happen on the way down or the way up.** `print` before the recursive call → descending order of calls; after → ascending as the stack unwinds. Knowing which side your work sits on is half of understanding any recursive function.

### Recursion vs iteration

Anything recursive can be rewritten iteratively (worst case: manage an explicit stack yourself — literally simulating the call stack) and vice versa. Choose by fit:

| Prefer recursion | Prefer iteration |
|---|---|
| Branching structures (trees, nested data) | Linear passes (sums, scans) |
| Divide & conquer (merge sort, binary search) | Very deep linear recursions (depth ≈ n) |
| Backtracking / "try all options" | Hot loops where frame overhead matters |

Python note: CPython does **not** optimize tail calls, so a linear recursion of depth 100,000 crashes where a loop is fine. JavaScript engines in practice don't reliably optimize tail calls either. When depth is O(n) and n is big, convert to a loop or an explicit stack.

### Analyzing recursive complexity

Count: (calls made) × (work per call), or draw the call tree.

- `factorial(n)`: n calls × O(1) each → **O(n)** time, O(n) stack space.
- Binary search (Chapter 8): halves each call → **O(log n)** calls.
- Naive `fib(n)`: each call spawns two → a tree of ~2ⁿ calls → **O(2ⁿ)**. (Chapter 13 fixes this with memoization.)

```
fib(5) call tree — note fib(2) is computed THREE times:
                    fib(5)
              /              \
          fib(4)              fib(3)
         /      \            /      \
     fib(3)    fib(2)     fib(2)   fib(1)
     /    \
  fib(2) fib(1)
```

## Code Examples

```python
# recursion_basics.py — the canon, annotated.

def factorial(n):
    """n! = n * (n-1)!  Time O(n), stack space O(n)."""
    if n <= 1:                # BASE: smallest input answered directly
        return 1
    return n * factorial(n - 1)   # RECURSE on strictly smaller input


def sum_list(items):
    """Sum via 'first element + sum of the rest'.
    Teaching example — in real code use sum() or a loop (depth = n!)."""
    if not items:                       # BASE: empty list sums to 0
        return 0
    return items[0] + sum_list(items[1:])   # NOTE: slice copies — see pitfalls


def reverse_string(s):
    """reverse(s) = reverse(everything after first char) + first char."""
    if len(s) <= 1:
        return s
    return reverse_string(s[1:]) + s[0]


def count_leaves(nested):
    """Where recursion earns its keep: arbitrarily nested lists.
    A loop can't do this cleanly — the DATA is recursive, so the code is.
    count_leaves([1, [2, [3, 4]], 5]) -> 5"""
    if not isinstance(nested, list):    # BASE: a non-list is one leaf
        return 1
    return sum(count_leaves(item) for item in nested)


def hanoi(n, source, target, spare):
    """Towers of Hanoi: move n disks from source to target.
    Move n-1 out of the way, move the big one, move n-1 back on top.
    Exactly 2^n - 1 moves — inherently exponential, and provably minimal."""
    if n == 0:
        return
    hanoi(n - 1, source, spare, target)         # clear the way (down)
    print(f"disk {n}: {source} -> {target}")    # the one real move
    hanoi(n - 1, spare, target, source)         # restack (up)


def permutations(items):
    """Backtracking: all orderings of items. n! results, so only for small n.
    Pattern: choose -> recurse -> unchoose."""
    result = []

    def backtrack(current, remaining):
        if not remaining:                       # BASE: nothing left to place
            result.append(current[:])           # snapshot (copy!) the answer
            return
        for i in range(len(remaining)):
            current.append(remaining[i])                        # choose
            backtrack(current, remaining[:i] + remaining[i+1:]) # explore
            current.pop()                                       # unchoose

    backtrack([], items)
    return result


if __name__ == "__main__":
    print(factorial(10))                  # 3628800
    print(sum_list([1, 2, 3, 4]))         # 10
    print(reverse_string("stack"))        # kcats
    print(count_leaves([1, [2, [3, 4]], 5]))   # 5
    hanoi(3, "A", "C", "B")               # 7 moves
    print(permutations([1, 2, 3]))        # 6 orderings
```

Converting recursion to iteration when depth threatens the stack:

```python
# depth_control.py

def sum_list_iterative(items):
    """The loop version: O(1) stack space, works on any length."""
    total = 0
    for x in items:
        total += x
    return total

def count_leaves_iterative(nested):
    """Recursion on branching data, converted with an EXPLICIT stack.
    This is exactly what the call stack was doing for us."""
    count = 0
    stack = [nested]                     # our own stack replaces call frames
    while stack:
        item = stack.pop()
        if isinstance(item, list):
            stack.extend(item)           # 'recurse' into children later
        else:
            count += 1
    return count

if __name__ == "__main__":
    deep = list(range(100_000))
    print(sum_list_iterative(deep))          # fine
    # sum_list(deep) would raise RecursionError — depth 100,000 > limit
    print(count_leaves_iterative([1, [2, [3, 4]], 5]))   # 5
```

JavaScript comparison — identical shape:

```javascript
function countLeaves(nested) {
  if (!Array.isArray(nested)) return 1;             // base case
  return nested.reduce((acc, x) => acc + countLeaves(x), 0);
}
```

## Common Pitfalls

**1. A base case that some inputs never reach.**

```python
# Bug: countdown(3.5) or countdown(-1) recurses forever (n == 0 never true).
def countdown(n):
    if n == 0:
        return
    print(n)
    countdown(n - 1)

# Corrected — base case as an inequality catches overshoot:
def countdown(n):
    if n <= 0:
        return
    print(n)
    countdown(n - 1)
```

**2. Forgetting to shrink the input.** `return helper(items)` instead of `return helper(items[1:])` — infinite recursion with no progress. Every recursive call must move measurably closer to a base case.

**3. Hidden O(n) work per call via slicing.** `sum_list(items[1:])` copies the remaining list every call: O(n²) time and memory churn for an O(n) job. Corrected: pass an index instead of slicing:

```python
def sum_list_fast(items, i=0):
    if i == len(items):
        return 0
    return items[i] + sum_list_fast(items, i + 1)
```

**4. Appending a shared mutable object to results.** In `permutations`, `result.append(current)` (no copy) stores the *same list object* every time — and later `pop()` calls mutate your saved answers into empty lists. The `current[:]` snapshot is load-bearing. This bug appears in virtually every first backtracking attempt.

**5. Forgetting to return the recursive result.**

```python
# Bug: recursion happens, result is computed... and dropped. Returns None.
def find(node, target):
    if node is None:
        return None
    if node.value == target:
        return node
    find(node.next, target)          # <- missing `return`

# Corrected:
    return find(node.next, target)
```

**6. Raising `sys.setrecursionlimit` as a fix.** It trades an exception for a possible interpreter crash and signals the wrong design. If depth is linear in input size, restructure to a loop or explicit stack instead.

## Practice Exercises

1. Write recursive `power(base, exp)` for non-negative integer `exp` two ways: the O(exp) version, and the O(log exp) fast version using `power(b, e//2)` squared (handle odd `e`). Verify both against `base ** exp`.
2. Write `flatten(nested)` that turns `[1, [2, [3, [4]]], 5]` into `[1, 2, 3, 4, 5]` recursively. Then write the iterative explicit-stack version and make it preserve order (the naive `stack.extend` version reverses — why?).
3. Write `binary_strings(n)` returning all strings of length n made of '0' and '1', using the choose/explore/unchoose pattern. How many results are there, and what does that say about the time complexity floor?
4. Trace `hanoi(4, "A", "C", "B")` **without running it**: predict the first four moves and the total move count, then run to confirm. Explain where in the call tree the very first printed move comes from.
5. Write `is_palindrome(s)` recursively with O(1) work per call and no slicing (use two indices). State its time and stack-space complexity, then convert it to a loop and compare.

---

**Next:** Chapter 7 puts recursion to work on the most-studied problem in computing: sorting — from intuition-builders to the divide-and-conquer workhorses.
