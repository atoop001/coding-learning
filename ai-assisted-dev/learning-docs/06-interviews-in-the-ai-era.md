# Chapter 6: Interviews in the AI Era

## Overview

Everything in this track converges here. Interviewers in 2026 assume candidates use AI tools — the question is no longer whether you did, it's whether you can explain, defend, and take responsibility for what you shipped. This chapter covers explaining AI-assisted code under direct questioning, the behavioral questions that probe your workflow rather than your output, how take-home assignments typically handle AI (and why you always ask rather than assume), live coding with and without an assistant, and how to describe your AI fluency on a resume without overclaiming or underselling it.

## Definitions & Explanations

**"Walk me through this" is a verification test, not a trivia test.** When an interviewer asks you to explain a piece of your own code — AI-assisted or not — they are checking exactly what Chapter 2 trained: do you understand what you shipped, or did you ship something you can't account for. The strongest answer covers, in order: what the code does, why you (or the AI, initially) chose this approach over alternatives, what its tradeoffs and limits are, and what you changed after you verified it. That last part is often the most convincing thing you can say in the whole interview — a specific bug you caught and fixed is direct, undeniable evidence of the verification habit, stronger than any claim of "I always review AI code carefully" on its own.

**Behavioral questions about your AI workflow are now common, and they have a specific shape.** Expect variants of: *"Tell me about a time AI gave you wrong or misleading code — how did you catch it?"* or *"How do you decide when to use AI versus writing something yourself?"* or *"Describe your process for reviewing AI-generated code."* These aren't gotchas designed to catch you using AI — using it is assumed and fine. They're designed to catch candidates with no process at all — someone who accepts suggestions without a verification habit, described honestly, reads as a liability regardless of how the code itself turned out. The strongest answers here are specific and drawn from real experience (this is exactly why Project 1 and Project 3 in this track exist — they hand you real material for this question) rather than a generic, rehearsed-sounding description of "best practices."

**Take-home assignments: policies vary, so always ask, and always disclose per whatever the rules turn out to be.** Some companies explicitly welcome AI assistance on take-homes and evaluate your process alongside the output; others explicitly prohibit it to test unassisted ability; many say nothing and leave it ambiguous, which is itself a reason to ask rather than guess. The one move that's wrong in every scenario is guessing incorrectly and getting caught — either using AI when it was implicitly expected not to be, or being needlessly slow because you assumed it wasn't allowed when it was. Ask directly, in writing if possible ("Is AI assistance okay for this assignment? I want to make sure I follow your process correctly"), and then disclose your actual usage in the submission exactly as agreed, using the honest-documentation habit from Chapter 4.

**Live coding, with and without an assistant.** Some live interviews explicitly allow an AI tool as part of the exercise (increasingly common, since it mirrors real work); many traditional ones still don't, especially earlier screening rounds meant to assess raw fundamentals. Two separate readiness levels follow from this, and you need both:

