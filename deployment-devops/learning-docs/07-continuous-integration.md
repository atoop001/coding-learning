# Chapter 7: Continuous Integration

## Overview

Here is a scene from every un-CI'd team in history: Friday afternoon, someone merges a branch that "worked on their machine," the main branch quietly breaks, three other people pull it Monday morning, and half a day evaporates figuring out whose change did it. **Continuous Integration (CI)** is the practice that killed that scene at professional shops: every push, every pull request, a robot checks out the code on a fresh machine, installs it from scratch, lints it, and runs the tests — and everyone sees green or red within minutes. It's not bureaucracy; it's the thing that makes shared code trustworthy, and teams that have it genuinely refuse to work without it. In this chapter you'll learn the anatomy of **GitHub Actions** — the CI system built into GitHub and the one you're most likely to meet in any job — build a real lint-and-test workflow, develop the essential skill of *reading a failed run*, and wire up branch protection so red builds physically can't merge. Job-market note: "set up CI with GitHub Actions" is one of the highest-signal, lowest-effort lines a junior portfolio can carry.

## Definitions & Explanations

**Continuous Integration** — the practice of merging everyone's work into the shared main branch frequently (daily or better), with every change automatically verified by an integration build: fresh checkout, clean install, lint, tests. The two halves matter equally: *integrate often* (so conflicts stay small) and *verify automatically* (so breakage is caught in minutes, by a machine, before it spreads to teammates). CI's deepest benefit is trust: a green main branch means anyone can pull, branch, and deploy at any moment.

**GitHub Actions** — GitHub's built-in automation platform. Competitors you'll hear named: GitLab CI, CircleCI, Jenkins (the self-hosted old guard). Concepts transfer almost one-to-one; syntax differs. Actions is free for public repos and has a generous free tier for private ones — for your purposes, free.

**Workflow** — one automation recipe, defined by one YAML file in `.github/workflows/` in your repo (the path is magic — files there are discovered automatically). A repo can have many: `ci.yml`, `deploy.yml`, etc. Workflows are code: versioned, reviewed, diffed.

**Event / trigger (`on:`)** — what starts a workflow: `push` (commits landing on specified branches), `pull_request` (a PR opened or updated — this is what checks PRs *before* merge), `schedule` (cron), `workflow_dispatch` (a manual "Run" button). The standard CI pair is `push` to main plus `pull_request` targeting main.

**Job** — a named unit of work within a workflow. Each job runs on a *fresh virtual machine*, and jobs run **in parallel** by default; `needs:` creates ordering between them. A starter CI workflow is usually one job.

**Step** — one action within a job, executed sequentially: either `run:` (a shell command) or `uses:` (a reusable action). A failing step fails the job, which fails the workflow, which turns the commit's check red.

**Action (reusable)** — a packaged, shareable step published on the GitHub Marketplace, referenced as `owner/name@version`. The indispensable ones: `actions/checkout` (clones your repo into the runner — nothing exists without it) and `actions/setup-node` / `actions/setup-python` (install a language runtime and can cache dependencies). Pin versions (`@v4`) — unpinned actions are a supply-chain risk and a reproducibility hole.

**Runner** — the machine executing a job: GitHub-hosted (`runs-on: ubuntu-latest` — a fresh Linux VM, discarded after the job) or self-hosted. Use `ubuntu-latest` unless you have a specific reason: it's the fastest, cheapest, and best-documented. Yes — your CI runs on Linux while you develop on Windows; that's a *feature*, catching platform assumptions (path separators, case-sensitive filenames) before production, which is also Linux.

**Check / status** — the green ✓ / red ✗ / yellow ● that workflows attach to commits and PRs. The PR page lists each required workflow's outcome; clicking through reaches the logs.

**Status badge** — the little "CI: passing" image in a README, generated from your workflow's latest result on main. Cosmetic, but it signals "this repo has standards" to visitors — including hiring managers skimming your GitHub.

**Branch protection** — repo settings (Settings → Branches → branch protection rules, or the newer Rulesets) that make main *enforce* CI: require status checks to pass before merging, require PRs (no direct pushes to main), optionally require reviews. This converts CI from advice into law: a red build physically cannot merge. Solo-project tip: protecting your own main and PR-ing your own changes is excellent job-behavior rehearsal.

**Dependabot** — GitHub's built-in dependency bot, flipped on in Settings → Security (or the older "Security & analysis" tab): **alerts** flag dependencies with known vulnerabilities by reading your lockfile against a CVE database, and **security updates** go further, auto-opening a PR that bumps the vulnerable package to a patched version — which your CI workflow then proves safe before you merge it. Enabling both takes under a minute, costs nothing, and is exactly the kind of near-zero-effort, high-signal habit that shows on a public repo: a visitor sees Dependabot PRs appearing and getting merged promptly, and reads that as active, competent maintenance.

