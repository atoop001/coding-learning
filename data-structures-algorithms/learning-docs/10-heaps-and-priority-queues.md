# Chapter 10: Heaps & Priority Queues

## Overview

A **priority queue** is a queue where "next" means "most important," not "least recently added": hospital triage, OS task schedulers, Dijkstra's shortest paths (Chapter 11), merging sorted streams, "top 10" leaderboards. The data structure that implements it well is the **binary heap** — a tree that maintains just enough order to answer "what's the minimum?" in O(1) and fix itself in O(log n) after changes, while cleverly living inside a plain array with zero pointers. Heaps also give you heapsort and the single most common interview trick after hash maps: solving "top-k" problems in O(n log k).

## Definitions & Explanations

### The heap property (and what it is *not*)

A **min-heap** is a binary tree where every node's key ≤ its children's keys. That's the entire rule.

```
            (1)                 min-heap: every parent <= its children
           /   \
         (3)    (2)      <- NOTE: 3 > 2 across siblings — perfectly legal!
        /  \    /  \
      (7)  (4) (6)  (5)
```

Consequences:
- The minimum is always the root → **peek is O(1)**.
- Siblings are unordered; left vs right means nothing. A heap is NOT a BST — it maintains far less order, which is exactly why its operations are cheaper to maintain.
- A **max-heap** flips the inequality (parent ≥ children).

### Shape: complete trees, stored in arrays

Heaps are **complete** binary trees: every level full except possibly the last, which fills left to right. Complete trees pack perfectly into an array — no `None` gaps, no child pointers:

```
array:   [1, 3, 2, 7, 4, 6, 5]
index:    0  1  2  3  4  5  6

parent(i)      = (i - 1) // 2
left_child(i)  = 2*i + 1
right_child(i) = 2*i + 2
```

The tree drawn above *is* this array. Navigation is arithmetic — cache-friendly and allocation-free. Completeness also pins the height at ⌊log₂ n⌋, which is what makes every reheapify O(log n).

### The two repair moves

All heap operations are one of two "bubble" procedures that fix a single out-of-place element:

- **Sift up** (after inserting at the end): while the new element is smaller than its parent, swap upward. ≤ height swaps → O(log n).
- **Sift down** (after moving the last element into the root on pop): while larger than its smaller child, swap downward with that child. O(log n).

```
push(0):  [1,3,2,7,4,6,5,0]      pop():  root 1 leaves; last elem 5 -> root
   0 < parent 7: swap                    [5,3,2,7,4,6]
   0 < parent 3: swap                    5 > min(3,2)=2: swap down-right
   0 < parent 1: swap                    [2,3,5,7,4,6]  5 <= 6: stop
result: [0,1,2,3,4,6,5,7]
```

### Operation costs

| Operation | Cost | How |
|---|---|---|
| peek min | O(1) | root, index 0 |
| push | O(log n) | append + sift up |
| pop min | O(log n) | swap root/last, shrink, sift down |
| build heap from n items | **O(n)** | sift-down from the middle backward — better than n pushes! |
| search for arbitrary value | O(n) | heaps don't do lookup — wrong tool |

The O(n) build (heapify) surprises people: most nodes are near the bottom where sift-down is short; the costs sum to O(n), not O(n log n).

### The top-k pattern

"Find the k largest of n items": keep a **min**-heap of size k (yes, min — its root is the *weakest current member*, the one candidates must beat). Scan all items; if an item beats the root, replace the root. O(n log k) time, O(k) space — dominant over full sorting (O(n log n)) when k ≪ n, and it works on streams too big to hold in memory.

## Code Examples

```python
# min_heap.py — a binary min-heap from scratch, array-backed.

class MinHeap:
    def __init__(self, items=None):
        self._a = []
        if items:
            self._a = list(items)
            self._heapify()                  # O(n) bulk build

    def __len__(self):
        return len(self._a)

    def peek(self):
        """O(1): the minimum lives at the root."""
        if not self._a:
            raise IndexError("peek at empty heap")
        return self._a[0]

    def push(self, item):
        """O(log n): append at the next free slot, repair upward."""
        self._a.append(item)
        self._sift_up(len(self._a) - 1)

    def pop(self):
        """O(log n): take the root; plug the hole with the LAST element
        (preserving completeness), repair downward."""
        if not self._a:
            raise IndexError("pop from empty heap")
        a = self._a
        a[0], a[-1] = a[-1], a[0]            # min to the end
        smallest = a.pop()                   # remove it (O(1), it's last)
        if a:
            self._sift_down(0)
        return smallest

    def _sift_up(self, i):
        a = self._a
        while i > 0:
            parent = (i - 1) // 2
            if a[i] >= a[parent]:            # heap property restored
                break
            a[i], a[parent] = a[parent], a[i]
            i = parent

    def _sift_down(self, i):
        a, n = self._a, len(self._a)
        while True:
            left, right = 2 * i + 1, 2 * i + 2
            smallest = i
            # Find the smallest among node and its (up to two) children.
            if left < n and a[left] < a[smallest]:
                smallest = left
            if right < n and a[right] < a[smallest]:
                smallest = right
            if smallest == i:                # node beats both children: done
                break
            a[i], a[smallest] = a[smallest], a[i]
            i = smallest

    def _heapify(self):
        """O(n): repair from the last PARENT down to the root.
        Leaves (the back half) are already valid one-node heaps."""
        for i in range(len(self._a) // 2 - 1, -1, -1):
            self._sift_down(i)


def heapsort(items):
    """O(n log n), demonstrates the heap earning its keep n times."""
    h = MinHeap(items)                       # O(n)
    return [h.pop() for _ in range(len(h))]  # n pops * O(log n)


def top_k_largest(items, k):
    """O(n log k): min-heap of the k best so far; root = current cutoff."""
    h = MinHeap()
    for x in items:
        if len(h) < k:
            h.push(x)
        elif x > h.peek():                   # beats the weakest keeper?
            h.pop()
            h.push(x)
    return sorted((h.pop() for _ in range(len(h))), reverse=True)


if __name__ == "__main__":
    import random
    data = [random.randint(0, 999) for _ in range(50)]
    assert heapsort(data) == sorted(data)
    print("heapsort OK")
    print("top 5:", top_k_largest(data, 5))
    h = MinHeap([7, 2, 9, 1])
    h.push(0)
    print(h.peek(), len(h))                  # 0 5
```

