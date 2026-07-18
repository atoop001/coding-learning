# Chapter 13: Dynamic Programming — An Introduction

## Overview

Dynamic programming (DP) has a fearsome reputation it doesn't deserve. It is one idea: **when a recursive solution solves the same subproblem many times, save each answer the first time and reuse it.** That's it. The naive `fib` from Chapter 6 took O(2ⁿ) because it recomputed `fib(2)` millions of times; caching turns it into O(n). DP problems are common in interviews (usually as the "hard" question), and — more importantly — the DP habit of asking *"what smaller version of this problem, if already solved, would make the current one easy?"* is one of the deepest problem-solving skills in this track. This chapter builds DP from recursion you already know, in two styles: memoization (top-down) and tabulation (bottom-up), through four classic problems.

## Definitions & Explanations

### When DP applies: the two ingredients

1. **Overlapping subproblems** — the recursion tree contains the same call repeatedly. (`fib(n-1)` and `fib(n-2)` both need `fib(n-3)`.) If subproblems are all distinct (merge sort's halves), plain divide-and-conquer is enough; caching buys nothing.
2. **Optimal substructure** — the best answer to the whole is composed from best answers to parts. ("Cheapest path to D via C = cheapest path to C + cost C→D.")

Both present → DP collapses exponential trees into polynomial tables.

### Style 1: Memoization (top-down)

Write the natural recursion, then add a cache keyed by the arguments. First call computes and stores; repeats return instantly.

```
fib(5) WITHOUT memo: ~15 calls        WITH memo: each value computed once
        fib(5)                              fib(5)
       /      \                            /      \
    fib(4)    fib(3)                    fib(4)   [cache hit]
    /    \    /    \                    /    \
 fib(3) fib(2) ...duplicates...      fib(3)  [cache hit]
                                      ...
  O(2^n)  ->                          O(n) calls
```

Pros: mechanical transformation from the brute-force recursion; only computes subproblems actually needed. Cons: recursion depth limits (Chapter 6), per-call overhead.

### Style 2: Tabulation (bottom-up)

Kill the recursion: figure out the dependency order (small subproblems first), fill a table iteratively.

```python
dp[0], dp[1] = 0, 1
for i in range(2, n + 1):
    dp[i] = dp[i-1] + dp[i-2]
```

Pros: no recursion limit, often less overhead, opens space optimizations (if `dp[i]` needs only the last two entries, keep two variables → O(1) space). Cons: you must know the fill order; computes every subproblem in range.

### The 4-step recipe (works for both styles)

1. **Define the state in one sentence.** "`dp[i]` = the length of the longest increasing subsequence *ending at* index i." If you can't phrase it, stop and rethink — most DP failures are state-definition failures, not coding failures.
2. **Write the recurrence.** How does `dp[i]` combine earlier states? (`dp[i] = 1 + max(dp[j] for j < i if a[j] < a[i])`)
3. **Set base cases.** The subproblems answerable without recursion. Get these wrong and everything downstream is wrong.
4. **Determine the answer's location.** Sometimes `dp[n]`, sometimes `max(dp)` — depends on whether the state says "using the first i" or "ending exactly at i."

Complexity = (number of states) × (work per state). LIS below: n states × O(n) each = O(n²).

### Recognizing DP in the wild

Signals: "count the number of ways...", "minimum/maximum cost to reach...", "longest/shortest sequence such that...", "can you form X from...", choices at each step with consequences later. Counter-signal: if greedy local choices provably work (coin change with US coins), DP is overkill — but note that greedy *fails* for general coin systems (coins `[1, 3, 4]`, target 6: greedy picks 4+1+1, best is 3+3), which is exactly why the DP version exists.

## Code Examples

```python
# dp_intro.py — four classics, each tying recursion to a table.
from functools import lru_cache

# ---- 1. Fibonacci: the hello-world of DP ---------------------------------
def fib_naive(n):
    """O(2^n). Try fib_naive(35) and feel the exponential."""
    if n <= 1:
        return n
    return fib_naive(n - 1) + fib_naive(n - 2)

def fib_memo(n, cache=None):
    """Top-down: same recursion + a dict. O(n) time, O(n) space."""
    if cache is None:
        cache = {}
    if n <= 1:
        return n
    if n not in cache:                      # compute only on first request
        cache[n] = fib_memo(n - 1, cache) + fib_memo(n - 2, cache)
    return cache[n]

def fib_table(n):
    """Bottom-up with O(1) space: only the last two values matter."""
    if n <= 1:
        return n
    prev2, prev1 = 0, 1
    for _ in range(2, n + 1):
        prev2, prev1 = prev1, prev2 + prev1
    return prev1

# ---- 2. Climbing stairs: counting paths ----------------------------------
def climb_stairs(n):
    """Ways to climb n steps taking 1 or 2 at a time.
    State: dp[i] = ways to reach step i.
    Recurrence: dp[i] = dp[i-1] + dp[i-2]  (arrive via a 1-step or a 2-step).
    It's Fibonacci wearing a costume — recognizing THAT is the skill."""
    if n <= 2:
        return n
    prev2, prev1 = 1, 2
    for _ in range(3, n + 1):
        prev2, prev1 = prev1, prev2 + prev1
    return prev1

# ---- 3. Coin change: minimization where greedy fails ---------------------
def coin_change(coins, amount):
    """Fewest coins totaling `amount`, or -1.
    State: dp[a] = fewest coins to make amount a.
    Recurrence: dp[a] = 1 + min(dp[a - c] for usable c).
    Base: dp[0] = 0. Time O(amount * len(coins)), space O(amount)."""
    INF = float("inf")
    dp = [0] + [INF] * amount
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a and dp[a - c] + 1 < dp[a]:
                dp[a] = dp[a - c] + 1
    return dp[amount] if dp[amount] != INF else -1

# ---- 4. Longest increasing subsequence: 2D thinking in 1D ----------------
def longest_increasing_subsequence(a):
    """Length of the longest strictly-increasing subsequence (not contiguous).
    State: dp[i] = length of the LIS ENDING exactly at index i.
    Recurrence: dp[i] = 1 + max(dp[j] : j < i and a[j] < a[i]), else 1.
    Answer: max(dp) — the LIS can end anywhere. O(n^2)."""
    if not a:
        return 0
    dp = [1] * len(a)
    for i in range(1, len(a)):
        for j in range(i):
            if a[j] < a[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)

# ---- Bonus: memoization for free -----------------------------------------
@lru_cache(maxsize=None)
def grid_paths(rows, cols):
    """Ways to walk a grid corner-to-corner moving only right/down.
    @lru_cache = Python's built-in memoizer (args must be hashable!)."""
    if rows == 1 or cols == 1:
        return 1
    return grid_paths(rows - 1, cols) + grid_paths(rows, cols - 1)


if __name__ == "__main__":
    assert fib_memo(40) == fib_table(40) == 102334155
    print("stairs(10):", climb_stairs(10))                     # 89
    print("coins [1,3,4] -> 6:", coin_change([1, 3, 4], 6))    # 2 (3+3, beats greedy's 3)
    print("LIS:", longest_increasing_subsequence([10, 9, 2, 5, 3, 7, 101, 18]))  # 4
    print("grid 3x7:", grid_paths(3, 7))                       # 28
```

Recovering the actual answer (not just its value) — interviewers ask this follow-up:

```python
def coin_change_with_coins(coins, amount):
    """Same DP + a `choice` table remembering WHICH coin won each state.
    Walk choices backward to reconstruct the coin list."""
    INF = float("inf")
    dp = [0] + [INF] * amount
    choice = [None] * (amount + 1)
    for a in range(1, amount + 1):
        for c in coins:
            if c <= a and dp[a - c] + 1 < dp[a]:
                dp[a] = dp[a - c] + 1
                choice[a] = c                  # remember the winning move
    if dp[amount] == INF:
        return None
    result, a = [], amount
    while a > 0:
        result.append(choice[a])
        a -= choice[a]
    return result                              # e.g. [3, 3] for coins [1,3,4], 6

print(coin_change_with_coins([1, 3, 4], 6))
```

JavaScript memoization for comparison:

```javascript
function fibMemo(n, cache = new Map()) {
  if (n <= 1) return n;
  if (!cache.has(n)) cache.set(n, fibMemo(n - 1, cache) + fibMemo(n - 2, cache));
  return cache.get(n);
}
```

## Common Pitfalls

**1. The mutable default argument trap.**

```python
# Bug: cache={} as a default is created ONCE and shared across ALL calls —
# here it accidentally "works" but leaks memory forever, and for functions
# whose answers depend on more context it returns stale wrong answers.
def fib(n, cache={}): ...

# Corrected — the None-sentinel idiom used in fib_memo above:
def fib(n, cache=None):
    if cache is None:
        cache = {}
```

**2. Caching on incomplete state.** The cache key must include *everything* the answer depends on. Memoizing a maze-walker on position only, while the answer also depends on collected keys, returns wrong hits. Symptom: memoized answers differ from brute force on small inputs — always diff-test against brute force.

**3. Fuzzy state definitions.** "dp[i] = best so far" is not a definition. *Ending at i* vs *within the first i* changes the recurrence AND where the answer lives (`dp[n-1]` vs `max(dp)`). LIS with "within first i" semantics but an "ending at i" recurrence is the classic hybrid bug.

**4. Wrong base cases / wrong fill order.** `dp[0] = 1` vs `dp[0] = 0` flips counting problems entirely (empty-way conventions!). In 2D tabulation, filling row-major when the recurrence needs the cell below is reading uninitialized garbage. Rule: a state may only be read after every state it depends on is final.

**5. Missing the "impossible" representation.** `coin_change` needs INF (or None) for unreachable amounts; initializing dp to 0 makes unreachable states look free and poisons everything above them. Decide upfront how "no solution" is encoded and test an impossible input (`coins=[2], amount=3`).

**6. Reaching for DP when subproblems don't overlap — or greedy suffices.** Memoized merge sort caches nothing (all subarrays distinct) and just wastes memory. And for problems with a provable greedy (activity selection, US-coin change), DP is a slower correct answer — in interviews, mention the greedy and why it works or fails.

## Practice Exercises

1. `house_robber(values)`: max sum of non-adjacent elements. Define the state in one written sentence *before* coding, implement both memoized and tabulated versions, then reduce space to O(1). Verify all three agree on 200 random inputs vs brute force.
2. `count_ways_coin_change(coins, amount)`: count *combinations* (order irrelevant) that make the amount. The loop order (coins outer vs amount outer) is the difference between counting combinations and permutations — implement both, and explain with `coins=[1,2], amount=3` which is which.
3. `min_path_sum(grid)`: min sum path from top-left to bottom-right moving only right/down. Tabulate in 2D, then reduce to one row of space. Follow-up: reconstruct the path.
4. `can_partition(nums)`: can the list split into two equal-sum halves? Reduce it to a subset-sum DP over amounts 0..total/2 and state the complexity — then explain in a sentence why this is called "pseudo-polynomial."
5. `edit_distance(a, b)`: minimum insertions/deletions/substitutions turning `a` into `b`. State: `dp[i][j]` = distance between the first i chars of `a` and first j of `b`. Work the recurrence out on paper with "cat" → "cut" before coding, and check `dp` matches your hand table.

---

**Next:** Chapter 14 — putting it all together: how to actually perform in interviews, and a practice plan that builds real skill instead of memorized answers.
