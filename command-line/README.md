# Command-Line Learning Track

A self-paced track for going from "I can open a terminal" to genuine developer fluency — on Windows first (PowerShell as your daily shell), with Bash equivalents throughout so Linux servers, macOS, Git Bash, and CI systems feel familiar rather than foreign by the end.

**Why this track is worth running alongside anything else you're learning:** the terminal is not a subject like a programming language — it's the *workbench* every other subject sits on. Skills from Chapter 2 pay off the same afternoon in your Python or JavaScript work (navigating, running tools, reading errors); by mid-track you're installing tools, fixing PATH problems, and scripting chores that used to eat evenings. This track has no prerequisites beyond basic computer use and some coding exposure, conflicts with nothing, and compounds everything.

## Structure

- `learning-docs/` — 10 chapters, the primary study material. Each contains an overview, plain-English definitions, worked command examples (PowerShell and Bash side by side once both are introduced), common pitfalls with corrections, and practice exercises.
- `projects/` — 5 guided project specs, easiest to hardest, each bundling several chapters into a hands-on build. Specs define requirements and hints only — no solution code. Do them in a sandbox folder; mistakes there are free.

## Chapters (in order)

| # | File | Covers |
|---|------|--------|
| 1 | `learning-docs/01-why-the-terminal.md` | What terminals and shells are, PowerShell vs cmd vs Bash/WSL, prompts, running first commands, getting help |
| 2 | `learning-docs/02-navigating-the-filesystem.md` | Paths (absolute/relative), cd, listing and inspecting, tab completion |
| 3 | `learning-docs/03-managing-files-and-folders.md` | Create/copy/move/rename/delete — and the safety habits that prevent rm disasters |
| 4 | `learning-docs/04-viewing-and-searching.md` | cat/Get-Content, head/tail, grep/Select-String, find/Get-ChildItem, wildcards vs regex |
| 5 | `learning-docs/05-the-pipeline.md` | Pipes, redirection, chaining with && and \|\|, PowerShell objects vs Bash text |
| 6 | `learning-docs/06-environment-and-configuration.md` | PATH demystified, environment variables, profiles, aliases and functions, fixing "command not found", processes and ports |
| 7 | `learning-docs/07-package-managers.md` | winget, npm, pip (+ venvs), apt preview, the PATH implications of installing tools, and HTTP requests from the terminal (curl, Invoke-RestMethod) |
| 8 | `learning-docs/08-writing-scripts.md` | .ps1 scripts + execution policy, .sh scripts + shebang/chmod, variables, arguments, if, loops, exit codes |
| 9 | `learning-docs/09-practical-automation.md` | Batch renaming, dated backups, dry runs, idempotency, Task Scheduler and cron, logging |
| 10 | `learning-docs/10-wsl-and-remote.md` | Installing WSL, ssh and keys, scp, Linux file permissions, chmod/chown/sudo |

## Projects (in order)

| # | File | Difficulty | After chapter | Chapters used | Drills |
|---|------|-----------|---------------|----------------|--------|
| 1 | `projects/01-terminal-scavenger-hunt.md` | Beginner | Ch. 4 | Ch. 1–4 | Build and solve a folder-maze with terminal only — navigation, file ops, tab completion |
| 2 | `projects/02-log-detective.md` | Beginner–Int. | Ch. 5 | Ch. 2, 4, 5 | Generate and investigate a service log with search + pipeline tools |
| 3 | `projects/03-environment-makeover.md` | Intermediate | Ch. 7 | Ch. 1, 2, 5, 6, 7 | Audit and rebuild your PATH, profiles, aliases; install tools cleanly; reproducible setup |
| 4 | `projects/04-project-scaffolder.md` | Int.–Advanced | Ch. 8 | Ch. 3, 5, 6, 8, 9 | Build a real `new-project` scaffolding utility in PowerShell, then port it to Bash |
| 5 | `projects/05-capstone-automated-workflow.md` | Advanced | Ch. 10 | Ch. 3–6, 8–10 | Nightly backup/report/cleanup pipeline, scheduled on Windows *and* deployed to WSL over ssh with cron |

## Suggested cadence

At **3–5 hours per week**, the track takes roughly **8–10 weeks** — comfortably in parallel with a language track, and most weeks it will *save* time in that other track.

- **Weeks 1–2:** Chapters 1–3. Short daily sessions beat one long one — the goal is reflexes (pwd, ls, tab completion), and reflexes come from repetition.
- **Week 3:** Chapter 4, then **Project 1** (scavenger hunt).
- **Week 4:** Chapter 5 — the conceptual heart of the track; take it slowly — then **Project 2** (log detective).
- **Week 5:** Chapters 6–7, then **Project 3** (environment makeover). Expect this week to immediately improve your daily workflow.
- **Weeks 6–7:** Chapter 8, then **Project 4** (scaffolder + Bash port). The biggest single build; don't rush the spec.
- **Week 8:** Chapter 9, start **Project 5** through the Windows scheduling phase (it needs elapsed days to prove itself).
- **Weeks 9–10:** Chapter 10, finish **Project 5** with the WSL/ssh/cron deployment, write the retrospective.

Habit that makes everything stick: from Week 1 onward, do your *other* coursework's file management and program-running from the terminal, even when a GUI would be easier. Ten real repetitions beat a hundred exercises.

## Ground rules

- Work in sandbox folders (each project spec names one). Destructive-command practice belongs where mistakes cost nothing.
- Type the commands — don't paste. Errors you cause and fix are the curriculum.
- When both shells are shown, run both. The Bash column is your future server life.
- No solution code exists in this track on purpose. The hints sections nudge; the chapters contain every technique required.
