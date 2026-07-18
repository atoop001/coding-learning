# Chapter 3: Linked Lists

## Overview

A linked list stores elements in separate *nodes* scattered anywhere in memory, each node holding a value and a pointer (reference) to the next node. It is the mirror image of the array: inserting or removing at a known spot is O(1), but finding the i-th element is O(n). Linked lists appear directly in real systems (undo chains, LRU caches, memory allocators, blockchain is literally one) and indirectly everywhere — they are the mental model for trees and graphs, which are "linked lists that branch." They are also a perennial interview topic because pointer manipulation exposes whether you truly understand references.

## Definitions & Explanations

### Nodes and links

```
head
  |
  v
+------+---+    +------+---+    +------+---+
|  17  | o-|--->|  42  | o-|--->|   8  | / |
+------+---+    +------+---+    +------+---+
 value  next     value  next     value  next = None (end of list)
```

- **Node** — an object holding `value` and `next` (a reference to another node, or `None`).
- **Head** — reference to the first node. Lose the head, lose the list.
- **Tail** — the last node (`next is None`). Some implementations keep a direct tail reference to make appending O(1).

In Python, "pointer" just means "variable holding a reference to an object" — no special syntax needed.

### Why insertion is O(1) (once you're there)

To insert 99 after the node holding 17, you rewire two references. Nothing shifts:

```
before:  [17] ---------------> [42]
insert:  [17] --> [99] ------> [42]
         (17.next = new node; new.next = old 17.next)
```

Compare the array's O(n) shift-everything insert. But note the catch: *getting to* the node before your insertion point costs O(n) unless you already hold a reference to it.

### Cost comparison vs dynamic arrays

| Operation | Dynamic array | Singly linked list |
|---|---|---|
| Index `a[i]` | O(1) | O(n) — walk i links |
| Insert/delete at front | O(n) | **O(1)** |
| Insert/delete at back | O(1) amortized | O(1) with tail ref (delete-at-back is O(n) without `prev` links) |
| Insert after a held node | O(n) | **O(1)** |
| Search by value | O(n) | O(n) |
| Memory | compact, contiguous | per-node overhead, scattered (worse cache behavior) |

That last row matters in practice: arrays are contiguous, so the CPU's cache prefetches neighbors and real-world array traversal is much faster than the Big-O tie suggests. Linked lists win only when their O(1) structural edits are the workload.

### Variants

- **Singly linked** — `next` only. Can't walk backward; deleting a node requires knowing its predecessor.
- **Doubly linked** — `next` and `prev`. O(1) delete of any held node; costs one extra reference per node. Python's `collections.deque` and functools' LRU cache use this idea.
- **Circular** — tail points back to the head; useful for round-robin scheduling.

```
doubly linked:
        +--------+     +--------+     +--------+
None <--| prev   |<----| prev   |<----| prev   |
        |   17   |     |   42   |     |    8   |
        |   next |---->|   next |---->|   next |--> None
        +--------+     +--------+     +--------+
```

### The two techniques that solve most linked-list problems

