# Chapter 4: Undoing Things

## Overview

The ability to undo is why version control exists — yet Git's undo commands are where beginners get hurt, because several similar-sounding commands do very different things. This chapter maps them all: `restore` (throw away edits), `reset` in its three modes (move a branch backwards), `revert` (undo publicly and safely), and `commit --amend` (touch up the last commit). For each one you'll learn *what it changes*, *what it can destroy*, and *when it's safe*.

The professional skill here isn't memorizing flags — it's choosing the right tool based on one question: **has anyone else seen this history yet?** Private mistakes can be rewritten; shared history should be corrected with new commits.

## Definitions & Explanations

### The undo toolbox at a glance

| Command | What it changes | Destroys work? | Safe when |
|---|---|---|---|
| `git restore <file>` | Working directory (discards edits) | YES — uncommitted edits gone | You truly don't want those edits |
| `git restore --staged <file>` | Staging area only | No | Always |
| `git commit --amend` | Replaces the last commit | No (old commit recoverable via reflog) | Last commit not pushed |
| `git reset --soft <ref>` | Branch pointer only | No | Commits not pushed |
| `git reset --mixed <ref>` (default) | Branch pointer + staging area | No | Commits not pushed |
| `git reset --hard <ref>` | Branch + staging + WORKING DIR | YES — uncommitted work gone | You've checked `git status` is clean-ish and commits not pushed |
| `git revert <ref>` | Nothing — ADDS a new opposite commit | No | Always, even on shared history |

### Reset's three modes, pictured

`git reset <mode> HEAD~1` moves your branch pointer back one commit. The mode decides what happens to the two other areas:

```
Before:   c1 ◀── c2 ◀── c3  ◀─ main ◀─ HEAD      (working dir & staging match c3)

git reset --soft HEAD~1:
          c1 ◀── c2  ◀─ main            c3's changes are now STAGED
                 (c3 still exists, just unlabeled — reflog remembers it)

git reset --mixed HEAD~1:
          c1 ◀── c2  ◀─ main            c3's changes are in the WORKING DIR, unstaged

git reset --hard HEAD~1:
          c1 ◀── c2  ◀─ main            c3's changes are GONE from disk
                                        (the commit is recoverable; uncommitted edits are not)
```

Mental model: **soft** = "un-commit", **mixed** = "un-commit and un-stage", **hard** = "make everything look exactly like that commit, discard the rest."

### Revert: the public undo

`git revert c3` doesn't remove `c3`. It computes the opposite of `c3`'s changes and commits *that* as a new commit:

```
c1 ◀── c2 ◀── c3 ◀── c3' ◀─ main        c3' = "Revert: undo what c3 did"
```

History only ever grows, so nothing anyone has pulled becomes invalid. This is why revert is the **only** history-undo you should use on commits that are already pushed and shared.

### Amend: the touch-up

`git commit --amend` takes the last commit, adds whatever is currently staged, lets you edit the message, and writes a **replacement commit** (new hash — remember, commits are immutable). Use it for "oops, typo in the message" or "forgot to include one file." Because it replaces the commit, treat it like reset: fine before pushing, problematic after.

### The golden rule

> **Never rewrite history that others may have based work on.**
> Local & unpushed → amend/reset freely. Pushed & shared → revert.

(Chapter 9 covers the narrow exception: force-pushing your *own* feature branch.)

## Command Examples

### Discarding uncommitted edits

```bash
echo "bad idea" >> notes.md
git status                       # modified: notes.md

git restore notes.md             # ⚠️ the edit is gone, permanently
git status                       # working tree clean
```

There is no undo for `restore` of never-committed content. Pause before running it.

### Unstage without losing anything

```bash
git add secret-config.txt        # oops
git restore --staged secret-config.txt   # file untouched, just unstaged
```

### Fixing the last commit

```bash
git commit -m "Add loign form"          # typo, and forgot a file
git add forgotten.css
git commit --amend -m "Add login form"
# The last commit now includes forgotten.css and the fixed message.
git log --oneline -1
# 9f8e7d6 (HEAD -> main) Add login form    <-- note: NEW hash
```

### Reset, all three modes