Python's built-in heap — a module of functions over a plain list:

```python
# heapq_idioms.py — what you'd use in practice/interviews.
import heapq

nums = [7, 2, 9, 1, 6]
heapq.heapify(nums)                  # O(n), in place; nums[0] is now the min
heapq.heappush(nums, 0)
smallest = heapq.heappop(nums)       # 0

# heapq is MIN-only. Max-heap trick: negate on the way in and out.
maxheap = [-x for x in [7, 2, 9]]
heapq.heapify(maxheap)
largest = -heapq.heappop(maxheap)    # 9

# Priority queue of tasks: (priority, tiebreaker, payload) tuples.
# The counter breaks ties so payloads are never compared (see pitfalls).
import itertools
counter = itertools.count()
pq = []
heapq.heappush(pq, (2, next(counter), "write report"))
heapq.heappush(pq, (1, next(counter), "fix outage"))
heapq.heappush(pq, (1, next(counter), "answer page"))
while pq:
    prio, _, task = heapq.heappop(pq)
    print(prio, task)                # outage before page (FIFO within priority)

# nlargest/nsmallest implement top-k for you:
print(heapq.nlargest(3, [5, 1, 8, 3, 9, 2]))     # [9, 8, 5]
```

JavaScript has **no built-in heap** — in interviews you either implement `MinHeap` above (it ports line-for-line) or state you would.

## Common Pitfalls

**1. Expecting a sorted array.** `[1, 3, 2, 7, 4, 6, 5]` is a valid heap; iterating it does NOT give sorted order, and `heap[1]` is not the second-smallest (the second-smallest is *somewhere* in indices 1–2, third could be deeper). To consume in order, pop repeatedly.

**2. Popping by deleting index 0 directly.** `a.pop(0)` on the backing list is the Chapter 2 O(n) shift *and* it breaks the tree shape. The swap-with-last dance exists to preserve completeness cheaply. Same for removal of arbitrary elements — naive `list.remove` corrupts the structure.

**3. Unorderable tie-breaking in tuple entries.**

```python
# Crashes when priorities tie: dicts aren't comparable.
heapq.heappush(pq, (1, {"task": "a"}))
heapq.heappush(pq, (1, {"task": "b"}))   # TypeError on comparison!

# Corrected — insert a unique counter between priority and payload:
heapq.heappush(pq, (1, next(counter), {"task": "a"}))
```

**4. Using a heap for lookup or "decrease priority".** "Is x in the heap?" is O(n). Updating an element's priority requires finding it first. The standard workaround (used in real Dijkstra implementations, Chapter 11) is **lazy deletion**: push the updated entry as a new one and skip stale entries when popped.

**5. Top-k with the wrong heap polarity.** For k *largest*, use a *min*-heap (root = eviction candidate). Reaching for a max-heap of all n items works but costs O(n + k log n) space O(n) — and misses the point on streams. Flip everything for k smallest.

**6. Off-by-one in `_heapify`'s start index.** The last parent is at `n // 2 - 1`. Starting at `n - 1` is wasted work (leaves are trivially heaps); starting at `n // 2` skips the last parent and can leave the heap invalid on even-sized arrays.

## Practice Exercises

1. Add `pushpop(item)` (push then pop, in one O(log n) sift) and `replace(item)` (pop then push) to `MinHeap`, each with a fast path that avoids sifting when possible. Explain when each beats a separate push+pop.
2. Convert `MinHeap` into a `MaxHeap` two ways: (a) copy-and-flip the comparisons, (b) wrap `MinHeap` with negation. Then generalize: accept a `key=` function like `sorted` does, and heap-sort a list of `(name, score)` tuples by score without tuple-comparison tricks.
3. Measure `_heapify` vs n pushes: build heaps of 10⁶ random ints both ways and time them. Then explain the O(n) heapify sum informally: how many nodes can require 1 swap? 2 swaps? h swaps?
4. **Running median**: maintain a stream's median with two heaps — a max-heap of the lower half and a min-heap of the upper half, kept within one element of each other in size. Implement `add(num)` and `median()` with O(log n) and O(1) costs, and test against `statistics.median` on random data.
5. **Merge k sorted lists** into one sorted list in O(N log k) (N = total items) using a heap holding one "cursor" per list. Why is the naive "concatenate and sort" O(N log N) — and when would it actually be fine?

---

**Next:** Chapter 11 — graphs: nodes connected any which way, and the two traversals (one uses your stack, one uses your queue) that unlock them.
