# Project 4: Build a Real Utility — Project Scaffolder

## Description

Write a genuine, reusable command-line utility: `new-project` — a scaffolder that creates a ready-to-go project skeleton (folders, starter files, git init, optional extras) from a single command like:

```
new-project -Name my-api -Type python
```

You'll build it properly in PowerShell — parameters, validation, dry-run mode, helpful errors, logging — then **port it to Bash** so it runs on any server. By the end you'll have a tool you actually use every time you start something new, and you'll have felt precisely where the two scripting worlds differ.

## Difficulty

**Intermediate–Advanced** — Estimated effort: 4–6 hours

## Chapters used

- Chapter 8: Writing Scripts (core)
- Chapter 3: File & folder operations (core)
- Chapter 5: Pipeline, exit codes, chaining
- Chapter 6: PATH (installing your tool so it runs by name)
- Chapter 9: Dry-run discipline, idempotency, logging

## Requirements checklist

Develop in `D:\atoop\coding-projects\command-line\projects\sandbox\scaffolder\`; test outputs go in a `test-output\` subfolder you can delete freely.

**Phase A — Specification first:**
- [ ] Write `SPEC.md` before any code: supported project types (at least `python`, `node`, and `notes` or another of your choice), the exact tree each type produces, all parameters and defaults, and every failure case with its intended message and exit code
- [ ] Each project type's skeleton must include: a `README.md` seeded with the project name and date, a sensible `.gitignore`, a source folder with one starter file, and one type-specific extra (e.g., `requirements.txt` / `package.json` placeholder / templates of your choosing)

**Phase B — PowerShell implementation (`new-project.ps1`):**
- [ ] `param(...)` block with: mandatory `-Name`, validated `-Type` (investigate `ValidateSet`), optional `-Path` defaulting to a sensible projects root, and a `-DryRun` switch
- [ ] Name validation: reject names with spaces or illegal path characters, with a clear error on stderr and a distinct nonzero exit code
- [ ] Refuses to overwrite: if the target folder already exists, fail loudly (distinct exit code) — no silent clobbering
- [ ] `-DryRun` prints every folder/file that *would* be created, creates nothing, and exits 0
- [ ] Real run creates the full skeleton per spec, seeds file contents (project name + creation date interpolated), and prints a concise success summary
- [ ] Runs `git init` in the new project only if git is available (detect it — don't crash without it)
- [ ] Appends one line per invocation (timestamp, name, type, outcome) to a `scaffolder.log` next to the script
- [ ] Exit codes: 0 success, and at least two distinct failure codes matching `SPEC.md`; prove them with `&&` / `||` demonstrations recorded in `SPEC.md`

**Phase C — Make it a real command:**
- [ ] Install the script so `new-project` works by bare name from any directory (PATH folder + wrapper, or a profile function — your choice, justified in `SPEC.md`)
- [ ] Create three projects of different types from three different working directories to prove it

**Phase D — Bash port (`new-project.sh`):**
- [ ] Same behaviors: argument parsing (positional or flags — investigate `getopts`), validation, no-overwrite rule, dry-run mode, logging, matching exit codes
- [ ] Correct shebang, executable bit, LF line endings; runs in Git Bash and/or WSL
- [ ] `PORT-NOTES.md`: at least 5 concrete differences you had to handle (parameter parsing, path handling, string interpolation, exists-tests, anything that surprised you)

**Phase E — Test protocol (manual is fine, scripted is stretch):**
- [ ] Write `TESTS.md` as a table of at least 8 cases (happy paths per type, bad name, existing target, dry-run, missing git, unknown type) with: command run, expected result, actual result, pass/fail — all executed against both implementations

## Requirements are the contract — a run-through checklist for "done"
- [ ] A stranger could read `SPEC.md` and predict the tool's behavior in every listed case
- [ ] Every checklist item above demonstrably true, not aspirational

## Hints

- Spec-first feels slow and is the fastest route: every validation case you write down becomes an obvious `if` block later.
- Build the skeleton-creation for *one* type end to end (with dry-run) before adding the second type; resist writing three half-features.
- A hashtable/associative-array mapping type → list of paths keeps the per-type logic in data rather than in triplicated code — both languages support this shape (Ch. 8 gives you the loop tools).
- `$PSScriptRoot` (and the Bash `dirname "$0"` idiom) is how the log file stays next to the script no matter where you invoke from — Ch. 8's working-directory pitfall, applied.
- For "is git available," both shells have a way to ask whether a command exists without running it destructively (Ch. 6 showed you the PowerShell one; Bash has `command -v`).
- Seeding multi-line file content: PowerShell here-strings (`@"..."@`) and Bash heredocs (`<<EOF`) are made for exactly this — worth ten minutes of experimentation each.
- When the port fights you, check the classics first: CRLF endings, missing quotes around variables, spaces around `=`.

## Stretch goals

- `--list-types` flag that prints available types with one-line descriptions, generated from the same data structure that drives creation.
- A `-Template <folder>` escape hatch: scaffold by copying a user-provided template tree, with `{{NAME}}` placeholders substituted in file contents.
- Automate Phase E: a `run-tests.ps1` that executes every case, checks exit codes and resulting trees, and prints a pass/fail summary (your first test harness!).
- Publish quality: `-Help`/`-h` output, a version number, and a `CHANGELOG.md` — then actually use the tool for your next real project and log the first improvement request against yourself.
