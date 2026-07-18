# Chapter 6: Merge Conflicts

## Overview

A merge conflict is not an error, a failure, or a punishment. It is Git saying: *"Two branches changed the same lines differently, and I'm not going to guess which version you want — you decide."* That's the entire event. Nothing is corrupted, nothing is lost, and until you commit the resolution, you can always back out.

Conflicts terrify beginners because they arrive with alarming output and mutilated-looking files. This chapter defuses that: you'll learn exactly why conflicts happen, how to read conflict markers, a calm step-by-step resolution routine, how VS Code helps, and the escape hatch (`--abort`) for when you want to regroup. Professionals hit conflicts weekly; resolving them calmly is a core teamwork skill.

## Definitions & Explanations

### Why conflicts happen

A three-way merge compares both branch tips against their common ancestor. For each hunk of each file:

- Changed on one side only → take that change automatically.
- Changed identically on both sides → take it, no fuss.
- Changed **differently on both sides** → **conflict**: human required.

```
                 base (common ancestor): line says  "color: blue"
                /                                   \
   main changed it to "color: red"      feature changed it to "color: green"
                \                                   /
                 merge → CONFLICT (which one did you mean? or something else?)
```

Only *overlapping* edits conflict. Two branches touching different files — or different parts of the same file — merge cleanly. Also conflict-prone: one branch deletes a file the other modified, or both branches add a file with the same name and different contents.

### Anatomy of conflict markers

During a conflicted merge, Git writes both versions into the file:

```
<<<<<<< HEAD
color: red
=======
color: green
>>>>>>> feature/recolor
```

- `<<<<<<< HEAD` to `=======` — the version on **your current branch** (the one you were on when you ran `merge`).
- `=======` to `>>>>>>> feature/recolor` — the version from the **branch being merged in**.

Resolving means: edit the file so it contains what the final result *should be* — one side, the other, both, or something new entirely — and **delete all three marker lines**. The markers are just text; nothing magic enforces them except your diligence in removing them.

### The merge-in-progress state

While a conflict is unresolved, the repo is in a special "MERGING" state (VS Code shows it; `git status` narrates it). Cleanly-merged files are already staged. Conflicted files are listed under "Unmerged paths." Your two moves:

1. **Resolve**: fix each file → `git add` it (this is how you tell Git "this one's decided") → when all are added, `git commit` finishes the merge.
2. **Abort**: `git merge --abort` returns everything to the exact pre-merge state. No penalty. You can retry any time.

That abort command is the anti-panic button. You are never trapped in a merge.

## Command Examples

### Manufacturing a conflict (do this — seriously)

The best inoculation against conflict fear is causing one on purpose:

```bash
mkdir conflict-lab && cd conflict-lab && git init
echo "color: blue" > style.txt
git add . && git commit -m "Base: blue"

git switch -c feature/recolor
echo "color: green" > style.txt
git commit -am "Feature: green"

git switch main
echo "color: red" > style.txt
git commit -am "Main: red"

git merge feature/recolor
# Auto-merging style.txt
# CONFLICT (content): Merge conflict in style.txt
# Automatic merge failed; fix conflicts and then commit the result.
```

Note the tone of that message: "fix conflicts and then commit." It's instructions, not an alarm.

### Reading the situation

```bash
git status
# On branch main
# You have unmerged paths.
#   (fix conflicts and run "git commit")
#   (use "git merge --abort" to abort the merge)
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#         both modified:   style.txt

git diff             # in conflicts, shows a combined view with ++/-- columns
cat style.txt
# <<<<<<< HEAD
# color: red
# =======
# color: green
# >>>>>>> feature/recolor
```

### Resolving

Open `style.txt` in any editor. Decide the true final content — say you want green — and make the file exactly:

```
color: green
```

Then:

```bash
git add style.txt        # "this file is resolved"
git status
# All conflicts fixed but you are still merging.
#   (use "git commit" to conclude the merge)

git commit               # editor opens with a prefilled merge message; save & close
# [main 4d5e6f7] Merge branch 'feature/recolor'

git log --oneline --graph
# *   4d5e6f7 (HEAD -> main) Merge branch 'feature/recolor'
# |\
# | * 2b3c4d5 (feature/recolor) Feature: green
# * | 3c4d5e6 Main: red
# |/
# * 1a2b3c4 Base: blue
```

### Resolving in VS Code

Open a conflicted file and VS Code highlights each conflict block with clickable actions: **Accept Current Change** (HEAD's side) | **Accept Incoming Change** (the merged branch's side) | **Accept Both Changes** | **Compare Changes**. Newer VS Code also offers a three-pane **Merge Editor** (click "Resolve in Merge Editor" in the bottom-right of a conflicted file) showing Incoming | Current | Result.

These buttons just edit the text the same way you would by hand — convenient, but verify the final file reads correctly before you `git add`. "Accept Both" in particular often needs manual stitching.

### The escape hatch and prevention

```bash
git merge --abort        # any time before the concluding commit: full rewind

# Preview trouble before merging:
git log --oneline main..feature/recolor    # what the branch would bring in
git diff main...feature/recolor            # its changes since the common ancestor
```

Prevention beats cure: small branches, merged promptly, conflict far less than long-lived ones. Teams keep branches short-lived partly for exactly this reason.

## Common Pitfalls

**Panic-quitting mid-merge.** People see CONFLICT, close the terminal, and hope. The repo stays in MERGING state and later commands act "weird." Recovery: `git status` to re-orient, then either finish resolving or `git merge --abort`. The state persists until you choose; it never resolves itself.

**Committing conflict markers.** Forgetting to delete `<<<<<<<`/`=======`/`>>>>>>>` lines and committing them — now the markers are *in your code*, usually breaking it. Recovery: it's just a bad commit; edit the file properly and commit again (or `--amend`, Chapter 4). Before concluding any merge, search the file for `<<<` as a final check. (`git diff --check` also flags leftover markers.)

**Choosing a side without reading.** Mashing "Accept Current" on every block ends the conflict fast — and silently throws away the other branch's work. The teammate whose changes vanished will eventually notice. Read each block; sometimes the right answer is *both*, interleaved by hand.

**Thinking the conflict deleted code.** Both versions are right there in the markers, and both branches' commits are untouched — `git log --all` proves it. A conflict cannot destroy committed work.

**"I resolved it but Git still says unmerged."** Editing the file isn't enough; `git add` is the "resolved" declaration. Status stays "both modified" until you add.

**Merging with a dirty working directory.** Uncommitted changes tangled into a conflicted merge is genuinely confusing. Git usually blocks this, but don't fight it: start merges from a clean `git status`. Commit or stash first (Chapter 10).

## Practice Exercises

1. **The inoculation.** Build the conflict-lab from this chapter three times, resolving differently each time: (a) keep your side, (b) keep theirs, (c) write a third thing that's neither. Confirm each resolution with `cat` and the final graph with `--graph`.

2. **Abort and retry.** Create a conflict, look around thoroughly (`status`, `diff`, the file itself), then `git merge --abort`. Verify the repo matches its pre-merge state exactly. Then redo the merge and resolve it for real. The goal: make "abort" feel like a normal, shame-free move.

3. **Multi-file, multi-hunk.** Engineer a merge where two files conflict and a third merges cleanly. Note what `git status` shows about each file mid-merge. Resolve the two conflicts with *different* strategies (one keep-mine, one combine-both) and finish the merge.

4. **The "accept both" trap.** Create a conflict where both branches added a different function/paragraph to the same spot, where the correct result is both pieces *in a specific order*. Resolve using VS Code's Accept Both, then fix the ordering by hand. Reflect: what would have shipped if you hadn't reread the file?

5. **Delete vs. modify.** On one branch, delete a file; on the other, modify that same file. Merge. Read what `git status` says about this special conflict shape and resolve it both ways across two attempts (keep the file with modifications / confirm the deletion). Use `--abort` between attempts.
