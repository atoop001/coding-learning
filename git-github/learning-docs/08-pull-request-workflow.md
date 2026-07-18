# Chapter 8: The Pull Request Workflow

## Overview

A pull request (PR) is GitHub's mechanism for proposing changes: "here's a branch with my work — please review it and, if it's good, merge it." Around that simple idea, the modern software industry has built its entire collaboration model: feature branches, code review, automated checks, and a permanent, searchable record of every change and the discussion that shaped it.

This chapter covers the full loop — feature branch → push → open PR → review → merge → clean up — plus forks (contributing to repos you can't write to), and *why* teams work this way even when, technically, everyone could just push to `main`. You'll practice the whole thing solo against your own repo, which is exactly how professionals recommend learning it: same mechanics, no stakes.

## Definitions & Explanations

**Pull request** — a GitHub construct (not a Git command) that says: "please pull branch `feature/x` into branch `main`." It bundles: the diff, a description, a conversation thread, line-by-line review comments, status checks, and finally a merge button. ("Merge request" on GitLab — same thing.)

**The feature-branch workflow** — the default professional loop:

```
        create branch      push       open PR      review,      merge on     delete branch,
main ──▶ feature/login ──▶ origin ──▶ on GitHub ─▶ maybe fix ─▶ GitHub ───▶ pull main locally
```

`main` stays protected and always releasable; all change flows through reviewed PRs. Many teams enforce this with **branch protection rules** (no direct pushes to `main`, N approvals required, checks must pass).

**Why teams bother** (the part that seems like ceremony until you've worked on a team):
- **Review catches bugs cheaply** — a second reader before merge is the cheapest QA that exists.
- **Knowledge spreads** — reviewers learn the codebase; authors learn the standards; nobody becomes the only person who understands module X.
- **History gets a why** — every change links to a PR with description and discussion. Future debuggers (often future-you) inherit context, not just diffs.
- **Automation gates** — CI (tests, linters) runs on every PR automatically; broken code is stopped at the door instead of discovered on `main`.

**Fork** — your own server-side copy of someone else's repo. When you lack push access (any open-source project), you fork it, push your branch *to your fork*, and open a PR from your fork's branch to the original ("upstream") repo. For repos you own or your team shares, you branch within the repo and skip forking.

```
 Shared-repo model (teams):              Fork model (open source):

   github.com/team/app                    github.com/them/project   ◀── PR ──┐
      ▲ push branch, PR inside            github.com/you/project (fork) ─────┘
   your clone                                ▲ push branch
                                          your clone
```

**Merge strategies on the GitHub button** — three flavors, chosen per-PR:
- **Merge commit** — a `--no-ff` merge; preserves the branch shape in history.
- **Squash and merge** — collapses the PR's commits into one tidy commit on `main`. Very popular: PRs can have messy WIP commits, `main` stays clean.
- **Rebase and merge** — replays the PR's commits onto `main` individually, no merge commit (Chapter 9 explains rebase).

**Draft PR** — a PR flagged "not ready yet"; used to share work-in-progress for early feedback. Turning it "Ready for review" is the signal.

## Command Examples

### The full loop, solo, on your own repo

```bash
# 0. Start synced
git switch main && git pull

# 1. Branch
git switch -c feature/add-glossary

# 2. Work in normal small commits
echo "# Glossary" > glossary.md
git add glossary.md && git commit -m "Add glossary skeleton"
echo "- repo: a tracked directory" >> glossary.md
git commit -am "Add first glossary entry"

# 3. Publish the branch
git push -u origin feature/add-glossary
# ...
# remote: Create a pull request for 'feature/add-glossary' on GitHub by visiting:
# remote:   https://github.com/yourname/practice-sync/pull/new/feature/add-glossary
#  * [new branch]      feature/add-glossary -> feature/add-glossary
```

**4. Open the PR** — visit that URL (or the "Compare & pull request" banner on the repo page). Confirm base: `main` ← compare: `feature/add-glossary`. Write a real title and description (what & why, how to verify). Click **Create pull request**.

**5. Review it** — even solo, do the ritual: open the **Files changed** tab, read every line of the diff as if a stranger wrote it, leave at least one line comment on your own code (hover a line → blue `+`). You will be surprised how often reviewing your own diff finds something.

```bash
# 6. Respond to review — just push more commits to the same branch:
echo "- commit: a recorded snapshot" >> glossary.md
git commit -am "Address review: add commit definition"
git push
# The PR updates automatically. This is the key mechanic: PR = branch, live.
```

**7. Merge** — back on the PR: **Merge pull request** (try "Squash and merge" at least once to see the difference) → confirm → **Delete branch** (the button afterward).

```bash
# 8. Sync up and clean locally
git switch main
git pull                              # brings in the merge/squash commit
git branch -d feature/add-glossary    # delete the local label
git fetch --prune                     # drop stale origin/feature/* bookmarks
git log --oneline --graph -5          # admire the result
```

### The fork variant (open-source style)

```bash
# On GitHub: click "Fork" on someone else's repo → creates github.com/you/project
git clone https://github.com/you/project.git
cd project
git remote add upstream https://github.com/them/project.git   # convention
git remote -v
# origin    https://github.com/you/project.git (your fork — push here)
# upstream  https://github.com/them/project.git (theirs — pull from here)

git switch -c fix/typo-in-readme
# ...commit...
git push -u origin fix/typo-in-readme
# Open PR on GitHub: base = them/project main  ◀—  compare = you/project fix/typo-in-readme

# Later, keep your fork's main current:
git switch main
git pull upstream main && git push origin main
```

### Optional: the `gh` CLI

```bash
gh auth login                 # one-time
gh pr create --fill           # open a PR from the current branch, from your terminal
gh pr view --web              # jump to it in the browser
gh pr merge --squash --delete-branch
```

Not required — the web UI is fine — but many professionals live in `gh`.

## Common Pitfalls

**PR shows way more commits/changes than you made.** Usually: (a) you branched from an outdated `main` — the "extra" commits are old ones your local main lacked; or (b) base branch is set wrong in the PR. Prevention: `git switch main && git pull` *before* creating branches. Fix: edit the PR's base, or rebase/merge main into your branch (Chapter 9).

**Committed to `main` instead of a branch — again.** In PR-world this matters more, since protected `main` will reject your push. Same rescue as Chapter 5: create the branch at your commit, reset `main` back, push the branch, open the PR.

**Pushed more commits but "the PR didn't update."** It always does — same branch, same PR. If it looks stale you likely pushed a *different* branch (check `git branch -vv` and the push output) or you're looking at a different PR.

**Fear of pushing "imperfect" commits to a PR.** WIP commits in a PR are normal and expected; that's what squash-merge exists for. What matters is the final diff and description, not commit-by-commit elegance mid-review.

**Merged on GitHub, confused locally.** After a squash-merge, your local feature branch's commits aren't literally on `main` (they were replaced by one new commit), so `git branch -d` may protest "not fully merged." If the PR is merged, force-delete the local label with `-D` — the *changes* are safely in main; only the duplicate commit objects are being discarded.

**Fork drift.** Weeks later, your fork's `main` is far behind upstream and new PRs contain ancient noise. Routine: sync fork main from upstream (commands above, or GitHub's "Sync fork" button) before every new branch.

**Review comments taken personally.** Half of professional readiness is treating review as collaboration on an artifact, not judgment of a person — in both directions. Practice the tone now, while your only reviewer is you.

## Practice Exercises

1. **Solo PR, full ceremony.** Run the complete loop from this chapter on your own repo: synced main → branch → 2–3 commits → push → PR with a genuine description → self-review with at least two line comments → push one "address review" commit → squash-merge → delete branch → pull main → prune. Repeat until the loop takes under ten minutes.

2. **Merge-strategy taste test.** Make three tiny PRs and merge one each way: merge commit, squash, rebase. Afterward, study `git log --oneline --graph` on main and write 2–3 sentences on how the histories differ and which you'd want on a team.

3. **The stale-base incident.** Deliberately branch from an out-of-date local `main` (don't pull first), make a commit, push, open a PR, and observe the extra noise in the diff. Then fix the situation and get the PR clean. Document what you did.

4. **Fork a real project.** Fork any small public repo (even a docs/awesome-list repo), clone your fork, add the `upstream` remote, make a genuinely tiny improvement on a branch, push to your fork, and open a PR *at least as far as the creation screen*. (Submitting for real is a stretch goal — see Project 4.)

5. **Write a review checklist.** From Chapters 2–8, draft a 6–10 item personal checklist for reviewing any PR (e.g., "diff contains only intended files," "no leftover conflict markers," "message explains why"). Commit it to your workspace repo — via a PR, naturally. You'll use it in Project 4.
