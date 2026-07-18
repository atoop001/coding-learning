# Chapter 9: Practical Automation — Renaming, Backups & Scheduling

## Overview

This chapter is where everything converges: navigation (Ch. 2), file operations (Ch. 3), searching (Ch. 4), pipelines (Ch. 5), environment (Ch. 6), and scripting (Ch. 8) combine into real automation — the tasks developers actually script. You'll batch-rename files, build a dated backup routine, and learn the concept of **scheduled tasks** so your scripts run themselves. The theme throughout: *if you'll do it three times, script it* — and always dry-run destructive automation before letting it loose.

## Definitions & Explanations

**Batch operation** — One command or loop applied to many files: rename 200 photos, convert every `.md`, compress old logs. The payoff of loops + wildcards + pipelines.

**Dry run** — Executing automation in "describe, don't do" mode first. PowerShell gives you `-WhatIf` nearly everywhere; in Bash you emulate it by `echo`-ing each intended command before switching to the real thing. Non-negotiable for anything that renames, moves, overwrites, or deletes in bulk — a buggy loop makes mistakes at machine speed.

**Idempotent** — A script that can run twice safely, producing the same result (the second run changes nothing or harmlessly repeats). Scheduled jobs *will* run repeatedly, so design for it: "create folder if missing" (`-Force`/`mkdir -p`), "copy only newer files," "skip if today's backup exists."

**Timestamped naming** — Embedding dates in filenames — `backup-2026-07-18.zip` — so runs never collide and sort chronologically. Use `yyyy-MM-dd` (ISO) ordering; `Get-Date -Format "yyyy-MM-dd"` / `date +%F`. Avoid `/` or `:` in names (illegal in paths).

**Archive/compression** — Bundling a tree into one file: `Compress-Archive` → `.zip` (PowerShell); `tar -czf` → `.tar.gz` (the Linux standard). Smaller, single-file, easy to move or upload.

**Scheduled task** — The OS running a command on a timetable with no human present.
- *Windows*: **Task Scheduler** — GUI (`taskschd.msc`) or the `schtasks` command / `ScheduledTasks` PowerShell module. Triggers (daily at 02:00, at logon, on idle) fire actions (run `pwsh -File backup.ps1`).
- *Linux*: **cron** — per-user tables edited via `crontab -e`; each line is five time fields plus a command. `0 2 * * *` = daily at 02:00. (Modern alternative: systemd timers.)
Key mental shift: scheduled scripts run *without you* — no visible window, possibly a different working directory and a minimal environment. Scripts must use absolute paths, assume nothing, and **log to a file** since nobody's watching the screen.

**Logging** — Automation's flight recorder: append a timestamped line per significant action to a log file. When a 2 a.m. job misbehaves, the log is all you have.

## Command Examples

Batch renaming — the canonical drill (photos to dated, numbered names):

```powershell
# Setup: a sandbox of fake photos
mkdir rename-lab; cd rename-lab
1..8 | ForEach-Object { New-Item "IMG_00$_.jpg" | Out-Null }

# DRY RUN first — always:
Get-ChildItem *.jpg | Rename-Item -NewName { $_.Name -replace '^IMG_', 'vacation-' } -WhatIf
# What if: Performing the operation "Rename File" on target
# "Item: ...\IMG_001.jpg Destination: ...\vacation-001.jpg".  (x8)

# Looks right — run for real:
Get-ChildItem *.jpg | Rename-Item -NewName { $_.Name -replace '^IMG_', 'vacation-' }
ls -Name
# vacation-001.jpg ... vacation-008.jpg

# Numbered renames with a counter:
$i = 1
Get-ChildItem *.jpg | Sort-Object Name | ForEach-Object {
    Rename-Item $_ ("photo-{0:d3}.jpg" -f $i)    # d3 => 001, 002...
    $i++
}
```

