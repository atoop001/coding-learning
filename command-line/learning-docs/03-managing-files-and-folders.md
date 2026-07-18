# Chapter 3: Creating, Copying, Moving & Deleting — Safely

## Overview

Now that you can navigate, it's time to change things: make folders, create files, copy, rename, move, and delete. These are the commands you'll use dozens of times a day. They're also where beginners do real damage — the terminal has **no Recycle Bin by default** and **no "Are you sure?" dialog** for most operations. `rm -rf` horror stories are a rite of passage precisely because deletion is instant and silent. This chapter teaches the operations *and* the safety habits that make disasters nearly impossible.

## Definitions & Explanations

**Creating** — `New-Item` (PowerShell) creates files or directories depending on `-ItemType`. Bash splits this: `mkdir` makes directories, `touch` creates empty files (technically it updates timestamps, creating the file if missing).

**Copying** — `Copy-Item` / `cp` duplicates files. Copying a directory requires asking for recursion (`-Recurse` / `-r`) so the shell knows you mean the folder *and its contents*.

**Moving vs. renaming** — They are the same operation to the filesystem: changing a file's path. `Move-Item` / `mv` does both. PowerShell also has a dedicated `Rename-Item` which only changes the name (can't relocate) — slightly safer when renaming.

**Deleting** — `Remove-Item` / `rm` removes files; with recursion flags it removes directories and everything inside. Deleted means *gone* — not in the Recycle Bin. (File-recovery tools exist but are a last resort, not a plan.)

**Overwriting** — Copy and move will happily replace an existing destination file, usually without asking. The overwritten content is unrecoverable.

**`-WhatIf` (PowerShell)** — The single best safety feature in this chapter. Add `-WhatIf` to almost any state-changing cmdlet and PowerShell *describes* what it would do without doing it. Bash has no universal equivalent — the closest habit is running `echo`/`ls` with the same pattern first to preview what will match.

**`-Confirm` / `-i`** — Ask before acting. `Remove-Item -Confirm` prompts per item; Bash `rm -i` does the same. `cp -i` / `mv -i` prompt before overwriting.

**Wildcards (globs)** — Patterns that match multiple names: `*` matches any run of characters, `?` matches exactly one. `*.txt` = all files ending in `.txt`. Both shells expand these before the command runs — which is why a mistyped wildcard can select far more than you intended (see Pitfalls).

**Trailing structure flags** — `mkdir` in PowerShell creates parent folders automatically. Bash needs `mkdir -p a/b/c` to create the whole chain (`-p` = parents).

## Command Examples

Set up a sandbox to practice in — everything in this chapter should happen inside a scratch folder so mistakes are free:

```powershell
# PowerShell
New-Item -ItemType Directory -Path D:\atoop\coding-projects\sandbox
Set-Location D:\atoop\coding-projects\sandbox

# mkdir is an alias that creates directories, parents included:
mkdir demo\src, demo\docs        # two folders at once (note comma)

New-Item -ItemType File demo\src\main.py       # create empty file
New-Item demo\notes.txt -Value "first line"    # create file WITH content
Get-ChildItem -Recurse -Name
# demo
# demo\docs
# demo\src
# demo\notes.txt
# demo\src\main.py
```

```bash
# Bash
mkdir -p ~/sandbox/demo/src ~/sandbox/demo/docs   # -p makes parents
cd ~/sandbox

touch demo/src/main.py            # create empty file
echo "first line" > demo/notes.txt   # create file with content (redirection, Ch.5)
ls -R demo
# demo:
# docs  notes.txt  src
# demo/src:
# main.py
```

Copying:

```powershell
# PowerShell
Copy-Item demo\notes.txt demo\notes-backup.txt          # copy + new name
Copy-Item demo\src\main.py demo\docs\                   # copy into folder (keep name)
Copy-Item demo demo-copy -Recurse                       # copy a whole tree
Copy-Item demo\*.txt D:\atoop\coding-projects\sandbox\  # wildcard copy

# Preview before a risky copy over existing files:
Copy-Item demo\notes.txt demo\notes-backup.txt -WhatIf
# What if: Performing the operation "Copy File" on target
# "Item: ...\notes.txt Destination: ...\notes-backup.txt".
```

```bash
# Bash
cp demo/notes.txt demo/notes-backup.txt
cp demo/src/main.py demo/docs/
cp -r demo demo-copy                 # -r required for directories
cp demo/*.txt ~/sandbox/             # wildcard copy
cp -i demo/notes.txt demo/notes-backup.txt   # -i: ask before overwrite
# cp: overwrite 'demo/notes-backup.txt'? y
```

Moving and renaming:

```powershell
# PowerShell
Move-Item demo\notes-backup.txt demo\docs\        # move into folder
Rename-Item demo\notes.txt README.txt             # rename in place
Move-Item demo\docs\notes-backup.txt demo\docs\old-notes.txt  # move+rename at once
```

```bash
# Bash — mv does all three jobs
mv demo/notes-backup.txt demo/docs/               # move
mv demo/notes.txt demo/README.txt                 # rename
mv demo/docs/notes-backup.txt demo/docs/old.txt   # move + rename
mv -i important.txt demo/                         # prompt if it would overwrite
```

Deleting — read this section twice:

