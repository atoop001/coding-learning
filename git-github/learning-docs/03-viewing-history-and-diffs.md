# Chapter 3: Viewing History & Diffs

## Overview

Once you have commits, the payoff is being able to *read* them: what changed, when, why, and by whom. This chapter covers `git log` (and its many useful options), `git diff` (comparing any two states), `git show` (inspecting a single commit), and `git blame` (who last touched each line). You'll also learn to safely visit old commits — including the infamous "detached HEAD" state, which is harmless once you understand it.

Professionally, these are the archaeology tools. Debugging often starts with "when did this break?" and code review starts with "what exactly changed?" Reading diffs fluently is as important as writing code.

## Definitions & Explanations

**`git log`** — walks the commit chain backwards from HEAD, printing each commit. Options control formatting, filtering, and which files/commits to include.

**Diff** — a line-by-line comparison between two states, in "unified diff" format:

```
diff --git a/notes.md b/notes.md
index 3b18e51..9daeafb 100644
--- a/notes.md              <-- the "before" version
+++ b/notes.md              <-- the "after" version
@@ -1,3 +1,4 @@            <-- hunk header: where in the file
 # My Notes                 <-- context line (unchanged)
-old line                    <-- removed (starts with -)
+new line                    <-- added (starts with +)
 another context line
```

Lines starting with `-` were removed, `+` were added; a changed line appears as a remove+add pair. The `@@ -1,3 +1,4 @@` header means "this hunk covers lines 1–3 of the old file and 1–4 of the new."

**Which diff compares what** — this trips everyone, so memorize the triangle:

```
        Working Directory
          ▲           ▲
          │ git diff  │ git diff HEAD
          ▼           │
     Staging Area     │
          ▲           │
          │ git diff  │
          │ --staged  │
          ▼           ▼
        Last commit (HEAD)
```

- `git diff` — working directory vs. staging area ("what have I edited but not staged?")
- `git diff --staged` — staging area vs. last commit ("what will my next commit contain?")
- `git diff HEAD` — working directory vs. last commit ("everything uncommitted")

**Commit references** — ways to name a commit without typing the full hash:
- A hash prefix: `a1b2c3d` (first ~7 characters is usually enough)
- `HEAD` — current commit; `HEAD~1` — its parent; `HEAD~3` — three back
- A branch name (`main`) — the commit at that branch's tip
- A tag name (Chapter 10)

**`git show <ref>`** — prints one commit: metadata, message, and its diff against its parent. `git show` alone shows HEAD.

**`git blame <file>`** — annotates every line of a file with the commit and author that last changed it. Despite the name, its professional use is not blame — it's "what commit introduced this line, so I can read its message and understand *why* it exists."

