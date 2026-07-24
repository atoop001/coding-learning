# Project 3: CI Pipeline

## Description

Give one of your existing projects the thing every professional repo has and almost no learner repo does: a continuous integration pipeline that installs, lints, and tests the code on every push and every pull request — plus a status badge in the README and branch protection that makes a red build physically unmergeable. This is the highest ratio of career-signal to effort in the entire track: a hiring manager who opens your repo and sees green checks, a real workflow file, and protected main immediately knows you've practiced how teams actually work. But the badge is the souvenir, not the goal — the goal is the *discipline loop*: push, watch, read failures, fix, green. You'll run that loop on purpose here, several times, so it's boring by the time a job makes it mandatory.

## Difficulty & Estimated Effort

Intermediate — 3–5 hours (more if the chosen project has no tests yet; writing a starter test suite is in scope).

## Chapters Used

- Chapter 7: Continuous Integration (core)
- Chapter 3: Configuration & Secrets (supporting, if your tests need config)

## Requirements

- [ ] Choose a project with (or give it) both a lint command and a test command that run locally and *mean something*: the linter has a real config, and there are at least five real tests that can genuinely fail. If the project has neither, adding them is the first milestone.
- [ ] Verify the exit-code contract locally: run lint and tests, then check the exit code in PowerShell after a passing run and after a deliberately failing run. If a failing test doesn't produce a non-zero exit code, fix the scripts before touching CI — the pipeline can only see what exit codes tell it.
- [ ] Create a workflow file in the magic directory, triggered on both pushes to main and pull requests targeting main.
- [ ] The job must: run on a GitHub-hosted Linux runner, check out the code, set up your language runtime with a pinned version matching your dev environment, install dependencies with the reproducible (lockfile-exact) command, then lint, then test — cheap checks before expensive ones.
- [ ] Enable dependency caching via the setup action and verify from run logs that the second run is faster than the first (find the cache-hit line).
- [ ] Push and get an honest green run. "Honest" means: prove the pipeline can fail by pushing a branch with a broken test, watching it go red, and reading the failure from the Actions log *bottom-up* before fixing it.
- [ ] Practice the failure-classification discipline: in your notes, log at least three distinct red runs from this project (test failure, lint failure, and one of your choice), each with: the decisive log line, your classification, and the fix. This is the "reading failed runs" skill made concrete.
- [ ] Add the status badge to the top of the README, pointing at the workflow's results on main.
- [ ] Turn on branch protection for main: require the CI check to pass, and require changes to arrive via pull request. Prove both walls exist: attempt a direct push to main (expect rejection) and open a PR with a failing check (expect a blocked merge button). Screenshot or transcript both.
- [ ] Merge at least two real changes through the full ceremony: branch → push → PR → checks green → merge. Solo, yes — the ceremony is the practice.
- [ ] Ensure the workflow needs no secrets; if your tests currently require real credentials or a real database, decouple them for CI (fakes, or skip-with-reason) and note the Chapter 10 service-container approach as the future fix.

## Hints

- The chapter's starter workflow is the right skeleton; the work here is fitting it to *your* project's scripts and versions, then earning the green honestly.
- If CI fails but local passes, Chapter 7 pitfall 1 has the reproduction recipe: clean clone into a temp folder, lockfile install, run the same commands. The difference between that clone and your working copy *is* the bug.
- YAML will humble you at least once. Two spaces, no tabs, quote version numbers, and remember the Actions tab shows parse errors for workflows that never triggered at all.
- "No tests specified" exiting zero is the classic decorative-pipeline trap — it's named in the chapter's pitfalls for a reason. Your deliberate-red requirement exists to catch exactly this.
- Branch protection settings live in repo Settings → Branches (or Rulesets — either satisfies the requirement, and noticing GitHub has two generations of this feature is itself useful knowledge).
- For the badge, the workflow page's "..." menu writes the markdown for you.
- Keep runs fast. If your pipeline takes more than ~3 minutes for a small project, something's uncached or overbuilt — a slow pipeline is a pipeline people stop respecting.

## Stretch Goals

- Split lint and test into two parallel jobs and compare total wall-clock time against the sequential version; write one sentence on when the split is worth it.
- Add a second workflow that runs the test job against two runtime versions via a matrix, and decide (in writing) whether an app like yours actually needs it.
- Make the pipeline run your Project 2 Docker build (`docker build` as a CI step) so image breakage is caught on every push — no pushing to a registry required yet.
- Add a scheduled weekly run (cron trigger) and explain in a comment what class of breakage a scheduled run catches that push-triggered runs cannot.
- Retrofit the same pipeline shape onto a second project in under 30 minutes, timed — the proof that this skill has become cheap for you.
