# Chapter 5: The Pipeline — Pipes, Redirection & Chaining

## Overview

The pipeline is the idea that makes the command line more than a bag of separate tools: **the output of one command becomes the input of the next**. Small, single-purpose commands snap together like LEGO into powerful one-liners — "list the files, keep only the big ones, sort them, show the top five." Add **redirection** (sending output to files instead of the screen) and **chaining** (running commands in sequence conditionally), and you have the core mental model of shell power. This chapter also explains the deepest difference between your two shells: Bash pipes **text**, PowerShell pipes **objects**.

## Definitions & Explanations

**Pipe (`|`)** — Connects the output of the left command to the input of the right command. `A | B | C` runs all three, streaming data left to right. Each stage transforms or filters.

**Text vs. objects — the big divide.**
- *Bash*: every command's output is plain text (lines of characters). Downstream tools (`grep`, `sort`, `awk`, `cut`) parse that text. Flexible, universal — but fragile: if a column moves or a filename contains a space, text parsing breaks.
- *PowerShell*: commands emit **objects** — structured data with named properties. `Get-ChildItem` doesn't output the *text* of a directory listing; it outputs FileInfo objects, each with `.Name`, `.Length`, `.LastWriteTime`, etc. Downstream cmdlets (`Where-Object`, `Sort-Object`, `Select-Object`) work with those properties directly — no parsing, no column counting. The table you *see* is just formatting applied at the very end.

**Core PowerShell pipeline verbs**:
- `Where-Object` (alias `?`) — filter: keep objects meeting a condition.
- `Sort-Object` — sort by a property.
- `Select-Object` (alias `select`) — pick properties or take first/last N.
- `ForEach-Object` (alias `%`) — do something with each object; `$_` is "the current object."
- `Measure-Object` — count/sum/average.

**Core Bash pipeline tools**:
- `grep` — filter lines by pattern.
- `sort` — sort lines (`-n` numeric, `-r` reverse, `-k` by column).
- `uniq` — collapse repeated adjacent lines (`-c` counts them; pair with `sort` first).
- `head` / `tail` — first/last N lines.
- `wc` — count lines/words/bytes.
- `cut` / `awk` — extract columns from lines.
- `xargs` — turn input lines into command arguments.

**Redirection** — Routing output to files:
- `>` — write stdout to a file, **overwriting** it.
- `>>` — append to a file.
- `2>` — redirect stderr (the error stream) — errors are a separate stream from normal output.
- `2>&1` — merge errors into normal output. Bash idiom: `cmd > all.log 2>&1`. PowerShell: `cmd *> all.log` catches every stream.
- `<` (Bash) — feed a file as input.

**Chaining operators** — Run multiple commands in one line:
- `;` — run B after A regardless of success.
- `&&` — run B only if A **succeeded**.
- `||` — run B only if A **failed**.
Bash has had these forever; PowerShell gained `&&`/`||` in version 7 (another reason to use pwsh).

**stdin / stdout / stderr** — The three standard streams every process has: input, output, errors. Pipes connect stdout→stdin. Keeping errors on a separate stream is why you can redirect results to a file while still seeing errors on screen.

## Command Examples

Filtering and sorting — the flagship comparison:

```powershell
# PowerShell: 5 largest files under the current tree
Get-ChildItem -Recurse -File |
    Sort-Object Length -Descending |
    Select-Object -First 5 Name, Length
# Name           Length
# ----           ------
# video.mp4    52428800
# dataset.csv   8388608
# ...
# No text parsing anywhere: Length is a real number the whole way through.

# Files over 1 MB, newest first
Get-ChildItem -Recurse -File |
    Where-Object Length -gt 1MB |
    Sort-Object LastWriteTime -Descending

# Where-Object with a full condition block ($_ = current object):
Get-ChildItem -File | Where-Object { $_.Name -like "*.log" -and $_.Length -gt 10KB }

# Count matching items
(Get-ChildItem -Recurse -Filter *.md | Measure-Object).Count
# 17

# ForEach-Object: act on each item
Get-ChildItem *.txt | ForEach-Object { "$($_.Name) is $($_.Length) bytes" }
# notes.txt is 2048 bytes
# todo.txt is 512 bytes
```

```bash
# Bash: 5 largest files under the current tree (text all the way)
find . -type f -exec du -b {} + | sort -nr | head -n 5
# 52428800  ./video.mp4
# 8388608   ./dataset.csv
# ...

# ls -l text piped through parsing tools:
ls -l | grep "^-" | sort -k5 -nr | head -n 3   # sort by column 5 (size)

# Classic log crunching: top repeated ERROR messages
grep "ERROR" app.log | sort | uniq -c | sort -nr
#   14 ERROR Connection timeout after 30s
#    3 ERROR Disk full

# Count files per extension
ls | awk -F. '{print $NF}' | sort | uniq -c | sort -nr
#   12 md
#    5 py
#    2 txt
```

Redirection:

