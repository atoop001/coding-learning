# Chapter 2: Navigating & Inspecting the Filesystem

## Overview

Everything in the terminal happens *somewhere* — every shell session has a **current working directory**, and most commands act relative to it. Before you can create, search, or run anything confidently, you need to move around the filesystem, know where you are at all times, and list what's around you. This chapter builds that spatial awareness: paths, `cd`, listing files, absolute vs. relative addressing, and tab completion (the skill that makes fast typists look like wizards).

## Definitions & Explanations

**Filesystem** — The tree of drives, folders (directories), and files. On Windows the tree has multiple roots, one per drive (`C:\`, `D:\`). On Linux there is exactly one root, `/`, and everything — including other disks — is mounted somewhere under it (e.g., `/mnt/data`).

**Directory vs. folder** — Same thing. "Directory" is the traditional term and what commands use (`mkdir` = *make directory*).

**Working directory** — The directory your shell is currently "standing in." Shown in the prompt. Commands that take file names without a full path look here first.

**Path** — The address of a file or directory.
- **Absolute path**: the full address from a root. Windows: `D:\atoop\coding-projects\notes.txt`. Linux: `/home/atoop/projects/notes.txt`. Absolute paths work no matter where you're standing.
- **Relative path**: an address starting from the working directory. If you're in `D:\atoop`, then `coding-projects\notes.txt` reaches the same file. Relative paths are shorter but only valid from the right starting point.

**Path separators** — Windows traditionally uses backslash `\`; Linux/Bash uses forward slash `/`. Helpfully, PowerShell accepts *both* (`cd D:/atoop` works). Bash accepts only `/`. When in doubt, forward slashes are the more portable habit.

**Special path names** (all shells):
- `.` — the current directory.
- `..` — the parent directory (one level up).
- `~` — your home directory (`C:\Users\atoop` on Windows, `/home/atoop` on Linux). Works in both PowerShell and Bash.

**Home directory** — Your personal area. Shells start here by default. On servers it's usually where you'll keep your work.

**Hidden files** — On Linux, any file starting with a dot (`.bashrc`, `.git`) is hidden from plain `ls`. On Windows, hidden is a file *attribute* instead, but dotfiles are so common in development that they matter on both systems.

**Tab completion** — Press `Tab` after typing part of a name and the shell completes it. PowerShell cycles through matches on repeated presses; Bash completes the common prefix and shows options on a second press. This isn't a luxury: it prevents typos, saves enormous time, and confirms a file exists before you act on it.

## Command Examples

Moving around — PowerShell:

```powershell
Get-Location                       # where am I? (alias: pwd)
# Path
# ----
# C:\Users\atoop

Set-Location D:\atoop\coding-projects    # alias: cd
Get-Location
# D:\atoop\coding-projects

cd ..                              # up one level
# now in D:\atoop

cd ~                               # jump home from anywhere
# now in C:\Users\atoop

cd -                               # PowerShell 7: back to previous location
# now in D:\atoop

# Switching drives is just a path with a drive letter:
cd D:\
```

The same moves in Bash:

```bash
pwd
# /home/atoop

cd projects/website                # relative path
pwd
# /home/atoop/projects/website

cd ..                              # up one level
cd ../..                           # up two levels
cd ~                               # home (or just: cd)
cd -                               # back to previous directory
# /home/atoop/projects/website
```

Listing and inspecting — PowerShell:

```powershell
Get-ChildItem                      # aliases: ls, dir, gci
#     Directory: D:\atoop\coding-projects
# Mode     LastWriteTime      Length Name
# ----     -------------      ------ ----
# d----    2026-07-10 08:15          command-line
# -a---    2026-07-09 17:22     2048 todo.md

Get-ChildItem -Force               # include hidden/system items (like .git)
Get-ChildItem -Name                # names only, compact
# command-line
# todo.md

Get-ChildItem -Directory           # folders only
Get-ChildItem -File                # files only

Get-ChildItem -Recurse             # everything beneath, recursively (can be long!)

# Inspect one item's details:
Get-Item .\todo.md | Format-List Name, Length, LastWriteTime, FullName
# Name          : todo.md
# Length        : 2048
# LastWriteTime : 2026-07-09 17:22:41
# FullName      : D:\atoop\coding-projects\todo.md

# Does a path exist? (invaluable in scripts)
Test-Path D:\atoop\coding-projects
# True
```

Listing and inspecting — Bash:

```bash
ls                                 # plain listing
# projects  todo.md

ls -l                              # long format: permissions, owner, size, date
# -rw-r--r-- 1 atoop atoop 2048 Jul  9 17:22 todo.md
# drwxr-xr-x 3 atoop atoop 4096 Jul 10 08:15 projects

ls -a                              # include hidden dotfiles
# .  ..  .bashrc  projects  todo.md

ls -la                             # both combined — the classic
ls -lh                             # human-readable sizes (2.0K instead of 2048)
ls -R                              # recursive listing

stat todo.md                       # detailed info about one file
# File: todo.md  Size: 2048 ... Modify: 2026-07-09 17:22:41 ...

file todo.md                       # what kind of file is this?
# todo.md: ASCII text

[ -e todo.md ] && echo "exists"    # existence test, script-style
# exists
```

Side-by-side cheat sheet:

```text
Task                    PowerShell                 Bash
----                    ----------                 ----
Where am I?             Get-Location / pwd         pwd
Change directory        Set-Location / cd          cd
List files              Get-ChildItem / ls         ls
List incl. hidden       ls -Force                  ls -a
Long/detailed list      ls | Format-List           ls -l
Recursive list          ls -Recurse                ls -R
Go home                 cd ~                       cd ~  (or cd)
Go up one               cd ..                      cd ..
Previous location       cd -                       cd -
Path exists?            Test-Path <p>              [ -e <p> ]
```

Tab completion drill (do this, don't just read it):

```powershell
# Type this much, then press Tab:
cd D:\atoop\cod<Tab>
# PowerShell completes to: cd D:\atoop\coding-projects\
# Press Tab again if multiple folders match — it cycles through them.

# It also completes command names and parameters:
Get-Chi<Tab>          # -> Get-ChildItem
Get-ChildItem -Rec<Tab>   # -> -Recurse
```

```bash
# Bash: completes as far as unambiguous, then beeps.
cd pro<Tab>           # -> cd projects/
ls ~/pro<Tab>j<Tab>   # press Tab twice on ambiguity to SEE all matches
```

## Common Pitfalls

**Pitfall: running commands in the wrong directory.**
You run a build or delete files, but you're standing somewhere unexpected — the command works on the wrong folder.
*Correction*: glance at your prompt before consequential commands, or run `pwd` explicitly. Make "where am I?" a reflex.

**Pitfall: spaces in paths break commands.**
`cd C:\Users\atoop\My Documents` fails — the shell reads `C:\Users\atoop\My` as the path and `Documents` as a second argument.
*Correction*: quote paths containing spaces: `cd "C:\Users\atoop\My Documents"`. Tab completion quotes automatically — another reason to use it.

**Pitfall: confusing relative and absolute paths in scripts and docs.**
A command using `.\data\input.txt` only works from one specific directory; run it elsewhere and you get "cannot find path."
*Correction*: understand *from where* a relative path resolves. When writing instructions or scripts meant to run from anywhere, prefer absolute paths or explicitly `cd` first.

**Pitfall: Windows backslashes pasted into Bash.**
`cd D:\atoop` in Bash does nothing like what you expect — backslash is Bash's *escape character*. Your WSL path for that folder is actually `/mnt/d/atoop`.
*Correction*: in Bash use forward slashes, and in WSL remember drives live under `/mnt/<letter>/`.

**Pitfall: case sensitivity on Linux.**
`cd Projects` fails on a server when the folder is `projects`. Windows lets you be sloppy; Linux doesn't.
*Correction*: use tab completion — it only completes names that actually exist, with correct casing.

**Pitfall: `ls -Recurse` on a huge tree floods the screen for minutes.**
*Correction*: Ctrl+C cancels. Scope recursion to a subfolder, or add limits (`-Depth 2` in PowerShell).

## Practice Exercises

1. From your home directory, navigate to `D:\atoop\coding-projects` using a single absolute-path `cd`, go back home, then reach it again using only relative `cd` steps (`..`, folder names). Count your keystrokes; then redo it leaning on Tab the whole way.
2. Create a mental map: run a recursive, depth-limited listing of `D:\atoop\coding-projects` (`Get-ChildItem -Recurse -Depth 2`). Sketch the tree on paper from the output — no GUI file explorer allowed.
3. Find your PowerShell profile's folder by running `Split-Path $PROFILE` and navigate there. Does the file exist? Prove it with `Test-Path $PROFILE`.
4. In Git Bash or WSL, list your home directory including hidden files, in long human-readable format, sorted so the newest files appear last (investigate `ls` flags with `man ls` — look at `-t` and `-r`).
5. Without a GUI: figure out the full absolute path of the deepest directory anywhere under `D:\atoop\coding-projects` (the one most levels down). Any listing flags are fair game.
