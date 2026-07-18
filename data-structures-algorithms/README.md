# Data Structures & Algorithms — Learning Track

A self-paced track for learners who already know **Python** (required — all primary examples are Python) and ideally some **JavaScript** (used for occasional comparisons). Two goals drive everything here: **genuine understanding of efficiency** — why code is fast or slow, and how to choose structures deliberately — and **technical-interview readiness**, built through pattern recognition and deliberate practice.

Everything runs on a stock Python 3 install (Windows: `py file.py` or `python file.py`). No external libraries are required anywhere in the track.

## How the track works

- **`learning-docs/`** — 14 chapters, in order, beginner → advanced. Each chapter contains an overview, plain-English explanations with ASCII diagrams, complete runnable from-scratch Python implementations, common pitfalls with corrections, and 3–5 practice exercises (no solutions provided — the struggle is the point). Chapters assume only prior chapters.
- **`projects/`** — 7 guided project specs, easiest → hardest, each bundling several chapters into something you build. Specs contain requirements checklists, hints, and stretch goals — **no solution code**. You write everything.

## Chapters (learning-docs/)

| # | File | Topic |
|---|------|-------|
| 1 | `01-why-dsa-and-big-o.md` | Why DS&A matters; Big-O, time vs space, analyzing code |
| 2 | `02-arrays-and-dynamic-arrays.md` | How Python lists / JS arrays really work; amortized growth |
| 3 | `03-linked-lists.md` | Nodes and pointers; singly/doubly linked; runner & sentinel tricks |
| 4 | `04-stacks-and-queues.md` | LIFO/FIFO, ring buffers, the call stack |
| 5 | `05-hash-tables.md` | How dicts/objects work; hashing, collisions, load factor |
| 6 | `06-recursion.md` | Base cases, the call stack, recursion vs iteration, backtracking |
| 7 | `07-sorting-algorithms.md` | Bubble/insertion for intuition; merge/quick for real; stability; Timsort |
| 8 | `08-searching-and-binary-search.md` | Binary search done correctly; boundaries; search-on-the-answer |
| 9 | `09-trees-and-binary-search-trees.md` | Tree vocabulary, BST operations, balance, traversals |
| 10 | `10-heaps-and-priority-queues.md` | Binary heaps in arrays; top-k; heapq |
| 11 | `11-graphs-and-traversal.md` | Representations; BFS/DFS; Dijkstra; topological sort |
| 12 | `12-problem-solving-patterns.md` | Two pointers, sliding window, frequency counting, prefix sums |
| 13 | `13-dynamic-programming-intro.md` | Memoization, tabulation, the 4-step recipe, classic problems |
| 14 | `14-interview-technique-and-practice.md` | The 6-phase method, communication, an 8–12 week practice plan |

## Projects (projects/)

| # | File | Builds | After chapters |
|---|------|--------|----------------|
| 1 | `01-dynamic-array-benchmark-lab.md` | Dynamic array from scratch + benchmark harness | 1–2 |
| 2 | `02-browser-history-engine.md` | Back/forward navigation with stacks, queues, linked lists | 1–4 |
| 3 | `03-build-your-own-hash-map.md` | A dict from scratch, two collision strategies, benchmarks | 1–5 |
| 4 | `04-sorted-leaderboard.md` | Always-sorted rankings via binary-search insertion + merge sort | 1–8 |
| 5 | `05-autocomplete-engine.md` | Prefix suggestions: BST vs trie, heap-ranked | 1–10 |
| 6 | `06-route-finder.md` | Transit routing & mazes: BFS, Dijkstra, connectivity | 1–11 |
| 7 | `07-interview-gym-capstone.md` | 18 classic problems under interview conditions + practice tracker | all |

## Suggested cadence (~14–18 weeks at 5–8 hrs/week)

| Weeks | Focus |
|-------|-------|
| 1–2 | Chapters 1–2 → **Project 1** |
| 3–4 | Chapters 3–4 → **Project 2** |
| 5–6 | Chapter 5 → **Project 3** (start Chapter 6 alongside) |
| 7–8 | Chapters 6–8 → **Project 4** |
| 9–10 | Chapters 9–10 → **Project 5** |
| 11–12 | Chapter 11 → **Project 6** |
| 13–14 | Chapters 12–14 (patterns, DP, interview technique) |
| 15+ | **Project 7 (capstone)** — spread over 3+ weeks; its spaced-repetition schedule is part of the design |

Working rules that make the track stick:

1. **Type every code example yourself and run it.** Reading is not learning; transcription with prediction is.
2. **Do the exercises before moving on** — at least 3 of the 5 per chapter. They have no posted solutions on purpose.
3. **Projects are checkpoints, not options.** If a project feels impossible, that's a signal to re-read a specific chapter, and each spec tells you which.
4. **When stuck, struggle for 25–35 minutes before seeking help** — then study the resolution actively and re-attempt from scratch the next day. (Chapter 14 explains why this schedule beats both extremes.)

## Prerequisites

- **Python (required):** functions, classes, lists/dicts/sets, `for`/`while`, exceptions, running scripts from a terminal. If any of that is shaky, finish a Python fundamentals track first.
- **JavaScript (optional but useful):** the chapters include short JS comparisons so the concepts anchor in both languages; all required work is Python.
- **Math:** arithmetic and comfort with the idea of exponents/logarithms. Chapter 1 (re)builds the logarithm intuition you need.
