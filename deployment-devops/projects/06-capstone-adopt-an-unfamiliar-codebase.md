# Project 6: CAPSTONE — Adopt an Unfamiliar Codebase

## Description

Every project until now happened inside code you wrote. This one doesn't — and that's the entire point. When hiring managers are surveyed about junior developers, the #1 complaint is not a missing framework or a weak algorithm — it's that juniors freeze inside code they didn't write: can't get it running, can't find where things happen, can't make a change without hand-holding. Yet "here's our codebase, get productive" is the actual first week of every job you'll ever take. This capstone is that first week, simulated honestly: you'll pick a small open-source project you have *never touched*, get it running locally using only what its README gives you (documenting every gap you fall into), containerize it, repair or extend its CI, fix one real open issue, and finish by either submitting an upstream pull request or deploying your fork to a live URL — plus a write-up of *how you oriented yourself*, because the orientation method is the transferable skill. Everything the track taught converges here, applied to code that never had you in mind.

## Difficulty & Estimated Effort

Advanced — 15–25 hours across 2–3 weeks. The variance is the codebase you pick; the selection criteria below exist to keep you in the range.

## Chapters Used

All of them. Chapters 4–7 do the heaviest lifting (containerize, CI); Chapters 1–3 shape the "get it running" phase; Chapters 8–12 come into play on the deploy path.

## Requirements

**Phase 0 — Selection (do not skip the criteria; a bad pick sinks the project):**

