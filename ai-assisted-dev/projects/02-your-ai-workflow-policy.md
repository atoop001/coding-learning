# Project 2: Your AI Workflow Policy

## Description

Write a one-page personal AI-use policy — the kind of document a thoughtful professional keeps for themselves even without an employer requiring it. Cover when you use AI tools, when you deliberately don't, and how you verify what they give you, grounded in what you've actually learned in this track rather than generic advice. Then put the policy to work: apply it to one real project from an earlier track in this workspace, and add an honest "AI usage" section to that project's README documenting how AI was (or wasn't) involved, following Chapter 4's disclosure habit.

## Difficulty & Estimated Effort

**Beginner–Intermediate.** Estimated effort: 2–4 hours.

## Chapters Used

- Chapter 3 — Prompting and Context (when not to use AI)
- Chapter 4 — AI in the Professional Workflow (honest documentation, data handling)
- Chapter 5 — Keeping Your Skills Real (leverage vs. crutch, deliberate practice)

## Requirements

- [ ] Write `MY-AI-POLICY.md` (roughly one page) with these sections, each with real, specific content — not restated chapter summaries:
  - [ ] **When I use AI** — be specific about task types (e.g., "boilerplate I already understand," "explaining an unfamiliar error," "practice problem generation") rather than "when it's helpful."
  - [ ] **When I don't use AI** — grounded in Chapter 3's two categories (fundamentals still being learned, security-critical code) applied to your actual current skill level, named concretely (e.g., "I don't use it yet for anything involving auth or password handling, because I can't fully verify that code myself").
  - [ ] **How I verify AI output** — your own restatement of Chapter 2's workflow, adapted to how you actually work (which steps you're most likely to skip if you're not careful, and how you'll catch yourself).
  - [ ] **Data I never share with AI tools** — specific to your own situation (secrets, any client/employer code you may have, personal data).
  - [ ] **How I document AI use** — your own standard for commit messages/READMEs, consistent with Chapter 4.
- [ ] Choose one completed or in-progress project from any earlier track in this workspace.
- [ ] Review that project's existing code against your new policy: identify at least one place AI was used (or would plausibly have been used) and evaluate honestly whether it was verified to the standard your policy now sets.
- [ ] Add an "AI usage" section to that project's own README, written in the honest, factual style from Chapter 4's examples — stating what was and wasn't AI-assisted, to the best of your actual memory or judgment.
- [ ] If reviewing the project surfaces something you'd now handle differently (an unverified assumption, a function you can't fully explain), fix it, and note the fix in the README's AI-usage section.

## Hints

- Specificity is the whole point of this project. "I use AI for boilerplate" is a placeholder; "I use AI for form-validation regexes and CRUD route scaffolding, always testing the boundary cases myself afterward" is an actual policy.
- If you're honestly unsure how much AI was involved in an older project (common — you may not have been tracking it carefully at the time), say that plainly in the README rather than guessing confidently either direction. "I don't have a clear record of how much AI assistance went into this project's early commits; going forward I'm tracking it per my policy" is a legitimate and honest thing to write.
- Keep the policy realistic for where you actually are right now, not aspirational for a senior engineer's workflow — it should change as you grow, and that's fine.
- Re-reading Chapter 5's leverage-vs-crutch distinction before writing the "when I don't use AI" section tends to produce a much sharper answer than writing it cold.

## Stretch Goals

- Set a calendar reminder to revisit and revise `MY-AI-POLICY.md` in 2–3 months, once you've used it for a while — note what changed and why.
- Apply the same README audit to a second project and compare what you found.
- Share your policy with a peer or mentor (if you have one) and ask them to push back on any part that seems vague or unrealistic.
