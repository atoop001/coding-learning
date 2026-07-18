# Project 2: Log Detective

## Description

A (fictional) web service misbehaved overnight, and you have its log file. Your job: answer an investigation brief using only command-line search and pipeline tools — no opening the log in an editor and scrolling. First you'll *generate* a realistic log file to your own specification (which is itself great practice), then you'll interrogate it: filtering, counting, sorting, and correlating until the story of the outage emerges.

This is the daily bread of real operations work: on a server, the log file and your shell are all you get.

## Difficulty

**Beginner–Intermediate** — Estimated effort: 2–3 hours

## Chapters used

- Chapter 4: Viewing & Searching File Contents (core)
- Chapter 5: The Pipeline (core)
- Chapter 2: Navigation (incidental)
- Chapter 8: Writing Scripts (only for the generator, and only lightly)

## Requirements checklist

Work in `D:\atoop\coding-projects\command-line\projects\sandbox\log-detective\`.

**Phase A — Manufacture the evidence:**
- [ ] Create `service.log` with **at least 300 lines** in this exact format (one entry per line):
      `YYYY-MM-DD HH:MM:SS LEVEL [component] message`
      e.g. `2026-07-17 02:14:09 ERROR [db] connection timeout after 30s`
- [ ] Use at least 4 levels (`INFO`, `WARN`, `ERROR`, `FATAL`) and at least 4 components (e.g. `web`, `db`, `auth`, `cache`)
- [ ] Build in a deliberate "incident": a time window where ERROR/FATAL lines cluster around one component, with a cause hinted earlier (e.g., WARN lines about the same component before the failures start)
- [ ] Generate the lines with a loop (a small script or a one-liner is fine — hand-typing 300 lines is not the exercise); some randomness or repetition patterns are welcome
- [ ] Do not consult the generator or its source while solving Phase B — investigate the log as if you'd never seen it

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
- Generator stuck? A loop over a range, a couple of arrays of sample components/messages, and an index picked with `Get-Random` / `$RANDOM` goes a long way. Keep it crude; realism comes from volume.

## Stretch goals

- Add a second log (`worker.log`) with overlapping timestamps and correlate: did the worker fail *before* or *after* the main service's first ERROR?
- Answer one brief question with a regex that captures only the millisecond values from "timeout after Nms" lines, and compute their average (PowerShell `Measure-Object -Average` or `awk`).
- Wrap your five favorite investigation commands into a `loginspect.ps1` that takes a log path and prints the whole dashboard at once (foreshadows Project 4).
- Re-run the entire investigation inside WSL using only Bash tools, timing yourself against your first pass.