```bash
# Bash equivalent
mkdir rename-lab && cd rename-lab
for i in 1 2 3 4 5 6 7 8; do touch "IMG_00$i.jpg"; done

# DRY RUN: echo the mv commands instead of running them
for f in IMG_*.jpg; do
    echo mv "$f" "${f/IMG_/vacation-}"     # ${var/find/replace} substitution
done
# mv IMG_001.jpg vacation-001.jpg   (x8 — read every line!)

# Real run: delete the word `echo`
for f in IMG_*.jpg; do mv "$f" "${f/IMG_/vacation-}"; done

# Numbered with a counter:
i=1
for f in *.jpg; do
    mv "$f" "$(printf 'photo-%03d.jpg' "$i")"
    i=$((i+1))
done
```

A dated backup script — the pattern to memorize:

```powershell
# backup.ps1 — zip a source folder to a dated archive, keep last 7, log everything
param(
    [string]$Source  = "D:\atoop\coding-projects\command-line",
    [string]$DestDir = "D:\backups",
    [int]$Keep       = 7
)

$stamp   = Get-Date -Format "yyyy-MM-dd_HHmm"
$logFile = Join-Path $DestDir "backup.log"

New-Item -ItemType Directory -Path $DestDir -Force | Out-Null   # idempotent
$zip = Join-Path $DestDir "backup-$stamp.zip"

try {
    Compress-Archive -Path $Source -DestinationPath $zip -ErrorAction Stop
    "$(Get-Date -Format o) OK   created $zip" | Add-Content $logFile
} catch {
    "$(Get-Date -Format o) FAIL $($_.Exception.Message)" | Add-Content $logFile
    exit 1
}

# Retention: delete oldest beyond $Keep
Get-ChildItem $DestDir -Filter "backup-*.zip" |
    Sort-Object LastWriteTime -Descending |
    Select-Object -Skip $Keep |
    Remove-Item

exit 0
```

```bash
#!/usr/bin/env bash
# backup.sh — same pattern in Bash
set -euo pipefail                       # strict mode: stop on errors/undefined vars

source_dir="${1:-$HOME/projects}"
dest_dir="${2:-$HOME/backups}"
keep=7

stamp="$(date +%F_%H%M)"
mkdir -p "$dest_dir"                    # idempotent
archive="$dest_dir/backup-$stamp.tar.gz"
log="$dest_dir/backup.log"

if tar -czf "$archive" -C "$(dirname "$source_dir")" "$(basename "$source_dir")"; then
    echo "$(date -Is) OK   created $archive" >> "$log"
else
    echo "$(date -Is) FAIL tar exited nonzero" >> "$log"
    exit 1
fi

# Retention: list newest-first, delete from line keep+1 on
ls -1t "$dest_dir"/backup-*.tar.gz | tail -n +$((keep+1)) | xargs -r rm --
```

Scheduling — Windows Task Scheduler:

```powershell
# Register a daily 02:00 run of backup.ps1 (run from an elevated/admin prompt
# for some options; basic user tasks work unelevated):
$action  = New-ScheduledTaskAction -Execute "pwsh.exe" `
           -Argument "-NoProfile -File D:\atoop\scripts\backup.ps1"
$trigger = New-ScheduledTaskTrigger -Daily -At 2:00AM
Register-ScheduledTask -TaskName "NightlyBackup" -Action $action -Trigger $trigger
# TaskName       Status
# --------       ------
# NightlyBackup  Ready

Get-ScheduledTask -TaskName "NightlyBackup"        # inspect
Start-ScheduledTask -TaskName "NightlyBackup"      # test-fire it NOW
Get-ScheduledTaskInfo -TaskName "NightlyBackup"    # last run time & result code
Unregister-ScheduledTask -TaskName "NightlyBackup" # remove when done testing
```

Scheduling — cron (WSL/Linux):

```bash
crontab -e            # opens your personal cron table in an editor
# Add this line — five time fields, then the command:
# 0 2 * * * /home/atoop/scripts/backup.sh >> /home/atoop/backups/cron.log 2>&1
#
# field order: minute hour day-of-month month day-of-week
#   0 2 * * *      = every day at 02:00
#   */15 * * * *   = every 15 minutes
#   0 9 * * 1-5    = 09:00 on weekdays

crontab -l            # list current entries to verify
```

Combining tools — small real-world one-liners:

```powershell
# Free disk space by finding node_modules folders and their sizes:
Get-ChildItem -Recurse -Directory -Filter node_modules |
    ForEach-Object {
        $mb = (Get-ChildItem $_ -Recurse -File | Measure-Object Length -Sum).Sum / 1MB
        "{0,8:n1} MB  {1}" -f $mb, $_.FullName
    }