1. **The runner (fast & slow pointers).** Two pointers advance at different speeds. Fast moves 2, slow moves 1 → when fast hits the end, slow is at the middle; if they ever meet, the list has a cycle (Floyd's algorithm).
2. **The dummy (sentinel) node.** A throwaway node placed before the head so "insert/delete at the front" stops being a special case — every operation becomes "edit the node after some node."

## Code Examples

```python
# linked_list.py — singly linked list from scratch.

class Node:
    """One cell of the chain."""
    def __init__(self, value, next=None):
        self.value = value
        self.next = next


class LinkedList:
    def __init__(self):
        self.head = None
        self.tail = None       # kept so append is O(1)
        self.size = 0

    def __len__(self):
        return self.size

    def push_front(self, value):
        """O(1): new node points at old head, becomes the new head."""
        node = Node(value, next=self.head)
        self.head = node
        if self.tail is None:              # list was empty
            self.tail = node
        self.size += 1

    def append(self, value):
        """O(1) thanks to the tail reference."""
        node = Node(value)
        if self.tail is None:              # empty list: node is head AND tail
            self.head = self.tail = node
        else:
            self.tail.next = node          # old tail links to new node
            self.tail = node
        self.size += 1

    def find(self, value):
        """O(n): walk the chain until we see the value."""
        current = self.head
        while current is not None:
            if current.value == value:
                return current
            current = current.next
        return None

    def delete(self, value):
        """O(n): find the node BEFORE the one to remove, then bypass it.
        Returns True if something was deleted."""
        # Sentinel trick: pretend there's a node before head, so deleting
        # the head is not a special case.
        dummy = Node(None, next=self.head)
        prev, current = dummy, self.head
        while current is not None:
            if current.value == value:
                prev.next = current.next   # bypass: unlinks `current`
                if current is self.tail:   # removed the last node?
                    self.tail = prev if prev is not dummy else None
                self.head = dummy.next     # head may have changed
                self.size -= 1
                return True
            prev, current = current, current.next
        return False

    def reverse(self):
        """O(n) time, O(1) space: re-point every arrow backward.
        The classic interview question."""
        prev = None
        current = self.head
        self.tail = self.head              # old head becomes new tail
        while current is not None:
            nxt = current.next             # 1. save where we're going
            current.next = prev            # 2. flip this node's arrow
            prev = current                 # 3. advance prev
            current = nxt                  # 4. advance current
        self.head = prev

    def middle(self):
        """O(n) one pass, runner technique: slow ends at the middle node."""
        slow = fast = self.head
        while fast is not None and fast.next is not None:
            slow = slow.next               # 1 step
            fast = fast.next.next          # 2 steps
        return slow                        # None if list is empty

    def has_cycle(self):
        """Floyd's tortoise & hare: if fast laps slow, there's a loop."""
        slow = fast = self.head
        while fast is not None and fast.next is not None:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:
                return True
        return False

    def __repr__(self):
        parts, current = [], self.head
        while current is not None:
            parts.append(str(current.value))
            current = current.next
        return " -> ".join(parts) + " -> None" if parts else "(empty)"


if __name__ == "__main__":
    ll = LinkedList()
    for x in [10, 20, 30, 40]:
        ll.append(x)
    ll.push_front(5)
    print(ll)                       # 5 -> 10 -> 20 -> 30 -> 40 -> None
    print("middle:", ll.middle().value)   # 20
    ll.delete(5)                    # delete head via sentinel path
    ll.delete(40)                   # delete tail
    print(ll)                       # 10 -> 20 -> 30 -> None
    ll.reverse()
    print("reversed:", ll)          # 30 -> 20 -> 10 -> None
    print("cycle?", ll.has_cycle()) # False
```

JavaScript node for comparison — objects and references work identically:

```javascript
class Node {
  constructor(value, next = null) { this.value = value; this.next = next; }
}
let head = new Node(17, new Node(42, new Node(8)));
// Traversal is the same walk:
for (let cur = head; cur !== null; cur = cur.next) console.log(cur.value);
```

## Common Pitfalls

**1. Losing the rest of the list by overwriting `next` too early.**

```python
# Bug: after this line, the old chain past `current` is unreachable.
current.next = new_node          # oops — old current.next is lost!
new_node.next = current.next     # now points at new_node itself (cycle!)

# Corrected — save first, or order the writes correctly:
new_node.next = current.next     # 1. new node adopts the tail of the chain
current.next = new_node          # 2. then splice it in
```

**2. Forgetting the empty-list and single-node cases.** Almost every linked-list bug lives at the boundaries: empty list, one node, head, tail. Habit: after writing an operation, mentally trace it on `(empty)`, `[x]`, and `[x, y]`.

**3. Special-casing the head instead of using a dummy node.** Head-deletion code duplicated from body-deletion code drifts out of sync. The `dummy` pattern in `delete()` above makes one code path handle everything.

**4. Infinite loop from an un-advanced pointer.**

```python
# Bug: `current` never advances when the value doesn't match — hangs forever.
while current is not None:
    if current.value == target:
        return current
# Corrected: the advance belongs inside the loop unconditionally.
while current is not None:
    if current.value == target:
        return current
    current = current.next
```

**5. Comparing with `==` when you mean identity.** `prev is dummy` and `current is self.tail` use `is` deliberately: we care whether they are the *same node*, not nodes with equal values. Two nodes can hold equal values and still be different nodes.

## Practice Exercises

1. Add `insert_after(node, value)` and `pop_front()` to `LinkedList`, each O(1). State what each should do on an empty list and implement that behavior.
2. Write `nth_from_end(k)` that returns the k-th node from the end in **one pass** using two pointers spaced k apart. (k=1 means the tail.)
3. Implement a `DoublyLinkedList` with `push_front`, `push_back`, `pop_front`, `pop_back`, and `remove(node)` — all O(1). Use a single circular sentinel node so there are no `None` checks anywhere in the five methods.
4. Write a function `merge_sorted(l1, l2)` that merges two already-sorted linked lists into one sorted list by re-linking existing nodes (no new nodes, no arrays). This becomes the heart of merge sort in Chapter 7.
5. A list has a cycle. Extend `has_cycle` into `cycle_start()` returning the node where the cycle begins (Floyd's algorithm, phase two: after the pointers meet, restart one at the head and advance both one step at a time). Explain in a comment why that works.

---

**Next:** Chapter 4 builds stacks and queues — restricted structures whose *limitations* are the feature — using both arrays and linked lists as the engine.
