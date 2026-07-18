# Project 3: Environment Makeover

## Description

Turn the stock terminal into *your* terminal. In this project you audit your current environment, then deliberately rebuild it: a version-controlled PowerShell profile with aliases and functions that match how you actually work, a matching Bash profile, tools installed cleanly through package managers, and a PATH you fully understand entry by entry. You'll finish with a setup you could re-create on a brand-new machine in minutes — which is exactly the final test.

This is the project that pays off every single day afterward: seconds saved per command, multiplied forever.

## Difficulty

**Intermediate** — Estimated effort: 2.5–4 hours

## Chapters used

- Chapter 6: Environment & Configuration (core)
- Chapter 7: Package Managers & Installing Tools (core)
- Chapter 1–2: Orientation & navigation (assumed)
- Chapter 5: Pipeline (for the audit)
- Chapter 8: Scripts (functions in your profile; execution policy)

## Requirements checklist

Keep all deliverables in `D:\atoop\coding-projects\command-line\projects\sandbox\env-makeover\` (profiles live where the shell demands, but keep *copies* here).

**Phase A — Audit (know what you have before changing it):**
- [ ] Produce `audit.md` containing: your PowerShell version, execution policy, whether a profile exists, your PATH one-entry-per-line, and for each PATH entry a one-phrase note of what lives there (or "unknown — investigate")
- [ ] Identify at least one PATH entry that is dead (folder missing or empty) or redundant, if any exist
- [ ] List the 10 commands you run most (mine your shell history — investigate `Get-History` / the PSReadLine history file / Bash `history`) — these decide what deserves an alias

**Phase B — PowerShell profile:**
- [ ] Create (or rebuild) your PowerShell profile, with every line commented
- [ ] At least 3 aliases for simple renames
- [ ] At least 4 functions, including: one navigation shortcut to your projects root, one enhanced listing, one that takes at least one parameter, and one multi-step convenience of your own design
- [ ] A customized prompt (investigate the `prompt` function) showing at minimum the current folder — bonus for git branch awareness
- [ ] Profile reloads cleanly with no errors, and a brand-new terminal starts without red text
- [ ] Execution policy set intentionally, with a comment in `audit.md` saying what you chose and why

**Phase C — Bash parity (Git Bash or WSL):**
- [ ] A `.bashrc` (or additions to it) mirroring your PowerShell shortcuts where they make sense — same muscle memory in both shells
- [ ] At least one Bash function with an argument
- [ ] Reloaded and verified in a fresh Bash session

**Phase D — Tooling via package managers:**
- [ ] Install at least 3 new CLI tools via winget (candidates: `jq`, a fuzzy finder, `ripgrep`, `bat`, a better prompt — research what appeals)
- [ ] For each: record in `tools.md` the winget Id, what it does, and one worked example command you ran successfully
- [ ] Demonstrate the PATH lifecycle for one tool: show it unavailable in the old terminal and available in a new one, and identify exactly which PATH entry provides it
- [ ] Install one npm global CLI and one pip tool (in a venv or via pipx-style isolation), and note where each ended up on PATH

**Phase E — Reproducibility (the real test):**
- [ ] Write `setup-notes.md`: the complete, ordered recipe to rebuild this environment on a fresh Windows machine — every winget Id, every file to copy, every setting to flip
- [ ] Copy your profile files into the sandbox folder as `profile.ps1.bak` and `bashrc.bak` alongside the notes
- [ ] Dry-run the recipe mentally (or in a fresh Windows user account / VM if available) and fix anything missing

## Hints

- The PATH audit is more interesting than it sounds: `Get-Command <tool>` for tools you use tells you which entries earn their keep.
- Chapter 6's pitfall list is effectively this project's safety briefing — especially the append-don't-replace PATH rule and the "alias with arguments needs a function" rule.
- For history mining, PSReadLine keeps a plain-text history file — find its location with `(Get-PSReadLineOption).HistorySavePath`, then Ch. 4/5 tools give you a frequency table of your own habits.
- A prompt function that errors will haunt every keystroke; test prompt changes by pasting into a live session before saving to the profile.
- If a new tool "isn't found" right after install, you already know the two-step diagnosis (Ch. 6/7). Don't skip writing down the answer — that's Phase D's point.
- Keep functions small. If one grows past ~10 lines, it wants to be a script in a folder on your PATH instead (a preview of Project 4).

## Stretch goals

- Put your profile copies under git version control and write a one-command "install my dotfiles" script that copies them into place (idempotently, Ch. 9).
- Add tab-completion or history-search quality-of-life upgrades: investigate PSReadLine options (`Get-PSReadLineOption`) for prediction and history search keybindings.
- Theme Windows Terminal per shell (different color scheme for PowerShell / WSL / ssh sessions) so context is visible at a glance — Ch. 10's wrong-machine pitfall, pre-empted.
- Benchmark yourself: pick 5 frequent tasks, do them with stock commands, then with your new shortcuts, and record the keystroke savings in `audit.md`.
