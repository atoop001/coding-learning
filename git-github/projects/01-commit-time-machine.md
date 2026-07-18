# Project 1: Commit Time Machine

## Description

Build a small commit history *deliberately* — then navigate it, read it, and undo through it in every direction. You'll create a throwaway repo containing a short story written one commit at a time, so the file's contents always tell you "when" you are. Then you'll travel through that history (log, diff, show, blame, detached HEAD) and practice every major undo tool (restore, all three resets, amend, revert) against it, finishing with a reflog rescue.

The point is to make history and undo *boring* — you should end this project having reset --hard and recovered so many times that neither impresses you anymore.

## Difficulty

**Easiest (1/5).** Estimated effort: 2–3 hours. Can be split across two sessions (build + navigate, then undo drills).

## Chapters used

- Chapter 1 — setup, git status habits
- Chapter 2 — the three areas, add/commit, what a commit is
- Chapter 3 — log, diff, show, blame, detached HEAD
- Chapter 4 — restore, reset (soft/mixed/hard), amend, revert
- Chapter 10 — reflog (rescue section only)

## Requirements checklist

Work in a brand-new throwaway repo (NOT your workspace-sync repo).

**Part A — Build the history**
- [ ] Create a fresh repo with your configured identity; confirm with `git config user.name` inside it
- [ ] Create `story.txt` and grow it across **exactly 8 commits**, each adding one numbered line ("Line 1: ...", "Line 2: ..."), each with a proper imperative commit message
- [ ] At least two commits also touch a second file (`characters.txt`), so some commits are multi-file
- [ ] One commit must be built with *selective staging*: change both files, but commit them as two separate commits
- [ ] Produce a `git log --oneline` that reads as a coherent story of the repo

**Part B — Navigate**
- [ ] Show the full history, then only the last 3 commits, then only commits touching `characters.txt`
- [ ] Use `git diff` between two non-adjacent commits and explain (in a `NOTES.md` file in the repo) what the output says
- [ ] Use `git show` to print an old *version of story.txt* without switching to it
- [ ] Use `git blame story.txt` and identify which commit added Line 5
- [ ] Enter detached HEAD at the 4th commit, verify the file contents match that point in time, and return safely to `main`

**Part C — Undo everything (and survive)**
- [ ] Make an uncommitted edit and discard it with `restore`
- [ ] Stage a change, unstage it without losing it
- [ ] Make a commit with a wrong message + missing file; repair both with a single `--amend`
- [ ] From the tip: `reset --soft` back 2 commits, examine state, and rebuild history with a new commit
- [ ] Restore the repo to 8+ commits, then `reset --hard` back 3 commits — and recover fully using **only** `git reflog`
- [ ] `revert` one middle commit (not the tip) so history *grows* rather than rewinds; resolve any conflict that results
- [ ] Record in `NOTES.md`: for each undo tool, one sentence on when it's the right choice

## Hints

- Numbered lines in `story.txt` are your instrument panel: after any operation, `cat story.txt` (or `type story.txt` in PowerShell) tells you instantly where you are.
- Run `git status` before and after *every* operation in Part C. Predict its output first; check second.
- If `git log` seems to be missing commits after a reset — that's expected. The log shows what's *reachable*. The reflog shows what *happened*.
- Reverting a middle commit conflicts when later commits touched the same lines. That's Chapter 6's material a bit early — read just the "anatomy of conflict markers" section, or choose a middle commit that later commits didn't touch.
- If you truly wedge the repo: delete the folder and rebuild Part A. Rebuilding is fast, and repetition is the actual product here.

## Stretch goals

- Do the entire Part C sequence a second time using VS Code's Source Control UI wherever possible, noting which operations the UI makes easier and which it hides.
- Use `git log -S` (pickaxe) to find which commit introduced and which removed a specific word you planted.
- Explore `git bisect` on your history: mark the tip "bad," an early commit "good," and let Git binary-search for the commit that introduced a chosen line — a preview of a professional debugging tool.
- Create a `git config --global` alias for your favorite log format and defend the choice in `NOTES.md`.