```bash
git log --oneline
# c333333 (HEAD -> main) Third
# c222222 Second
# c111111 First

git reset --soft HEAD~1     # back one commit; "Third"'s changes staged
git status                   # Changes to be committed: ...
git commit -m "Third, but better"   # easy way to redo a commit

git reset --mixed HEAD~1    # back one; changes present but unstaged
git status                   # Changes not staged for commit: ...

# The dangerous one — check status first, every time:
git status                   # make sure nothing uncommitted matters
git reset --hard HEAD~1     # ⚠️ working dir forcibly matches the older commit
# HEAD is now at c111111 First
```

### Revert

```bash
git log --oneline
# b222222 (HEAD -> main) Add broken feature
# b111111 Good commit

git revert b222222
# Opens editor with "Revert 'Add broken feature'" — save & close.
# [main b333333] Revert "Add broken feature"

git log --oneline
# b333333 (HEAD -> main) Revert "Add broken feature"
# b222222 Add broken feature
# b111111 Good commit
```

Reverting an older (not most-recent) commit may hit conflicts — resolve with Chapter 6's technique. `git revert --abort` backs out cleanly if you change your mind mid-way.

### The safety net preview

```bash
git reflog
# b333333 HEAD@{0}: revert: Revert "Add broken feature"
# b222222 HEAD@{1}: commit: Add broken feature
# c111111 HEAD@{2}: reset: moving to HEAD~1
# ...
```

`reflog` records everywhere HEAD has been — even commits that reset "removed." Chapter 10 covers using it to resurrect anything. Knowing it exists is why you don't need to fear reset.

## Common Pitfalls

**Ran `--hard` and lost committed work.** Not actually lost. `git reflog`, find the hash from before the reset, `git reset --hard <that-hash>`. Back in seconds. (This is the single most reassuring trick in Git — practice it deliberately in Exercise 4.)

**Ran `--hard` and lost *uncommitted* work.** This one is real: edits that were never committed or staged are unrecoverable by Git. The defense is habit, not heroics: run `git status` before any `--hard`, and when in doubt, commit or stash (Chapter 10) first. Commits are cheap; make them freely.

**Amended or reset after pushing.** Now your history disagrees with GitHub's; a normal `git push` gets rejected ("non-fast-forward"). Don't reach for `--force` reflexively. If others may have pulled: `git pull` to reconcile, or `git revert` instead. Force-push rules are covered in Chapter 9.

**`git restore .` wiped every edit in the repo.** The `.` means "everything below here." Same recovery reality as `--hard`: committed work is safe, uncommitted work is gone. Restore one file at a time unless you're certain.

**Confusing `revert` with "go back."** People run `git revert` expecting reset behavior, or type `git reset` when the commit is public. Before undoing, ask the golden-rule question, then pick from the table above.

**Detached HEAD after `git checkout <hash>` while trying to undo.** You wanted reset but got sightseeing mode (Chapter 3). No harm done: `git switch main`, then do the reset you intended.

## Practice Exercises

1. **Build the undo lab.** Fresh repo, five commits (c1–c5) each adding one line to `story.txt` so you can always tell "when" you are by reading the file. This repo is the setting for the next exercises — recreate it whenever you trash it, which is the point.

2. **Three flavors of reset.** From c5: `--soft` back to c3 and inspect status + file; recreate the lab; `--mixed` back to c3 and inspect; recreate; `--hard` back to c3 and inspect. Write one line each describing where c4/c5's content ended up.

3. **Amend drill.** Make a commit with a deliberately wrong message AND a missing file. Fix both with a single `--amend`. Compare `git log --oneline` hashes before and after to confirm the commit was replaced, not edited.

4. **Reflog resurrection.** From your 5-commit lab, `git reset --hard HEAD~3`. Confirm the log shows only c1–c2 and the file has shrunk. Now use `git reflog` to find pre-reset state and restore the branch to c5. Do the whole cycle twice until it feels boring instead of scary.

5. **Public vs. private undo.** Scenario planning, on paper first: for each case, choose amend / reset / revert / restore and justify it in one sentence — (a) typo in a commit message you haven't pushed; (b) a pushed commit broke the app for everyone; (c) three local commits you want to squash-redo as one; (d) uncommitted experiment you want gone; (e) you staged a password file but haven't committed. Then act out each scenario in the lab repo.
