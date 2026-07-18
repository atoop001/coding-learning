# Project 3: Build Your Own Hash Map

## Description

Implement a dictionary from scratch — twice. Version 1 uses separate chaining; version 2 uses open addressing with tombstones. Both must pass the same test suite and support Python's dict protocol (`d[k]`, `k in d`, `len(d)`, iteration). Then race them: a benchmark suite compares your two maps against each other and against the built-in `dict` under different load factors, and a "sabotage" experiment shows what happens when the hash function is bad. This is the project that makes Chapter 5 permanent.

## Difficulty

**Intermediate.** Estimated effort: 6–9 hours.

## Chapters used

- Chapter 1 (Big-O, amortized analysis, benchmarking from Project 1)
- Chapter 2 (arrays, resizing)
- Chapter 3 (chains are linked lists — or small arrays; you choose and justify)
- Chapter 5 (hash tables)

## Requirements checklist

### Version 1 — separate chaining (`ChainedHashMap`)
- [ ] `put(key, value)`, `get(key, default=None)`, `remove(key)`, `__len__`, `__contains__`, `keys()`, `values()`, `items()`
- [ ] Dunder protocol: `__getitem__` (raising `KeyError` on miss), `__setitem__`, `__delitem__`, `__iter__`
- [ ] Collision handling by chaining; key comparison must compare **keys**, never hashes
- [ ] Automatic resize (rehash all entries) when load factor exceeds a threshold you choose and document
- [ ] Correctly distinguishes "key absent" from "key maps to None" (sentinel technique)

### Version 2 — open addressing (`ProbedHashMap`)
- [ ] Same public interface, backed by a single flat slot array with linear probing
- [ ] Deletion via tombstone markers, with a comment explaining precisely what breaks without them (construct the breaking sequence!)
- [ ] Resize also purges tombstones; track "used slots" (entries + tombstones) separately from "live entries" for the load-factor decision

### Shared verification & benchmarks
- [ ] One test file run against BOTH classes (parametrize over the class), covering: 1,000 random insert/lookup/delete ops mirrored against a real `dict`; deleting then re-inserting the same key; collision-heavy keys; iteration completeness; `KeyError` behavior
- [ ] A `stats()` method on both: load factor, entry count, and (chained) max/mean chain length or (probed) max/mean probe distance
- [ ] Benchmark: 100k inserts + 100k hits + 100k misses for both maps and built-in `dict`; results in a text table
- [ ] Sabotage experiment: swap in `hash = lambda k: 0` (or subclass) and re-run a scaled-down benchmark; record the O(1) → O(n) collapse in numbers
- [ ] `RESULTS.md`: which of your maps won, at what load factor each degrades, and two sentences on why the built-in beats both

## Hints

- Build chaining first — it's forgiving. Open addressing shares the resize/sentinel machinery, so factor common logic into a base class or shared helpers.
- The tombstone-breaking sequence needs only three keys that collide: insert A, insert B (probes past A), delete A, look up B. Walk it on paper with slot numbers.
- `hash()` of Python strings changes between interpreter runs (hash randomization) — if a test depends on which keys collide, build collisions deliberately with a custom `__hash__` on a small key class instead.
- For `__iter__` on the probed version, skip both empty slots and tombstones.
- Load-factor experiment: force fixed capacities (disable resizing) and measure lookups at 25%, 50%, 75%, 90% full. Probing should degrade much more sharply than chaining near 90% — check that it does.

## Stretch goals

- Implement `Counter`-style helpers on top of your map: `increment(key)`, `most_common(k)` (partner it with a heap after Chapter 10).
- Add quadratic probing or double hashing to `ProbedHashMap` and benchmark against linear probing on the clustering-heavy workload.
- Make iteration order insertion-ordered (like modern `dict`): add a compact entries array plus slot indirection — a simplified version of CPython's actual design.
- Build a `HashSet` on your map and use it to deduplicate a large word list; compare timing with the built-in `set`.
