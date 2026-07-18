# Chapter 8: Writing Scripts — PowerShell .ps1 & Bash .sh

## Overview

Everything you've typed so far vanishes when the window closes. A **script** is those same commands saved in a file — repeatable, shareable, versionable, schedulable. Scripting is where terminal knowledge compounds: a fiddly 10-step task becomes one command you (or a scheduler, Chapter 9) can run forever. This chapter teaches the fundamentals in both dialects: files and execution (including PowerShell's infamous **execution policy**), variables, arguments, conditionals, and loops. Same concepts, two syntaxes — learning them side by side makes both stick.

## Definitions & Explanations

**Script** — A plain-text file of shell commands executed top to bottom. PowerShell scripts end in `.ps1`; Bash scripts in `.sh` (the extension is convention — Linux cares about the *shebang* and execute permission instead).

**Execution policy (PowerShell/Windows only)** — A safety gate controlling whether `.ps1` files may run. Fresh Windows machines often ship with `Restricted` (no scripts at all), producing the error every PowerShell user eventually meets: *"...cannot be loaded because running scripts is disabled on this system."* The standard developer setting is `RemoteSigned`: local scripts run freely; internet-downloaded scripts must be signed (or unblocked). This is *not* a security boundary against determined attackers — it's a seatbelt against accidental execution.

**Shebang (`#!`) (Bash)** — The first line of a Linux script, e.g. `#!/usr/bin/env bash`, telling the OS which interpreter runs the file. Paired with the **execute permission bit**: `chmod +x script.sh` marks the file runnable (full permissions story in Chapter 10).

**Variables** — Named storage. PowerShell: `$name = "value"` (spaces around `=` are fine). Bash: `name="value"` (**no spaces around `=`** — `name = "value"` is an error), read back with `$name`. Quote Bash variables when using them (`"$name"`) to survive spaces.

**Arguments/parameters** — Values passed to a script at run time. PowerShell's first-class way is a `param(...)` block with named, typed, defaulted parameters. Bash uses positional variables: `$1`, `$2`, ... plus `$#` (count), `$@` (all args), `$0` (script name).

**Exit codes** — Every command finishes with a number: `0` = success, anything else = failure. This is what `&&`/`||` (Chapter 5) test. Read the last code with `$LASTEXITCODE` (PowerShell, for external programs; `$?` gives `$true/$false`) or `$?` (Bash). Scripts should `exit 1` (or similar) on failure so callers can react.

**Conditionals** — `if`/`elseif`/`else`. PowerShell comparisons use operator words: `-eq -ne -gt -lt -ge -le -like -match`. Bash tests use brackets: `[ "$x" = "yes" ]`, `[ -f file ]` (file exists), `[ $n -gt 5 ]` — the spaces inside `[ ]` are mandatory.

**Loops** — `foreach`/`for`/`while` repeat work over collections or until a condition. The bread and butter of automation: "for every file matching X, do Y."

**Comments** — `#` starts a comment in both languages. Comment the *why*, not the obvious what.

## Command Examples

Hello, script — PowerShell:

```powershell
# Create hello.ps1 with these contents (use any editor, e.g. notepad hello.ps1):
# ----------------------------------------
# My first script
Write-Output "Hello from a script!"
Write-Output "Today is $(Get-Date -Format 'yyyy-MM-dd')"
# ----------------------------------------

.\hello.ps1                          # run it — note the .\ prefix is REQUIRED
# Hello from a script!
# Today is 2026-07-18

# If instead you see: "running scripts is disabled on this system":
Get-ExecutionPolicy                  # probably: Restricted
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned   # one-time fix, no admin needed
.\hello.ps1                          # now it runs
```

Hello, script — Bash:

