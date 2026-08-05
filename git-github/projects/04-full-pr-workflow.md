# Project 4: Full Pull Request Workflow

## Description

Run the complete professional collaboration loop — repeatedly, until it's routine. Against your own throwaway GitHub repo you'll ship a series of small "features," each through the full ceremony: synced main → feature branch → commits → push → pull request with a real description → disciplined self-review using a checklist → revision commits → merge (trying all three merge strategies) → cleanup. You'll also enable branch protection so the repo *forces* the workflow on you, wire a PR to an issue, and take one trip through the fork model.

This is the project that most directly converts "knows Git" into "ready to work on a team."

## Difficulty & Estimated Effort

**Medium-Hard (4/5).** Estimated effort: 4–6 hours across several sessions — the value compounds with repetition, so spreading it out is better than cramming.

## Chapters used

- Chapter 8 — the PR workflow (core)
- Chapter 7 — push/pull mechanics, upstreams
- Chapter 5 — feature branches
- Chapter 9 — tidying a branch before review; force-with-lease
- Chapter 10 — issues; reading Actions checks (light)

## Requirements checklist

**Part A — Stage the theater**
- [ ] Create a throwaway GitHub repo (e.g., `pr-dojo`) with a README; clone it
- [ ] Plan 4+ small "features" as content changes (e.g., sections of a study-guide site in markdown) — each shippable in 2–4 commits
- [ ] Commit your review checklist from Chapter 8's exercises as `REVIEW-CHECKLIST.md` — via the project's very first PR
- [ ] After that first PR merges, enable **branch protection** on `main`: require a pull request before merging. Verify by attempting a direct push to `main` and getting rejected
- [ ] **Optional:** add a `CODEOWNERS` file (paths → GitHub usernames who must approve changes to them) and set branch protection to require review from a code owner; verify a PR touching an owned path shows the requirement

**Part B — Reps (at least 3 more PRs, one per feature)**
For every PR:
- [ ] Branch from a freshly pulled `main`, with a descriptive branch name
- [ ] Small, well-messaged commits; push with `-u`
- [ ] PR description with: what changed, why, and how a reviewer could verify it
- [ ] Self-review on the **Files changed** tab against your checklist, leaving at least two line comments; at least one must result in an actual fix
- [ ] Push revision commits and confirm the PR updates
- [ ] Merge and delete the branch (remote and local), then sync local main
Across the set of PRs:
- [ ] Use each merge strategy at least once (merge commit, squash, rebase) and record in the repo how each shaped `main`'s history
- [ ] For one PR, dirty up the branch with deliberate "wip"/"typo" commits, then tidy with interactive rebase and `--force-with-lease` *before* marking it ready — the pro polish move
- [ ] For one PR, open it as a **Draft** first, then mark ready
- [ ] For one PR, file a GitHub **Issue** first describing the need; reference it with `Fixes #N` so the merge auto-closes it

**Part C — The stale-base recovery**
- [ ] Engineer the classic mess: branch from an outdated local main, commit, push, open the PR, and observe the polluted diff
- [ ] Repair the PR to a clean diff (your choice of technique — Chapter 9 has two) and document what you did in the PR conversation

**Part D — The fork model**
- [ ] Fork a real public repo, clone the fork, add the `upstream` remote, and verify both remotes
- [ ] Make a genuinely tiny, genuinely correct improvement on a branch; push to your fork; proceed to the PR creation screen against upstream (actually submitting is a stretch goal)
- [ ] Sync your fork's main from upstream afterward

## Hints

- Solo self-review feels silly for about two PRs, then starts catching real mistakes. The checklist is what makes it work — review against *it*, not against vibes.
- PR descriptions have a shape: what / why / how-to-verify. Write for a reviewer who has 90 seconds.
- "PR = branch, live" is the master key: pushing to the branch *is* updating the PR. Nothing special to do.
- Branch protection is under repo **Settings → Branches** (or Rules). The rejection you get when pushing to protected `main` is the same one professionals see daily — meet it now, calmly.
- For Part C: the pollution is commits that were already on GitHub's main but not in the main you branched from. Both standard repairs end with your branch containing only your commits *on top of* current main.
- For Part D: docs typo fixes and dead-link repairs are the classic first contribution — small, verifiable, welcome. Read the project's CONTRIBUTING.md if it has one.

## Stretch goals

- Actually submit the open-source PR from Part D, respond to any maintainer feedback, and (fingers crossed) get your first merged outside contribution.
- Add a starter GitHub Actions workflow (e.g., a markdown link checker) to `pr-dojo` via its own PR, then make its green check *required* in branch protection — now your repo has real CI gating.
- Do one full PR cycle entirely from the `gh` CLI: `gh pr create`, `gh pr view`, `gh pr merge`. Compare the feel.
- Invite a friend (or your other GitHub account) to review one PR for real, and practice responding to feedback you didn't write yourself.
- Write `WORKFLOW.md` in the repo: your end-to-end team-ready routine, from `git switch main && git pull` to post-merge cleanup, as a reusable reference.
