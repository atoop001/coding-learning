# Project 1: Terminal-Only Scavenger Hunt

## Description

Build, explore, and solve a puzzle folder structure using **only the terminal** — the GUI file explorer is strictly off-limits for the entire project. You'll play both roles: first the *game master*, scripting nothing but hand-typing commands to construct a nested "dungeon" of folders and files with clues hidden inside; then the *treasure hunter*, navigating and searching your way to the prize. (Even better: swap dungeons with a friend, or rebuild the dungeon a week later from your notes and hunt it cold.)

This project drills the foundation everything else stands on: knowing where you are, moving with confidence, creating and inspecting files, and using tab completion until it's automatic.

## Difficulty

**Beginner** — Estimated effort: 1.5–2.5 hours

## Chapters used

- Chapter 1: Why the Terminal (orientation, running commands, help)
- Chapter 2: Navigating & Inspecting the Filesystem
- Chapter 3: Creating, Copying, Moving & Deleting
- Chapter 4: Viewing & Searching (light use — reading clue files)

## Requirements checklist

Work inside a sandbox: `D:\atoop\coding-projects\command-line\projects\sandbox\scavenger\`.

**Phase A — Build the dungeon (game master):**
- [ ] Create a folder tree at least 4 levels deep with at least 12 directories total, themed however you like (castle wings, ship decks, cave systems...)
- [ ] Create at least 10 files scattered through the tree; at least 6 contain a line of text written at creation time (no GUI editor — use the terminal techniques from Ch. 3)
- [ ] Plant a chain of at least 4 clue files, where each clue's text names the *relative path* to the next clue (e.g., "the next clue lies two levels up, then under armory/")
- [ ] Plant one `treasure.txt` at the end of the chain, plus at least 2 decoy files with misleading names elsewhere
- [ ] Include at least one folder name containing a space, and one hidden item (dotfile or hidden attribute)
- [ ] Produce a map of your dungeon using a recursive listing command, saved to `map.txt` (you may not look at it during Phase B!)

**Phase B — Run the hunt (treasure hunter):**
- [ ] Starting from the dungeon root, follow the clue chain to the treasure using only `cd`, listing commands, and file-viewing commands
- [ ] Use tab completion for every path you type (honor system — no full hand-typed paths)
- [ ] At each clue, record in a `journal.txt` (appended via terminal, Ch. 5's `>>` welcome): your current absolute path and the command you ran next
- [ ] Find the hidden item without using your map
- [ ] Prove completion: display the treasure's contents and its full absolute path in one final journal entry

**Phase C — Both shells:**
- [ ] Complete Phase B once in PowerShell
- [ ] Repeat the hunt (or a shortened version) in Git Bash or WSL, noting in your journal the 3 biggest command differences you felt

## Hints

- Stuck creating a file with content and no editor? Chapter 3 shows a `New-Item -Value` way and Chapter 5 shows a redirection way. Either counts.
- If a `cd` fails on the folder with a space, the error message is telling you how the shell split your input. Chapter 2's pitfalls cover the fix.
- `..` chains (`..\..`) make excellent clue material — force yourself to *think* in relative paths before moving.
- Lost? Two commands re-orient you completely: one prints where you are, one lists what's around. Make them reflexes before reaching for the map.
- For the hidden item, remember: PowerShell and Bash reveal hidden things with *different* flags (Ch. 2).
- Rebuilding for a second run goes faster if your Phase A commands came from your history — explore what the up-arrow and `Ctrl+R` give you.

## Stretch goals

- Write the clue texts as riddles ("I sleep where the logs are kept...") so the hunter must *search* filenames (Ch. 4 patterns) rather than follow literal paths.
- Add a locked door: a clue that can only be read after renaming or moving a "key" file to a specific location.
- Time yourself on a fresh rebuild-and-hunt a week later; aim to halve your original time.
- Do a full run inside WSL over ssh from PowerShell (after Chapter 10) — dungeon-crawling on a "remote server."