```bash
# Create hello.sh:
# ----------------------------------------
#!/usr/bin/env bash
echo "Hello from a script!"
echo "Today is $(date +%F)"
# ----------------------------------------

bash hello.sh                        # run via interpreter — works immediately
chmod +x hello.sh                    # OR mark executable...
./hello.sh                           # ...and run directly (./ prefix required)
# Hello from a script!
# Today is 2026-07-18
```

Variables and arguments:

```powershell
# greet.ps1 — PowerShell parameters: named, defaulted, typed
param(
    [string]$Name = "world",
    [int]$Times = 1
)

for ($i = 1; $i -le $Times; $i++) {
    Write-Output "Hello, $Name! (greeting $i of $Times)"
}
```

```powershell
.\greet.ps1                          # Hello, world! (greeting 1 of 1)
.\greet.ps1 -Name Ada -Times 2
# Hello, Ada! (greeting 1 of 2)
# Hello, Ada! (greeting 2 of 2)
```

```bash
#!/usr/bin/env bash
# greet.sh — Bash positional arguments
name="${1:-world}"        # $1, defaulting to "world" if absent
times="${2:-1}"

for ((i=1; i<=times; i++)); do
    echo "Hello, $name! (greeting $i of $times)"
done
```

```bash
./greet.sh Ada 2
# Hello, Ada! (greeting 1 of 2)
# Hello, Ada! (greeting 2 of 2)
```

Conditionals and exit codes — a file-checker in both dialects:

```powershell
# check.ps1
param([Parameter(Mandatory)][string]$Path)   # Mandatory: prompts if omitted

if (Test-Path $Path -PathType Leaf) {
    $size = (Get-Item $Path).Length
    if ($size -gt 1MB) {
        Write-Output "$Path exists and is large ($size bytes)"
    } else {
        Write-Output "$Path exists ($size bytes)"
    }
    exit 0
} elseif (Test-Path $Path -PathType Container) {
    Write-Output "$Path is a directory, not a file"
    exit 1
} else {
    Write-Output "$Path does not exist"
    exit 2
}
```

```bash
#!/usr/bin/env bash
# check.sh
path="$1"

if [ -z "$path" ]; then              # -z: string is empty
    echo "Usage: $0 <path>" >&2      # errors go to stderr
    exit 64
fi

if [ -f "$path" ]; then              # -f: regular file exists
    size=$(stat -c %s "$path")
    if [ "$size" -gt 1048576 ]; then
        echo "$path exists and is large ($size bytes)"
    else
        echo "$path exists ($size bytes)"
    fi
elif [ -d "$path" ]; then            # -d: directory exists
    echo "$path is a directory, not a file"
    exit 1
else
    echo "$path does not exist"
    exit 2
fi
```

Loops over files — the automation workhorse:

```powershell
# For every .log file here: report name and line count
foreach ($f in Get-ChildItem -Filter *.log) {
    $lines = (Get-Content $f).Count
    Write-Output "$($f.Name): $lines lines"
}

# The pipeline flavor of the same idea:
Get-ChildItem -Filter *.log |
    ForEach-Object { "$($_.Name): $((Get-Content $_).Count) lines" }

# while loop:
$n = 3
while ($n -gt 0) { Write-Output "T-minus $n"; $n-- }
```

```bash
# For every .log file here: report name and line count
for f in *.log; do
    [ -e "$f" ] || continue          # guard: pattern matched nothing
    echo "$f: $(wc -l < "$f") lines"
done

# while loop:
n=3
while [ "$n" -gt 0 ]; do echo "T-minus $n"; n=$((n-1)); done

# read a file line by line (the safe idiom):
while IFS= read -r line; do
    echo "got: $line"
done < input.txt
```

## Common Pitfalls

