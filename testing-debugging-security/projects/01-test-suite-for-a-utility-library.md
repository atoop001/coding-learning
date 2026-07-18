# Project 1: Test Suite for a Utility Library

## Description

Your first real testing project: you will *build a small utility library from a provided spec, then write a professional-grade test suite for it*. The library is deliberately simple so the focus stays on the tests — structure, naming, AAA, boundaries, and error paths. Do it twice: once in Python with pytest, once in JavaScript with Vitest (same functions), so the frameworks stop feeling foreign. By the end you'll have a reusable mental template for "what does a well-tested module look like?"

The library — `textkit` — has five functions. Implement them to this spec exactly:

1. `slugify(text)` — lowercase, trim, spaces/underscores → single hyphens, strip all other non-alphanumerics; empty result raises/throws.
2. `truncate(text, max_len)` — return `text` unchanged if within `max_len`; otherwise cut and append `"..."` so the *total* length equals `max_len`; `max_len < 4` is an error.
3. `word_count(text)` — number of whitespace-separated words; `""` and whitespace-only → 0.
4. `initials(full_name)` — `"ada lovelace"` → `"A.L."`; handles 1–4 name parts; extra whitespace tolerated; empty input is an error.
5. `ordinal(n)` — `1→"1st"`, `2→"2nd"`, `3→"3rd"`, `4→"4th"`, `11→"11th"`, `21→"21st"`, `112→"112th"`; negative numbers are an error.

## Difficulty

**Beginner.** Estimated effort: 4–6 hours (roughly 2–3 per language).

## Chapters used

- Chapter 1 — Why Testing & the Testing Mindset
- Chapter 2 — Writing Your First Unit Tests
- Chapter 4 — Edge Cases & Test Design (boundaries, parameterization)

## Requirements checklist

- [ ] Python project with a venv, pytest installed, and `textkit.py` + `test_textkit.py`
- [ ] JS project with Vitest configured, `textkit.js` + `textkit.test.js`, `npm test` working
- [ ] All five functions implemented in both languages, matching the spec
- [ ] Every test follows Arrange–Act–Assert with behavior-describing names
- [ ] At least 3 tests per function: one typical case, one boundary, one edge or error case
- [ ] `truncate` boundary trio covered: length exactly `max_len`, one under, one over
- [ ] `ordinal` tested with a parameterized table (`@pytest.mark.parametrize` / `test.each`) of at least 10 cases including 11, 12, 13, 21, 101, 111
- [ ] Every specified error condition has an exception/throw test (`pytest.raises` / `expect(...).toThrow`)
- [ ] Unicode input tested somewhere (e.g., `slugify("Crème Brûlée")` — decide and document your policy)
- [ ] Both suites pass, and you have deliberately broken each function once to confirm at least one test catches it (the "mutation check")
- [ ] A short `NOTES.md` recording every spec ambiguity you hit and the decision you made

## Hints

- The spec is intentionally slightly ambiguous in places (What does `slugify` do with leading hyphens after stripping? Is `"..."` part of `max_len` even for `max_len=4`?). Deciding and *encoding the decision as a test* is the professional move — that's what tests are for.
- Write the test list for a function *before* implementing it — enumerate boundaries and equivalence classes on paper first (Chapter 4's checklist). Implementation goes faster when the target is precise.
- For `ordinal`, the teens (11/12/13) are the classic trap — if your first implementation handles 21 correctly but not 111, your parameterized table is doing its job.
- The mutation check is the most valuable step: comment out a line, flip a `>=` to `>`, swap `-` for `+` — if no test fails, you found a hole in the suite, not proof of genius.
- Keep the two languages' test suites structurally parallel; the comparison itself teaches you what is framework and what is principle.

## Stretch goals

- Add coverage reporting (`pytest --cov`, `vitest run --coverage`) and get statement coverage above 95% — then write down one bug the 95% would still miss.
- Add a sixth function of your own design, spec'd in writing first, with the test list written before the implementation.
- Try property-based style: write one test that generates 100 random strings and asserts an invariant (e.g., `len(truncate(s, n)) <= n` for all of them).
