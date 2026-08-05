# Chapter 2: The Verification Workflow

## Overview

Chapter 1 established *why* AI output needs checking. This chapter gives you the actual habit — the one thing in this whole track that matters most if you only ever adopt one: **treat every piece of AI-generated code like a pull request from a fast, talented, overconfident stranger who has never seen your codebase before and cannot be trusted to know what "done" means.** You wouldn't merge that PR unread. You wouldn't merge it because it compiled. You'd read it, run it, test it, ask why it's built that way, and check any claim it makes about a library against the library's own docs. That's the whole workflow — this chapter breaks it into five concrete steps and walks a real bug through all five.

## Definitions & Explanations

**The five-step verification workflow:**

1. **Read every line.** Before you run anything, read the code the way you'd read a teammate's diff — not skimming for "does this look like code," but tracing what each line actually does. If you can't explain a line, that's not a green light to move on; it's the exact line to stop on.
2. **Run it.** Confirm it executes, on real inputs, in your actual environment — not just that it has no red squiggles. "It compiled" and "it ran once without crashing" are the two weakest forms of evidence about correctness that exist; treat them as a prerequisite to checking, not a substitute for it.
3. **Test it — and write the test yourself.** Don't ask the AI to write the tests for the code it just wrote; that's asking the stranger to review their own PR. You write tests independently, from what the function is *supposed* to do (its spec), not from what it *happens* to do (which just recreates any bug it already has). This is the single highest-leverage step in the whole workflow.
4. **Question it.** Ask, out loud or in writing: *why this approach, specifically? What would break it? What's the worst input I could hand it?* If you're using a chat or agentic tool, ask the model these questions directly — "what edge cases does this not handle?" often surfaces a real gap, because the same model that wrote confidently the first time will often self-critique honestly the second time, once explicitly prompted to look for problems rather than produce a solution.
5. **Check the docs for any API it claims exists.** Any function, method, parameter, or config key the AI used from a library you didn't already verify in Chapter 1's exercises gets a quick trip to the official docs. This step alone catches most hallucinations before they cost you debugging time.

**Verification effort scales with two things: risk and unfamiliarity.** A throwaway script for your own one-time use, in a language you know cold, warrants a lighter pass than an auth function, a payment calculation, or code you're about to ship in a language you're still learning — because in the second case, you're both less able to catch a subtle wrongness by eye *and* the cost of missing one is higher. Calibrate; don't apply either "audit everything with equal paranoia" or "trust everything that runs" as a blanket rule.

**"I don't fully understand this" is a stop sign, not a footnote.** If AI output does something you can't explain, three real options exist: ask the AI to explain it (Chapter 3 covers prompting for this), look it up yourself, or don't ship it. "Ship it because it seems to work" is not on that list — you cannot verify what you can't explain, and code you can't explain is exactly the code that fails an interview walkthrough (Chapter 6) and exactly the code most likely to hide the kind of bug this chapter is built to catch.

## Code Examples

### Worked example: a subtle bug, and the steps that catch it

Suppose you prompted an AI tool: *"write a Python function that returns True if a date falls within a given date range, inclusive of both ends."* You get this back, confidently, with no caveats:

```python
from datetime import date

def in_range(check_date, start, end):
    """Return True if check_date falls within [start, end], inclusive."""
    return start <= check_date <= end
```

**Step 1 — read every line.** It reads clean: a chained comparison, inclusive on both ends as asked, type hints would help but aren't required. Nothing jumps out yet — which is exactly the point of not stopping at step 1.

**Step 2 — run it.** 

```python
>>> in_range(date(2026, 3, 15), date(2026, 3, 1), date(2026, 3, 31))
True
>>> in_range(date(2026, 4, 1), date(2026, 3, 1), date(2026, 3, 31))
False
```

Looks correct on the obvious cases. This is exactly where an unverified merge would happen — it "works."

**Step 3 — test it, from the spec, not from the code.** The spec said *inclusive of both ends* and *a date range* — that implies boundary cases and implies the caller might reasonably pass the range backwards. Write tests for exactly what the spec promises, independent of what you just read:

