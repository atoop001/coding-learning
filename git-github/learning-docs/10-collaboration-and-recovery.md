# Chapter 10: Collaboration & Recovery

## Overview

This closing chapter collects the tools that turn a competent Git user into a *calm* one: `stash` (pocket your work-in-progress), `cherry-pick` (copy one commit anywhere), `reflog` (the safety net under every "destructive" command), and tags & releases (naming the versions that matter). It finishes with a working tour of the GitHub features that surround code on real teams — issues and a first look at GitHub Actions.

The through-line: **in Git, almost nothing committed is ever truly lost.** Once you've personally resurrected "destroyed" commits a few times, the fear that makes beginners freeze simply stops firing — and that composure is professional readiness.

## Definitions & Explanations

### Stash — the pocket

`git stash` takes your uncommitted changes (staged and unstaged), stores them on a stack, and leaves you a clean working directory. Classic uses: an urgent bug interrupts feature work; you started editing on the wrong branch; you need to pull but your directory is dirty.

- `stash push -m "message"` — save with a label (always label; future-you forgets).
- `stash list` — the stack: `stash@{0}` is newest.
- `stash pop` — reapply newest and drop it from the stack.
- `stash apply` — reapply but *keep* it stashed (safer when unsure).
- Stashes apply onto any branch — which makes stash the standard fix for "started work on the wrong branch."
- Untracked files need `stash -u` to be included.

Stash is for *hours*, not weeks. A stash that lingers becomes a mystery. For anything longer-lived, a real branch with a WIP commit is strictly better.

### Cherry-pick — copy one commit

