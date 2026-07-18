# Chapter 5: Branching

## Overview

Branches are Git's killer feature: cheap, instant, parallel lines of development. A branch lets you build a feature, fix a bug, or run an experiment in isolation, then *merge* the result back — or throw it away — without ever endangering your main line of work. Where older systems made branching heavyweight and rare, Git makes it so cheap that professionals branch for nearly everything ("branch per task" is the industry norm, and it's the foundation of the pull request workflow in Chapter 8).

This chapter covers creating and switching branches, what a branch physically *is* (spoiler: almost nothing), merging, and the difference between fast-forward merges and merge commits.

## Definitions & Explanations

### A branch is just a movable label

A branch is a tiny file containing one commit hash. That's all. `main` is a 41-byte file in `.git/refs/heads/` that says "the tip of main is commit c3." Creating a branch creates one small file; nothing is copied. This is why Git branching is instant even in huge repos.

```
                 c1 ◀── c2 ◀── c3   ◀─ main ◀─ HEAD
```

When you commit, the *current* branch label moves forward automatically. HEAD points to whichever branch you're "on."

```
git switch -c feature        (create feature at c3, move HEAD to it)

                 c1 ◀── c2 ◀── c3   ◀─ main
                                 ▲
                                 └─ feature ◀─ HEAD

...two commits later on feature:

                 c1 ◀── c2 ◀── c3   ◀─ main
                                 ▲
                          c4 ◀── c5*  wait, arrows point parentward:

                 c1 ◀── c2 ◀── c3 ◀── c4 ◀── c5   ◀─ feature ◀─ HEAD
                                ▲
                                └─ main    (main hasn't moved)
```

Switching branches rewrites your working directory to match that branch's tip. Your files literally change on disk — same folder, different contents. This surprises people the first time; it's normal and safe (Git refuses to switch if it would overwrite uncommitted work).

### Merging: two shapes

**Fast-forward merge** — if `main` hasn't moved since `feature` split off, "merging feature into main" requires no merging at all: Git just slides the `main` label forward. No new commit is created.

```
Before:  c1 ◀── c2 ◀── c3 ◀── c4 ◀── c5 ◀─ feature
                        ▲
                        └─ main

After 'git merge feature' (on main):     ...c3 ◀── c4 ◀── c5 ◀─ main, feature
```

**Three-way merge (merge commit)** — if *both* branches gained commits, histories diverged. Git finds the common ancestor (c3), combines both sets of changes, and records a **merge commit** with *two parents*:

```
Before:                 c4 ◀── c5   ◀─ feature
                       /
         c1 ◀── c2 ◀── c3 ◀── c6    ◀─ main

After 'git merge feature' (on main):

                        c4 ◀── c5
                       /         \
         c1 ◀── c2 ◀── c3 ◀── c6 ◀── M   ◀─ main
                                     (M has parents c6 AND c5)
```

If both branches changed the *same lines*, Git can't combine automatically — that's a merge conflict, and it gets its own chapter (6).

**`--no-ff`** — forces a merge commit even when fast-forward is possible, so the branch's existence stays visible in history. Teams often prefer this (it's what GitHub's default merge button does).

**Deleting a branch** — deletes the *label*, not the commits. After merging, the branch name has done its job; deleting it is routine hygiene, not destruction.

## Command Examples

### Creating, listing, switching

```bash
git branch                     # list branches, * marks current
# * main

git switch -c feature/greeting # create AND switch (-c = create)
# Switched to a new branch 'feature/greeting'

git branch
#   main
# * feature/greeting

echo "hello" > greeting.txt
git add greeting.txt
git commit -m "Add greeting"

git switch main                # back to main
# 'greeting.txt' vanishes from the folder — it only exists on the branch
ls                             # no greeting.txt. Not lost — just elsewhere.

git switch feature/greeting    # and it's back
```

Branch naming: slashes are conventional and fine — `feature/login`, `fix/crash-on-save`, `experiment/new-layout`. (Older tutorials use `git checkout -b`; same effect.)

### Fast-forward merge

