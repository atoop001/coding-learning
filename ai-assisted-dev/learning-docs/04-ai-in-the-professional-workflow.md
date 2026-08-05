# Chapter 4: AI in the Professional Workflow

## Overview

Everything so far has been about you and a tool. This chapter is about you, a tool, and an employer — the rules, risks, and responsibilities that show up the moment AI-assisted code leaves your machine and becomes part of a team's codebase. Team AI policies, data you must never paste anywhere, licensing questions, reviewing AI-assisted pull requests (yours and a teammate's), documenting your AI use honestly, and the one rule that swallows all the others: you own every line you ship, regardless of who — or what — typed it first.

## Definitions & Explanations

**Why companies have AI policies at all.** Once you're inside a real codebase, the code you're pasting into a chat window or letting an agent read often isn't just yours — it's the company's, sometimes a client's, sometimes covered by contracts you've never seen. A policy exists to answer questions before they become incidents: which tools are approved, what data can and can't be shared with them, whether output needs disclosure, and who's accountable if AI-generated code causes a problem in production. Policies vary a lot company to company (some are permissive, some restrict AI to certain tasks, a few ban it outright for specific codebases) — the professional habit isn't memorizing one policy, it's the reflex to *find out* the policy on day one of any new job and follow it, not assume your personal workflow transfers.

**Never paste secrets or proprietary code into a tool that trains on input.** Two separate risks live here, and it's worth keeping them separate:

- **Secrets** — API keys, passwords, tokens, connection strings, `.env` contents. Once pasted into a chat box, that text has left your machine and your company's control, full stop, regardless of what the vendor's data policy says about training. Treat "pasted into a chat" as equivalent to "posted publicly" for the purposes of deciding whether to do it. If a key ever does get exposed this way, it needs to be rotated (replaced with a new one, old one revoked) immediately — the exposure isn't undone by deleting the message.
- **Proprietary code** — your employer's actual source, which may be confidential by contract even if it contains no secrets. Some AI tools offer an explicit setting or plan tier that guarantees your input isn't used for training; many free/consumer tiers do not, and the default should be assumed "this may be retained or used" unless your company's approved tool and settings say otherwise. This is exactly the kind of thing the policy above should specify — check it rather than guess.
- **The pattern that catches both:** before pasting anything into an AI tool, ask "would I be comfortable if this text were public tomorrow?" If no, don't paste it — describe the shape of the problem instead, or use a tool your company has explicitly vetted for this data.

**Licensing and provenance.** Code an LLM generates is statistically influenced by the (often unlicensed-for-this-purpose) code it trained on; the legal questions around whether AI-generated code carries any license obligations from training data are genuinely unsettled and vary by jurisdiction and tool. You don't need to resolve this as a junior developer, but you do need to know it's a live question — which means treating an AI tool's output as license-free-by-default is not a safe assumption, and copying a large, distinctive block of AI-generated code (especially anything that looks like it could be reproducing a specific known project) deserves the same scrutiny you'd give copy-pasting from Stack Overflow into commercial code: check your company's policy, and when a chunk feels suspiciously specific rather than generic, don't ship it without asking.

**Reviewing an AI-assisted pull request — yours or a teammate's.** The verification workflow from Chapter 2 doesn't stop being necessary because the code is now in a PR instead of a chat window; if anything, a PR is exactly the artifact that workflow was modeled on. A few things specific to PR review:

- **Your own AI-assisted PR** should already have gone through Chapter 2's five steps before you open it — a PR is not where verification starts.
- **A teammate's AI-assisted PR** gets the same review any PR gets: does it solve the stated problem, is it tested, does it fit the codebase's conventions, are there edge cases it misses. The fact that AI was involved doesn't change the review bar — it's not evidence of extra quality *or* extra risk on its own; the code is the code, checked as always.
- **Disclosed AI involvement is a review help, not a red flag.** If a PR description says "drafted with Copilot, then modified — see commit 3 for the fix to the date-range bug," that's useful context for a reviewer, the same way "based on the pattern in `utils/auth.js`" would be. It tells the reviewer where to look harder, not that they should distrust the PR by default.

**Documenting AI use honestly.** Commit messages, PR descriptions, and code comments should reflect what actually happened, not perform either extreme — neither "written entirely by hand" when a tool did the first draft, nor an apologetic disclaimer on every line. A simple, factual note is enough: "Initial implementation drafted with Claude Code, reviewed and tested; fixed a missing null check in the date-parsing branch." This habit matters for the same reason disclosure matters in interviews (Chapter 6) and in Project 2's policy: honesty about process is a trust signal, and the inverse — getting caught having implied something was hand-written when it wasn't — costs far more trust than the disclosure ever would have.