**Detached HEAD** — normally HEAD points at a *branch*, and the branch points at a commit. When you check out a commit directly (`git switch --detach <hash>`), HEAD points at the commit itself — you're "detached" from any branch. It's a read-mostly sightseeing mode: perfect for looking around old versions. The only risk is *committing* while detached (those commits belong to no branch and are easy to lose track of — though Chapter 10's reflog can still rescue them).

```
 Normal:                       Detached:
 HEAD ─▶ main ─▶ c3           HEAD ──────▶ c1
                  │                          │
        c1 ◀── c2 ◀── c3            c1 ◀── c2 ◀── c3 ◀── main
```

## Command Examples

### Log, from verbose to skimmable

```bash
git log                    # full detail, newest first (q to quit the pager)
git log --oneline          # one line per commit
# 789abcd (HEAD -> main) Extend alpha
# f6e5d4c Add alpha and beta files
# a1b2c3d Add notes file

git log --oneline -5       # only the last 5
git log --oneline --graph --all   # ASCII graph of all branches — use constantly from Ch.5 on
git log --stat             # which files changed in each commit, +/- counts
git log -p                 # full diff of every commit (powerful, verbose)

# Filtering
git log --oneline -- notes.md          # only commits touching notes.md
git log --oneline --author="Your Name"
git log --oneline --since="2 weeks ago"
git log --oneline --grep="typo"        # search commit messages
git log -S "function_name"             # "pickaxe": commits that added/removed this string
```

The pager: long output opens in `less`. Navigate with space (page down), `b` (back), `/text` (search), `q` (quit). Not knowing about `q` has trapped many beginners.

### Diff in daily use

```bash
echo "work in progress" >> notes.md

git diff                   # unstaged changes
# diff --git a/notes.md b/notes.md
# ...
# +work in progress

git add notes.md
git diff                   # (empty — nothing unstaged now)
git diff --staged          # the same change, now shown as staged

git diff HEAD~2 HEAD       # everything that changed across the last two commits
git diff HEAD~2 HEAD -- notes.md   # ...restricted to one file
git diff --stat HEAD~2 HEAD        # summary only: files + line counts
```

Read `git diff A B` as "what would turn A into B."

### Inspecting single commits and lines

```bash
git show HEAD~1            # parent of current commit: metadata + message + diff
git show HEAD~1 --stat     # just the file summary
git show HEAD~1:notes.md   # print that file AS IT WAS at that commit (note the colon)

git blame notes.md
# a1b2c3d4 (Your Name 2026-07-17 10:00:00 -0500 1) # My Notes
# 789abcde (Your Name 2026-07-17 11:30:00 -0500 2) work in progress
# Then: git show a1b2c3d4  to read WHY line 1 exists

git blame -L 5,12 notes.md   # only lines 5-12
```

In VS Code: right-click a line → "View Line History" (with GitLens extension) is `blame` with a friendlier face. Hovering changes in the gutter shows inline diffs.

### Time travel (safely)

```bash
git log --oneline                 # pick a commit, e.g. f6e5d4c
git switch --detach f6e5d4c
# HEAD is now at f6e5d4c Add alpha and beta files

git status
# HEAD detached at f6e5d4c        <-- expected, not an error
# Look around, open files, run the code as it was back then...

git switch main                   # come home
# Previous HEAD position was f6e5d4c...
# Switched to branch 'main'
```

(`git checkout f6e5d4c` does the same thing — `checkout` is the older command that `switch`/`restore` split into. You'll see both in documentation; prefer the new ones.)

## Common Pitfalls

**Trapped in the pager.** Screen fills with log output and typing does nothing. Press `q`. If you're in vim instead (an editor opened): `Esc`, then `:q!`, Enter.

**Detached HEAD panic.** The message "You are in 'detached HEAD' state" reads like an injury report. It isn't. You asked to visit a commit; Git took you there. Recovery is always: `git switch main` (or whatever branch). If you *made commits* while detached and then switched away, they're not shown in `git log` anymore — recover with `git reflog` (Chapter 10) and `git branch rescue <hash>`.

**`git diff` shows nothing after you staged.** Correct behavior — plain `diff` compares working dir vs. *staging area*, and they now match. Use `git diff --staged` to see what's queued for commit.

**Reading diff direction backwards.** In `git diff old new`, `-` lines belong to *old*, `+` to *new*. Swap the arguments and every sign flips, which can send you debugging in exactly the wrong direction. When confused, check the `---`/`+++` header lines.

**`git show HEAD~1:notes.md` fails with "invalid object name."** Path is relative to the repo root and case-sensitive on Git's side — `Notes.md` ≠ `notes.md`. Check the exact path with `git show HEAD~1 --stat`.

**Blaming the wrong person (or the formatter).** `blame` shows the *last* touch, which might be a reformat or rename, not the meaningful change. `git blame -w` ignores whitespace changes, and `git log -S` digs for when content actually appeared.

## Practice Exercises

1. **Build a readable history.** In a practice repo, make 8–10 small commits to 2–3 files with good messages. Then produce: (a) a one-line log, (b) the log for just one file, (c) the log filtered by a word in your messages, (d) a `--stat` view. Save the exact commands you used as notes in the repo — then commit that too.

2. **The diff triangle.** Set up a state where `git diff`, `git diff --staged`, and `git diff HEAD` all show *different* output. (Think: one change staged, a different change unstaged, same file.) Explain to yourself out loud what each one is comparing before running it.

3. **Time-travel report.** Detach HEAD onto your third-oldest commit. Confirm from the files that you're really in the past. Return to `main`. Then, without detaching, print the old version of one file from that same commit using `git show`.

4. **Archaeology with blame.** Pick a line in one of your files. Use `blame` to find its commit, `show` to read that commit's full context, and `log -- <file>` to see the file's whole story. Write one sentence: "This line exists because ___."

5. **Pickaxe hunt.** Add a distinctive word to a file in one commit, remove it a few commits later. Then use `git log -S "<word>" --oneline` to find both commits. Compare with what `git log --grep` finds and articulate the difference between searching *content* and searching *messages*.