- **With AI allowed:** the interview is now testing your *workflow* as much as your output — how you prompt, how quickly you catch a wrong suggestion, whether you read before accepting, whether you can explain what the tool gave you. Narrating your verification process out loud (Chapter 2's steps, said aloud as you do them) turns an invisible habit into visible signal for the interviewer.
- **With AI not allowed:** this is exactly what Chapter 5's no-AI practice sessions prepare you for. If your fundamentals only exist in AI-assisted form, this round exposes the gap immediately and under pressure — which is the single strongest argument in this entire track for taking Chapter 5 seriously rather than treating it as optional.
- **The practical move:** always ask, before the interview starts if it wasn't already stated, which mode applies — the same "always ask" habit as take-homes.

**Presenting AI fluency on a resume, honestly.** Two failure modes exist in opposite directions, and both cost you: overclaiming ("Expert in AI-driven development," listed like a framework, with nothing concrete behind it) reads as buzzword-chasing to anyone who interviews you about it for thirty seconds; omitting it entirely, when you do use these tools as part of your real workflow, is an inaccurate picture of how you actually work and wastes a chance to show a skill that's now genuinely relevant. The middle path: describe it the way you'd describe any other real skill — specific and tied to outcomes, not a certification-style claim. "Use AI-assisted tools (Claude Code, Copilot) as part of a test-driven workflow — draft, verify, and own every change" is concrete, checkable in an interview, and true. A bullet like that also directly sets up the "walk me through your AI workflow" behavioral question — you've already previewed the answer.

## Code Examples

### A strong "walk me through this" answer, structured

```text
INTERVIEWER: "Walk me through this pagination function."

WEAK ANSWER (no verification story):
"Yeah, I used Copilot for this, it just generates the offset from
the page number and limit."

STRONG ANSWER (verification story, specific):
"This calculates the SQL OFFSET from a page number and page size --
(page - 1) * page_size. I drafted it with Copilot, but the first
version didn't handle page=0, which returned a negative offset and
would've thrown a database error in production instead of a clean
400. I wrote a test for that case, confirmed it failed, then added
validation that rejects page < 1 before the calculation runs. I
chose to validate at this layer rather than in the route handler
because [reason specific to their codebase/your design]."
```

The strong version demonstrates exactly the chain this whole track builds: used AI, verified it, found a real gap, fixed it, and can justify the design choice independent of where the first draft came from.

### Asking about a take-home's AI policy, without sounding like you're fishing for an excuse

```text
"Before I start the take-home -- is AI assistance (Copilot, ChatGPT,
etc.) okay to use for this? Just want to follow your process
correctly either way, and I'll note in my submission how I used it
if so."
```

### Disclosure line in a take-home submission

```markdown
## AI usage
Used Claude for the initial draft of the CSV-parsing function and
to explain the pandas `groupby` API I hadn't used before. Wrote all
tests myself; found and fixed a bug where the AI's version dropped
rows with missing values instead of raising an error, which the
assignment's spec required.
```

### A resume line that's specific instead of a buzzword

```text
WEAK:   "AI-powered developer. Expert in prompt engineering."

BETTER: "Use AI coding assistants (Claude Code, Copilot) as part of
a test-first workflow -- draft with AI, verify independently, own
every line shipped."
```

## Common Pitfalls

- **Being unable to explain a line because "the AI wrote that part."** This is the single most damaging answer available in a technical interview in this era — it directly contradicts the ownership expectation from Chapter 4 and answers the exact question being tested. Correction: never submit or bring to an interview code you haven't personally verified well enough to explain.
- **Rehearsing a generic "best practices" answer to the AI-workflow behavioral question instead of a real, specific story.** Interviewers can tell the difference immediately. Correction: have one or two real, specific examples ready — a genuine bug you caught, a real prompt-iteration story — from your own project history (this is what Projects 1 and 3 are for).
- **Guessing at a take-home's AI policy instead of asking.** Guessing wrong in either direction costs you — used AI when it was silently expected not to, or spent 3x as long assuming it wasn't allowed when it was. Correction: ask directly and in writing when possible.
- **Not disclosing AI use in a take-home after agreeing to, or after the policy required it.** Getting caught (mismatched code style, an interviewer directly asking and getting an evasive answer) costs far more trust than an upfront disclosure would have. Correction: disclose exactly as the stated policy requires, honestly, per Chapter 4's habit.
- **Assuming a live-coding round allows AI without confirming.** Some traditional rounds specifically test unassisted fundamentals, and using a tool you weren't supposed to can end the interview badly. Correction: ask before the round starts if it wasn't stated.
- **Overclaiming AI expertise on a resume with nothing concrete behind it.** This invites a follow-up question you can't answer well. Correction: keep resume claims specific and tied to real workflow habits and outcomes, per the example above.
- **Never practicing live coding without an assistant at all**, because most of your recent practice has been AI-assisted. This is the direct, high-stakes cost of skipping Chapter 5's no-AI sessions. Correction: run unassisted practice regularly enough that an unassisted round isn't a surprise.

## Practice Exercises

1. Pick a real piece of your own AI-assisted code (ideally from Project 1 or Project 3). Write out, in full sentences, the "strong answer" structure from the example above: what it does, why this approach, tradeoffs, what you changed after verifying.
2. Draft your answer to "tell me about a time AI gave you wrong or misleading code — how did you catch it?" using a real, specific example from your own work, not a generic description.
3. Write the exact message you'd send to ask about a take-home's AI policy, and a disclosure paragraph you'd include in the submission assuming AI assistance was allowed.
4. Do a timed, unassisted live-coding practice problem (pick anything from `data-structures-algorithms/` if you've covered it) with no AI tool open, narrating your thought process out loud as if an interviewer were listening.
5. Write your own one- or two-line resume bullet describing your AI-assisted workflow, following the "specific and checkable" pattern from the example above. Have it ready to defend if someone asks about it directly.
