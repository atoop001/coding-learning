# Project 7: Capstone — The Interview Gym

## Description

The capstone is different: instead of building one system, you build a *practice discipline* and run yourself through it. You will solve a curated set of 18 classic problems spanning every pattern in the track, each one under interview conditions (timed, out loud, brute-force-first), and produce a written solution log entry for every problem — the signal you spotted, the pattern you chose, the complexity, the bug you wrote, and the redo schedule. You'll also build the small tool that makes the discipline stick: a CLI practice tracker that schedules your spaced-repetition redos. When this project is done you don't just *know* the material — you have evidence you can perform it, and a machine for maintaining it.

## Difficulty

**Advanced (capstone).** Estimated effort: 20–30 hours spread over 3+ weeks — the spacing is part of the design; do not binge it.

## Chapters used

All of them. Chapter 14 is the operating manual; Chapters 1–13 supply the techniques. Each problem below names its chapters.

## The problem set

Solve these 18, in roughly this order (they interleave patterns deliberately — real interviews don't group by topic). All are classic problems findable on LeetCode/NeetCode by name; solve from the problem statement, not from solutions.

| # | Problem | Pattern | Chapters |
|---|---------|---------|----------|
| 1 | Two Sum | hash map | 5 |
| 2 | Valid Parentheses | stack | 4 |
| 3 | Merge Two Sorted Lists | linked list / merge | 3, 7 |
| 4 | Best Time to Buy & Sell Stock | one-pass min tracking | 1, 12 |
| 5 | Valid Anagram | frequency count | 5, 12 |
| 6 | Binary Search (+ first/last position) | binary search | 8 |
| 7 | Reverse Linked List | pointer manipulation | 3 |
| 8 | Longest Substring Without Repeating Characters | sliding window | 12 |
| 9 | Maximum Subarray | running state / Kadane | 12, 13 |
| 10 | K Closest / Kth Largest Element | heap top-k | 10 |
| 11 | Merge Intervals | sort-then-sweep | 7, 12 |
| 12 | Product of Array Except Self | prefix/suffix | 12 |
| 13 | Validate Binary Search Tree | tree recursion + bounds | 6, 9 |
| 14 | Binary Tree Level Order Traversal | BFS / queue | 4, 9, 11 |
| 15 | Number of Islands | grid DFS/BFS | 11 |
| 16 | Course Schedule | topological sort / cycle detection | 11 |
| 17 | Coin Change | DP (tabulation) | 13 |
| 18 | House Robber (+ LIS if time allows) | DP (state design) | 13 |

## Requirements checklist

### The tracker tool (build first, ~1 evening)
- [ ] A CLI program managing a log file (JSON or CSV) of solve entries with fields: date, problem, source, pattern, signal sentence, solved-unaided (bool), time taken, bug-I-wrote, redo dates
- [ ] `add` command that computes redo dates automatically (+2 days, +7 days, +21 days)
- [ ] `due` command listing redos due today or overdue, sorted
- [ ] `stats` command: solve rate unaided, per-pattern counts, average time — so weak patterns are visible in numbers
- [ ] Uses structures you built or mastered in this track where they naturally fit (e.g., a heap or sorted insert for the due queue) — note which, in comments

### The 18 solves
- [ ] Every problem attempted under protocol: 40-minute timer, phases 1–6 from Chapter 14, thinking out loud (talk to the empty room; it works), brute force stated before optimizing
- [ ] Every problem gets a solution file in Python: your final code, with the Phase 1–4 reasoning as comments at the top (see Chapter 14's `interview_format.py` for the shape) and your own test cases at the bottom
- [ ] Every problem gets a tracker entry — including honest `solved_unaided: false` when you needed the solution; those are the entries that matter most
- [ ] Problems you failed: re-solved from a blank file on the +2-day redo — no peeking at your previous attempt
- [ ] At least 5 problems additionally solved in JavaScript (your choice which) with a note on any language trap encountered
- [ ] All scheduled +2 and +7 day redos actually executed for failed problems (the +21s extend past the project; keep the tracker alive)

### The write-up
- [ ] `GYM-REPORT.md`: your per-pattern win rate, your three most repeated bug *types* (not instances) with the habit you're adopting against each, your average solve time trend, and your triage table (Chapter 14, exercise 2) rewritten from memory at the end — plus a 2-week maintenance plan
- [ ] At least 2 full mock interviews (self-recorded audio, or a partner/platform) on problems from the set's patterns but NOT from the set; one paragraph of honest review of each recording in the report

## Hints

- Build the tracker first even though it delays the "real" work — logging by hand collapses within a week, and the tool is itself a mini-review of files, dicts, sorting, and dates.
- If a 40-minute timer expires: stop, log the attempt honestly, and spend 20 minutes studying the solution *actively* — reproduce it from memory immediately, then schedule the blank-file redo. That loop is where the learning happens.
- The "signal sentence" is the single highest-value field in the log. Force yourself to write it even when the pattern feels obvious — especially then.
- Speaking aloud alone feels absurd for about two problems, then becomes automatic. Recording even one session and listening back is worth more than any hint in this file.
- Order interleaving is intentional but not sacred: if you fail two problems of the same pattern back-to-back, pause and re-read that chapter's pitfalls section before the next attempt.
- Don't let the tracker scope-creep into a web app. A file and three commands. The lifting is the solving.

## Stretch goals

- Extend the set to the full NeetCode 150 at a sustainable 3–4 problems/week cadence, tracker-driven.
- Add `mock` mode to the tracker: picks a random due-or-new problem, starts a 40-minute countdown, and logs the session on completion.
- Write a one-page "cheat sheet from memory" (complexities table + triage table + your personal pitfalls) and check it against the chapters — the diff is your final study list.
- Schedule a real interview (or a graded platform mock) within 4 weeks of finishing, while the training is hot. The gym was for something.
