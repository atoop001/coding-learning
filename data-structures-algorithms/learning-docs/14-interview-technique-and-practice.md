# Chapter 14: Interview Technique & Practice Strategy

## Overview

Knowing the material (Chapters 1–13) and performing under interview conditions are different skills. Interviews are 35–45 minutes, out loud, with a stranger, under pressure — and the interviewer is grading your *process* at least as much as your answer: how you clarify, how you reason about trade-offs, how you test, and how you respond to hints. This final chapter gives you a repeatable procedure for any coding question, the communication habits that distinguish strong candidates, a triage map from problem signals to techniques, and a practice plan measured in weeks, not vibes. Nothing here replaces the earlier chapters; this is the delivery system for them.

## Definitions & Explanations

### The interview loop, and what's actually graded

A typical loop: 1–2 coding rounds (this chapter), possibly a systems-design round and a behavioral round. In coding rounds, most rubrics score four axes:

1. **Problem solving** — did you find a correct, reasonably efficient approach?
2. **Coding** — is the implementation clean, idiomatic, and actually finished?
3. **Communication** — could the interviewer follow your thinking the whole way?
4. **Verification** — did you test your own work, or declare victory and hope?

A working solution delivered silently scores worse than a near-complete solution narrated well. Plan for that explicitly.

### The 6-phase method (use it every single time)

**Phase 1 — Restate and clarify (2–3 min).** Repeat the problem in your own words. Ask: input size (drives complexity targets)? Sorted? Duplicates? Negative numbers/empty inputs? What to return on no-answer? Mutate input or not? Asking is scored positively — silence here is the #1 rookie tell.

**Phase 2 — Work an example by hand.** Small but not trivial (4–6 elements, include an edge). This verifies your understanding *and* often reveals the algorithm — watch what your own brain does to solve it manually.

**Phase 3 — State the brute force, out loud, with its complexity.** "Brute force: check all pairs, O(n²). Let me look for better." This banks a fallback, proves you can analyze (Chapter 1), and frames the optimization search. Do not skip it to look clever; do not *code* it unless time runs short.

**Phase 4 — Optimize using the triage table** (below). Say the trade-offs: "A hash set gets me O(n) time for O(n) space." Get the interviewer's nod *before* coding — "I'll sort, then two-pointer, O(n log n). Sound good?" This is a checkpoint where hints are cheap.

**Phase 5 — Code it, narrating intent, not syntax.** Say "now I shrink the window until valid again," not "now I write a while loop." Name variables well (`left`, `best`, `seen` — not `x`, `tmp`). If stuck on a sub-piece, stub it (`# helper: returns first index >= target`) and continue; fill in after.

**Phase 6 — Test before being asked.** Trace your code line by line — actually tracking variable values, not skimming and nodding — on the Phase-2 example, then hit edges: empty, single element, duplicates, extremes, no-answer case. Finding your own bug is a *positive* signal; the interviewer finding it is not. Close by restating final time/space complexity.

### Triage: from problem signals to technique

Compressed from Chapters 5–13 — this table should feel like an index into the whole track:

| Signal | First technique to consider | Chapter |
|---|---|---|
| "Seen before?" / duplicates / counting | Hash set / Counter | 5 |
| Sorted input, or you can afford to sort | Binary search / two pointers | 8, 12 |
| Contiguous subarray/substring, max/longest/count | Sliding window (check monotonicity) | 12 |
| Top-k / k-th largest / streaming extremes | Heap | 10 |
| Nesting, matching, undo, most-recent-first | Stack | 4 |
| Level-by-level, shortest steps (unweighted) | BFS + queue | 11 |
| Any-path, connectivity, exhaustive exploration | DFS / backtracking | 6, 11 |
| Weighted shortest path | Dijkstra (heap + BFS) | 10, 11 |
| Ordering with dependencies | Topological sort | 11 |
| Count ways / min cost / longest sequence with choices | DP — define the state first | 13 |
| Hierarchical / sorted-order / range queries | Tree, BST | 9 |
| O(1) ops at a specific position you hold | Linked list | 3 |

