# Chapter 7: Remotes & GitHub

## Overview

Everything so far happened on one machine. Remotes are how Git repos talk to each other: your local repo exchanges commits with a copy hosted on GitHub, and through that copy, with your other devices and other people. This chapter covers `clone`, `push`, `pull`, and `fetch`; what `origin` and `origin/main` actually are; authenticating from Windows; and — directly relevant to you — how syncing one workspace across multiple machines really works, and how to fix it when the machines diverge.

You've been doing this already. After this chapter you'll know what each step of your sync routine does, which failure messages mean what, and why the routine sometimes breaks after you forget to push from one device.

## Definitions & Explanations

**Remote** — a named reference to another copy of the repository, usually a URL. The conventional name for your primary remote is **`origin`** — it's just a nickname, auto-created by `clone`, with no special powers.

**Clone** — copy an entire repository (all commits, all branches, full history) from a remote to your machine, wiring up `origin` automatically. Cloning is how you "install" a repo on a new device.

**Push** — upload your new local commits to the remote, moving the remote's branch pointer forward. Push is only accepted if it's a fast-forward for the remote (i.e., you're strictly adding to what's there). If the remote has commits you don't, push is rejected — that's divergence, and you reconcile with pull first.

**Fetch** — download new commits *from* the remote **without touching your local branches**. Purely informational: "go see what's new." Always safe.

**Pull** — `fetch` + merge the fetched branch into your current branch. One command, two steps. (It can also be configured as fetch + rebase; Chapter 9.)

**Remote-tracking branches** — after fetching, your repo contains read-only bookmarks like `origin/main`: "where `main` was on the remote, last time I checked." Your `main` and `origin/main` are different pointers and can point at different commits — that gap is exactly what "ahead/behind" messages describe.

```
Two machines syncing through GitHub:

   [Desktop: main]  ── push ──▶  [GitHub: main]  ◀── push ──  [Laptop: main]
                    ◀── pull ──                  ── pull ──▶

The classic sync divergence — Desktop pushed c4, Laptop (which never pulled) made c5:

   GitHub:   c1 ◀── c2 ◀── c3 ◀── c4
   Laptop:   c1 ◀── c2 ◀── c3 ◀── c5        ← push rejected! (GitHub has c4, laptop doesn't)

   Fix: pull (merging or rebasing c5 with c4), then push:
   c1 ◀── c2 ◀── c3 ◀── c4 ◀── M(erge) ── (push) → everyone has everything
                    ◀── c5 ◀──┘
```

**Authentication** — GitHub no longer accepts account passwords for Git operations. On Windows, Git ships with **Git Credential Manager (GCM)**: the first push/clone of a private repo opens a browser window to sign in to GitHub, then stores a token in Windows Credential Manager. After that, it's invisible. (Alternatives you'll meet later: SSH keys, the `gh` CLI's `gh auth login`.)

**Upstream branch** — the remote branch your local branch pushes to and pulls from by default (`main` ↔ `origin/main`). Set once with `push -u`; after that, bare `git push`/`git pull` know where to go.

## Command Examples

### Starting from GitHub (the common direction)

```bash
# Create an empty repo on github.com (web UI: New repository), then:
git clone https://github.com/yourname/practice-sync.git
# Cloning into 'practice-sync'...
# remote: Enumerating objects: 3, done.
# ...
cd practice-sync
git remote -v
# origin  https://github.com/yourname/practice-sync.git (fetch)
# origin  https://github.com/yourname/practice-sync.git (push)
```

### Starting locally, then connecting (the other direction)

```bash
cd existing-local-repo
git remote add origin https://github.com/yourname/existing.git
git push -u origin main        # -u sets the upstream link; needed once per branch
# Enumerating objects: 12, done.
# ...
# To https://github.com/yourname/existing.git
#  * [new branch]      main -> main
# branch 'main' set up to track 'origin/main'.
```

### The daily rhythm

```bash
git pull                # start of session: get what other machines/people pushed
# Already up to date.          (or a list of updated files)

# ...work, add, commit, possibly several times...

git push                # end of session: publish your commits
# To https://github.com/yourname/practice-sync.git
#    a1b2c3d..e4f5a6b  main -> main
```

For a personal sync repo, that's the whole discipline: **pull when you sit down, push when you stand up.**

### Fetch: look before you leap