# Archive all logs older than 30 days, then remove originals (note the order!)
$old = Get-ChildItem *.log | Where-Object LastWriteTime -lt (Get-Date).AddDays(-30)
$old | Compress-Archive -DestinationPath "old-logs-$(Get-Date -Format yyyy-MM-dd).zip"
$old | Remove-Item -WhatIf       # dry run... then run without -WhatIf
```

## Common Pitfalls

**Pitfall: bulk rename collisions and clobbering.**
A rename maps two source files to the same target name — one silently overwrites the other; or re-running the rename script mangles already-renamed files (`vacation-vacation-001.jpg`).
*Correction*: dry-run and *read every proposed line*; design patterns to be idempotent (only rename files still matching the original pattern, e.g. anchored `^IMG_`); when unsure, copy to a new folder instead of renaming in place.

**Pitfall: the scheduled task "works when I run it, fails at 2 a.m."**
Interactively, your script enjoyed your working directory, your PATH, your profile, maybe a venv. The scheduler grants none of that.
*Correction*: absolute paths for everything (script, inputs, outputs, even executables like `pwsh.exe` if needed); `-NoProfile` so you're not depending on profile magic; log every run to a file; after registering a task, *always* test-fire with `Start-ScheduledTask` and read the log rather than trusting it.

**Pitfall: backup script that has never been restored from.**
The backup "ran green" for a year, but the archives are empty/corrupt/missing the folder you actually cared about.
*Correction*: test the restore path — expand an archive to a temp folder and diff against the source (`Compare-Object`, or spot-check counts). A backup you haven't restored from is a hope, not a backup.

**Pitfall: retention logic that deletes the wrong files.**
An overly broad wildcard in the cleanup step (`Remove-Item $DestDir\*`) takes the log file — or worse, unrelated files someone stored in that folder.
*Correction*: make retention patterns exactly as specific as the creation pattern (`backup-*.zip`), and dry-run the deletion branch separately with `-WhatIf` before first deployment.

**Pitfall: cron jobs failing silently forever.**
Cron's environment is even barer than Task Scheduler's (minimal PATH, no aliases); errors go to a mail spool nobody reads.
*Correction*: absolute paths in cron lines; redirect both streams to a log (`>> log 2>&1`) as shown above; check the log after the first scheduled cycle, not just after manual runs.

**Pitfall: automating before verifying the manual steps.**
Scripting a workflow you haven't done by hand bakes misunderstandings into a loop.
*Correction*: do it manually once, note the exact commands, script them, dry-run, run on a sandbox, then trust it with real data. Automation is the *last* step, not the first.

## Practice Exercises

1. Seed a sandbox with 20 files named `report_final_v1.txt` ... `report_final_v20.txt` (script the seeding itself!). Batch-rename them to `2026-report-01.txt` ... `2026-report-20.txt` with zero-padded numbers — dry run first, in both PowerShell and Bash.
2. Write your own version of the dated backup script for a folder you care about (target a sandbox copy while testing): dated archive name, idempotent destination creation, append-only log, retention of the newest 5. Run it three times and verify: 3 archives, 3 log lines, correct retention behavior once you exceed 5.
3. Prove the restore: expand your newest archive into a fresh temp folder and verify the file count matches the source (one pipeline per shell). Document the command you'd run "in anger" at the top of your backup script as a comment.
4. Register a Windows scheduled task that runs your backup script daily at a time of your choosing, test-fire it with `Start-ScheduledTask`, confirm via the log and `Get-ScheduledTaskInfo`, then unregister it. If you have WSL: add (then remove) an equivalent crontab line, using `*/5 * * * *` briefly to see it actually fire.
5. Build a "downloads janitor" one-pipeline-per-rule script: in a sandbox modeled on a messy Downloads folder, move images to `Pictures-sorted/`, archives to `Archives-sorted/`, and installers (`.exe`, `.msi`) older than 30 days into `Quarantine/` — with a `-WhatIf`-style dry-run mode you demonstrate before the real run.
