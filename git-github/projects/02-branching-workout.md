# Project 2: Branching Workout

## Description

A structured gym session for branches and merges. In a throwaway repo simulating a tiny website (a few text/markdown files standing in for pages), you'll develop several "features" in parallel branches, perform both fast-forward and true merges, and then — the main event — deliberately engineer merge conflicts of increasing nastiness and resolve them calmly, including the abort-and-retry path and a delete-vs-modify conflict.

By the end, the words "CONFLICT (content)" should trigger a checklist in your head instead of adrenaline.

## Difficulty

**Easy-Medium (2/5).** Estimated effort: 3–4 hours. Best done in one or two sittings; the conflict drills (Part C) benefit from being done in a single focused block.

## Chapters used

- Chapter 2 — commits and staging (assumed fluent)
- Chapter 3 — `log --oneline --graph --all` as your map
- Chapter 5 — branching, switching, fast-forward vs merge commits
- Chapter 6 — conflicts: markers, resolution, abort
- Chapter 4 — wrong-branch rescue (pitfall drill)

## Requirements checklist

Fresh throwaway repo. Seed it with 3 files (`home.md`, `about.md`, `menu.md`) in one initial commit.

**Part A — Parallel development**
- [ ] Create three feature branches off the same commit: `feature/home-copy`, `feature/about-page`, `feature/menu-prices`, each with 2+ commits touching *its own* file
- [ ] Before merging anything, draw the commit graph on paper from memory, then verify against `git log --oneline --graph --all`
- [ ] Merge the first branch and confirm it **fast-forwards** (predict it beforehand)
- [ ] Merge the second branch with `--no-ff` and confirm a merge commit exists even though none was strictly required
- [ ] Commit something to `main` *before* merging the third branch, so that merge produces a genuine three-way **merge commit**; identify its two parents with `git log` or `git show`
- [ ] Delete all three branch labels after merging; confirm with `git branch` and confirm no commits were lost

**Part B — The wrong-branch rescue**
- [ ] Deliberately commit a "feature" change directly onto `main`, then rescue it: move the commit to a proper feature branch and return `main` to its prior state (unpushed history, so reset is fair game)
- [ ] Verify with the graph that `main` and the new branch look exactly as if you'd done it right the first time

**Part C — Conflict gauntlet** (each drill: cause it, read `git status` fully, then act)
- [ ] **Drill 1 — same line:** two branches change the same line of `home.md` differently; merge; resolve keeping *your current branch's* version
- [ ] **Drill 2 — abort:** recreate a conflict, explore the mid-merge state (status, diff, the marked-up file), then `merge --abort`; verify the repo is bit-for-bit back to pre-merge; then redo and resolve keeping the *incoming* version
- [ ] **Drill 3 — combined resolution:** engineer a conflict where the correct answer is *both* changes woven together in a specific order; resolve by hand-editing (not just an "accept" button)
- [ ] **Drill 4 — multi-file:** one merge where two files conflict and one merges clean; resolve the two conflicts using different strategies
- [ ] **Drill 5 — delete vs modify:** one branch deletes `menu.md`, the other edits it; merge; read what status says about this shape; resolve it once each way (across two attempts, using abort between)
- [ ] After every resolution: prove no conflict markers survived (search the files for `<<<`) and the graph shows the expected merge commit
- [ ] Keep a running `CONFLICT-LOG.md` in the repo: for each drill, 2–3 lines — what conflicted, why, what you chose

## Hints

- The graph command worth aliasing for this project: `git log --oneline --graph --all --decorate`.
- Whether a merge fast-forwards is decided entirely by whether the *target* branch moved after the split. You can always predict it from the graph before merging.
- Conflicts only happen where edits *overlap*. To manufacture one on demand: both branches must change the same line (or one must delete a file the other edits). To manufacture a clean merge: touch different files.
- Mid-merge, `git status` literally lists your options, including the abort command. Read it slowly — it is the calmest voice in the room.
- VS Code's merge editor is allowed, but for at least two drills resolve in a plain text editor so the markers hold no mystery.
- If a drill goes sideways: `git merge --abort` and re-attempt. If the *repo* goes sideways: reflog (Project 1 skills), or rebuild — seed commits take two minutes.

## Stretch goals

- Redo Drill 1 but resolve via `git checkout --ours home.md` / `--theirs` variants (research these flags first) and articulate when file-level resolution is appropriate vs dangerous.
- Create a fourth branch that stays unmerged for the whole project, then rebase it onto final `main` (Chapter 9 preview) instead of merging, and compare the graphs.
- Time yourself: run the entire conflict gauntlet a second time and try to halve your total time — fluency, not heroics.
- Explore `git rerere` (reuse recorded resolutions): enable it, resolve the same conflict twice, and observe the second resolution auto-apply.
