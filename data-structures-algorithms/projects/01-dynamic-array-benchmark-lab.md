# Project 1: Dynamic Array + Benchmark Lab

## Description

Build your own dynamic array class from scratch, then build a small benchmarking harness that *proves* the Big-O claims from the chapters with real measurements and simple text charts. You'll finish with an empirical answer to "what does amortized O(1) actually look like on my machine?" — and a reusable timing tool you'll want again in later projects.

This project deliberately mirrors Chapter 2's `DynamicArray`, but you write yours without looking at the chapter code, extend it further, and — the new part — measure it against Python's built-in `list`.

## Difficulty

**Beginner.** Estimated effort: 3–5 hours.

## Chapters used

- Chapter 1 (Big-O, timing methodology)
- Chapter 2 (arrays & dynamic arrays)

## Requirements checklist

### Part A — the data structure
- [ ] A `DynamicArray` class backed by a fixed-size block (a pre-allocated Python list you never call `append` on), tracking `length` and `capacity` separately
- [ ] `append`, `get(i)`, `set(i, value)`, `insert(i, value)`, `delete(i)`, `pop()`, `__len__`, and `__repr__`
- [ ] Doubling growth on full `append`; shrink to half capacity when occupancy drops to 1/4 (floor of 4 slots)
- [ ] Negative index support (`arr[-1]`) via `__getitem__`/`__setitem__`
- [ ] Proper `IndexError` on all out-of-range access, including after shrink
- [ ] A test script that exercises every method, including: empty array edge cases, a grow-then-shrink cycle, and a 10,000-append stress run verified against a plain list kept in parallel

### Part B — the benchmark lab
- [ ] A `timeit`-or-`perf_counter`-based harness function: `measure(fn, sizes) -> list of (n, seconds)` that runs each size several times and keeps the median
- [ ] Benchmark 1: `append` × n for your class vs built-in `list`, for n in at least [1k, 10k, 100k]
- [ ] Benchmark 2: front-`insert` × n vs `append` × n (demonstrate O(n) vs O(1) amortized)
- [ ] Benchmark 3: instrument your `_resize` to count copies; report total elements copied across n appends and show it's ≈ 2n, not n²
- [ ] Output a plain-text table AND a crude ASCII bar chart of the results (no plotting libraries required)
- [ ] A short `RESULTS.md` (5–15 lines) in the project folder stating what you measured and whether it matched the theory — and if something didn't, your best explanation

## Hints

- Simulate "raw memory" with `[None] * capacity` and index assignment only. If you catch yourself calling `.append` on the backing block, you've dissolved the exercise.
- For the shrink rule, write down a sequence of ops that would thrash (grow, shrink, grow, shrink...) if you shrank at 1/2 instead of 1/4 — that reasoning belongs in a comment.
- Timing noise is real: run each measurement 5+ times, take the median, and keep sizes big enough that runs take at least a few milliseconds.
- For the ASCII chart, scale bars to the largest value: `"#" * int(40 * value / max_value)`.
- If your class is 5–20× slower than `list`, that's expected (Python-level loops vs C). The *shape* of the growth curves is what you're comparing, not absolute speed.

## Stretch goals

- Add slicing support (`arr[2:5]`) returning a new `DynamicArray`, and state its complexity.
- Benchmark `pop(0)`-as-queue vs `collections.deque.popleft` to preview Chapter 4's lesson.
- Plot the per-append cost of 100,000 appends individually (time each one) and find the resize "spikes" in the output — the amortization picture, live.
- Implement a `GrowthPolicy` parameter (doubling vs +10 fixed vs 1.5×) and benchmark all three to reproduce Chapter 2's exercise 5 empirically.
