# Chapter 4: Viewing & Searching File Contents

## Overview

Half of terminal life is *finding things*: reading a config file without opening an editor, hunting a function name across a codebase, finding which log file mentions an error, locating that file you know exists *somewhere*. GUI search is slow and limited; command-line search is instant, precise, and scriptable. This chapter covers reading files (`cat` / `Get-Content`), searching *inside* files (`grep` / `Select-String`), finding files *by name or property* (`find` / `Get-ChildItem`), and the wildcard and regex patterns that power all of them.

## Definitions & Explanations

**Viewing vs. paging** — `cat`/`Get-Content` dump a whole file to the screen — fine for short files, useless for a 10,000-line log. A **pager** (`less` in Bash, `more`/`Out-Host -Paging` in PowerShell) shows one screen at a time with scrolling and internal search.

**Head and tail** — Often you only need the start (headers, first error) or the end (latest log entries) of a file. Bash has dedicated `head`/`tail`; PowerShell uses `-TotalCount` (first N) and `-Tail` (last N) on `Get-Content`.

**Following a file** — `tail -f` (Bash) and `Get-Content -Wait` (PowerShell) keep the file open and print new lines as they're written — the standard way to watch a live log.

**Searching content: grep / Select-String** — `grep` ("global regular expression print") scans text for lines matching a pattern; it's arguably the most-used tool on Linux. PowerShell's counterpart is `Select-String` (alias `sls`), which returns *match objects* carrying the filename, line number, and line text as properties.

**Finding files: find / Get-ChildItem** — `find` (Bash) walks a directory tree applying tests: name patterns, size, modification time, type. PowerShell reuses `Get-ChildItem -Recurse` with filters (`-Filter`, `-Include`) and pipes to `Where-Object` for richer conditions (Chapter 5 explains piping fully).

**Wildcards (globbing)** — Filesystem patterns: `*` = any characters, `?` = one character, `[abc]` = one of these characters. Used for *file names*: `*.md`, `report-202?.txt`, `data[12].csv`.

**Regular expressions (regex)** — A richer pattern language for *text content*. Key differences from wildcards: `.` means "any one character", `.*` means "anything", `^` anchors to line start, `$` to line end, `\d` is a digit, `+` means "one or more of the previous". `grep` and `Select-String` both speak regex. You don't need regex mastery yet — but recognize that `grep "error.*timeout"` is regex, not a wildcard.

**Case sensitivity in search** — `grep` is case-sensitive by default (`-i` to ignore case). `Select-String` is case-*insensitive* by default (`-CaseSensitive` to make it strict). Opposite defaults — a classic trip-up when switching shells.

## Command Examples

First, generate a practice file so outputs match:

```powershell
# PowerShell — create a fake log
@"
2026-07-15 09:00:01 INFO  Service started
2026-07-15 09:00:05 INFO  Connected to database
2026-07-15 09:12:44 WARN  Slow query: 2103ms
2026-07-15 09:13:02 ERROR Connection timeout after 30s
2026-07-15 09:13:10 INFO  Retrying connection
2026-07-15 09:13:41 ERROR Connection timeout after 30s
2026-07-15 09:14:00 INFO  Connected to database
"@ | Set-Content app.log
```

Viewing files:

```powershell
# PowerShell
Get-Content app.log                 # aliases: cat, gc, type — whole file
Get-Content app.log -TotalCount 3   # first 3 lines
# 2026-07-15 09:00:01 INFO  Service started
# 2026-07-15 09:00:05 INFO  Connected to database
# 2026-07-15 09:12:44 WARN  Slow query: 2103ms

Get-Content app.log -Tail 2         # last 2 lines
# 2026-07-15 09:13:41 ERROR Connection timeout after 30s
# 2026-07-15 09:14:00 INFO  Connected to database

Get-Content app.log -Wait           # live-follow; Ctrl+C to stop

(Get-Content app.log).Count         # how many lines?
# 7
```

```bash
# Bash
cat app.log                         # whole file
head -n 3 app.log                   # first 3 lines
tail -n 2 app.log                   # last 2 lines
tail -f app.log                     # follow live; Ctrl+C to stop
wc -l app.log                       # line count
# 7 app.log

less app.log                        # pager: arrows/PgDn scroll, /text searches, q quits
```

Searching inside files:

```powershell
# PowerShell
Select-String "ERROR" app.log       # alias: sls
# app.log:4:2026-07-15 09:13:02 ERROR Connection timeout after 30s
# app.log:6:2026-07-15 09:13:41 ERROR Connection timeout after 30s
#        ^ filename : line number : line text

Select-String "error" app.log                    # still matches! case-insensitive default
Select-String "error" app.log -CaseSensitive     # no matches now

Select-String "timeout" *.log                    # search every .log in the folder
Select-String "TODO" -Path .\src\* -Recurse:$false  # multiple files via wildcard

# Search a whole tree: recurse for files, pipe them in (full pipe story in Ch.5)
Get-ChildItem -Recurse -Filter *.py | Select-String "def main"

# Count matches / invert:
(Select-String "ERROR" app.log).Count            # 2
Select-String "INFO" app.log -NotMatch           # lines WITHOUT "INFO"

# Regex example: lines whose message starts with "Conn"
Select-String "^\S+ \S+ \w+\s+Conn" app.log
```