When lost: (a) sort the input mentally and see what opens up; (b) ask "what would I precompute into a hash map?"; (c) solve size-3 by hand and generalize; (d) say your dead-end aloud — interviewers steer talkers, not statues.

### Handling the hard moments

- **Totally stuck:** narrate your options honestly. "Window fails because negatives break monotonicity; let me try prefix sums." That sentence *is* competence.
- **Given a hint:** take it seriously and visibly — restate it, connect it. Fighting hints is a red flag.
- **Bug during testing:** stay calm, localize with the trace, fix, re-test. Debugging composure is being graded.
- **Out of time:** describe the rest: "I'd handle the empty-graph case here, and this helper is binary search from Chapter-8 shape." Partial credit is real.

### The practice plan (8–12 weeks, ~5–7 hrs/week)

**Principles.** (1) *Patterns over volume*: 60 problems chosen across patterns beat 300 random ones. (2) *Struggle is the workout*: 25–35 minutes genuinely stuck before reading a solution — but then *always* read it, and re-solve from scratch the next day. (3) *Spaced repetition*: re-solve every problem you failed after 2 days, then 1 week, then 3 weeks. Keep a log (the capstone project builds one).

**Weeks 1–2 — refresh + easy volume.** Re-skim Chapters 1–8. 15–20 easy problems on arrays/strings/hash maps. Goal: implement without pausing on syntax.
**Weeks 3–5 — pattern immersion.** 3–4 medium problems per pattern: two pointers, window, stack, BFS/DFS, heap, binary search. After each, write one sentence: "the signal was ___, the pattern was ___."
**Weeks 6–8 — weak spots + DP.** Your log tells you what's weak. Add 8–10 DP problems (Chapter 13's recipe, every time).
**Weeks 9+ — simulation.** 2–3 timed mock interviews weekly: 40-minute timer, talk aloud (yes, alone at your desk), no pauses, no docs. Trade mocks with a friend or use a mock platform; recording yourself once is worth ten articles about communication.

