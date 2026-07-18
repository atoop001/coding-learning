# Git & GitHub Learning Track

A self-paced track that takes you from "I copy git commands and hope" to professional workflow readiness: clean commits, confident branching, calm conflict resolution, the full pull-request ceremony, history rewriting, and disaster recovery. Designed for a Windows machine with VS Code, and for someone who already uses a GitHub repo to sync a workspace across devices — that habit becomes a first-class skill here.

**This track runs alongside any language track.** Git is language-agnostic; every exercise uses plain text/markdown files so nothing here depends on (or interferes with) whatever programming you're learning. In fact, pairing them is ideal: use the Git habits from each chapter immediately in your language projects.

## Structure

- `learning-docs/` — 10 chapters, the primary study material. Each has an overview, plain-English definitions with commit-graph diagrams, real command sequences with expected output, common pitfalls **with recovery steps**, and practice exercises.
- `projects/` — 5 guided drills done in throwaway practice repos, each bundling several chapters. Specs only — no answer keys. The struggle is the curriculum.

## Chapters (in order)

| # | File | Covers |
|---|------|--------|
| 1 | `learning-docs/01-version-control-and-setup.md` | What version control is; installing & configuring Git on Windows; Git in VS Code |
| 2 | `learning-docs/02-repositories-and-the-three-areas.md` | Working dir / staging / history; add, commit, status, log; what a commit really is |
| 3 | `learning-docs/03-viewing-history-and-diffs.md` | log options, diff, show, blame; detached HEAD demystified |
| 4 | `learning-docs/04-undoing-things.md` | restore, reset (soft/mixed/hard), revert, amend — and when each is safe |
| 5 | `learning-docs/05-branching.md` | Creating/switching branches; merging; fast-forward vs merge commits |
| 6 | `learning-docs/06-merge-conflicts.md` | Why conflicts happen; conflict markers; calm resolution; the abort escape hatch |
| 7 | `learning-docs/07-remotes-and-github.md` | clone/push/pull/fetch; origin; authentication on Windows; multi-machine syncing |
| 8 | `learning-docs/08-pull-request-workflow.md` | Feature branches, forks, opening/reviewing/merging PRs; why teams work this way |
| 9 | `learning-docs/09-rebase-and-history-management.md` | Rebase vs merge; interactive rebase; force-push dangers; .gitignore & untracking |
| 10 | `learning-docs/10-collaboration-and-recovery.md` | stash, cherry-pick, reflog safety net, tags/releases; GitHub issues & Actions overview |

## Projects (easiest → hardest)

| # | File | Drill | After chapters |
|---|------|-------|----------------|
| 1 | `projects/01-commit-time-machine.md` | Build a deliberate history; navigate and undo through it every way | 1–4 |
| 2 | `projects/02-branching-workout.md` | Parallel features, every merge shape, a five-drill conflict gauntlet | 5–6 |
| 3 | `projects/03-two-device-sync-drill.md` | Simulate two devices with two clones; break and repair sync every way | 7 |
| 4 | `projects/04-full-pr-workflow.md` | Repeated full PR ceremony with branch protection, review checklist, fork model | 8 (uses 9–10) |
| 5 | `projects/05-rescue-and-rewrite-capstone.md` | Manufacture a disaster; rescue via reflog; rewrite history; tag & release | everything |

## Suggested cadence

At **3–5 hours/week**, the track takes roughly **7–9 weeks**:

- **Weeks 1–2:** Chapters 1–4, then Project 1. (Foundations + undo. Don't rush Chapter 2 — it's the load-bearing mental model.)
- **Weeks 3–4:** Chapters 5–6, then Project 2. (Branches + conflict inoculation.)
- **Week 5:** Chapter 7, then Project 3. (This week directly upgrades your real device-sync routine — adopt the playbook you write.)
- **Weeks 6–7:** Chapter 8, then Project 4 spread over several short sessions. (Team readiness lives here.)
- **Weeks 8–9:** Chapters 9–10, then the Project 5 capstone.

Rules of thumb:
- Do every chapter's exercises before its project — projects assume them.
- All drills happen in **throwaway repos**, never in your real workspace repo (exceptions are explicitly marked in the projects).
- When something scary happens mid-exercise, that *is* the exercise: read `git status`, breathe, consult the chapter's pitfalls section.
- From Week 3 on, practice trickles into real life automatically: branch for each experiment in your language track, and sync your workspace with the Chapter 7 rhythm.

## Done when

You can: explain the three areas from memory; predict whether a merge fast-forwards; resolve a conflict without adrenaline; recover a hard-reset commit in under a minute; run the full PR loop in under ten; and articulate when rebase is safe. That's professional workflow readiness — the capstone's `RETRO.md` is your certificate.