```bash
# Bash
grep "ERROR" app.log
# 2026-07-15 09:13:02 ERROR Connection timeout after 30s
# 2026-07-15 09:13:41 ERROR Connection timeout after 30s

grep -i "error" app.log             # -i: ignore case
grep -n "timeout" app.log           # -n: show line numbers
# 4:2026-07-15 09:13:02 ERROR Connection timeout after 30s
# 6:2026-07-15 09:13:41 ERROR Connection timeout after 30s

grep -c "ERROR" app.log             # -c: count matching lines
# 2
grep -v "INFO" app.log              # -v: invert (lines NOT matching)
grep -r "def main" ~/projects       # -r: recursive through a directory tree
grep -rn --include="*.py" "import" .   # recursive, only .py files, with line numbers
grep -E "timeout|refused" app.log   # -E: extended regex, | means OR
grep -A 2 "ERROR" app.log           # show 2 lines After each match (-B before, -C both)
```

Finding files by name and properties:

```powershell
# PowerShell
Get-ChildItem -Recurse -Filter *.md            # all markdown files beneath here
Get-ChildItem -Recurse -Include *.jpg,*.png -File   # multiple patterns
Get-ChildItem -Recurse -Directory -Filter "test*"   # folders starting with test

# Property-based finds use Where-Object (preview of Chapter 5):
Get-ChildItem -Recurse -File | Where-Object Length -gt 1MB        # files over 1 MB
Get-ChildItem -Recurse -File |
    Where-Object LastWriteTime -gt (Get-Date).AddDays(-7)          # changed this week
```

```bash
# Bash
find . -name "*.md"                 # by name pattern (quote it!)
find . -iname "readme*"             # case-insensitive name
find . -type d -name "test*"        # directories only
find . -type f -size +1M            # files over 1 MB
find . -type f -mtime -7            # modified in last 7 days
find . -name "*.tmp" -delete        # find AND delete (preview without -delete first!)
```

## Common Pitfalls

**Pitfall: unquoted wildcards in Bash `find`.**
`find . -name *.md` — Bash expands `*.md` *before* find runs, using files in the current directory, giving wrong results or errors.
*Correction*: always quote the pattern: `find . -name "*.md"`. The quotes deliver the pattern to `find` intact.

**Pitfall: opposite case-sensitivity defaults.**
You `grep error` on a server and miss every `ERROR`; you assume `Select-String` is case-sensitive and add redundant patterns.
*Correction*: memorize the pair — grep: sensitive (add `-i` to relax); Select-String: insensitive (add `-CaseSensitive` to tighten).

**Pitfall: treating regex like wildcards.**
`grep "*.txt" file` doesn't mean "ends in .txt" — in regex, `*` modifies the previous character. Similarly `.` matches *any* character, so `grep "1.5"` also matches `125`.
*Correction*: for a literal ".", escape it (`\.`); for "any text", use `.*`; to search for a literal string with no regex meaning at all, use `grep -F` or `Select-String -SimpleMatch`.

**Pitfall: dumping huge files with cat.**
`cat massive.log` scrolls for minutes and buries your history.
*Correction*: reach for `less`, `head`/`tail`, or `-TotalCount`/`-Tail` first. Ask "what part do I actually need?"

**Pitfall: searching binary files.**
`grep` on a directory full of images prints `Binary file ... matches` noise, or garbage characters that can scramble your terminal display.
*Correction*: restrict by extension (`--include="*.log"`, `-Filter *.log`); if the display scrambles, run `clear` (or `reset` in Bash).

**Pitfall: forgetting Select-String returns objects, not text.**
You pipe `Select-String` output to a file and get odd formatting, or wonder how to get *just* the line text.
*Correction*: use the properties: `(Select-String "ERROR" app.log).Line` gives clean text; `.LineNumber`, `.Filename` are also available. This object model becomes a superpower in Chapter 5.

## Practice Exercises

1. Recreate the `app.log` file from this chapter (or write a longer one by hand). Using one command per question, find: (a) all ERROR lines with line numbers, (b) how many lines are *not* INFO, (c) the last 3 lines, (d) every line where a number of milliseconds appears (regex: digits followed by `ms`).
2. Search your entire `D:\atoop\coding-projects` tree for files containing the word `TODO` (any case), limiting the search to `.md`, `.py`, `.js`, and `.ps1` files. Do it in PowerShell; if you have Git Bash or WSL, repeat in Bash and compare the command shapes.
3. Find the 5 largest files under your Downloads folder using the terminal only. (PowerShell hint: `Sort-Object` and `Select-Object -First` exist — experiment; full treatment in Chapter 5.)
4. Find every file under `coding-projects` modified in the last 48 hours. Then narrow it: only files, only `.md` extension.
5. Open a long file (any big log or a book-length text) in `less` (Bash) and practice: jump to the end (`G`), back to the top (`g`), search forward for a word (`/word`, then `n` for next match), and quit (`q`). No mouse allowed.
