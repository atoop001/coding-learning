# Chapter 9: Rebase & History Management

## Overview

Merge and rebase are two answers to the same question — "how do I combine two lines of history?" — with opposite philosophies. Merge *records* what happened, diamonds and all. Rebase *rewrites* your commits as if you'd started from the latest code, producing a straight line. Professionals use both, and flame-war about the mix; what you need is to understand exactly what rebase does, when it's safe, and the one rule that keeps it safe.

This chapter also covers the supporting cast of history management: interactive rebase (cleaning up your own commits before sharing), force-push and its dangers (plus the safer `--force-with-lease`), and repository hygiene — `.gitignore` and untracking files you never meant to commit.

## Definitions & Explanations

### What rebase actually does

`git rebase main` (run while on `feature`) means: *take my branch's commits and replay them, one by one, on top of main's current tip.*

```
Before:                  c4 ◀── c5     ◀─ feature (HEAD)
                        /
          c1 ◀── c2 ◀── c3 ◀── c6      ◀─ main

git rebase main   (on feature):

          c1 ◀── c2 ◀── c3 ◀── c6      ◀─ main
                                 \
                                  c4' ◀── c5'   ◀─ feature
```

Crucial details:
- `c4'` and `c5'` are **new commits** — same changes, new parents, therefore **new hashes**. The originals still exist (reflog) but the branch label moved to the copies.
- `main` is untouched. Rebase rewrites the branch you're *on*.
- Conflicts can occur during replay, one commit at a time. You resolve exactly as in Chapter 6, then `git rebase --continue` (or `--abort` to rewind the whole thing).
- After rebasing, merging `feature` into `main` fast-forwards — straight-line history, no diamond.

**Merge vs. rebase, honestly:**

| | Merge | Rebase |
|---|---|---|
| History | True record, includes diamonds | Linear, tidy, "as if written today" |
| Hashes | Preserved | Rewritten |
| Safe on shared branches | Yes | **No** (the golden rule) |
| Typical pro use | Integrating finished PRs into main | Updating *your own* feature branch onto latest main; tidying before review |

### The golden rule, now with teeth

> **Never rebase commits that exist outside your own unshared branch.**

If others have the old commits and you rewrite them, their history and yours disagree about "the same" work — producing duplicate commits, baffling conflicts, and misery. Corollary: rebasing your own not-yet-shared (or explicitly single-author) feature branch is fine and normal.

### Force-push

After you rebase a branch you'd already pushed, `git push` is rejected — the remote has the old commits, and your rewrite isn't a fast-forward. Overwriting the remote requires force:

- `git push --force` — "remote, become exactly my version, discard whatever you have." If a teammate pushed to that branch since you last fetched, **their commits are destroyed**.
- `git push --force-with-lease` — same, but *only if the remote still looks like the last state you fetched*. If anyone pushed in between, it refuses. **Always prefer this.** It converts a silent data loss into a polite error.

Force-pushing your own feature branch (e.g., after tidying it for a PR): routine. Force-pushing `main` or any shared branch: an incident report waiting to happen — protected branches usually forbid it outright.

### Interactive rebase (concepts)

`git rebase -i HEAD~4` opens a script listing your last 4 commits, oldest first, each with a verb you can edit:

```
pick a1a1a1 Add login form
pick b2b2b2 wip
pick c3c3c3 fix typo
pick d4d4d4 Actually finish login form

# Verbs: pick   = keep as-is          reword = keep, edit message
#        squash = meld into previous  fixup  = meld, discard message
#        edit   = pause here to amend  drop  = delete commit
#        (you can also reorder lines to reorder commits)
```

Change `wip`, `fix typo`, and the finisher to `fixup`/`squash` and you emerge with one clean "Add login form" commit. This is how messy-but-honest local history becomes a reviewable PR. Same rewrite rules apply: only do this to unshared commits (or expect a `--force-with-lease` afterward on your own branch).

**Note for Windows:** interactive rebase opens your configured editor; with `core.editor "code --wait"` (Chapter 1), the todo list appears as a VS Code tab — edit, save, close the tab to proceed.

### .gitignore & untracking

**`.gitignore`** — a committed file of patterns Git should never track: build output, dependencies, editor droppings, secrets.

```gitignore
node_modules/        # a directory, anywhere in the repo
*.log                # by extension
build/               # generated output
.env                 # secrets — env files stay local
Thumbs.db            # Windows Explorer litter
!important.log       # '!' re-includes an exception
```

**The catch everyone hits:** `.gitignore` only affects **untracked** files. A file already committed keeps being tracked no matter what you add to `.gitignore`. To stop tracking it while keeping it on disk:

```bash
git rm --cached secrets.env     # remove from the INDEX only; file stays on disk
# then ensure .gitignore covers it, and commit both facts
```

(And if the file contained real secrets, removing it from the tip does **not** remove it from history — every old commit still contains it. Rotate the secret; history-scrubbing tools exist but rotation is the real fix.)

