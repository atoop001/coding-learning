# Chapter 4: Stacks & Queues

## Overview

Stacks and queues are not new machinery — they are arrays or linked lists with *fewer* allowed operations. That restriction is the point: by promising you'll only add/remove at certain ends, you get guaranteed O(1) operations and, more importantly, a crisp way to model real processes. A stack models "most recent thing first": undo history, the browser back button, the function call stack, matching brackets. A queue models "first come, first served": print jobs, message queues, breadth-first search (Chapter 11). Interviewers love them because many problems that look hard collapse instantly once you say "this is a stack problem."

## Definitions & Explanations

### Stack — LIFO (Last In, First Out)

Operations:
- **push(x)** — add to the top. O(1)
- **pop()** — remove and return the top. O(1)
- **peek()** — look at the top without removing. O(1)

```
push 5, push 9, push 2:        pop() -> 2:

  |  2  | <- top                 |  9  | <- top
  |  9  |                        |  5  |
  |  5  |                        +-----+
  +-----+
```

Think: a stack of plates. You can only touch the top one.

### Queue — FIFO (First In, First Out)

Operations:
- **enqueue(x)** — add at the back. O(1)
- **dequeue()** — remove and return the front. O(1)
- **peek()** — look at the front. O(1)

```
enqueue 5, 9, 2:                    dequeue() -> 5:

  front -> | 5 | 9 | 2 | <- back      front -> | 9 | 2 | <- back
```

Think: a line at a checkout.

### Deque — double-ended queue

Add/remove at *both* ends in O(1). A deque can act as a stack, a queue, or both at once (used in the sliding-window pattern, Chapter 12). Python ships one: `collections.deque`, implemented as a doubly linked list of blocks.

### Choosing an implementation

| Backing | Stack | Queue |
|---|---|---|
| Python `list` | Perfect: `append`/`pop` at the end are O(1) | **Bad**: `pop(0)` is O(n) (Chapter 2) |
| `collections.deque` | Fine | Perfect: `append`/`popleft` O(1) |
| Linked list | O(1) at head | O(1) with head+tail refs |
| Ring buffer (circular array) | — | Perfect when max size is known |

The **ring buffer** deserves a picture. Use a fixed array plus `front` and `count`; indices wrap around with modulo, so nothing ever shifts:

```
capacity 5, after enqueue a,b,c,d then dequeue twice, enqueue e,f:

  index:   0     1     2     3     4
         +-----+-----+-----+-----+-----+
         |  f  | (x) |  c  |  d  |  e  |
         +-----+-----+-----+-----+-----+
                       ^front             back wrapped to index 0
```

### The call stack (a stack you use every day)

Every function call pushes a *frame* (locals + where to return); every return pops it. That's why it's called a "stack trace," and why runaway recursion raises "stack overflow." Chapter 6 leans on this heavily.

## Code Examples

```python
# stack_queue.py — stacks and queues from scratch, plus classic uses.

class Stack:
    """LIFO on top of a Python list. All ops O(1) (append amortized)."""
    def __init__(self):
        self._items = []

    def push(self, item):
        self._items.append(item)          # add at the end = the "top"

    def pop(self):
        if not self._items:
            raise IndexError("pop from empty stack")
        return self._items.pop()          # remove from the end: O(1)

    def peek(self):
        if not self._items:
            raise IndexError("peek at empty stack")
        return self._items[-1]

    def __len__(self):
        return len(self._items)


class RingBufferQueue:
    """FIFO on a fixed circular array. Every operation O(1), no shifting.
    `front` marks the oldest element; back is computed with modulo."""
    def __init__(self, capacity):
        self._data = [None] * capacity
        self._capacity = capacity
        self._front = 0
        self._count = 0

    def enqueue(self, item):
        if self._count == self._capacity:
            raise OverflowError("queue is full")
        back = (self._front + self._count) % self._capacity   # wrap!
        self._data[back] = item
        self._count += 1

    def dequeue(self):
        if self._count == 0:
            raise IndexError("dequeue from empty queue")
        item = self._data[self._front]
        self._data[self._front] = None
        self._front = (self._front + 1) % self._capacity      # wrap!
        self._count -= 1
        return item

    def peek(self):
        if self._count == 0:
            raise IndexError("peek at empty queue")
        return self._data[self._front]

    def __len__(self):
        return self._count


def is_balanced(text):
    """Classic stack application: are (), [], {} properly nested?
    Push openers; each closer must match the most recent opener."""
    pairs = {")": "(", "]": "[", "}": "{"}
    stack = Stack()
    for ch in text:
        if ch in "([{":
            stack.push(ch)
        elif ch in ")]}":
            if len(stack) == 0 or stack.pop() != pairs[ch]:
                return False
        # other characters: ignore
    return len(stack) == 0                # leftovers mean unclosed openers


def evaluate_rpn(tokens):
    """Evaluate Reverse Polish Notation, e.g. ['3','4','+','2','*'] = 14.
    Numbers get pushed; operators pop two operands. How calculators
    and compilers actually evaluate expressions."""
    stack = Stack()
    for tok in tokens:
        if tok in "+-*/":
            b = stack.pop()               # note: b first — order matters for - and /
            a = stack.pop()
            if   tok == "+": stack.push(a + b)
            elif tok == "-": stack.push(a - b)
            elif tok == "*": stack.push(a * b)
            else:            stack.push(a / b)
        else:
            stack.push(float(tok))
    return stack.pop()


if __name__ == "__main__":
    print(is_balanced("def f(x): return [x, {1: (2)}]"))   # True
    print(is_balanced("([)]"))                              # False — interleaved
    print(evaluate_rpn("3 4 + 2 *".split()))                # 14.0

    q = RingBufferQueue(3)
    for job in ["print A", "print B", "print C"]:
        q.enqueue(job)
    print(q.dequeue(), "| next up:", q.peek())              # print A | print B
```

