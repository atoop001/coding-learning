# Project 1: Audit an AI-Generated Module

## Description

Have an AI tool generate a small, self-contained module — a date-range utility, an input validator, a string-formatting helper, or something similar with clear inputs, outputs, and edge cases. Then put on your reviewer hat and formally audit it, exactly the way Chapter 2's verification workflow describes: read it line by line, write your own tests (from the spec you gave the AI, not from the code it produced) until you find at least one real flaw or unhandled edge case, fix that flaw yourself, and write a short audit report documenting what you found and how you found it. The deliverable isn't just working code — it's evidence that you can catch what an unreviewed merge would have missed.

## Difficulty & Estimated Effort

**Beginner–Intermediate.** Estimated effort: 3–5 hours.

## Chapters Used

- Chapter 1 — How AI Coding Tools Work (why the flaw is likely there in the first place)
- Chapter 2 — The Verification Workflow (the five-step process this whole project runs)

## Requirements

- [ ] Choose (or write) a clear one-paragraph spec for a small module before prompting the AI — e.g., "a function that checks whether a password meets: 8+ characters, at least one digit, at least one letter; returns True/False and, separately, a list of which rules failed."
- [ ] Prompt an AI tool to generate the module from that spec. Save the AI's original, unmodified output in the project folder (e.g., `original_ai_output.py`) before you touch it.
- [ ] Read every line of the generated code and write a short note (a few sentences) on what each function is supposed to do, in your own words, before running anything.
- [ ] Run the code manually on a few obvious inputs to confirm it executes.
- [ ] Write your own test suite, derived from your original spec — not from the AI's code — covering: the happy path, at least three boundary/edge cases, and at least one case your spec was ambiguous or silent about (decide what the "right" behavior should be and test for it).
- [ ] Run your test suite against the AI's original code. Find at least one genuine failure — a real bug, an unhandled edge case, or a case where behavior doesn't match a reasonable reading of the spec. (If your first module doesn't produce a real failure, try a second, slightly more complex module — most will surface something.)
- [ ] Fix the flaw yourself, in your own words and code, without asking the AI to fix it for you.
- [ ] Re-run your full test suite against the fixed version; confirm everything passes.
- [ ] Write a short audit report (`AUDIT.md`, 1–2 pages) covering: what was generated, what the flaw was, why it's a real problem (not just a style nitpick), how your test caught it, and the fix.
- [ ] Check the docs for any library function the AI's code used that you hadn't personally verified before (Chapter 2, step 5) — note in the report whether everything checked out.

## Hints

- Pick a module with real edge-case surface area — date/time handling, input validation, string parsing, and numeric range logic are all reliable sources of a genuine flaw, more so than something purely arithmetic.
- Write your tests from the spec you gave the AI, not by reading the code and guessing what it was "trying" to do — Chapter 2's `in_range` example shows exactly why this distinction matters.
- If the module has no library calls at all, that's fine — just note it in the report; the point of step 5 is the check, not forcing a finding where none exists.
- A flaw doesn't have to be a crash. A silent wrong answer on an edge case (the kind Chapter 2's worked example shows) is often more valuable to find than an obvious exception, because it's the kind that ships unnoticed.
- If you genuinely can't find a flaw after real effort, that's a legitimate outcome to report — but be honest with yourself about whether you tested the boundaries hard enough first. Try at least one deliberately adversarial or unusual input before concluding the module is clean.

## Stretch Goals

- Repeat the audit with a different AI tool (a different chat assistant, or an agentic tool if you have access) on the same spec, and compare which flaws each version had, if any.
- Ask the AI, after your audit is done, to review your own fix — see whether it agrees with your reasoning or raises something you missed.
- Add a short "what I'd check differently next time" section to your `AUDIT.md`, reflecting on which part of the five-step workflow was actually the one that found the flaw.