```python
import pytest
from datetime import date
from date_range import in_range

def test_start_boundary_is_inclusive():
    assert in_range(date(2026, 3, 1), date(2026, 3, 1), date(2026, 3, 31)) is True

def test_end_boundary_is_inclusive():
    assert in_range(date(2026, 3, 31), date(2026, 3, 1), date(2026, 3, 31)) is True

def test_single_day_range():
    d = date(2026, 3, 1)
    assert in_range(d, d, d) is True

def test_reversed_range_raises_or_is_documented():
    # The spec never said what happens if start > end. The function
    # SILENTLY returns False for every date, which looks like "not in
    # range" instead of "invalid range" -- a caller bug hiding as a
    # false negative. This test is the one that finds the real flaw:
    with pytest.raises(ValueError):
        in_range(date(2026, 3, 15), date(2026, 3, 31), date(2026, 3, 1))
```

That last test fails against the AI's version — not because the chained comparison is wrong, but because the function has *no input validation* for a caller error the spec never ruled out. This is the flaw: not a crash, not an obviously wrong answer, just a silent wrong answer for a case nobody happened to try by hand. This is exactly the class of bug that step 2 (run it a few times) will never catch and step 3 (test it against the spec) reliably does.

**Step 4 — question it.** Asked directly, *"what happens if start is after end?"*, most AI tools will correctly identify the gap once prompted — the model is quite capable of finding this bug, it just didn't surface it unprompted, because generating a solution and critiquing a solution are different tasks even for the same model.

**Step 5 — check the docs.** Nothing here uses a library call, so this step is a no-op for this example — but note that step 5 would have been the *first* thing to catch a hallucinated `datetime` method, had one been used.

**The fix**, written by you once the real spec is clear (validate, then compare):

```python
def in_range(check_date, start, end):
    """Return True if check_date falls within [start, end], inclusive.

    Raises ValueError if start is after end.
    """
    if start > end:
        raise ValueError("start date must not be after end date")
    return start <= check_date <= end
```

Notice what made this catchable: not superior reading skill, but a test written from the *spec* the AI was given, not from the code the AI produced. That's the reusable lesson.

## Common Pitfalls

- **Testing the code instead of the spec.** Asking the AI "write tests for this function" tests what the function *does*, bugs included — it will happily write a passing test for `in_range` with a reversed range returning `False`, because that's what the code does. Correction: derive tests from the original request/requirements, written or reasoned through independently.
- **Stopping at "it ran once."** A single manual run through the obvious case is evidence of almost nothing. Correction: step 3 (real tests, including boundaries and invalid input) is not optional for anything you intend to keep.
- **Skimming instead of reading.** Eyes moving over code without tracing what each line does *feels* like step 1 but isn't step 1. Correction: if you can't paraphrase what a line does without re-reading it, you haven't read it yet.
- **Verifying once, then trusting the next output from the same session at face value.** Confidence and correctness are independent every single time, not just the first time in a session. Correction: the workflow runs on new output, every time, no matter how good the last three answers were.
- **Treating "I don't understand this" as embarrassing rather than informative.** Some learners skip questioning unfamiliar AI code because asking feels like admitting weakness. Correction: not understanding a line is data about what to check, not a verdict on your ability — and shipping what you don't understand is the actual risk, not asking about it.
- **Applying full audit-depth to every single AI suggestion, including trivial ones.** Over-applying this workflow to a one-line, no-risk autocomplete accept burns time you need for the code that matters. Correction: calibrate depth to risk and unfamiliarity, as described above — this is a skill, not a checklist to run identically every time.

## Practice Exercises

1. Ask an AI tool for a small function with a real spec (pick anything with a boundary condition: "clamp a number to a range," "check if a password meets length requirements," "split a full name into first/last"). Run the full five-step workflow and write down what step 3 caught, if anything, that step 1 and step 2 didn't.
2. Take the `in_range` example above yourself: implement the fixed version, then write two more tests the AI's original spec doesn't obviously call for but a careful reviewer would ask about (What if `check_date` is `None`? What if the arguments are strings, not `date` objects?).
3. Find or recreate a case where asking the AI "what could go wrong with this?" surfaced a real gap it didn't mention when first generating the code. Write the before/after: what it said unprompted vs. what it said once explicitly asked to critique.
4. Pick one piece of AI-generated code from your own past work (or generate one now) that calls a library function. Do step 5 — check the actual docs for every parameter used — and note any mismatch, even a minor one (wrong default, deprecated parameter, wrong return type).
5. Write your own one-paragraph description of "how much verification is enough," calibrated the way this chapter describes (by risk and unfamiliarity). Apply it to three examples: a personal script, a form-validation function on a real project, and an authentication check. Justify why each gets a different amount of scrutiny.