```powershell
# PowerShell
Remove-Item demo\docs\old-notes.txt               # delete one file. Gone. No bin.

# ALWAYS preview recursive/wildcard deletes first:
Remove-Item demo-copy -Recurse -WhatIf
# What if: Performing the operation "Remove Directory" on target "...\demo-copy".
Remove-Item demo-copy -Recurse                    # now for real

# Delete by wildcard — preview by listing the SAME pattern first:
Get-ChildItem *.txt                               # <- what will match?
Remove-Item *.txt                                 # <- then delete

# -Force is needed for hidden/read-only items; treat it as a red flag to slow down
Remove-Item .hidden-thing -Force
```

```bash
# Bash
rm demo/docs/old.txt                # delete one file. Instant. Permanent.
rmdir empty-folder                  # delete an EMPTY directory only
rm -r demo-copy                     # delete folder recursively
rm -ri demo-copy                    # recursive but confirm each item
rm -rf demo-copy                    # recursive + force, NO prompts. Danger tier.

# The preview habit: run ls with the exact same pattern before rm
ls *.log                            # see what matches...
rm *.log                            # ...then remove exactly that
```

Side-by-side summary:

```text
Task                    PowerShell                        Bash
----                    ----------                        ----
New directory           mkdir name                        mkdir -p name
New empty file          New-Item -ItemType File f         touch f
Copy file               Copy-Item src dst                 cp src dst
Copy folder             Copy-Item src dst -Recurse        cp -r src dst
Move / rename           Move-Item / Rename-Item           mv
Delete file             Remove-Item f                     rm f
Delete folder + all     Remove-Item d -Recurse            rm -r d
Dry run                 add -WhatIf                       ls the pattern first
Ask first               add -Confirm                      rm -i / cp -i / mv -i
```

## Common Pitfalls

**Pitfall: the classic `rm` disaster — a space in a wildcard.**
`rm -rf ~/sandbox /demo` (accidental space) deletes `~/sandbox` *and* tries to delete `/demo`. Worse: `rm -rf $DIR/` when `$DIR` is empty becomes `rm -rf /`. People have erased servers this way.
*Correction*: never type `rm -rf` casually. Preview with `ls` first, avoid trailing slashes after variables, quote variables (`rm -rf "$DIR"`), and in scripts verify the variable is non-empty before deleting. In PowerShell, use `-WhatIf` as a dress rehearsal for every recursive delete.

**Pitfall: deleting or overwriting with no undo.**
Terminal deletes skip the Recycle Bin; `cp`/`mv`/`Copy-Item` silently replace existing destination files.
*Correction*: before bulk operations in an important folder, make a quick copy of the folder itself (`Copy-Item proj proj-bak -Recurse`). Use `-i` flags in Bash. Slow down whenever a command combines *recursion*, *force*, and a *wildcard* — that trio is where disasters live.

**Pitfall: wildcard matches more than you think.**
`rm *` in the wrong directory. Or `Remove-Item *.tmp` when a folder is *named* `important.tmp` — with `-Recurse` present, that folder and its contents go too.
*Correction*: list the pattern first (`ls *.tmp`), read the output, then delete. This two-step habit costs 3 seconds and prevents 90% of accidents.

**Pitfall: `mv`/`Move-Item` into a non-existent folder renames instead of moving.**
`mv report.txt archive/` when `archive/` doesn't exist creates a *file* named `archive` (Bash without trailing slash) or errors confusingly.
*Correction*: create the destination folder first, or `Test-Path archive` / `ls -d archive/` to confirm it exists.

**Pitfall: PowerShell's `rm` is not Bash's `rm`.**
In PowerShell, `rm` is an alias for `Remove-Item` — flags differ! `rm -rf x` in PowerShell fails (`-rf` isn't a parameter); the equivalent is `Remove-Item x -Recurse -Force`.
*Correction*: know which shell you're in; when a flag errors, run `Get-Help Remove-Item` rather than guessing.

**Pitfall: "directory not empty" errors.**
`rmdir` (Bash) and `Remove-Item` without `-Recurse` refuse to delete non-empty folders. This is a *safety feature*, not a bug.
*Correction*: if you truly mean to delete the contents too, say so explicitly with `-Recurse` / `-r` — after previewing.

## Practice Exercises

Work entirely inside a fresh `sandbox` folder. If a command scares you, that's the point — practice the preview-first habit.

1. Build this tree using only the terminal, in as few commands as you can: `practice/` containing `src/`, `docs/`, `archive/`, with empty files `src/app.py`, `src/util.py`, `docs/readme.md`. Verify with a recursive listing.
2. Copy the entire `practice` tree to `practice-backup`, then rename `practice-backup` to `practice-2026-07`. Confirm both trees exist and match.
3. Create five throwaway files: `temp1.log` through `temp5.log` plus one file named `keep.log`. Delete *only* the `temp*` files using one wildcard command — but run the preview step first and make sure `keep.log` survives.
4. In PowerShell, run a recursive `Remove-Item` on your dated backup folder with `-WhatIf`, read every line of the output, then run it for real. Repeat the equivalent in Bash using `rm -ri` and answer the prompts.
5. Reflection drill: write down (in a text file created from the terminal) the three warning signs that a delete command deserves extra scrutiny, and the preview technique you'd use in each shell. Keep this file — it becomes part of your notes folder for later chapters.