**Pitfall: the execution policy error, met at the worst time.**
`.\deploy.ps1 : File ... cannot be loaded because running scripts is disabled on this system.`
*Correction*: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` — once, no admin required, scoped to you. For a file downloaded from the internet that still complains, `Unblock-File .\script.ps1` removes the "from the web" mark. Do *not* reflexively use `-Scope LocalMachine Bypass`; keep the seatbelt sensible.

**Pitfall: "why won't `hello.ps1` / `hello.sh` run by name?"**
Typing `hello.ps1` gives "not recognized"; `hello.sh` gives "command not found" — because the current directory is *not* on PATH (deliberately, for security, in both ecosystems).
*Correction*: prefix with `.\` (PowerShell) or `./` (Bash). For Bash also ensure `chmod +x` was done — "Permission denied" means the execute bit is missing.

**Pitfall: spaces around `=` in Bash.**
`count = 5` → `count: command not found`. Bash parsed `count` as a program name.
*Correction*: `count=5`, no spaces. (PowerShell is forgiving here; Bash never is.)

**Pitfall: unquoted variables in Bash.**
`rm $file` where `file="My Documents.txt"` deletes `My` and `Documents.txt` — two wrong targets. Unquoted variables split on spaces.
*Correction*: quote nearly every expansion: `rm "$file"`. This habit plus Chapter 3's preview habit prevents most script disasters.

**Pitfall: Windows line endings break Bash scripts.**
A script edited on Windows may carry CRLF endings; Bash sees `\r` at each line end: `$'\r': command not found` or the cryptic `/usr/bin/env: 'bash\r': No such file or directory`.
*Correction*: convert endings (`dos2unix script.sh`, or set your editor to LF for `.sh` files — VS Code shows CRLF/LF in the status bar). Add `.gitattributes` rules (`*.sh text eol=lf`) on shared projects.

**Pitfall: comparing with `=` or `==` in PowerShell conditionals.**
`if ($x == 5)` is a syntax error; `if ($x = 5)` *assigns* 5 (and is always truthy) — a silent logic bug.
*Correction*: PowerShell comparisons are `-eq -ne -gt -lt -ge -le`. Bash arithmetic uses `-eq` in `[ ]` too, but string equality is `=`. Keep a cheat card until it's muscle memory.

**Pitfall: scripts that assume the current directory.**
A script using relative paths works from its own folder and corrupts data when run from elsewhere (or from a scheduler — Chapter 9).
*Correction*: PowerShell: build paths from `$PSScriptRoot` (the script's own folder). Bash: `script_dir="$(cd "$(dirname "$0")" && pwd)"`. Never trust the caller's working directory for anything important.

## Practice Exercises

1. Write `sysinfo.ps1`: prints your username, computer name, PowerShell version, current directory, and free space on `C:` — labeled, one per line. Port it to `sysinfo.sh` (username, hostname, Bash version, `pwd`, free space via `df -h /`). Run both.
2. Write `countdown.ps1` taking `-From <int>` (default 5): counts down to zero, one number per second (`Start-Sleep 1`), then prints "Liftoff!". Bash port takes `$1` with a default (`sleep 1`). Bonus: reject negative numbers with a message on stderr and a nonzero exit code.
3. Write `wordhunt.ps1` taking a `-Pattern` and a `-Folder`: reports each file under the folder containing the pattern, with match counts, and finishes with a total. Exit code 0 if matches were found, 1 if not. Prove the exit code works using `&&` / `||` from Chapter 5. Then write the Bash version with `grep -rc`.
4. Deliberately trigger, observe, and fix all of these — collecting the exact error text in a notes file: (a) execution policy block, (b) running a script without `.\` or `./`, (c) Bash "Permission denied" from a missing execute bit, (d) Bash `count = 5` spacing error, (e) a CRLF-damaged `.sh` file run in WSL/Git Bash.
5. Write `organize.ps1`: for every file in a target folder, create a subfolder named after its extension (`txt`, `jpg`, ...) and move the file in. Parameters: `-Path` (mandatory) and `-WhatIfMode` (switch) that only *prints* what would move without moving. Test on a sandbox folder seeded with a dozen dummy files of mixed extensions. (Keep this script — Project 4 builds on the same skills.)
