# Project 2: Browser History Engine

## Description

Build the navigation core of a web browser as a command-line program: visiting pages, going back, going forward, plus a "recently closed tabs" feature and a bounded "most recent downloads" list. Under the hood this is two stacks (back/forward), a queue, and a linked list — chosen deliberately, with a written justification for each choice. You'll interact with it through a simple REPL loop (`visit x`, `back`, `forward`, ...), so the finished program feels like a real tool.

## Difficulty

**Beginner+.** Estimated effort: 4–6 hours.

## Chapters used

- Chapter 2 (arrays — and why they're the wrong tool for some of this)
- Chapter 3 (linked lists)
- Chapter 4 (stacks & queues)

## Requirements checklist

### Core navigation
- [ ] A `BrowserHistory` class with `visit(url)`, `back()`, `forward()`, and `current()` built on **two stacks you implement yourself** (no bare `list.pop(0)`, no `collections.deque` for this part)
- [ ] `visit` while somewhere in the middle of history must clear the forward stack (matching real browser behavior — verify in your own browser first)
- [ ] `back`/`forward` on empty stacks must not crash: return the current page and print a friendly message
- [ ] `history()` prints the full back-trail, oldest first, with the current page marked — without permanently destroying the stacks

### Recently closed & downloads
- [ ] `close_tab(url)` / `reopen_tab()`: last-closed-first-reopened, capacity 10 — justify in a comment which structure fits and why
- [ ] A downloads list showing the **last 5** downloads in order: implement with a ring buffer (fixed array + wrapping indices) that overwrites the oldest — no shifting allowed
- [ ] A doubly linked list powering an unbounded "full session log" with O(1) append, and a `session_log(n)` command that prints the last n entries by walking backward from the tail

### The program
- [ ] A REPL: reads commands (`visit`, `back`, `forward`, `history`, `close`, `reopen`, `download`, `downloads`, `log`, `quit`), dispatches, and never crashes on bad input
- [ ] A test script (separate file) that scripts a realistic session — at least 25 commands including every edge case above — and asserts the outputs
- [ ] A `DESIGN.md` (10–20 lines): for each feature, which structure you chose, the alternative you rejected, and the complexity of each operation

## Hints

- The two-stack model: `back()` pops the back-stack onto the forward-stack; `forward()` does the reverse. Work one example on paper (visit A, B, C; back; back; visit D) before coding — the forward-clearing rule falls out of the paper trace.
- For non-destructive `history()`, remember a stack built on your own class can expose safe internal iteration without violating the stack discipline for *mutations* — or pour into a temp stack and pour back.
- Ring buffer: `write_index = (write_index + 1) % capacity`. Displaying in order when the buffer has wrapped is the part that needs thought — where is the *oldest* element?
- In the doubly linked list, a sentinel node (Chapter 3, exercise 3) removes every `None` special case from append and backward-walk.
- REPL robustness: `command, *args = line.split()` plus a dict of handler functions beats a wall of `if/elif`.

## Stretch goals

- Persist the session log to a file on `quit` and reload on start.
- Add `back n` / `forward n` (jump n pages) in O(n) — then discuss in `DESIGN.md` what structure change would make jumping O(1) and what it would cost you.
- Add an LRU cache of "page contents" (capacity 3): visiting a cached page prints instantly, uncached pages "load" — hash map + doubly linked list, the classic combo (this previews Chapter 5).
- Implement undo for `close_tab` beyond capacity 10 by spilling the overflow to a second structure of your choice, and defend the choice.