## Command Examples

### Updating a feature branch onto latest main

```bash
git switch feature/report
git fetch
git rebase origin/main
# Successfully rebased and updated refs/heads/feature/report.
#   -- or, per conflicting commit: --
# CONFLICT (content): Merge conflict in report.py
# ...resolve the file (Ch.6), then:
git add report.py
git rebase --continue
# ...repeat if further commits conflict. Escape hatch at any point:
git rebase --abort              # branch returns to exactly its pre-rebase state
```

### Tidying before a PR, then force-pushing your own branch

```bash
git log --oneline -4
# d4d4d4 (HEAD -> feature/report) Actually finish login form
# c3c3c3 fix typo
# b2b2b2 wip
# a1a1a1 Add login form

git rebase -i HEAD~4            # mark the middle two as 'fixup', last as 'squash'
git log --oneline -1
# 9e9e9e (HEAD -> feature/report) Add login form     <-- one clean commit

git push --force-with-lease
# To github.com:you/repo.git
#  + d4d4d4...9e9e9e feature/report -> feature/report (forced update)
```

### Ignore hygiene on an existing repo

```bash
# Realize node_modules/ was committed long ago:
git rm -r --cached node_modules
echo "node_modules/" >> .gitignore
git add .gitignore
git commit -m "Stop tracking node_modules; ignore it"

git check-ignore -v some/file.log     # debug WHY a path is (or isn't) ignored
git status --ignored                  # see what's being ignored right now
```

### Choosing pull's behavior (from Chapter 7's cliffhanger)

```bash
git config --global pull.rebase false   # pull = fetch + merge (safe default)
git config --global pull.rebase true    # pull = fetch + rebase (linear; fine for
                                        # personal sync repos where all commits are yours)
```

Now you can make this choice deliberately: for your workspace-sync repo, `pull.rebase true` keeps history a clean line; on team repos, follow the team's convention.

## Common Pitfalls

**Rebased a shared branch; teammate (or your other machine) now sees chaos.** Symptoms: duplicate-looking commits, "diverged by N and M," recurring conflicts. Recovery: coordinate — one side hard-resets to the other's version (`git fetch` + `git reset --hard origin/branch` on the machine whose copy you're discarding, after saving any local-only work to a temp branch). Prevention is the golden rule.

**Force-pushed over someone's work.** With plain `--force` there's no warning. The overwritten commits are recoverable from the victim's local repo (their reflog still has them) — but only if they still have their clone. Make `--force-with-lease` muscle memory and this pitfall mostly disappears.

**Stuck mid-rebase, repo feels haunted.** Prompt shows `REBASE 2/5`, files conflict, panic sets in. Same defusal as merges: `git status` explains the state; `git rebase --abort` is a full, penalty-free rewind. Nothing about a paused rebase is urgent.

**Same conflict repeating on every replayed commit.** A long branch rebased across big upstream changes can conflict repeatedly (each commit replays separately). Options: resolve patiently, or `--abort` and use a single merge instead (one conflict resolution total). Merge being *easier* here is a legitimate reason pros sometimes prefer it.

**"I added it to .gitignore but Git still tracks it!"** By design — see above. `git rm --cached`, commit, done.

**Rebasing when you meant to be done.** Some learners get tidiness-obsessed and rebase constantly, losing time and occasionally work. History polish matters at the moment of sharing (PR time); before that, honest messy commits are fine, and after merging, it's out of your hands. Tidy once, at the boundary.

## Practice Exercises

1. **Merge vs. rebase, side by side.** Build a divergence (feature +2 commits, main +1) twice, in two throwaway repos. Resolve one with merge, one with rebase, then compare `git log --oneline --graph --all` outputs. Note the hash difference between the original and replayed commits in the rebase repo.

2. **Conflicted rebase, both exits.** Engineer a rebase that conflicts (both branches edit the same line). First time: `--abort` and verify the branch is untouched. Second time: resolve, `--continue`, and finish. Confirm the final file content is what you intended.

3. **The cleanup special.** Make five deliberately scruffy commits ("wip", "typo", "oops", ...) on a branch. Use interactive rebase to end with two well-named commits, exercising at least `squash`, `fixup`, and `reword`. Verify with `git log --stat` that no *content* was lost in the polish.

4. **Force-with-lease in action.** Push a branch to GitHub, tidy it with interactive rebase locally, and push again — watch the rejection, then succeed with `--force-with-lease`. Bonus round: simulate the race by pushing a commit to that branch from a second clone first, and observe `--force-with-lease` refuse.

5. **Ignore-file surgery.** In a practice repo, commit some junk (a `logs/` dir with `.log` files, a fake `.env`). Now write a proper `.gitignore`, untrack the junk with `git rm --cached`, and commit. Prove with a fresh clone (clone it locally to a second folder) that the junk no longer arrives — and then find it in an *old* commit with `git show`, and write one sentence about what that implies for real secrets.