```powershell
# PowerShell
Get-ChildItem -Recurse -Name > filelist.txt      # overwrite file with listing
"one more line" >> filelist.txt                  # append
Get-Content missing.txt 2> errors.txt            # errors to a file
some-build-command *> build-full.log             # ALL streams to one file

# Out-File gives encoding control; Tee-Object shows AND saves:
Get-ChildItem | Tee-Object listing.txt           # screen + file simultaneously
```

```bash
# Bash
ls -R > filelist.txt              # overwrite
echo "one more line" >> filelist.txt   # append
cat missing.txt 2> errors.txt     # stderr to file
make > build.log 2>&1             # stdout AND stderr to one file (order matters!)
ls | tee listing.txt              # screen + file simultaneously
sort < unsorted.txt               # file as stdin
```

Chaining:

```powershell
# PowerShell 7+
mkdir build && cd build                    # cd only if mkdir succeeded
npm test && git commit -m "green tests"    # commit only if tests pass
Test-Path data.csv || Write-Output "missing data file"   # runs only on failure
git pull; npm install; npm test            # all three, regardless of failures
```

```bash
mkdir build && cd build
npm test && git commit -m "green tests"
[ -f data.csv ] || echo "missing data file"
git pull; npm install; npm test
```

Seeing the object model (PowerShell only — this is the "aha" demo):

```powershell
Get-ChildItem app.log | Get-Member          # what IS this thing?
#    TypeName: System.IO.FileInfo
# Name            MemberType   Definition
# ----            ----------   ----------
# Delete          Method       void Delete()
# Length          Property     long Length {get;}
# LastWriteTime   Property     datetime LastWriteTime {get;set;}
# ... dozens of properties you can filter/sort on directly

(Get-Item app.log).LastWriteTime.DayOfWeek   # drill into properties
# Tuesday
```

## Common Pitfalls

**Pitfall: `>` silently destroys files.**
`results > important.txt` — if you meant to append, the original content is already gone. In Bash, even `> file` alone (no command) truncates the file to zero bytes.
*Correction*: think "clobber" every time you type a single `>`. Use `>>` for append. Treat `>` toward any file you didn't just create as a stop-and-check moment.

**Pitfall: parsing `ls` output in Bash scripts.**
`for f in $(ls)` breaks on filenames with spaces and is a famously buggy pattern.
*Correction*: use globs (`for f in *.txt`) or `find ... -exec`/`while read` patterns. This fragility is exactly what PowerShell's object pipeline eliminates — one reason to appreciate both models.

**Pitfall: expecting text tools to work on PowerShell objects (and vice versa).**
`Get-ChildItem | grep foo` may "work" oddly (objects get stringified through formatting), and `findstr`-style habits miss properties entirely. Conversely, expecting `Where-Object Length -gt 1MB` semantics from Bash's `grep` disappoints.
*Correction*: in PowerShell stay object-native (`Where-Object`, `Select-String`); convert to text only at the end. In Bash, embrace text tools fully.

**Pitfall: `2>&1` in the wrong position (Bash).**
`cmd 2>&1 > file` sends errors to the *screen*, not the file — redirections apply left to right.
*Correction*: the idiom is `cmd > file 2>&1` (or in modern Bash, `cmd &> file`). Memorize it as a unit.

**Pitfall: overusing `;` where `&&` is meant.**
`build; deploy` deploys even when the build failed.
*Correction*: use `&&` whenever step B only makes sense after step A succeeded. Reserve `;` for genuinely independent steps.

**Pitfall: formatting cmdlets in mid-pipeline.**
`Get-ChildItem | Format-Table | Sort-Object Length` fails — `Format-*` cmdlets emit formatting instructions, not your objects, so nothing downstream can use them.
*Correction*: `Format-Table`/`Format-List` must be the *last* stage, used only for display.

## Practice Exercises

1. In PowerShell, produce a list of the 10 most recently modified files anywhere under `D:\atoop\coding-projects`, showing only name, length, and timestamp — then write that list to `recent.txt` while *also* displaying it (one pipeline, no temporary steps).
2. Reusing your Chapter 4 `app.log` (make it longer: duplicate lines, add more ERROR variants), build a pipeline in each shell that outputs each distinct ERROR message with its occurrence count, most frequent first. (Bash path: `grep`/`sort`/`uniq -c`. PowerShell path: `Select-String`/`Group-Object` — investigate `Group-Object` with `Get-Help`.)
3. Write a one-liner that counts how many `.md` files exist under your projects folder — once in PowerShell (objects), once in Bash (`find` piped to `wc -l`). Confirm both agree.
4. Demonstrate stream separation: run a command that produces both normal output and an error (e.g., `Get-Content` / `cat` on one real file and one missing file in the same call). Send output to `out.txt` and errors to `err.txt`. Show the contents of each to prove the split worked.
5. Build a chained "mini CI" line: create a folder, enter it, create a file, and print DONE — using `&&` so any failure stops the chain. Then intentionally sabotage step one (make the folder already exist, or make it invalid) and observe what runs and what doesn't. Explain why in a comment saved to your notes file.
