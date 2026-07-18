# Chapter 9: Trees & Binary Search Trees

## Overview

Chapter 8 left a tension unresolved: sorted arrays give O(log n) search but O(n) insertion; linked lists give O(1) insertion but O(n) search. The **binary search tree (BST)** resolves it — O(log n) search *and* insert *and* delete (when balanced), plus something hash tables can't do at all: ordered traversal, min/max, and range queries. Trees are also everywhere beyond BSTs: your file system, HTML's DOM, JSON documents, database indexes (B-trees), and compilers' syntax trees are all trees. Structurally, a tree is just a linked list whose nodes may point to *multiple* nexts — everything you learned in Chapters 3 and 6 transfers directly.

## Definitions & Explanations

### Tree vocabulary

```
                (8)          <- root (no parent)
               /   \
            (3)     (10)     <- children of 8; 3 and 10 are siblings
           /   \       \
         (1)   (6)     (14)  <- 1, 6 are leaves' parents...
              /  \     /
            (4)  (7) (13)    <- leaves (no children)
```

- **Node / edge** — the circles / the connections.
- **Root** — the top node. **Leaf** — a node with no children.
- **Height** — longest path from root to a leaf (tree above: height 3).
- **Depth of a node** — its distance from the root.
- **Binary tree** — every node has ≤ 2 children ("left" and "right").
- **Subtree** — any node plus all its descendants is itself a tree. This self-similarity is why recursion (Chapter 6) is the native language of trees.

### The BST property

A binary tree is a **binary search tree** iff for every node: *all* keys in its left subtree are smaller, *all* keys in its right subtree are larger. The tree above is a valid BST. The property must hold for entire subtrees, not just immediate children — the #1 misconception (see pitfalls).

Consequence: from any node, one comparison tells you which subtree the target must be in. Search walks one root-to-leaf path → cost is O(height).

### Balance is everything

- **Balanced tree**: height ≈ log₂ n → all operations O(log n).
- **Degenerate tree**: insert 1, 2, 3, 4, 5 in order and every node goes right — you've built a linked list with extra steps. Height = n → all operations O(n).

```
balanced (h=2):        degenerate (h=4):
      (3)                (1)
     /   \                 \
   (1)    (4)              (2)
      \      \                \
      (2)    (5)              (3)
                                 \
                                 (4)
                                    \
                                    (5)
```

Production trees (AVL, red-black — used by C++ `std::map`, Java `TreeMap`; B-trees in databases) *rebalance themselves* on insert/delete to guarantee O(log n). The rebalancing rotations are beyond this chapter, but you must know they exist and *why*: naive BSTs degrade on sorted input, which real data loves to be.

### Traversals — visiting every node

Depth-first, defined recursively; the name says where the *root* goes:

- **In-order** (left, node, right): visits a BST's keys **in sorted order** — the killer feature.
- **Pre-order** (node, left, right): useful for copying/serializing a tree.
- **Post-order** (left, right, node): children before parents — deleting, computing sizes.
- **Level-order**: row by row using a *queue* (Chapter 4) — this is BFS, formalized in Chapter 11.

For the tree at the top: in-order = 1 3 4 6 7 8 10 13 14 (sorted!), pre-order = 8 3 1 6 4 7 10 14 13.

### BST vs hash table — when to choose which

| Need | Hash table (Ch. 5) | Balanced BST |
|---|---|---|
| exact-key lookup | O(1) avg | O(log n) |
| min / max / successor | O(n) | **O(log n)** |
| range query ("keys 10..20") | O(n) | **O(log n + k)** |
| ordered iteration | O(n log n) (sort keys) | **O(n)** in-order |

Rule: exact lookups only → hash. Anything about *order* → tree.

## Code Examples

