# Chapter 2: Repositories & the Three Areas

## Overview

This chapter is the heart of Git. Almost every moment of Git confusion — "why isn't my change in the commit?", "what does staging even mean?", "why does status show my file twice?" — dissolves once you understand the **three areas**: the working directory, the staging area (index), and the commit history. You'll learn what `add`, `commit`, `status`, and `log` actually do, and — critically — what a commit *really is* under the hood.

Professionals care about this because good commits are the unit of professional work: they're what gets reviewed, reverted, cherry-picked, and bisected. Learning to build clean, deliberate commits (rather than "stuff everything in and hope") is the single biggest habit upgrade this track offers.

## Definitions & Explanations

### The three areas

```
 Working Directory          Staging Area (Index)         Repository (History)
 ─────────────────          ────────────────────         ────────────────────
 The actual files           A "loading dock" where       The chain of commits
 you see and edit           you assemble the NEXT        permanently recorded
 in VS Code                 commit, piece by piece       in .git

        │                           │                            │
        │ ───── git add ──────────▶ │                            │
        │                           │ ────── git commit ───────▶ │
        │                           │                            │
        │ ◀──────────────── git restore / checkout ───────────── │
```

**Working directory** — your files as they exist on disk right now. Edit freely; Git watches but records nothing automatically.

**Staging area (also called the index)** — a middle layer where you *stage* exactly the changes you want in your next commit. `git add file.txt` copies the current state of `file.txt` into the staging area. This lets you commit some changes but not others — e.g., commit the bug fix now, keep the half-finished experiment out of it.

**Repository / history** — the permanent record. `git commit` takes everything in the staging area and seals it into a new commit.

### What a commit really is

A commit is **not a diff**. A commit is a **complete snapshot** of every tracked file at that moment, plus:

- a pointer to its **parent commit** (the one before it),
- an author, timestamp, and message,
- a unique **SHA-1 hash** (like `a1b2c3d...`, 40 hex characters) computed from all of the above.

Because each commit points to its parent, history forms a chain:

```
  a1b2c3 ◀── d4e5f6 ◀── 789abc ◀── main (branch label)
  "init"     "add        "fix
              notes"      typo"

  (arrows point PARENT-ward: each commit knows what came before it)
```

The hash is a fingerprint of the entire snapshot *and* its parent. Change anything — even one character, even the message — and the hash changes completely. This is why Git history is tamper-evident, and why "changing" a commit actually means creating a new one (Chapter 4).

Snapshots sound wasteful, but Git stores unchanged files once and reuses them, so it's extremely compact in practice.

**Tracked vs. untracked** — a file is *tracked* once it has been added/committed at least once. *Untracked* files are ones Git has never been told about; they sit in `git status` under "Untracked files" until you `add` them or ignore them (Chapter 9 covers `.gitignore`).

**HEAD** — a pointer to "where you are now" — normally the tip of your current branch, i.e., the commit your next commit will build on. You saw the `HEAD` file in Chapter 1's exercises.

### Commit messages

Convention used almost everywhere:
- A short summary line (~50 characters, imperative mood: "Add login form", not "Added" or "Adds").
- Optionally a blank line, then a longer body explaining *why*.

The message is for the future reader — usually you, three weeks later, wondering why this change exists.

## Command Examples

### The core loop

```bash
cd git-practice          # the repo from Chapter 1

# Create a file
echo "# My Notes" > notes.md

git status
# On branch main
# No commits yet
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         notes.md

git add notes.md         # stage it (copy to the staging area)

git status
# On branch main
# No commits yet
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#         new file:   notes.md

git commit -m "Add notes file"
# [main (root-commit) a1b2c3d] Add notes file
#  1 file changed, 1 insertion(+)
#  create mode 100644 notes.md

git status
# On branch main
# nothing to commit, working tree clean     <-- the calmest sentence in Git

git log
# commit a1b2c3d4e5f6... (HEAD -> main)
# Author: Your Name <admin@tnt-tutoring.com>
# Date:   Fri Jul 17 10:00:00 2026 -0500
#
#     Add notes file
```

### The part everyone trips on: edit after staging

