# AI-Assisted Development

A self-paced track for working *with* AI coding tools the way a hireable professional does in 2026: fast, but never blind. The through-line is a mindset shift — from "the AI wrote it" to "I reviewed it, tested it, and I can defend every line in an interview."

This is a **late track**. Take it after you've built real fluency in at least one language and one web-dev track (HTML/CSS, JavaScript, and ideally `node-express/` or `react/`), and run it alongside your interview-prep work. Everything here assumes you already know enough to *judge* code — that's the whole point.

## Why this track exists

Entry-level hiring changed fast. AI coding assistants (Copilot, Claude Code, ChatGPT, Cursor, and their peers) are now standard-issue at most companies, which means the junior developer's job has shifted: less "type code from scratch," more "read, verify, and integrate code — some of it AI-generated — correctly and safely." Interviewers know this. It's now common for interviews to include AI-assisted portions, and for behavioral questions to probe your AI workflow directly: *When do you use it? When don't you? How do you know the output is right?* Candidates who can't explain code they "wrote" with an assistant's help — or who used one without saying so where disclosure was expected — lose offers over it, not over the code itself.

None of this makes fundamentals optional. It raises the bar: you now need the fundamentals *and* the judgment to supervise a fast, confident, occasionally-wrong collaborator. That judgment is what this track builds.

## Who this is for & prerequisites

- You've completed (or are well into) at least one full-stack-relevant track and can read/write code independently without an assistant.
- You already use, or have access to, some AI coding tool — an inline autocomplete (Copilot), a chat assistant (ChatGPT, Claude), or an agentic tool (Claude Code, Cursor). You don't need a paid subscription to any specific one; free tiers are enough for every exercise here.
- You're comfortable with your track's testing basics (`testing-debugging-security/` chapters 1–4 are assumed background for Chapter 2 here).

## How to use this track

Study the chapters in order — each is primary material, not a summary: read actively, try the prompts, do the exercises with a real AI tool open next to you. Interleave the projects as you reach the chapters they draw on. Chapters are in `learning-docs/`; projects are in `projects/`.

Every chapter contains: Overview, Definitions & Explanations, Code Examples, Common Pitfalls, and Practice Exercises. Every project contains: Description, Difficulty + effort, Chapters Used, a Requirements checklist, Hints, and Stretch Goals. **No project ships solution code** — the whole discipline here is that you supply the judgment, whether or not you also supply every keystroke.

## Chapters (in order)

1. `learning-docs/01-how-ai-coding-tools-work.md` — what LLMs actually do (prediction, not lookup), the tool landscape (autocomplete vs. chat vs. agentic), and the failure modes that bite: hallucinated APIs, stale training data, plausible-but-wrong logic, context limits, unwavering confident tone.
2. `learning-docs/02-the-verification-workflow.md` — the core skill: treat AI output like a PR from a fast, overconfident stranger. Read every line, run it, test it, question it, check the docs. Worked example: a bug hunt.
3. `learning-docs/03-prompting-and-context.md` — giving good context, iterating vs. restarting, decomposing tasks, prompting to *learn* vs. prompting for *output*, and when not to reach for AI at all.
4. `learning-docs/04-ai-in-the-professional-workflow.md` — team AI policies, never pasting secrets or proprietary code into tools that train on input, licensing questions, reviewing AI-assisted PRs, honest documentation, and owning every line you ship.
5. `learning-docs/05-keeping-your-skills-real.md` — the atrophy risk, and deliberate-practice patterns that keep AI a leverage tool instead of a crutch.
6. `learning-docs/06-interviews-in-the-ai-era.md` — explaining AI-assisted code live, behavioral questions about your workflow, take-home policies, live coding with/without assistants, and presenting AI fluency on a resume honestly.

## Projects (easiest → hardest)

1. `projects/01-audit-an-ai-module.md` — generate a small module with an AI tool, then formally audit it: read it line by line, write tests until you find a real flaw, fix it, write up the audit. *(Ch. 1, 2)*
2. `projects/02-your-ai-workflow-policy.md` — write a one-page personal AI-use policy, apply it to a project from an earlier track, and add an honest "AI usage" section to that project's README. *(Ch. 3, 4, 5)*
3. `projects/03-explain-it-like-you-own-it.md` — interview drill: prepare and deliver a walkthrough of a piece of your own AI-assisted code — what it does, why this approach, its tradeoffs, what you changed after verifying it. *(Ch. 2, 6)*

## Suggested cadence

A comfortable part-time pace is **2–3 weeks**, run alongside interview prep rather than blocking it.

| Week | Chapters | Project work |
|------|----------|---------------|
| 1 | 1–2 | Start Project 1 |
| 2 | 3–4 | Finish Project 1; start Project 2 |
| 3 | 5–6 | Finish Project 2; do Project 3 |

Guidance, not law: Chapter 2's verification workflow is the load-bearing chapter — don't rush it, and keep applying it in every track you touch afterward, including the ones you've already finished.

## A note on timing

This track is sequenced late deliberately: verifying AI output well requires already knowing what "right" looks like. But its central habit — never ship a line you can't explain, AI-assisted or not — isn't actually late-stage knowledge. If you're using an AI tool in earlier tracks already (most learners do), skim Chapter 2 now and apply its verification workflow from day one. Come back for the full track once you're closer to interviews.

## The habits this track leaves you with

- Every AI suggestion gets read, run, and tested before it's yours.
- You can explain — out loud, to a stranger — why any line in your codebase exists.
- Context and specificity go into a prompt the same way they'd go into a good PR description.
- Secrets and proprietary code never leave your machine through a chat box.
- "I used AI for this" is a sentence you say plainly, not one you hide or over-explain.

Carry these into every interview, every team, and every track you touch again. That's the point.