```python
# bst.py — a binary search tree from scratch.

class Node:
    def __init__(self, key):
        self.key = key
        self.left = None
        self.right = None


class BST:
    def __init__(self):
        self.root = None

    def insert(self, key):
        """O(height). Walk down as if searching; attach where you fall off."""
        self.root = self._insert(self.root, key)

    def _insert(self, node, key):
        if node is None:                 # fell off the tree: this is the spot
            return Node(key)
        if key < node.key:
            node.left = self._insert(node.left, key)
        elif key > node.key:
            node.right = self._insert(node.right, key)
        # equal: ignore duplicates (a design choice — document yours!)
        return node

    def contains(self, key):
        """O(height): one root-to-leaf path, guided by comparisons."""
        node = self.root
        while node is not None:
            if key == node.key:
                return True
            node = node.left if key < node.key else node.right
        return False

    def min(self):
        """Leftmost node = smallest key. O(height)."""
        if self.root is None:
            raise ValueError("empty tree")
        node = self.root
        while node.left is not None:
            node = node.left
        return node.key

    def delete(self, key):
        """O(height). Three cases: leaf, one child, two children."""
        self.root = self._delete(self.root, key)

    def _delete(self, node, key):
        if node is None:
            return None                              # key not present
        if key < node.key:
            node.left = self._delete(node.left, key)
        elif key > node.key:
            node.right = self._delete(node.right, key)
        else:                                        # found it
            if node.left is None:                    # 0 or 1 child:
                return node.right                    #   splice over the node
            if node.right is None:
                return node.left
            # Two children: replace key with in-order successor
            # (smallest key in right subtree), then delete THAT node.
            succ = node.right
            while succ.left is not None:
                succ = succ.left
            node.key = succ.key
            node.right = self._delete(node.right, succ.key)
        return node

    def in_order(self):
        """Yield keys in sorted order. O(n) total."""
        yield from self._in_order(self.root)

    def _in_order(self, node):
        if node is not None:
            yield from self._in_order(node.left)     # everything smaller...
            yield node.key                           # ...then me...
            yield from self._in_order(node.right)    # ...then everything bigger

    def range_query(self, lo, hi):
        """All keys in [lo, hi]. Prunes subtrees that can't contain answers,
        so cost is O(height + matches), not O(n)."""
        out = []
        def walk(node):
            if node is None:
                return
            if node.key > lo:            # left subtree can hold keys >= lo
                walk(node.left)
            if lo <= node.key <= hi:
                out.append(node.key)
            if node.key < hi:            # right subtree can hold keys <= hi
                walk(node.right)
        walk(self.root)
        return out

    def height(self):
        """Post-order recursion: my height = 1 + max(child heights)."""
        def h(node):
            if node is None:
                return -1                # empty tree convention: height -1
            return 1 + max(h(node.left), h(node.right))
        return h(self.root)


if __name__ == "__main__":
    t = BST()
    for k in [8, 3, 10, 1, 6, 14, 4, 7, 13]:
        t.insert(k)
    print(list(t.in_order()))            # [1, 3, 4, 6, 7, 8, 10, 13, 14]
    print(t.contains(6), t.contains(5))  # True False
    print("min:", t.min(), "height:", t.height())
    print("range 4..10:", t.range_query(4, 10))     # [4, 6, 7, 8, 10]
    t.delete(3)                          # two-children case
    print(list(t.in_order()))            # [1, 4, 6, 7, 8, 10, 13, 14]

    # Degeneracy demo: sorted inserts -> height n-1
    worst = BST()
    for k in range(10):
        worst.insert(k)
    print("sorted-insert height:", worst.height())   # 9 — a linked list!
```

Level-order traversal — a queue, not recursion:

```python
from collections import deque

def level_order(root):
    """Visit nodes row by row. The queue holds 'discovered but unvisited'."""
    if root is None:
        return []
    out, q = [], deque([root])
    while q:
        node = q.popleft()
        out.append(node.key)
        if node.left:  q.append(node.left)
        if node.right: q.append(node.right)
    return out
```

JavaScript node — same anatomy:

```javascript
class Node { constructor(key) { this.key = key; this.left = this.right = null; } }
// In-order without generators:
function inOrder(node, out = []) {
  if (!node) return out;
  inOrder(node.left, out); out.push(node.key); inOrder(node.right, out);
  return out;
}
```

## Common Pitfalls

**1. Validating a BST by checking only parent vs children.**

```
      (10)
     /    \
   (5)    (15)
          /
        (6)     <- 6 < 15 ✓ locally... but 6 < 10 sits in 10's RIGHT subtree ✗
```

```python
# Bug: local check passes the broken tree above.
def is_bst_wrong(n):
    if n is None: return True
    if n.left and n.left.key >= n.key: return False
    if n.right and n.right.key <= n.key: return False
    return is_bst_wrong(n.left) and is_bst_wrong(n.right)

# Corrected — carry the valid (lo, hi) window down the tree:
def is_bst(n, lo=float("-inf"), hi=float("inf")):
    if n is None: return True
    if not (lo < n.key < hi): return False
    return is_bst(n.left, lo, n.key) and is_bst(n.right, n.key, hi)
```

**2. Forgetting to reassign the subtree on insert/delete.** The pattern `node.left = self._insert(node.left, key)` looks redundant but is how a *new or replacement* child gets attached. Calling `self._insert(node.left, key)` without the assignment silently drops the insertion.

**3. Quoting O(log n) without the word "balanced".** A plain BST is O(height), and height is only O(log n) if insertions were lucky or the tree self-balances. On sorted input a naive BST is O(n) per op. In interviews, say "O(log n) if balanced" and mention AVL/red-black as the fix.

**4. Botching two-child deletion.** Copying the successor's key but forgetting to delete the successor node duplicates a key; taking the *left* subtree's max works too, but mixing the two conventions corrupts the tree. Pick "min of right subtree," implement it once, test on a node whose successor has a right child.

**5. Recursion depth on degenerate trees.** A recursive traversal of a 10,000-node degenerate tree blows Python's recursion limit (Chapter 6). The iterative in-order with an explicit stack (exercise 2) is the defense.

## Practice Exercises

1. Add `successor(key)` to `BST`: the smallest key strictly greater than `key`, or None — O(height), no full traversal. Two cases to handle: the node has a right subtree; it doesn't (track the last "turned left" ancestor while descending).
2. Write **iterative** in-order traversal using an explicit stack ("push left spine, pop, hop right"). Verify it matches the recursive generator on 1,000 random inserts, then on a degenerate 5,000-node tree where recursion would fail.
3. Write `from_sorted(a)` that builds a *height-balanced* BST from a sorted list (recursively: middle element becomes the root). Prove to yourself with `height()` that 1,023 sorted values give height 9, not 1,022.
4. Implement `kth_smallest(k)` two ways: (a) in-order traversal that stops early, (b) augment each node with a `size` field (maintained on insert) enabling O(height) selection. State each version's complexity.
5. Serialize a BST to a flat list with pre-order traversal and write `deserialize` that reconstructs it *exactly* — using the BST property and min/max bounds instead of storing None markers. Round-trip test on random trees.

---

**Next:** Chapter 10 — the heap: a tree that gives up total order to make "give me the smallest, fast" its entire personality.
