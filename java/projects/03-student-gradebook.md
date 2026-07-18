# Project 3: Student Gradebook

## Description

A console gradebook for one class of students. The teacher enters student names and their scores, then gets reports: per-student averages and letter grades, class statistics (mean, high, low, range), a ranked list, and a text histogram of grade distribution. All data lives in **arrays** managed by hand — this project is deliberately pre-collections, so you feel exactly what `ArrayList` will later automate.

## Difficulty

**Beginner-plus** — estimated effort: 4–6 hours.

## Chapters used

- 04 (Loops) — accumulation, nested iteration
- 05 (Methods & static) — every report is a method taking arrays as parameters
- 06 (Strings & Text) — formatted table output, name handling
- 07 (Arrays) — parallel arrays, 2D arrays, searching, sorting, manual resizing

## Requirements checklist

- [ ] Fixed maximum capacity (e.g., 30 students), with a `count` variable tracking how many slots are used
- [ ] Student data stored in parallel arrays: `String[] names` and `double[][] scores` (each student has the same number of assignment scores, e.g., 5)
- [ ] Menu loop: add student (name + their scores), list all students, student report, class report, rankings, quit
- [ ] Adding beyond capacity is refused with a clear message
- [ ] Scores are validated on entry (0–100); bad values are re-prompted, not stored
- [ ] Per-student report: all scores, average, letter grade (A ≥ 90, B ≥ 80, C ≥ 70, D ≥ 60, else F) via a dedicated `letterGrade(double avg)` method
- [ ] Class report: class average, highest and lowest student average (with names), and range
- [ ] Rankings: students printed best-to-worst by average — sort *without* losing the name–score pairing (hint below)
- [ ] Grade histogram: one line per letter, e.g., `B: ****` counting students per grade
- [ ] Looking up a student by name works case-insensitively and reports "not found" cleanly
- [ ] Every report is its own method; `main` contains only the menu loop and dispatch

## Hints

- Parallel arrays live or die on the invariant *index i means the same student everywhere*. When you sort by average, either sort an array of indices, or swap entries in **all** arrays together. (Sorting just the averages array scrambles the pairing — the classic bug of this project.)
- A simple approach to ranking: compute `double[] averages` once, then a selection-sort style loop that finds the next-best index each pass.
- `Arrays.toString` is your debugging friend; print state after every mutation while developing.
- Write `average(double[] row)` once and reuse it everywhere — class average is the average of averages *or* of all scores; decide which you mean and document it in a comment (they differ when you later allow varying score counts).
- Test with 0 students (reports shouldn't crash), 1 student, and a full roster.

## Stretch goals

- Variable scores per student using a jagged 2D array plus a per-student score count.
- Drop-the-lowest policy: averages computed excluding each student's worst score — as a toggleable setting.
- Weighted categories: scores tagged as homework/exam with different weights.
- "Edit score" and "remove student" menu options — removal means shifting array elements left; feel the pain that `ArrayList.remove` hides.
- A `curve(double points)` operation that adds points to every score, capped at 100.