```bash
echo "line two" >> notes.md      # modify the file
git add notes.md                  # stage that version
echo "line three" >> notes.md    # modify it AGAIN

git status
# Changes to be committed:
#         modified:   notes.md      <-- the staged version (with line two)
# Changes not staged for commit:
#         modified:   notes.md      <-- the newer edit (line three)
```

The same file appears **twice** because the staging area holds the version from when you ran `add` — not a live link to the file. If you commit now, only "line two" goes in. Run `git add notes.md` again to stage the latest version. This is the #1 source of "my change didn't make it into the commit."

### Staging several files, and shortcuts

```bash
echo "alpha" > a.txt
echo "beta"  > b.txt

git add a.txt b.txt      # stage specific files (the deliberate way)
git add .                # stage EVERYTHING changed/new under current dir (the blunt way)
git commit -m "Add alpha and beta files"

# Skip staging for already-tracked files (does NOT include new/untracked files):
echo "gamma" >> a.txt
git commit -a -m "Extend alpha"
# [main 789abcd] Extend alpha
#  1 file changed, 1 insertion(+)
```

Habit to build: prefer naming files (or using `git add -p`, Chapter 3+) over `git add .`. Deliberate staging is what produces clean, reviewable commits.

### Unstaging and useful log views

```bash
git add b.txt            # oops, didn't mean to stage this
git restore --staged b.txt   # remove from staging; file on disk is untouched

git log --oneline
# 789abcd (HEAD -> main) Extend alpha
# f6e5d4c Add alpha and beta files
# a1b2c3d Add notes file

git log --oneline --graph    # you'll appreciate --graph once branches exist
```

## Common Pitfalls

**"I committed but my change isn't in it."** — You edited after staging (see above). Recovery: `git add` the file again and either make a new commit or amend the last one (`git commit --amend`, detailed in Chapter 4). Nothing is lost.

**Same file listed under both "to be committed" and "not staged."** — Not a bug, not a corruption. It's the staged-then-edited situation. Re-run `git add` if you want the newest version staged.

**`git add .` swept in files you didn't want** (build junk, secrets, a 300 MB video). Recovery *before committing*: `git restore --staged <file>`. If already committed, see Chapter 4 (amend) and Chapter 9 (removing tracked files / .gitignore). The lesson: glance at `git status` before `add`, and at `git status` again before `commit`.

**Commit rejected / weird editor opened.** — `git commit` without `-m` opens your configured editor for the message. If VS Code opens: type the message, save, **close the tab**, and the commit completes. If you see a strange full-terminal editor (vim) instead: press `Esc`, type `:q!`, press Enter to escape without committing, then fix your editor config (Chapter 1).

**Empty commit message aborts the commit.** — Closing the editor without typing anything cancels the commit. Nothing happened; just run it again.

**Panic at "detached HEAD" from an early `checkout` experiment.** — If you ran `git checkout <some-hash>` while exploring, `git status` says "HEAD detached at a1b2c3d". You are not broken; you're visiting an old snapshot. Recovery: `git switch main` returns you to normal. Full explanation in Chapter 3.

## Practice Exercises

1. **Three-area walkthrough.** In a fresh practice repo, create `journal.md`. Move it through all three areas, running `git status` between every single command and predicting the output *before* you look. Do this until your predictions are right twice in a row.

2. **The double-listing.** Deliberately reproduce the staged-then-edited state so one file appears in both sections of `git status`. Commit *without* re-adding, then use `git log` and open the file to prove which version made it into the commit. Then fix things so the working directory, staging area, and last commit all match.

3. **Selective staging.** Modify three different files, then build **two** commits: the first containing exactly two of the files, the second containing the third. Verify with `git log --oneline` and `git show --stat HEAD` / `git show --stat HEAD~1` (peek at Chapter 3 for `show` if needed).

4. **Message discipline.** Make five small commits to a practice file, each with a proper imperative summary line under 50 characters. Then run `git log --oneline` and judge honestly: could a stranger understand the story of this repo from the messages alone?

5. **Snapshot proof.** Commit a file, change one character, commit again. Run `git log` and compare the two hashes. Then use `git cat-file -p <hash-of-second-commit>` (a plumbing command — safe, read-only) and identify the `parent` line and `tree` line. Relate what you see to the "what a commit really is" section.