In production Python you'd reach for the built-in:

```python
from collections import deque
stack = []            # list IS the idiomatic Python stack
stack.append(1); stack.pop()

queue = deque()       # deque IS the idiomatic Python queue
queue.append(1)       # enqueue at the back
queue.popleft()       # dequeue from the front, O(1)
```

JavaScript comparison:

```javascript
const stack = [];  stack.push(1);  stack.pop();     // O(1), same as Python
const queue = [];  queue.push(1);  queue.shift();   // shift() is O(n)!
// JS has no built-in deque — for hot paths, use an index pointer or a
// linked-list-based queue instead of shift().
```

## Common Pitfalls

**1. Using `list.pop(0)` as a queue.** The classic O(n²) trap from Chapter 2. Use `deque.popleft()`.

**2. Popping without checking for emptiness.** An empty-stack `pop` in the middle of `is_balanced("())(")` would crash instead of returning `False`. Guard first (as the code above does), or catch the exception deliberately — but decide, don't hope.

**3. Wrong operand order on non-commutative operators.**

```python
# Bug in RPN: "6 2 /" should be 3, this computes 2/6.
a = stack.pop()
b = stack.pop()
stack.push(a / b)

# Corrected: the SECOND pop is the left operand.
b = stack.pop()
a = stack.pop()
stack.push(a / b)
```

**4. Forgetting the modulo in a ring buffer.** `back = self._front + self._count` without `% capacity` walks off the end of the array as soon as the queue wraps. Every index computation in a circular structure needs the wrap.

**5. Reaching into the middle.** If you find yourself indexing `stack._items[3]`, you don't have a stack problem — you have an array problem, or your model is wrong. The discipline *is* the data structure.

## Practice Exercises

1. Implement `MinStack`: `push`, `pop`, `peek`, and `get_min` — all O(1). (Hint: a second stack that tracks the minimum *so far* alongside the main one.)
2. Implement a queue using **two stacks** (`enqueue` pushes to one, `dequeue` pops from the other, refilling it only when empty). Argue why `dequeue` is amortized O(1) even though a single call can be O(n).
3. Extend `RingBufferQueue` to auto-resize when full (double the capacity), keeping all operations amortized O(1). Careful: after resizing you must un-wrap the elements into the new array in order.
4. Write `next_greater(nums)` returning, for each element, the first element to its right that is larger (or −1). O(n) using a stack of "elements still waiting for their answer." Example: `[2, 1, 5, 3]` → `[5, 5, -1, -1]`.
5. Simulate a print-queue: jobs arrive with `(name, priority)` where priority is 0 or 1 (urgent). Urgent jobs go ahead of normal ones but behind other urgent ones. Implement it with two queues and O(1) operations. (Chapter 10 generalizes this to arbitrary priorities.)

---

**Next:** Chapter 5 opens the hood on Python's most-used structure after the list: the dict — hash tables, collisions, and why lookup is O(1).