- [ ] Find a candidate open-source project meeting ALL of: under ~5,000 lines of code (excluding dependencies/lockfiles — check with a line counter or GitHub's language stats plus judgment); written in a language you know (JS/TS or Python); has an existing test suite; has open issues; shows activity within the last year (commits or maintainer responses); and is NOT something you've contributed to or followed closely.
- [ ] Vet three candidates and pick one; record the runners-up (you may need a fallback). In your notes: what each does, size estimate, test/CI status, issue count, and why the winner won.
- [ ] Read the project's LICENSE and CONTRIBUTING file (if any) and note what they permit and expect — this determines your Phase 4 options.

**Phase 1 — Get it running from the README alone:**

- [ ] Fork and clone. Attempt to run the project locally following ONLY its README/docs — no external tutorials about the project, no issues-searching yet. Work in your normal environment (PowerShell/WSL2 as appropriate).
- [ ] Keep `ONBOARDING.md` in your fork from minute one: every command you ran, every error, every gap between what the README said and what reality required (missing prerequisite, undocumented env var, wrong version assumption, Windows-specific friction), and how you closed each gap. Timestamps encouraged — the gap log is a primary deliverable, not scratch paper.
- [ ] Get the test suite running and record the honest baseline: how many pass/fail/skip on your machine before you changed anything.
- [ ] Produce your orientation map (Chapter 6's "read a stranger's stack" and Chapter 7's workflow-reading drills, now for real): entry point(s), how config flows in, where the core logic lives, what talks to what, where tests live and how they're organized. A page of prose or a diagram — made by *systematic exploration*, and your write-up must say what your system was (reading order, search strategies, running things and watching).

**Phase 2 — Containerize it:**

- [ ] Write a `.dockerignore` and `Dockerfile` for the project (multi-service projects: a compose file too). All Chapter 5 standards apply: pinned base, caching-ordered layers, env-var config honored, no secrets baked in, reasonable size.
- [ ] Prove it: the containerized version runs and passes its test suite in (or against) the container. Update `ONBOARDING.md` with the containerized quick-start — the two-command version of the setup that took you hours.

**Phase 3 — CI:**

- [ ] If the project has no CI, or broken/rotten CI: add or repair a GitHub Actions workflow on your fork that installs, lints (if the project has a linter), and tests on push and PR. If its CI works, extend it meaningfully (e.g., add the Docker build as a checked step, add a missing lint gate) — copying their green is not the exercise.
- [ ] Get it green on your fork honestly, and protect your fork's main behind it. Log at least one red→diagnose→green cycle in `ONBOARDING.md`.

**Phase 4 — One real fix:**

- [ ] Pick one open issue (or one bug you found and can reproduce — a README gap you documented counts if the maintainers would take a docs PR). Smallest-real is the right size: a genuine bug, a missing small feature, a docs repair. Record why you chose it and how you reproduced it.
- [ ] Fix it on a branch, with a test that fails before and passes after (docs fixes exempt from the test, not from verification). Ship it through your own CI ceremony: branch → PR on your fork → green checks → merge.
- [ ] Then EITHER: **(a) submit the fix upstream** — following the project's contribution conventions, with a PR description a stranger-maintainer can act on (what, why, how verified); OR **(b) deploy your fork** to a live URL via your Project 5 playbook (PaaS, env-injected config, deploy-on-green from your fork's CI, health-checked and monitored where the project's nature allows). Do (a) if the project plausibly accepts contributions; (b) if it's deployable and (a) isn't realistic. Overachievers do both.

**Phase 5 — The write-up:**

- [ ] Finish `ONBOARDING.md` (or a separate `RETRO.md`) with: the orientation method you used and what you'd do differently; the three biggest gaps between README and reality; how long each phase actually took vs. your guess; what this codebase taught you about writing your *own* READMEs, Dockerfiles, and CI; and the state of your upstream PR or live deployment. Write it for the audience that matters: a hiring manager who asks "tell me about working in code you didn't write." This document is your answer, with receipts.

## Hints

- Finding candidates: GitHub search filters (`language:python stars:50..2000 pushed:>2025-07-01`), "good first issue" labels, awesome-lists of small tools, or CLIs/bots/small web apps you've seen mentioned. Small utilities and self-hostable mini-apps fit the size band; frameworks and anything with "monorepo" in it do not.
- Expect the README to be wrong somewhere. That's not a defect in your selection — it's the norm, it's half the learning, and it's why the gap log is a deliverable. The moment of "the docs say X but reality says Y" *is the simulation working*.
- Orientation strategy that works in any codebase: start from the entry point and trace one real request/invocation end to end; grep for a string you can see in the running app's output to find where things happen; read the tests — they're executable documentation of intended behavior; run `git log --oneline -30` to see what the project has been worrying about lately.
- Budget discipline: if Phase 1 exceeds ~5 hours with no forward motion, that's data — consider the fallback candidate. (Also data worth writing down: what made the first one impenetrable?)
- For upstream PRs: read how the maintainers responded to other people's PRs before submitting yours, match the project's conventions even where you'd choose differently, keep the diff minimal, and be genuinely gracious in review — maintainer time is a gift. A rejected-with-feedback PR still fully satisfies this project; document the exchange.
- Windows friction (path separators, line endings, a script assuming bash) is likely somewhere in Phase 1 — WSL2 is your escape hatch, and the friction itself belongs in the gap log; it's authentic onboarding pain.
- Resist the refactor itch. You will see things you'd write differently. First-week-at-a-job rule: smallest change that delivers the fix, in the codebase's own style. Save your opinions for the retro.

## Stretch Goals

- Get the upstream PR actually merged (partly outside your control — the attempt and the interaction are the value; a thoughtful maintainer exchange is résumé material either way).
- Submit a second upstream PR fixing the worst README gap you logged, turning your onboarding pain directly into the next person's smooth setup.
- Do both Phase 4 endings: upstream PR *and* live deployment of your fork with a custom domain (Chapter 12) on it.
- Add your full Project 5 operations rig to the deployed fork: structured logs, health checks, uptime monitor, error tracking, rollback drill — an operated adoption, not just a hosted one.
- Repeat the whole capstone on a second codebase and compare your `ONBOARDING.md` timelines: the delta between attempt one and attempt two is the skill, measured. Show that number to interviewers.
