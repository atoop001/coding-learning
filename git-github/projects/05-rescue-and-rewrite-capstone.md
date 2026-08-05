# Project 5: Rescue & Rewrite — Capstone Drill

## Description

The final exam you give yourself. First you'll *manufacture a disaster site*: a messy repo with scruffy commits, junk files that never belonged, an accidentally committed "secret," work stranded on the wrong branch, and commits obliterated by hard resets and branch deletions. Then — playing the senior engineer arriving on the scene — you'll rescue every lost commit with the reflog, relocate stranded work with cherry-pick and stash, scrub the junk into a proper `.gitignore`, rewrite the surviving mess into a clean, reviewable history with interactive rebase, and ship the result: pushed, tagged, and published as a GitHub release with honest notes.

If Project 1 made undo boring, this one makes *catastrophe* boring. That calm is the capstone.

## Difficulty & Estimated Effort

**Hardest (5/5).** Estimated effort: 5–7 hours. Do Part B in one focused sitting if possible — mid-rescue is a bad place to lose your mental stack.

## Chapters used

- Chapter 10 — reflog, stash, cherry-pick, tags/releases (core)
- Chapter 9 — interactive rebase, force-with-lease, .gitignore & untracking
- Chapter 4 — reset/revert/amend judgment calls
- Chapters 5–6 — branch surgery and any conflicts it causes
- Chapters 7–8 — pushing the rescued result; optional PR polish

## Requirements checklist

**Part A — Build the disaster (yes, on purpose; keep a `DISASTER-MANIFEST.md` *outside* the repo listing everything you break so you can grade your own rescue)**
- [ ] Fresh throwaway repo, pushed to GitHub, with ~6 legitimate commits of believable "project" content
- [ ] Pollute it: commit a `logs/` directory with `.log` files, a `build/` folder, and a fake secret file (`.env` with `API_KEY=not-real-obviously`) — spread across 2–3 commits
- [ ] Scruff it up: 4+ commits with messages like "wip", "asdf", "fix", including one commit that mixes two unrelated changes
- [ ] Strand some work: 2 good commits on a branch, then `git branch -D` it; note the tip hash in your manifest, then close the terminal (no cheating from scrollback)
- [ ] Orphan a commit: make one useful commit in detached HEAD and switch away
- [ ] Destroy some history: `git reset --hard` main back past at least 2 good commits (after noting, in the manifest, *what* the commits contained — not their hashes)
- [ ] Leave uncommitted WIP for a *different* feature sitting dirty in the working directory

**Part B — The rescue (recover everything; the manifest is your scorecard)**
- [ ] Stabilize first: get the dirty WIP safely out of the way (stash with a message, or a WIP branch — justify your choice in writing)
- [ ] Using **only** `git reflog` and `git show` for navigation: find and restore the hard-reset commits to `main`
- [ ] Resurrect the force-deleted branch by re-creating its label at the right commit; verify both commits survived intact
- [ ] Rescue the orphaned detached-HEAD commit onto an appropriately named branch (or cherry-pick it where it belongs)
- [ ] Reconcile everything into `main` (merges or cherry-picks — your call), resolving any conflicts
- [ ] Audit against the manifest: every item accounted for, nothing missing. Score yourself honestly

**Part C — The rewrite**
- [ ] Untrack all junk (`logs/`, `build/`, `.env`) while keeping the files on disk; add a proper `.gitignore`; commit the cleanup
- [ ] Verify with a fresh local clone that the junk no longer transfers — then demonstrate (with `git show`) that the "secret" is still visible in an old commit, and write three sentences in the repo about what this means for real leaked credentials
- [ ] With interactive rebase on the unpushed portion, transform the scruffy commits into a clean sequence: proper messages, fixups melded, and the mixed-changes commit **split into two** (research `edit` + `reset` for splitting — this is the black-belt move)
- [ ] Every surviving commit message now imperative, specific, and honest
- [ ] Push the final history (using `--force-with-lease` where the rewrite requires it, and articulating why that's acceptable *here*)

**Part D — Ship it**
- [ ] Annotated tag `v1.0` on the final commit; push the tag
- [ ] GitHub **Release** for `v1.0` whose notes summarize: what the project "is", what was rescued, and what was rewritten
- [ ] Close out: `RETRO.md` in the repo — the three Git behaviors that most surprised you across the whole track, and your personal "when disaster strikes" first-move checklist

## Hints

- The reflog reads newest-first and every entry is labeled with what caused it (`commit:`, `reset:`, `checkout:`). Rescues are mostly *reading*, slowly, then one small command.
- Rescue order matters less than rescue *calm*: before each recovery move, say out loud what you expect the graph to look like afterward, then check with `git log --oneline --graph --all`.
- `git branch <name> <hash>` is the gentlest rescue tool in Git — it changes nothing, moves nothing, only labels. When unsure, label first, decide later.
- Deleted branch tips don't appear in `git reflog` under the branch's name — but HEAD was *on* that branch when its commits were made. Look for the commit messages.
- Splitting a commit: mark it `edit` in the rebase todo, and when the rebase pauses, `git reset HEAD~1` un-commits it into your working directory — then build two commits with selective staging (Project 1 skills) and `--continue`.
- Force-with-lease on this repo is safe because you are provably the only writer. Note that reasoning — it's exactly the check a professional makes.
- Truly stuck mid-rebase? `git rebase --abort` costs you nothing but the attempt. Truly stuck generally? You have a manifest and a reflog; there is no unrecoverable state here except panic.

## Stretch goals

- Run the entire disaster+rescue again from scratch, timed, without opening any chapter. Under an hour means you've genuinely internalized the toolkit.
- Research and use `git filter-repo` (or study why BFG existed) to *actually* scrub the fake secret from all history, then verify with a fresh clone and `git log -S "API_KEY"` — and note why rotating a real credential is still mandatory.
- Hand a friend the disaster-generation script (write Part A as a `.ps1` or `.sh`) and race rescues.
- Publish `v1.1`: one more feature shipped through the full Project 4 PR ceremony on this repo, proving the rescued repo is a healthy, living project.
- Write a one-page "Git first aid" cheat sheet distilled from your RETRO.md and pin it in your real workspace repo — the artifact of the whole track.