```bash
git fetch
# remote: Enumerating objects: 5, done.
# ...
#    e4f5a6b..9c8d7e6  main       -> origin/main

git status
# Your branch is behind 'origin/main' by 2 commits, and can be fast-forwarded.

git log --oneline main..origin/main    # exactly what's new on the remote
git log --oneline origin/main..main    # what you have that the remote lacks
git merge origin/main                  # apply it (this is pull's second half)
```

### The rejected push, and the fix

```bash
git push
# ! [rejected]        main -> main (fetch first)
# error: failed to push some refs to 'https://github.com/...'
# hint: Updates were rejected because the remote contains work that you do not
# hint: have locally. ... Integrate the remote changes (e.g. 'git pull ...')

git pull
# (Git may ask how to reconcile divergence — pick merge for now:)
#   git config pull.rebase false      # merge (safe default; set --global if you like)
# Merge made by the 'ort' strategy.   (resolve conflicts here if any — Chapter 6)

git push
# ...success
```

Read the rejection as: "someone (probably you, on another device) pushed first; absorb their commits, then push." It's traffic control, not an error.

### Odds and ends

```bash
git remote show origin        # detailed view: tracked branches, ahead/behind
git push origin feature/x     # push a branch explicitly
git push -u origin feature/x  # ...and set upstream while at it
git branch -vv                # local branches + their upstreams + ahead/behind
git pull --ff-only            # pull that refuses to create surprise merge commits
```

## Common Pitfalls

**Forgot to push on machine A, worked on machine B.** The universal sync accident, producing the rejected-push above on whichever machine pushes second. Recovery: `git pull` (resolve any conflicts — usually there are none if you touched different files), then `git push`. Prevention: the sit-down/stand-up ritual, plus `git status` (which shows ahead/behind) whenever unsure.

**"Everything up-to-date" but GitHub shows old files.** You committed but never pushed, or pushed a different branch than you're viewing on GitHub. Check `git status` (ahead by N?) and `git branch -vv`. Also confirm you're looking at the right branch on the GitHub page.

**Password authentication fails.** GitHub removed password auth for Git in 2021; guides that say "enter your password" are outdated. Use the GCM browser popup (default on Windows), a personal access token, or `gh auth login`. If credentials got stale: Windows → "Credential Manager" → Windows Credentials → remove the `git:https://github.com` entry, and the next push re-prompts.

**Pulling with uncommitted changes.** Git may refuse ("your local changes would be overwritten") or tangle the merge. Commit or stash (Chapter 10) before pulling. A clean `git status` before sync operations prevents a whole category of mess.

**Divergence prompt confusion.** Newer Git versions stop and ask how to reconcile (`pull.rebase` / `pull.ff`). Until Chapter 9 gives you an informed opinion, `git config --global pull.rebase false` (merge) is the safe, traditional answer.

**Treating `origin/main` like a live view.** `origin/main` only updates when you fetch/pull/push. "Behind by 0" may just mean you haven't fetched lately. When it matters, `git fetch` first, *then* read `git status`.

**Cloning inside another repo.** Clone into a fresh directory (e.g., `D:\atoop\practice\`), not inside an existing repo's folder — nested repos behave confusingly (Chapter 1).

## Practice Exercises

1. **Round trip.** Create a new (private is fine) repo on GitHub with a README. Clone it, add a file locally, commit, push, and confirm the file appears on github.com. Then edit the README *in GitHub's web editor*, commit there, and `git pull` it down. Narrate each pointer (`main`, `origin/main`) as it moves.

2. **Two-machine simulation.** Clone the same GitHub repo into two separate local folders, `sim-desktop` and `sim-laptop`. Commit + push from one; pull from the other. Do three round trips. (This is exactly the setup Project 3 expands into a full drill.)

3. **Manufacture the rejection.** Using your two clones: commit + push from `sim-desktop`; then commit in `sim-laptop` *without pulling first*, and try to push. Read the full rejection message aloud. Fix it with pull-then-push. Inspect the resulting graph with `git log --oneline --graph`.

4. **Fetch vs. pull, felt difference.** Push a commit from clone A. In clone B run `git fetch` only. Show that: the working directory is unchanged, `git status` reports "behind," and `git log main..origin/main` reveals the incoming commit. Only then merge. Write one sentence on when you'd prefer this two-step over plain `pull`.

5. **Credential surgery.** Find your stored GitHub credential in Windows Credential Manager. Delete it. Run a `git fetch` on a private repo and complete the re-authentication flow. (Knowing this cold turns a future "auth suddenly broken" day into a two-minute fix.)