**Production-readiness: you own every line you ship.** This is the rule everything else in this chapter serves. When a bug ships, "the AI wrote that part" is not a defense a team accepts, and it shouldn't be one you accept from yourself either — the moment you commit, review-approve, or merge a line, it's yours, exactly as much as if you'd typed every character. This isn't about blame; it's the practical definition of professional ownership, and it's the reason Chapter 2's verification workflow exists in the first place: verification is how you earn the right to say "I own this" honestly.

## Code Examples

### A commit message that documents AI use honestly

```text
Fix off-by-one in pagination offset calculation

Drafted the initial fix with GitHub Copilot suggesting the
`(page - 1) * page_size` change; verified against the existing
test suite, then added a new test for page=1 (previously the
bug this fix addresses) and page=0 (invalid input, now raises
ValueError instead of returning garbage).

Co-authored-by: (AI-assisted, see note above)
```

### A PR description that gives a reviewer the right context

```markdown
## Summary
Adds email validation to the signup form.

## AI usage note
Initial validation regex and inline-error UI drafted with
Claude, then modified: the AI's regex rejected valid `+`-tagged
emails (e.g. `me+test@example.com`), so I replaced it with a
simpler `contains @ and a . after it` check per our team's
existing convention in `LoginForm.jsx`.

## Testing
- Added unit tests for empty, no-@, no-domain, and valid cases
- Manually tested in the signup flow
```

### A "would this be fine if it were public tomorrow?" gut check, made concrete

```python
# BEFORE pasting into a chat tool to ask "why is this failing":
DATABASE_URL = "postgresql://admin:Sup3rSecret!@prod-db.internal:5432/app"
def connect(): ...

# The fix isn't "trust the tool's privacy settings" -- it's don't paste
# it at all. Redact first, every time, regardless of which tool:
DATABASE_URL = "postgresql://<user>:<password>@<host>:5432/<db>"
def connect(): ...
# Now the actual bug (a connection-handling issue, say) is still fully
# askable, with zero secret exposure.
```

## Common Pitfalls

- **Assuming your personal AI habits transfer unchanged to a new job.** What's fine solo may violate a client contract or company policy at work. Correction: ask about (or read) the AI policy in your first week, before pasting anything company-related anywhere.
- **Pasting a real secret "just this once" to debug faster.** One paste is enough exposure to require rotation; there's no partial-credit version of this mistake. Correction: redact before pasting, always, even under deadline pressure — it costs seconds.
- **Treating AI-assisted code as needing *less* review because "the AI already checked it."** AI tools don't self-verify against your codebase's actual behavior; a PR's review bar doesn't move based on how it was drafted. Correction: same review standard, every time, informed but not lowered by disclosure.
- **Treating AI-assisted code as needing *more* suspicion by default**, over-scrutinizing disclosed AI-assisted PRs in a way you wouldn't scrutinize equivalent hand-written code. Correction: review the code on its merits; use the disclosure to know where to look, not as a presumption of lower quality.
- **Hiding AI use out of fear it looks like less effort.** This tends to surface anyway (in a review question you can't answer, in an interview follow-up) and costs more trust than the honest note would have. Correction: a factual, unemotional disclosure line, every time it's relevant.
- **Copying a large, distinctive AI-generated block without a second thought about provenance.** Correction: generic boilerplate is low-risk; anything that reads like it could be a specific known project's actual code deserves a pause and, if in doubt, a question to a senior teammate or your policy.
- **Saying "the AI wrote that bug, not me" after something ships broken.** This isn't accepted as a defense on real teams and undermines your own credibility. Correction: internalize the ownership rule before you need it in a hard conversation.

## Practice Exercises

1. Find (or imagine, if you're not yet employed) a company's public AI usage policy — many are published in engineering blogs or handbooks. Summarize it in three bullet points: what's allowed, what's restricted, what requires disclosure.
2. Write the redacted version of a snippet of your own code that currently contains something you'd never want pasted publicly (a real or placeholder secret, an internal hostname). Practice the "public tomorrow?" gut check on it.
3. Take a real AI-assisted piece of code from your own project history. Write both a commit message and a PR description for it that honestly documents the AI involvement, following the examples above.
4. Role-play a PR review: pick a teammate's (or your own past) AI-assisted change and review it as if you didn't write it — check it against the codebase's conventions, look for untested edge cases, and write two review comments as you would on a real team.
5. Write two sentences, in your own words, defending or explaining the "you own every line you ship" rule to someone who pushes back with "but I didn't write that part." Make the argument concrete, not just repeat the rule.