**Matrix build (awareness)** — `strategy: matrix:` runs a job several times across variations (Node 20 *and* 22, Ubuntu *and* Windows). Library authors need this; app CI usually doesn't. Recognize it when you see it.

**Contexts and expressions (`${{ ... }}`)** — the template syntax inside workflow YAML for reading values at run time: `${{ github.sha }}` (the commit being built), `${{ github.actor }}` (who pushed), `${{ secrets.MY_TOKEN }}` (Chapter 8's bread and butter), `${{ runner.os }}`. When a workflow needs to know something about *why it's running*, a context has it.

**Workflow artifacts** — files a job saves for humans or later jobs (`actions/upload-artifact` / `download-artifact`): test reports, coverage HTML, build output. Runners are destroyed after the job; artifacts are how anything survives. Not to be confused with *deployment* artifacts (Chapter 8) — same word, related idea, different lifetime.

**Concurrency control** — a `concurrency:` block can cancel a still-running workflow when a newer push to the same branch supersedes it. Saves minutes and money on busy branches; know it exists the first time you watch three stale runs of the same PR grinding away.

**Fail fast** — the CI virtue: cheap checks (lint) before expensive ones (tests) so obviously-broken pushes die in seconds, and a red result within minutes of the push that caused it — while the change is still fresh in the author's head. The economics of CI are entirely about shortening the distance between mistake and discovery.

## Code Examples

A production-quality starter CI workflow for a Node project — this exact shape, adjusted for scripts, covers most real repos:

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]        # verify everything landing on main
  pull_request:
    branches: [main]        # verify PRs BEFORE they land — the important one

jobs:
  build:
    runs-on: ubuntu-latest  # fresh Linux VM per run

    steps:
      - uses: actions/checkout@v4
        # clones the repo into the runner — without this, the VM is empty

      - uses: actions/setup-node@v4
        with:
          node-version: 22        # match your dev/prod version — pin it
          cache: npm              # caches ~/.npm between runs; big speedup, still exact installs

      - run: npm ci
        # ci, not install: exact lockfile versions or loud failure (Chapter 5's lesson, again)

      - run: npm run lint
        # cheap check first — fail fast

      - run: npm test
        # any step exiting non-zero fails the job and turns the check red
```

The Python equivalent differs only in dialect:

```yaml
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"    # quoted! unquoted 3.10 becomes the number 3.1 — YAML strikes again
          cache: pip
      - run: pip install -r requirements.txt
      - run: ruff check .           # linting
      - run: pytest                 # tests; pytest exits non-zero on failure, which is the whole contract
```

That contract deserves a spotlight, because *it is how CI works at all*:

```text
Every step is judged by its EXIT CODE. Zero = success, anything else = failure.
Linters, test runners, and compilers all follow this convention — that's why
gluing them into CI is just listing commands.
(You can check the code yourself: `$LASTEXITCODE` in PowerShell, `echo $?` in bash.)
```

Reading a failed run — the skill, as a procedure:

```text
1. Red ✗ on the PR (or an email) → click "Details".
2. The failed JOB is marked in the left sidebar; the failed STEP is expanded in red.
3. Read the step's log BOTTOM-UP: the actual error is usually in the last 30 lines
   (test summary, stack trace, lint rule name). Everything above is context.
4. Classify before touching anything:
   - test failure -> reproduce locally (`npm test`), fix, push
   - lint failure -> `npm run lint` locally, fix or auto-fix, push
   - install/infra failure (npm registry hiccup, action outage) -> "Re-run failed jobs"
     is legitimate ONLY for this class. Never re-run to see if a test failure "goes away";
     that's how flaky tests colonize a codebase.
5. Push the fix — the PR re-runs automatically. Green resumes.
```

What the runner actually is — worth one look so "fresh machine" stops being abstract:

```yaml
      # Drop this debugging step into any job when you're confused about the environment:
      - run: |
          pwd                # /home/runner/work/<repo>/<repo> — where checkout put you
          whoami             # runner
          node --version     # whatever setup-node installed
          df -h /            # tens of GB free — plenty for builds
          printenv | grep GITHUB_ | sort | head -20    # the context, as real env vars
      # Every value here is identical on every run — THAT sameness is what makes
      # CI results trustworthy where "works on my machine" isn't.
