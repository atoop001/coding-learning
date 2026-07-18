# Project 5: Capstone — The Self-Running Workspace

## Description

The finale combines everything: scripting, pipelines, environment, scheduling, and remote Linux. You'll build a small **automated workspace maintenance system** — a nightly pipeline that backs up your work, digests activity into a report, and tidies clutter — running unattended on a schedule, with logs good enough to debug at a distance. Then you'll take the show on the road: deploy the Bash side of the system to your WSL "server" over ssh and schedule it there with cron.

When this works — when you check a morning report generated while you slept, on two operating systems — you have crossed from "person who can use the terminal" to "person the terminal works for."

## Difficulty

**Advanced** — Estimated effort: 5–8 hours (spread over several days — scheduling needs real elapsed time to prove itself)

## Chapters used

- Chapter 9: Practical Automation & Scheduling (core)
- Chapter 8: Writing Scripts (core)
- Chapter 10: WSL, SSH & Permissions (core)
- Chapter 5: Pipeline & redirection (throughout)
- Chapters 3, 4, 6: file ops, searching, environment (assumed fluency)

## Requirements checklist

Develop in `D:\atoop\coding-projects\command-line\projects\sandbox\capstone\`. Point the automation at a **sandbox "workspace" folder you seed with realistic content** (docs, code files, junk files, logs) — never at real data until the final phase, and only then if you choose.

**Phase A — Design document:**
- [ ] `DESIGN.md` written before code: the three jobs (backup, report, cleanup), their order and data flow, file/folder layout, naming conventions, log format, schedule times, and a failure-handling policy (what happens when a step fails? does the next step run?)

**Phase B — The three jobs (PowerShell first):**
- [ ] **Backup job**: dated archive of the workspace to a backups folder; idempotent (safe to re-run same day — define and implement your collision policy); retention keeps newest N; verifies the archive exists and is non-empty before declaring success
- [ ] **Report job**: writes `report-YYYY-MM-DD.md` summarizing the workspace: total files/size, files modified in the last 24h, largest 5 files, count of TODO markers across text files, and latest backup status pulled from the log — all computed with pipelines, no hardcoded numbers
- [ ] **Cleanup job**: moves (never deletes) clutter — temp/cache patterns you define, files in a `scratch/` area older than 7 days — into a dated `quarantine/` folder; supports a dry-run mode; logs every file moved
- [ ] Each job: absolute-path safe (runs correctly from any working directory), one timestamped log line minimum per run to a shared `automation.log`, meaningful exit codes

**Phase C — The orchestrator:**
- [ ] `nightly.ps1` runs the three jobs in your designed order, honoring your failure policy (conditional chaining — a failed backup should probably block cleanup!)
- [ ] Writes a final `SUMMARY` log line: which jobs ran, which succeeded, total duration
- [ ] A `-DryRun` switch that cascades to every job

**Phase D — Scheduled and proven (Windows):**
- [ ] Registered as a Windows scheduled task running the orchestrator (pick a time you'll be awake for the first live run, e.g., in 10 minutes)
- [ ] Proof it ran *from the scheduler, not by hand*: log lines + `Get-ScheduledTaskInfo` result captured into `EVIDENCE.md`
- [ ] At least two consecutive scheduled runs verified, including one where you deliberately caused a job to fail (e.g., temporarily rename the workspace) and confirmed the failure policy and logs behaved as designed
- [ ] Task then re-pointed to a sane nightly time (or unregistered, your call — documented either way)

**Phase E — Deploy to "the server" (WSL over ssh):**
- [ ] Bash ports of the three jobs plus `nightly.sh` (jobs may be simplified where Windows-specific, but backup/report/cleanup/logging/exit-code behavior must survive)
- [ ] Deployment done *over ssh from PowerShell* (key-based auth, Ch. 10): copy scripts with `scp`, set permissions with `chmod`, seed a small Linux-side workspace, test-run remotely with `ssh <alias> "..."`
- [ ] Scheduled with cron (a fast interval like `*/10` for verification, then relaxed); two cron-triggered runs proven via the Linux-side log, evidence into `EVIDENCE.md`
- [ ] `RUNBOOK.md`: how a stranger would check system health on either machine (which log, which commands, what "healthy" looks like), plus the recovery steps for the two most likely failures

**Phase F — Retrospective:**
- [ ] `RETRO.md`: what broke, what you'd redesign, which chapter's pitfall you personally re-proved, and the next automation you now want to build

## Hints

- Build order that works: jobs interactively → orchestrator interactively → scheduled with a 5-minutes-from-now trigger → real schedule. Never debug two layers at once.
- The scheduler's barren environment is the boss fight of Phase D. Chapter 9's pitfall list is the walkthrough: absolute paths, `-NoProfile`, log-first design. When the task "does nothing," the log (or its absence) is the clue.
- For the report's TODO count and largest-files sections, you already built these pipelines in Chapter 4/5 exercises and Project 2 — steal from yourself freely.
- Quarantine-not-delete is what makes an automation bug survivable. Your future self will make a pattern mistake; design so it's an "oops, move it back" not a disaster.
- On the Linux side, if cron seems dead: is the service running? are your paths absolute? did you redirect output so you can even tell? (`grep CRON /var/log/syslog` on Ubuntu helps.)
- Elapsed time is a real dependency here — start Phase D on a day you can check back twice.

## Stretch goals

- Add a weekly job (different schedule) that rolls the daily reports into a `week-summary.md` — two schedules coexisting.
- Make the report multi-machine: the Windows report pulls the Linux log over ssh and includes a "server status" section — one document covering both worlds.
- Add simple alerting: if any job fails, the orchestrator drops an `ATTENTION.txt` on your Desktop (Windows) / writes a wall-visible message (Linux) so failures find *you*.
- Parameterize the whole system with a single config file (paths, retention counts, patterns) read by both the PowerShell and Bash implementations — one config, two runtimes.
- Run the system against your real `coding-projects` workspace for two weeks; log every intervention it needed, then ship a "v2" fixing the top annoyance.
