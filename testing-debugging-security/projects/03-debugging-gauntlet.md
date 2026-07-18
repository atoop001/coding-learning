# Project 3: The Debugging Gauntlet

## Description

A deliberate practice project for debugging skill. You will build a small "Grade Book" application from a spec whose features are *classic bug magnets* — off-by-ones, float math, mutation, timezone-ish date logic, state leaking between operations. The twist: you build it **fast and loose first** (no tests, minimal care — timeboxed!), then switch roles and become the maintenance programmer: find the bugs systematically using stack traces, bisect probes, and the VS Code debugger; write a regression test for each before fixing it; and log the whole hunt. The point is not to avoid writing bugs — it's to build a repeatable process for hunting them. One language of your choice (Python or JS).

The Grade Book spec (build all of it):

1. `add_student(book, name)` — names unique, case-insensitive.
2. `record_score(book, name, assignment, score)` — score 0–100; re-recording overwrites.
3. `student_average(book, name)` — mean of that student's scores.
4. `assignment_stats(book, assignment)` — min/max/mean across students; students missing that assignment are excluded (not counted as 0).
5. `letter_grade(avg)` — A ≥ 90 > B ≥ 80 > C ≥ 70 > D ≥ 60 > F.
6. `top_n(book, n)` — the n highest-averaging students, ties broken alphabetically.
7. `curve(book, assignment, points)` — add points to every recorded score for that assignment, capping at 100.
8. `format_report(book)` — a text table: name, average to 1 decimal, letter grade, sorted by average descending.

## Difficulty

**Intermediate.** Estimated effort: 7–10 hours (phase 1 timeboxed to 2).

## Chapters used

- Chapter 7 — Debugging Fundamentals (stack traces, bisecting, hypotheses)
- Chapter 8 — Using Real Debuggers (breakpoints, watches, conditional breaks)
- Chapter 9 — Systematic Debugging & Prevention (minimal repros, regression tests, assertions)
- Chapter 4 — Edge Cases & Test Design (the bug-shaped inputs)

## Requirements checklist

**Phase 1 — build fast (max 2 hours, enforce it)**
- [ ] All 8 features implemented in one sitting, no tests, speed over care
- [ ] Commit as-is: this snapshot is your "legacy codebase"

**Phase 2 — the hunt**
- [ ] A `HUNT-LOG.md` where every bug found gets an entry: symptom → hypothesis → experiment → root cause → fix (one hypothesis at a time, as per Chapter 7)
- [ ] Probe the classic suspects with targeted inputs, at minimum: averages at exactly 90/80/70/60 (boundary vs `letter_grade`); a student with zero scores; float sums (e.g., scores 33.1 + 33.2 + 33.7); `top_n` with n larger than the roster and with tied averages; `curve` pushing a 98 past the cap; case-insensitive name collisions (`"Ada"`/`"ada"`); two grade books used in the same session (shared-state leak); calling `assignment_stats` for an assignment nobody has
- [ ] Each confirmed bug gets a **failing regression test first**, then the fix, then a green run (Chapter 9's iron rule) — minimum 8 regression tests by the end
- [ ] At least two bugs diagnosed *with the VS Code debugger* (not prints): note in the log which panels/technique cracked each (watch expression, conditional breakpoint, call stack…)
- [ ] At least one bug reduced to a ≤10-line minimal repro pasted into the log
- [ ] At least two assertions added to the code guarding invariants the hunt revealed (e.g., "every stored score is 0–100")

**Phase 3 — proof**
- [ ] Full test suite green; `format_report` output verified against a hand-computed fixture roster of 4 students
- [ ] Log concludes with your top-3 personal bug patterns — the mistakes *you* apparently make under time pressure

## Hints

- Phase 1's time pressure is the bug generator — don't fight it, don't secretly test. Whatever you ship is perfect raw material.
- If Phase 2 finds fewer than five bugs, your probes are too gentle, not your code too good. The suspect list maps one-to-one onto classic bug classes; push each input harder (boundaries, empties, duplicates, big n).
- Conditional breakpoints shine in `curve` and `top_n` loops: break only when `score > 100` or when two averages are equal.
- When a probe produces a wrong answer with no crash, bisect: check the intermediate collections midway through the computation before reading any code closely.
- Write the log entry *as you hunt*, not after — the discipline of stating a hypothesis before testing it is most of the skill.

## Stretch goals

- Swap "legacy codebases" with a friend doing the same spec, and hunt each other's bugs cold — debugging unfamiliar code is the real professional experience.
- Practice `git bisect`: pick one fixed bug, find which Phase-1-era commit introduced it (make a few extra commits in Phase 1 to enable this).
- Add logging (Chapter 9 style) to the module, then re-diagnose one already-fixed bug using only the log output at DEBUG level, prints and debugger forbidden.