`git cherry-pick <hash>` applies the *changes* of any single commit onto your current branch as a new commit (new hash — it's a copy, not a move).

```
        c4 ◀── c5 ◀── c6      ◀─ feature (c5 is a critical fix)
       /
 c1 ◀── c2 ◀── c3             ◀─ main

git switch main && git cherry-pick <c5>:

 c1 ◀── c2 ◀── c3 ◀── c5'     ◀─ main    (feature untouched)
```

Real uses: hotfix needed on main before the feature merges; backporting a fix to a release branch; salvaging one good commit from an abandoned branch. Conflicts resolve Chapter-6 style (`--continue` / `--abort`).

### Reflog — the safety net

Every time HEAD moves — commit, switch, merge, reset, rebase, anything — Git appends a line to the **reflog**, a private, local, per-machine journal kept for ~90 days. Commits "destroyed" by reset, orphaned by detached HEAD, or replaced by rebase are all still in there, findable by hash.

```
git log:     shows commits reachable from branches — the official story
git reflog:  shows everywhere HEAD has BEEN — including the deleted scenes
```

The universal recovery recipe:
1. `git reflog` — find the hash from just before disaster.
2. Inspect it: `git show <hash>` (make sure it's the right moment).
3. Rescue: `git branch rescue <hash>` (safe — makes a label) or `git reset --hard <hash>` (moves your branch back).

Limits: reflog is local (a fresh clone has none of your old reflog) and cannot recover work that was *never committed or stashed*. Hence the deepest habit in this track: **commit early, commit often — commits are the things Git can always save.**

### Tags & releases

A **tag** is a permanent label on one commit — unlike a branch, it never moves. Used overwhelmingly for versions.

- **Lightweight tag** (`git tag v1.0`) — just a pointer.
- **Annotated tag** (`git tag -a v1.0 -m "..."`) — a full object with tagger, date, message. **Prefer annotated** for anything meaningful.
- Tags aren't pushed by default: `git push origin v1.0` (or `--tags`).
- Convention: semantic versioning — `v<major>.<minor>.<patch>`, e.g. `v2.1.3`.

A **GitHub Release** wraps a tag with a title, release notes, and optional downloadable assets — the public face of a version.

### GitHub around the code

**Issues** — the repo's threaded to-do list: bugs, ideas, questions. Cross-linking is the magic: writing `Fixes #12` in a commit message or PR description auto-closes issue #12 on merge, permanently linking problem → discussion → fix. Labels, milestones, and assignees organize the pile. On teams, issues are where work is born; PRs are where it's finished.

**GitHub Actions (orientation only)** — GitHub's automation platform. Workflow files in `.github/workflows/*.yml` say "when X happens (push, PR, schedule), run these steps on a fresh VM" — typically tests and linters on every PR (that's "CI"), deployments on merge. For now you only need to *recognize* it: the green ✓ / red ✗ beside commits and PRs is Actions reporting whether checks passed. Reading a failed check's log is a professional daily routine.

**Local hooks & commit conventions (orientation only)** — two more pieces of the tooling landscape worth *recognizing*, not yet mastering:

- **Git hooks** are scripts Git runs automatically at points like `pre-commit` or `pre-push`. Most teams don't hand-write them — they install a hook manager (**Husky** is the common one in JS projects) paired with tools like **lint-staged** (run linters/formatters only on staged files) or **gitleaks** (scan the diff for accidentally-committed secrets before they ever reach a commit). If a `commit` you expected to succeed instead prints tool output and refuses, a hook just did its job — read the message.
- **Conventional Commits** is a message format some teams require: `type(scope): subject`, e.g. `fix(auth): handle expired token on refresh` or `feat(api): add pagination to /users`. Common types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`. The payoff is automation — changelogs and version bumps can be generated straight from commit history. If a repo's `CONTRIBUTING.md` or a rejected commit mentions this format, that's what it means.

## Command Examples

### Stash in anger

```bash
# Mid-feature, an urgent fix is needed elsewhere:
git stash push -m "half-done glossary rework"
# Saved working directory and index state On feature/glossary: half-done...
git status                      # clean — free to switch
git switch main                 # ...fix the urgent thing, commit, push...

git switch feature/glossary
git stash list
# stash@{0}: On feature/glossary: half-done glossary rework
git stash pop
# ...changes are back...
# Dropped refs/stash@{0}

# Wrong-branch rescue:
git stash push -m "meant this for a branch"
git switch -c feature/right-place
git stash pop                   # work lands where it belonged
```

### Cherry-pick a hotfix

```bash
git log --oneline feature/big-thing
# 7g8h9i0 More WIP
# 4d5e6f7 Fix crash on empty input      <-- needed on main NOW
# 1a2b3c4 Start big thing

git switch main
git cherry-pick 4d5e6f7
# [main 8h9i0j1] Fix crash on empty input
git log --oneline -1            # note the NEW hash — it's a copy
```

### Reflog rescues (the confidence builders)

```bash
# Disaster 1: over-eager reset
git reset --hard HEAD~3         # "...wait, no."
git reflog
# 0aa1bb2 HEAD@{0}: reset: moving to HEAD~3
# 9cc8dd7 HEAD@{1}: commit: The commit I still want   <-- pre-disaster state
git reset --hard 9cc8dd7        # everything back

# Disaster 2: committed in detached HEAD, then switched away
git switch main
# Warning: you are leaving 1 commit behind, not connected to any of your branches
git reflog                      # find that orphaned commit's hash
git branch rescued 5ee4ff3      # give it a home; merge or cherry-pick at leisure

# Disaster 3: force-deleted an unmerged branch
git branch -D experiment        # "...it had the good version, didn't it."
git reflog                      # find experiment's tip (look for its last commit line)
git branch experiment 3ab2cd1   # branch restored
```

### Tags and a release

```bash
git tag -a v1.0 -m "First complete version of the learning workspace"
git tag                         # v1.0
git show v1.0                   # tag message + the tagged commit

git push origin v1.0
# To github.com:you/repo.git
#  * [new tag]         v1.0 -> v1.0

git tag -a v0.9 9cc8dd7 -m "Retroactive tag on an older commit"   # tags can point anywhere
```

On GitHub: **Releases → Draft a new release → choose tag v1.0 →** write notes → **Publish**. Your repo now has a Releases page with a permanent snapshot download.

### Issues wired to commits

```bash
# After opening issue #3 ("Typo in chapter 2 notes") on GitHub:
git commit -am "Fix chapter 2 typo. Fixes #3"
git push
# On GitHub: issue #3 closes automatically and links to this commit.
```

## Common Pitfalls

**Stash conflicts.** `stash pop` can conflict like any merge if the branch changed underneath. Resolve normally; note that on conflict, pop does *not* drop the stash — after resolving, `git stash drop` it yourself. When nervous, use `apply` (keeps the stash) instead of `pop`.

**Stash as a junk drawer.** `stash list` showing six unlabeled entries from last month is unusable. Label everything, pop promptly, and prefer WIP commits on branches for anything that might outlive the afternoon.

**Cherry-pick double-trouble.** A cherry-picked commit is a *copy*; when the original branch later merges, both versions arrive. Git usually reconciles identical changes silently, but it can cause odd conflicts. Know it happens; when possible, prefer merging a shared fix branch into both places over heavy cherry-picking.

**Waiting too long to rescue.** Reflog entries expire (~90 days) and `git gc` eventually collects unreachable commits. Practically: recover promptly, and don't rely on reflog as long-term storage — branches and tags are the durable labels.

**Expecting reflog on another machine.** Reflog is per-clone. The rescue must happen on the machine where the commits existed. (One more argument for pushing often: pushed commits live on GitHub regardless of local mishaps.)

**Tag didn't show up on GitHub.** Tags require explicit pushing (`git push origin <tag>`). Also: moving/re-pointing a tag after pushing it creates confusion for anyone who fetched the old one — treat pushed tags as immutable; make `v1.0.1` instead.

**Red ✗ paralysis.** A failed Actions check on your PR isn't a verdict on you — it's a log file. Click Details, read the last ~30 lines, fix, push again; checks re-run automatically. Not reading the log is the only real mistake.

## Practice Exercises

1. **Interruption drill.** On a feature branch, get genuinely mid-edit (unsaved-quality mess). Stash with a message, switch to main, make and commit an unrelated "urgent fix," return, pop, finish, and commit. Then run the wrong-branch variant: start edits on main, stash, and land them on a new branch instead.

2. **Hotfix surgery.** Build a feature branch of three commits where only the middle one is an urgent fix. Cherry-pick exactly that commit onto main. Then merge the full branch later and inspect the graph — find both copies of the fix and confirm the file content ended up correct.

3. **Three disasters, three rescues.** Reproduce and then reflog-rescue each of: (a) `reset --hard` past two good commits; (b) a commit created in detached HEAD and abandoned; (c) a `-D` force-deleted branch. Do the full set twice — the second time without looking at this chapter.

4. **Ship v1.0.** In your workspace repo: choose a commit that represents a milestone, tag it (annotated), push the tag, and publish a GitHub Release with honest release notes summarizing what your workspace contains so far. Then tag your current tip as the next version and reflect on what changed between them using `git log v1.0..v1.1 --oneline`.

5. **Run the whole loop with an issue.** Open a real issue on your own repo describing a small improvement. Branch, fix it, reference `Fixes #N` in the PR (Chapter 8 loop), merge, and confirm the issue auto-closed with links intact. Optional stretch: add a trivial Actions workflow (GitHub's "starter workflow" for your language, or a simple markdown-lint) and get a green check on your next PR.