```bash
git switch main
git merge feature/greeting
# Updating c3f9a2b..8d7e6f5
# Fast-forward
#  greeting.txt | 1 +
#  1 file changed, 1 insertion(+)

git log --oneline --graph
# * 8d7e6f5 (HEAD -> main, feature/greeting) Add greeting
# * c3f9a2b Previous commit
#   (straight line — no merge commit)

git branch -d feature/greeting     # clean up the label
# Deleted branch feature/greeting (was 8d7e6f5).
```

### Divergence and a real merge commit

```bash
git switch -c feature/footer
echo "footer" > footer.txt
git add . && git commit -m "Add footer"

git switch main
echo "header" > header.txt
git add . && git commit -m "Add header"     # now BOTH branches moved

git merge feature/footer
# An editor opens with "Merge branch 'feature/footer'" — save & close.
# Merge made by the 'ort' strategy.
#  footer.txt | 1 +

git log --oneline --graph
# *   9a8b7c6 (HEAD -> main) Merge branch 'feature/footer'
# |\
# | * 5f4e3d2 (feature/footer) Add footer
# * | 1a2b3c4 Add header
# |/
# * 8d7e6f5 Add greeting
```

That diamond shape is the signature of parallel work. `git log --oneline --graph --all` should become your reflexive orientation command.

### Odds and ends you'll need

```bash
git branch new-idea            # create WITHOUT switching
git switch -                   # toggle to the previous branch (like 'cd -')
git branch --merged            # branches fully merged into current (safe to delete)
git branch -d name             # delete (refuses if unmerged)
git branch -D name             # force delete an unmerged branch — you're declaring the work disposable
git merge --abort              # bail out of a conflicted merge (Chapter 6)
```

In VS Code: the branch name sits in the bottom-left status bar; clicking it opens a branch switcher. The Source Control panel's `...` menu has Branch and Merge submenus. Same operations, same rules.

## Common Pitfalls

**"My files disappeared!"** — You switched branches, and files that only exist on the other branch left the working directory. Nothing is lost; switch back. Verify with `git log --oneline --all` that the commits still exist. This is the branching version of detached-HEAD panic: alarming output, zero damage.

**Committed to the wrong branch** (usually `main` instead of a feature branch). Recovery when it's the latest commit and unpushed:
```bash
git branch feature/where-it-belongs   # label the current commit
git reset --hard HEAD~1               # pull main back one (Ch.4)
git switch feature/where-it-belongs   # commit is safe here
```

**"error: Your local changes would be overwritten by checkout."** — Git is *protecting* you: switching would clobber uncommitted edits. Options: commit them, stash them (Chapter 10), or discard them (`git restore`, Chapter 4) — then switch.

**Expected a fast-forward, got an editor asking for a merge message** (or vice versa). Whether a merge fast-forwards depends only on whether the target branch moved since the split. Check first with `git log --oneline --graph --all` and it will never surprise you.

**Deleted a branch and think the work is gone.** `-d` refuses unless merged, so usually the commits are reachable from `main` — nothing lost. If you forced with `-D`, the commits are unlabeled but still recoverable via `git reflog` (Chapter 10).

**Branch name soup.** After a while you have `test`, `test2`, `fix-final`, `fix-final-REAL`. Cure: descriptive names, and prune merged branches promptly (`git branch --merged`, then `-d` each).

## Practice Exercises

1. **Label mechanics.** In a practice repo with a few commits, run `git branch idea` and then open `.git/refs/heads/idea` in a text editor. Compare its contents to `git log --oneline -1`. Make a commit on `main` and check both files again — which label moved and why?

2. **Two features in parallel.** From the same starting commit, create `feature/a` and `feature/b`. Add different files to each (2 commits per branch). Draw the commit graph on paper *from memory*, then check yourself with `git log --oneline --graph --all`. Merge both into `main` — predict for each merge whether it will fast-forward *before* you run it.

3. **The disappearing file drill.** Create a branch, add a file, commit, switch to main, watch it vanish, switch back, watch it return. Repeat until it feels mundane. Then explain in one written sentence where the file "goes."

4. **Wrong-branch rescue.** Deliberately commit to `main` something that should have been on a feature branch, then perform the three-step rescue from the pitfalls section. Verify with `--graph --all` that `main` is back where it started and the feature branch holds the commit.

5. **`--no-ff` comparison.** Set up a fast-forwardable branch twice. Merge once normally and once with `git merge --no-ff`. Compare the two resulting graphs and write down one reason a team might prefer each style.
