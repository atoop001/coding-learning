# Project 4: Sorted Leaderboard Service

## Description

Build a game leaderboard that stays sorted at all times: players submit scores, and the board answers "top 10," "what's my rank?", "who's around me?", and "how many players scored between X and Y?" — all fast. The core is a sorted array maintained by **binary-search insertion**, a hand-rolled sort for bulk imports, and a head-to-head comparison against re-sorting on every insert. A small CLI (or menu loop) makes it usable; a benchmark proves your design choices.

## Difficulty

**Intermediate.** Estimated effort: 5–8 hours.

## Chapters used

- Chapter 2 (array insertion costs)
- Chapter 5 (hash map for name → entry lookup)
- Chapter 7 (sorting: your own merge sort for bulk import; stability for tie-breaking)
- Chapter 8 (binary search: bisect-style boundary searches, written yourself)

## Requirements checklist

### Core structure
- [ ] A `Leaderboard` class keeping entries `(score, name)` in a list sorted descending by score; ties broken alphabetically by name (define this ordering once, use it everywhere)
- [ ] `submit(name, score)`: find the insertion index with **your own** binary search (no `bisect` module), then insert — O(log n) search + O(n) insert, stated in a docstring
- [ ] Resubmission policy: a player's new score replaces their old entry only if higher; locate the old entry via a hash map `name -> current score` so you never linearly scan
- [ ] `top(k)`: the top k entries, O(k)
- [ ] `rank(name)`: 1-based rank, with standard competition ranking for ties ("1, 2, 2, 4") — document your tie behavior and test it
- [ ] `around(name, window)`: the window entries above and below a player
- [ ] `count_in_range(lo, hi)`: number of players with lo ≤ score ≤ hi in **O(log n)** using left/right boundary binary searches — no scanning

### Bulk import & sorting
- [ ] `import_scores(pairs)`: load thousands of entries at once by sorting with **your own merge sort** (stable, and explain in a comment why stability plus your comparison choice keeps tie-breaking consistent with `submit`)
- [ ] Verify your merge sort against `sorted()` on 1,000 random inputs

### Proof & polish
- [ ] Benchmark: 10,000 sequential `submit`s via (a) binary-insert (yours) vs (b) append-then-re-`sorted()` each time vs (c) append-then-sort-once-at-the-end; table of timings and a paragraph on when each strategy is right
- [ ] A CLI or menu loop: submit, top, rank, around, range-count, import from a CSV/text file of `name,score` lines
- [ ] Tests covering: empty board, one player, all-tied scores, resubmission (higher and lower), range with no matches, rank of a missing player

## Hints

- Sort descending by score but ascending by name is easiest with a key tuple like `(-score, name)` kept ascending internally — pick ONE canonical representation early; converting between "user view" and "internal order" in exactly one place saves you from mirror-image bugs everywhere.
- `count_in_range` is Chapter 8's `bisect_left`/`bisect_right` pair on your internal ordering. Write them as standalone functions taking a key — you'll reuse them for `rank`.
- The hash map and the sorted list must never disagree. Route every mutation through one private method that updates both, and add an `assert` (or a `check_invariants()` test helper) that rebuilds one from the other and compares.
- For `around(name, ...)`, clamp the window at both ends of the board rather than special-casing "near the top."
- Benchmark (b) is intentionally terrible — O(n log n) per insert. Predict the total complexity of all three strategies *before* running, write the predictions down, then check.

## Stretch goals

- Persist the leaderboard to disk (JSON or CSV) and reload on start; make import idempotent.
- Add `decay()`: all scores drop 10% weekly — is your sorted order preserved automatically? Prove why or why not in a comment, then implement it correctly either way.
- Support multiple leaderboards (per game mode) behind one interface, sharing the player hash map.
- After Chapter 10: re-implement "top k of a live stream where you never need full ranking" with a heap, and write a paragraph on when the heap design beats the sorted array (and what queries it gives up).
