# Project 3: Two-Device Sync Drill

## Description

You already sync a workspace across devices — this project makes you *good* at it, including every way it goes wrong. Using one throwaway GitHub repo and **two local clones on the same machine** (standing in for your desktop and laptop), you'll run the healthy sync rhythm, then systematically break it: forget to push, forget to pull, diverge on different files, diverge on the *same* lines, and fetch-inspect-merge your way back to order every time.

Everything here transfers 1:1 to your real multi-device workflow — after this drill, a rejected push on your actual laptop is a 60-second non-event.

## Difficulty & Estimated Effort

**Medium (3/5).** Estimated effort: 3–4 hours, plus a GitHub account. Requires working GitHub authentication (Chapter 7's setup).

## Chapters used

- Chapter 7 — remotes, clone/push/pull/fetch, origin/main, auth, divergence
- Chapter 6 — resolving the conflicts that divergence produces
- Chapter 5 — reading the graphs that merges create
- Chapter 3 — `log A..B` range syntax for "what's incoming/outgoing"

## Requirements checklist

**Part A — Setup**
- [ ] Create a fresh throwaway repo on GitHub (README initialized)
- [ ] Clone it twice on your machine into sibling folders named `sim-desktop` and `sim-laptop`
- [ ] In each clone, verify `git remote -v` and `git branch -vv` show what you expect; note in a `SYNC-LOG.md` (in the repo) what `origin/main` means in each clone

**Part B — The healthy rhythm**
- [ ] Perform 3 full round trips: commit+push from one clone, pull into the other, alternating direction — always pulling before starting "work"
- [ ] During one round trip, use `git fetch` + `git log main..origin/main` in the receiving clone to inspect exactly what's incoming *before* merging it
- [ ] After each round trip, confirm both clones and GitHub show identical `git log --oneline` tips

**Part C — Breaking it (the point of the project)**
- [ ] **Scenario 1 — forgot to pull, different files:** desktop commits+pushes file A; laptop (without pulling) commits file B and tries to push. Get the rejection, read the full message, recover with pull-then-push. Examine the resulting merge commit in the graph
- [ ] **Scenario 2 — forgot to pull, same lines:** engineer the same race but with both clones editing the same line of the same file, so the reconciling pull *conflicts*. Resolve it, push, and verify both clones converge to identical content
- [ ] **Scenario 3 — stale view:** make GitHub move ahead (push from desktop), then show that laptop's `git status` claims up-to-date *until* a fetch; document why `origin/main` lied
- [ ] **Scenario 4 — dirty pull:** with uncommitted changes in the laptop clone, attempt a pull that would touch the same file; deal with Git's refusal two ways across two attempts (commit first / stash first — stash is a small Chapter 10 preview)
- [ ] After every scenario, append to `SYNC-LOG.md`: what broke, what the error actually said, what fixed it

**Part D — Distill it**
- [ ] Write `SYNC-PLAYBOOK.md`: your personal, tested, step-by-step routine for real multi-device life — sit-down steps, stand-up steps, and a "when push is rejected" recipe. Commit it via a normal round trip
- [ ] Adopt the playbook in your *actual* workspace repo for one week (honor system)

## Hints

- Two clones on one machine are a perfect simulation — Git neither knows nor cares that "both devices" share a keyboard. Just be disciplined about which folder you're in; putting the clone name in your prompt or window title helps.
- The range syntax is your radar: `git log --oneline main..origin/main` = incoming, `origin/main..main` = outgoing (unpushed). Ahead/behind in `git status` and `git branch -vv` summarize the same facts.
- If Git asks how to reconcile divergent branches on pull, Chapter 7 gave you the safe answer (`pull.rebase false`) — after Chapter 9 you may choose differently for a personal repo. Either is defensible; note your choice in the playbook.
- A rejected push never damages anything. It is purely "the remote knows things you don't yet." Nothing about it is urgent.
- Scenario 2's conflict is just Project 2 material wearing a trench coat. Same markers, same routine.

## Stretch goals

- Add a third clone (`sim-tablet`) and create a three-way divergence: two clones push merges reconciling with each other while the third drifts. Bring all three home.
- Redo Scenario 1 with `git pull --rebase` in the laptop clone and compare the resulting history (linear, no merge commit) to the merge version — then decide, in writing, which your personal sync repo should use.
- Simulate "laptop died mid-work": clone a fresh third copy and confirm exactly what was and wasn't preserved (pushed commits: yes; local-only commits and reflog: no). Let the result adjust how often you push.
- Investigate `git config push.autoSetupRemote true` and decide whether it belongs in your global config.
