# Project 2: Log Detective

## Description

A (fictional) web service misbehaved overnight, and you have its log file. Your job: answer an investigation brief using only command-line search and pipeline tools — no opening the log in an editor and scrolling. First you'll *generate* a realistic log file to your own specification (which is itself great practice), then you'll interrogate it: filtering, counting, sorting, and correlating until the story of the outage emerges.

This is the daily bread of real operations work: on a server, the log file and your shell are all you get.

## Difficulty

**Beginner–Intermediate** — Estimated effort: 2–3 hours

## Chapters used

- Chapter 4: Viewing & Searching File Contents (core)
- Chapter 5: The Pipeline (core — also covers the generator below: `ForEach-Object` over a range, no scripting required)
- Chapter 2: Navigation (incidental)

## Requirements checklist

Work in `D:\atoop\coding-projects\command-line\projects\sandbox\log-detective\`.

**Phase A — Manufacture the evidence:**
- [ ] Create `service.log` with **at least 300 lines** in this exact format (one entry per line):
      `YYYY-MM-DD HH:MM:SS LEVEL [component] message`
      e.g. `2026-07-17 02:14:09 ERROR [db] connection timeout after 30s`
- [ ] Use at least 4 levels (`INFO`, `WARN`, `ERROR`, `FATAL`) and at least 4 components (e.g. `web`, `db`, `auth`, `cache`)
- [ ] Build in a deliberate "incident": a time window where ERROR/FATAL lines cluster around one component, with a cause hinted earlier (e.g., WARN lines about the same component before the failures start)
- [ ] Generate the baseline lines using the provided setup snippet below (tweak the values freely) — it's scaffolding for the exercise, not the skill being tested here; some randomness or repetition patterns are welcome
- [ ] Do not consult the generator or its source while solving Phase B — investigate the log as if you'd never seen it

**Provided setup snippet** (paste into PowerShell, or tweak the arrays/timestamps first — no scripting knowledge required, it's just a `ForEach-Object` pipeline over a range, straight from Ch. 5):

```powershell
$levels     = "INFO","WARN","ERROR","FATAL"
$components = "web","db","auth","cache"
$messages   = @{
    INFO  = "request completed","cache hit","session refreshed"
    WARN  = "slow query detected","retrying connection","cache miss"
    ERROR = "connection timeout after 30s","query failed","auth token rejected"
    FATAL = "service unresponsive","out of memory"
}
$start = Get-Date "2026-07-17 00:00:00"

# Baseline noise: 280 lines, one every ~15s, level/component/message picked at random
1..280 | ForEach-Object {
    $level     = $levels | Get-Random
    $component = $components | Get-Random
    $message   = $messages[$level] | Get-Random
    $timestamp = $start.AddSeconds($_ * 15)
    "$($timestamp.ToString('yyyy-MM-dd HH:mm:ss')) $level [$component] $message"
} | Set-Content service.log

# The incident: hand-write one component's failure story (edit freely — make it yours)
@(
    "2026-07-17 01:10:00 WARN [db] slow query detected"
    "2026-07-17 01:12:30 WARN [db] retrying connection"
    "2026-07-17 01:15:00 ERROR [db] connection timeout after 30s"
    "2026-07-17 01:15:20 ERROR [db] connection timeout after 30s"
    "2026-07-17 01:16:05 FATAL [db] service unresponsive"
) | Add-Content service.log
```

Change the component, messages, and timestamps in the incident block so it's not identical to this example — that's what keeps Phase B honest.

**Phase B — The investigation brief (answer each with a command; save every command + answer into `case-notes.md` as you go):**
- [ ] How many lines does the log contain? How many per level?
- [ ] Show the first 5 and last 5 entries (establish the time span)
- [ ] Extract every ERROR and FATAL line into a separate file `incidents.txt` using redirection
- [ ] Which distinct error messages occur, and how many times each? (frequency table, most common first)
- [ ] Which component produced the most ERROR lines?
- [ ] Identify the incident window: the earliest and latest timestamp of the error cluster
- [ ] Find the "smoking gun": the earliest WARN line about the failing component *before* the first ERROR
- [ ] Produce a one-screen summary (5–10 lines) of the outage story, written into `case-notes.md`
- [ ] At least half the brief solved in PowerShell (`Select-String`, `Where-Object`, `Group-Object`, `Sort-Object`, `Measure-Object`) and at least three questions *also* solved in Bash (`grep`, `sort`, `uniq -c`, `head`, `tail`, `wc`) — note where the two approaches felt different

**Phase C — Live tail:**
- [ ] In one terminal, run a loop that appends a new log line every few seconds; in a second terminal, watch the file grow with a follow/tail command; demonstrate spotting an ERROR the moment it appears

## Hints

- The frequency-table question has a classic Bash idiom built from three small tools chained twice (Ch. 5 shows the shape), and a PowerShell answer built on `Group-Object`. If you're piping and it feels clumsy, you're one cmdlet away.
- "Per level" counts don't need four separate commands — grouping can do it in one.
- For the incident window, remember timestamps in this format sort *alphabetically = chronologically*. Sorting the incidents file gets you both ends cheaply.
- `Select-String` gives you objects with a `.Line` property — useful when redirected output looks odd (Ch. 4 pitfalls).
- Case sensitivity defaults differ between `grep` and `Select-String` — decide explicitly, per query, whether case matters.
- Generator stuck? The provided snippet in Phase A is the whole thing — just edit the arrays and incident block to make it yours. Keep it crude; realism comes from volume.

## Stretch goals

- Add a second log (`worker.log`) with overlapping timestamps and correlate: did the worker fail *before* or *after* the main service's first ERROR?
- Answer one brief question with a regex that captures only the millisecond values from "timeout after Nms" lines, and compute their average (PowerShell `Measure-Object -Average` or `awk`).
- Wrap your five favorite investigation commands into a `loginspect.ps1` that takes a log path and prints the whole dashboard at once (foreshadows Project 4).
- Re-run the entire investigation inside WSL using only Bash tools, timing yourself against your first pass.