**Platforms.** LeetCode (the standard — filter by topic; company lists if targeting a specific employer; the "NeetCode 150" list maps almost 1:1 onto this track's chapters), HackerRank (gentler ramp), Codewars (fluency reps), Pramp/interviewing.io (free-ish live mocks). Difficulty calibration: interviews at most companies live at LeetCode easy-medium; hards are rare and usually signal a specialized team.

## Code Examples

The method itself, as an artifact — a solution written the way you'd narrate it. The comments mirror the phases; practicing this format trains the habit:

```python
# interview_format.py — "Longest substring with at most k distinct characters"
# solved in the 6-phase format. The COMMENTS are the technique.

def longest_k_distinct(s, k):
    # Phase 1 (clarified): s may be empty; k >= 0; ASCII; return a LENGTH.
    #   Edge decided with interviewer: k == 0 -> answer 0.
    # Phase 2 (hand example): s="araaci", k=2 -> "araa" -> 4.
    # Phase 3 (brute force, stated not coded): all O(n^2) substrings,
    #   each checked with a set in O(n) -> O(n^3). Too slow for n ~ 10^5.
    # Phase 4 (plan, confirmed): "contiguous substring + constraint" ->
    #   variable sliding window (Ch. 12) + count dict (Ch. 5). O(n)/O(k).
    if k == 0 or not s:
        return 0
    counts = {}                       # mirror of the window's contents
    left = best = 0
    for right, ch in enumerate(s):    # Phase 5: narrate INTENT per block
        counts[ch] = counts.get(ch, 0) + 1        # grow window rightward
        while len(counts) > k:                    # constraint broken:
            counts[s[left]] -= 1                  #   evict from the left
            if counts[s[left]] == 0:              #   (keep mirror exact —
                del counts[s[left]]               #    Ch. 12 pitfall #2)
            left += 1
        best = max(best, right - left + 1)        # window valid: record
    return best

# Phase 6: test my own work BEFORE the interviewer asks.
if __name__ == "__main__":
    cases = [
        (("araaci", 2), 4),      # the hand example from phase 2
        (("araaci", 1), 2),      # "aa"
        (("", 3), 0),            # empty input edge
        (("aaaa", 1), 4),        # all-same edge
        (("abc", 0), 0),         # k == 0 edge (clarified in phase 1)
        (("abcde", 5), 5),       # k >= distinct count
    ]
    for args, want in cases:
        got = longest_k_distinct(*args)
        status = "ok " if got == want else "FAIL"
        print(f"{status} {args} -> {got} (want {want})")
    # Close-out sentence for the interviewer:
    # "O(n) time — each index enters and leaves the window once — O(k) space."
```

A reusable practice-log entry format (the capstone project automates this):

```python
# One dict per solved problem. Reviewing `signal` lines weekly IS the studying.
entry = {
    "date": "2026-07-18",
    "problem": "Longest Substring with K Distinct",
    "source": "LeetCode 340",
    "pattern": "sliding window (variable)",
    "signal": "contiguous substring + 'at most k' constraint",
    "solved_unaided": False,
    "bug_i_wrote": "forgot to delete zero-count keys -> len(counts) stuck",
    "redo_on": ["2026-07-20", "2026-07-27", "2026-08-17"],
}
```

## Common Pitfalls

**1. Jumping straight to code.** The most common failure. Charging into an O(n²) solution unprompted — or the *wrong problem* because you skipped clarification — wastes ten minutes you can't recover. The corrected behavior is literally Phases 1–4: minutes of talk before the first keystroke, ending with an approved plan.

**2. Silent coding.** Two minutes of quiet is an eternity to an interviewer, and it converts a process interview into a pass/fail code review. Fix: narrate *intent* at block level. If you truly need silence for a tricky line, say "give me 30 seconds to get this loop boundary right" — announced silence is fine.

**3. Memorizing solutions instead of patterns.** Memorizers shatter on one-word variations ("at most k" → "exactly k"). If you can't explain *why* the window shrink is valid, you don't own the solution. The one-sentence "signal → pattern" log line after every problem is the antidote.

**4. Declaring done without testing.** "Looks right" is not verification (the earlier chapters' pitfalls exist because code that looks right often isn't). Trace with real values; check the empty/single/duplicate/no-answer edges. Interviewers frequently withhold approval specifically to see whether you test unprompted.

**5. Practicing only problems, or only reading.** Reading solutions feels productive and transfers little; grinding problems without post-mortems repeats mistakes efficiently. The loop that works: attempt hard → study solution → write the pattern sentence → re-solve cold on the review schedule.

**6. Ignoring the human across the table.** Behavioral basics decide close calls: have 3–4 concise stories ready (conflict, failure, ownership, learning — STAR shape: Situation, Task, Action, Result), ask real questions at the end, and treat hints as collaboration, not correction. "Strong solve, difficult to work with" is a rejection.

## Practice Exercises

1. Take a medium problem you have *already* solved and re-do it in full 6-phase format, out loud, with a 40-minute timer, recording audio. Listen back and note every silence over 30 seconds and every place you said "uh, wait, actually." Redo it once more a week later and compare.
2. Build your personal triage table from memory (don't look at this chapter's), covering at least 10 signals. Then diff it against the chapter's table — each miss marks a chapter to re-skim.
3. For each of these one-line prompts, write ONLY Phases 1–4 (clarifying questions, hand example, brute force + complexity, planned approach + complexity) with no code: (a) "find all pairs of songs whose durations sum to a multiple of 60"; (b) "serialize and deserialize a binary tree"; (c) "min cost to paint n houses with 3 colors, adjacent houses differing".
4. Solve one easy problem in Python, then immediately in JavaScript. Note every point where language differences (integer division, sort comparators, dict vs Map) would have bitten you live — keep that list; it's your pre-interview warm-up sheet.
5. Draft your 8–12 week plan as actual calendar entries: which patterns which weeks, which days are mock days, and where the spaced-repetition redos land. Then execute week 1 and revise the plan based on what actually happened — the revision habit is the plan.

---

**You've reached the end of the chapters.** The projects directory turns this knowledge into working artifacts — finish with the capstone "interview gym," which is exactly the practice log and problem set this chapter prescribes.