```

A status badge for your README (grab the exact snippet from the workflow page's "..." menu → Create status badge):

```markdown
![CI](https://github.com/<user>/<repo>/actions/workflows/ci.yml/badge.svg)
```

And useful local tooling — the GitHub CLI you met in the git track speaks Actions:

```powershell
gh run list                 # recent runs, statuses, durations
gh run view --log-failed    # dump just the failing step's log into your terminal
gh run watch                # live-follow the run you just triggered — push, then watch
```

Small upgrades you'll add to the starter workflow within a month, shown so you recognize them in the wild:

```yaml
# A manual "Run workflow" button in the Actions tab — invaluable for testing the
# workflow itself without dummy commits:
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:          # <- the button

# Cancel superseded runs on the same branch (don't test commit 3 when commit 4 exists):
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

```yaml
# Keeping evidence from a failed run — upload test output as a downloadable artifact:
      - run: npm test
      - uses: actions/upload-artifact@v4
        if: failure()               # only bother when something went wrong
        with:
          name: test-results
          path: test-results/       # whatever your runner writes
```

That `if: failure()` is your first taste of step-level conditionals — steps normally run only while everything upstream is green; `if:` bends that when you have a reason.

## Common Pitfalls

1. **"It passes on my machine" but fails in CI — and blaming CI.** The runner installed from the lockfile on clean Linux; your machine has globally-installed tools, uncommitted files, stale `node_modules`, a case-insensitive filesystem. CI is *right*: your repo doesn't self-contain. Correction: reproduce with a clean clone in a temp folder (`git clone . ../fresh && cd ../fresh && npm ci && npm test`) — the missing file or undeclared dependency reveals itself. Docker gives an even closer Linux reproduction (Chapter 4 pays off).

2. **Forgetting the `pull_request` trigger.** Workflow only fires on `push` to main, so PRs merge unverified and CI reports breakage *after* it lands — precisely backwards. Correction: the standard pair, `push: [main]` + `pull_request: [main]`. The PR trigger is the one doing the guarding.

3. **CI green because it checks nothing.** `npm test` echoing "no tests specified" and exiting 0 produces a beautiful, meaningless badge. Correction: green must be earned — real lint config, at least a handful of real tests. Sanity-check by pushing a deliberately broken commit to a branch and confirming CI goes red (if it doesn't, your pipeline is decorative).

4. **Skipping branch protection because it's a solo project.** Unprotected main means CI is a suggestion — you'll merge a red PR "just this once" at some point, guaranteed. Correction: protect main (require the CI check, require PRs) even solo; it builds the exact muscle memory teams expect, and your capstone (Project 6) assumes it.

5. **Secrets pasted into workflow YAML.** The workflow file is code in the repo; a token in it is a token published. Correction: repo Settings → Secrets and variables → Actions; reference as `${{ secrets.MY_TOKEN }}`. Actions masks secret values in logs, but don't test that boundary by printing them. (Chapter 8 uses this heavily for deploys.)

6. **Re-running failures until they pass.** A flaky test that passes on attempt three didn't pass — it's a bug in the test (timing, ordering, shared state) that just burned twenty minutes and some trust in green. Correction: re-runs are for infrastructure hiccups only; flaky tests get fixed or quarantined *as their own tracked issue*, never ignored.

7. **YAML structure errors: a step outdented into the void, tabs, unquoted versions.** The workflow either doesn't trigger at all (silent — check the Actions tab for parse errors) or fails bizarrely. Correction: two-space indents, quote version numbers, and lean on editor validation — VS Code's GitHub Actions extension flags schema errors as you type.

## Practice Exercises

1. **First pipeline.** Add the starter workflow to a real project of yours with genuine lint and test scripts. Push, watch the run in the Actions tab (or `gh run watch`), and get to green honestly. Add the status badge to the README.

2. **Autopsy a red build.** On a branch, deliberately break one test and push. Practice the bottom-up reading procedure: find the failing step, quote the decisive log line in your notes, classify the failure, fix, push, confirm green. Then repeat with a lint error instead — notice how much faster it dies (that's fail-fast ordering working).

3. **Lock the door.** Enable branch protection on main requiring your CI check and PRs. Prove it works twice: try to push directly to main (rejected), and open a PR with a failing test (merge button blocked). Then fix, watch the check flip, merge. This exercise *is* professional Git workflow; make it reflex.

4. **Clean-clone debugging.** Simulate the classic CI-only failure: create a file your code imports but that's listed in `.gitignore`, verify everything works locally, push, and watch CI fail. Diagnose from the CI log alone (pretend your working copy is unavailable), then fix properly.

5. **Second language, same shape.** Add CI to a Python project (ruff + pytest). In your notes, list exactly what changed versus the Node workflow and what stayed identical — the "what transfers between CI systems" insight, in miniature.

6. **Read a production workflow.** Open the `.github/workflows/` folder of a major open-source project you use (Express, Flask, Vite...). Pick their main CI workflow and annotate a copy line by line: trigger, jobs and their parallelism/`needs`, matrix if any, caching strategy, anything you can't identify. Write down two techniques worth stealing and one question to answer as you continue the track.
