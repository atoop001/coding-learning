# Chapter 8: Continuous Delivery & Deployment

## Overview

Chapter 7 gave you a robot that *verifies* every change. This chapter gives the robot the keys: when tests pass on main, the change ships to production — automatically, in minutes, with no human ceremony. That's **Continuous Deployment**, and to someone who hasn't lived it, it sounds reckless; in practice it's the opposite. Teams that deploy on every green merge ship tiny, easily-reversed changes many times a day, while teams that deploy quarterly ship enormous, terrifying bundles where any of three hundred commits could be the bug. Small batches, always releasable, rollback in one click — that's the deploy-on-green mindset, and it's backed by a decade of industry evidence (the DORA research consistently finds that the teams who deploy *most often* also break *least often*). Here you'll learn the CI/CD vocabulary precisely (CI vs delivery vs deployment), how build artifacts flow through a pipeline, how to wire GitHub Actions to a deployment target with environment secrets, and — the part that makes bosses trust the whole apparatus — how rollbacks work.

## Definitions & Explanations

**Continuous Delivery (CD)** — the discipline of keeping main *always deployable*: every green merge produces a verified, deploy-ready artifact, and shipping it is a single button-press whenever a human chooses. The pipeline is fully automated *up to* the release decision.

**Continuous Deployment (also CD, confusingly)** — one step further: remove the button. Every green merge to main deploys to production automatically, no human in the loop. The distinction to keep crisp: **delivery = automated up to a manual release decision; deployment = the release decision is automated too.** When someone says "CD," it's worth asking which they mean. This track builds toward continuous deployment, because for your solo projects there's no reason to keep the button.

**Pipeline** — the full automated path a change travels: push → build → test → (deliver artifact) → deploy → verify. In GitHub Actions terms: one or more workflows whose jobs are chained with `needs:`, deploy stages gated on test stages.

**Build artifact (in CD context)** — the immutable output of the build stage that flows *unchanged* through the rest of the pipeline: a Docker image, a `dist/` bundle, a zip. The governing principle: **build once, deploy that exact artifact everywhere.** Rebuilding per environment invites "staging tested one build, production ran another" — a bug class you eliminate entirely by promoting a single artifact.

**Immutable tagging** — artifacts get identified by something that can never move: the Git commit SHA (`myapp:3f2a91c`) rather than only `latest`. Now "what exactly is running in production?" has a one-word answer, and rollback means "run the previous SHA" — trivially precise.

**DORA metrics (awareness tier)** — the research-backed yardstick for delivery performance, from the annual State of DevOps studies: deployment frequency, lead time for changes (commit → production), change failure rate, and time to restore. The headline finding, worth quoting in interviews because it's counterintuitive and true: the four metrics improve *together* — teams that ship fastest also break least, because the same practices (small batches, automation, fast feedback) drive all four. This chapter is you installing those practices at personal scale.

**Deploy hook / API deploy** — how an external pipeline tells a platform to deploy. Simplest form: a **deploy hook** — a secret URL the platform gives you; an HTTP POST to it triggers a deploy (Render and Railway both offer these). Richer form: platform CLIs and official GitHub Actions (e.g., Fly.io's `flyctl deploy` in a workflow step). Either way the shape is identical: *deploy job runs after tests pass and calls the platform with a credential stored as a secret.*

**Git-push auto-deploy** — the PaaS convenience where the platform watches your repo and deploys every push to main *by itself* (how Project 1's static hosts worked). Perfectly fine — but note who's in control: the platform deploys whether or not your CI passed, unless the platform is configured to wait for checks or you disable auto-deploy and trigger from your own pipeline. Understanding this control-flow difference is the difference between having CD and merely having a deploying repo.

**Environment secrets (GitHub Actions)** — Chapter 7 introduced repo-level secrets; Actions also has **Environments** (Settings → Environments): named contexts like `production` holding their own secrets and *protection rules* (required reviewers — a manual-approval gate, i.e., continuous *delivery* — wait timers, branch restrictions). A deploy job declares `environment: production` and gets that environment's secrets, its deploy history, and its gates.

**`GITHUB_TOKEN`** — an automatic, short-lived credential every workflow run receives, scoped to the repo. It's how workflows push to GHCR (GitHub's container registry) or comment on PRs without you creating anything. Know it exists; grant it only the permissions a workflow needs.

**Rollback** — restoring the previously-good version after a bad deploy. With immutable artifacts it's *redeploying the old artifact* — one click in Render's deploy history, `flyctl releases` + rollback on Fly.io, or re-running your deploy job pinned to the prior SHA. What rollback is **not**: `git revert` + wait-for-pipeline as your *only* option (that's minutes of downtime you didn't need), or hand-editing the server (Chapter 2 already burned that bridge). Design rule: **know your rollback move *before* you need it** — a deploy process without a practiced rollback is a trap that hasn't sprung yet. Caveat that matters early: rollbacks revert *code*, not *database schema* — Chapter 10 handles that interaction.

**Blue-green & canary deployment (awareness tier)** — the two release strategies you'll hear named in job conversations. **Blue-green**: run two identical production environments; deploy to the idle one; flip traffic instantly (and flip back just as instantly — rollback as a switch). **Canary**: send a small slice of traffic (1–5%) to the new version, watch error rates, then widen. PaaS platforms give you a light version of blue-green automatically (the zero-downtime switchover from Chapter 1); full canary machinery is big-company territory. Concepts to speak, not tools to build yet.

**Feature flags (awareness tier)** — decoupling *deploying code* from *releasing features*: ship the new code dark behind an `if (flags.newCheckout)`, turn it on later (for everyone, or 5% of users) without a deploy, turn it *off* in seconds if it misbehaves — a rollback that doesn't touch infrastructure at all. Even a homegrown env-var flag delivers the core benefit; know the pattern exists before someone asks how you'd ship a risky change.

**Deploy-on-green mindset** — the cultural core: main is sacred (always releasable), everything ships through the pipeline (no side doors), deploys are boring (small, frequent, reversible), and a failed deploy is a pipeline bug to fix, not evidence that automation "doesn't work here." Fear of deploying is a symptom; the treatment is deploying *more often, in smaller pieces*, not less.

## Code Examples

First, the whole chapter as one diagram — the pipeline shapes side by side:

```text
CI only (Chapter 7):
  push/PR -> [install -> lint -> test] -> green check. Humans deploy, whenever, however.

Continuous DELIVERY:
  merge to main -> [test] -> [build artifact] -> ready & waiting
                                   -> a HUMAN clicks "release" -> production

Continuous DEPLOYMENT:
  merge to main -> [test] -> [build artifact] -> [deploy] -> [verify health]
  no hands. Merge IS the release decision. Red anywhere = the change never ships.
```

The canonical deploy-on-green workflow — tests gate, main-only, secret-authenticated:

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]          # merges to main only — PRs get CI, not deploys

jobs:
  test:                       # same verification as ci.yml — never deploy unverified code
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm test

  deploy:
    needs: test               # THE gate: this job exists only if `test` succeeded
    runs-on: ubuntu-latest
    environment: production   # binds to the Actions Environment: its secrets, history, and any approval rules
    steps:
      - name: Trigger Render deploy
        run: curl -fsS -X POST "${{ secrets.RENDER_DEPLOY_HOOK_URL }}"
        # -f: non-2xx responses become failures (otherwise curl exits 0 and you get a lying green)
        # The hook URL is itself a secret — anyone holding it can trigger deploys.
```

Where that secret lives: repo → Settings → Environments → New environment `production` → add secret `RENDER_DEPLOY_HOOK_URL` (value from Render's dashboard → your service → Settings → Deploy Hook). While there, notice the protection rules — adding "Required reviewers: you" converts this pipeline from continuous deployment to continuous delivery with one checkbox. That's the entire practical difference between the two terms.

Building and pushing an immutably-tagged Docker image to GHCR — the artifact half of the story:

```yaml
  build-image:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write          # allow GITHUB_TOKEN to push to GHCR
    steps:
      - uses: actions/checkout@v4
      - name: Log in to GHCR
        run: echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - name: Build and push, tagged with the commit SHA
        run: |
          docker build -t ghcr.io/${{ github.repository }}:${{ github.sha }} .
          docker push ghcr.io/${{ github.repository }}:${{ github.sha }}
        # ${{ github.sha }} = the exact commit — the immutable tag.
        # Production runs THIS; rollback = deploy the previous SHA. No archaeology.
```

A deploy step using a platform CLI instead of a hook (Fly.io's official pattern):

```yaml
      - uses: superfly/flyctl-actions/setup-flyctl@master
      - run: flyctl deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
        # Same shape as the hook version: gated job + platform credential from secrets.
        # Every platform's variant of this differs only in spelling.
```

Making "what is live?" answerable from the outside — a tiny version endpoint pays for itself the first confusing incident:

```js
// Set COMMIT_SHA as an env var at deploy time (Actions has it as github.sha;
// most platforms also inject their own render/railway/fly equivalents):
app.get("/version", (req, res) =>
  res.json({ sha: process.env.COMMIT_SHA || "unknown", node: process.version })
);
// Now `curl https://myapp.example.com/version` during any incident answers the
// first question anyone asks — "is the bad deploy actually live yet?" — in one second.
```

And the operational commands you'll actually use around deploys:

```powershell
gh run watch                        # follow the pipeline after merging a PR
gh run list --workflow=deploy.yml   # deploy history from the terminal

# "What is production running right now?" — if you deploy by SHA, answer instantly:
git log --oneline -5                # match the deployed SHA to a commit message
```

Closing the loop — a deploy job that *verifies* the deployment it just triggered, instead of trusting it:

```yaml
      - name: Trigger deploy
        run: curl -fsS -X POST "${{ secrets.RENDER_DEPLOY_HOOK_URL }}"

      - name: Wait for the new version to answer
        run: |
          # Poll the health endpoint (Chapter 11 builds it) until it's green or we give up.
          for i in $(seq 1 30); do
            if curl -fsS https://myapp.onrender.com/health; then
              echo "healthy"; exit 0
            fi
            echo "not yet ($i)"; sleep 10
          done
          echo "deploy never became healthy"; exit 1
          # A red result here means: deploy went out, site isn't serving. That's
          # exactly the moment you want a red workflow instead of a false green.
```

(That's bash, because the runner is Linux — one more reason the Unix half of the command-line track keeps paying rent.)

Rollback, concretely, in the two forms you'll meet first:

```text
PaaS dashboard route (Render): service → "Deploys" tab → previous successful deploy
  → "Rollback to this deploy". Restores the previous BUILD (artifact), not a rebuild. ~1 minute.

Pipeline route: GitHub → Actions → deploy workflow → the last good run → "Re-run all jobs"
  (redeploys that commit's artifact), or `git revert <bad-sha>` + push for a
  history-honest fix-forward. Know both; prefer the fastest during an incident,
  the honest one after the smoke clears.
```

## Common Pitfalls

1. **Deploying whatever's on main without re-verifying it.** A deploy workflow with no test job (or no `needs:`) will happily ship a broken direct-push. Correction: the deploy job always sits behind `needs: test`, and branch protection (Chapter 7) makes unverified pushes to main impossible in the first place — belt and suspenders.

2. **A deploy step that can't fail.** `curl` without `-f` returns exit 0 on an HTTP 500; your pipeline glows green while nothing deployed. Correction: every deploy step must be able to report failure — `-fsS` on curl, check CLI exit codes, and ideally verify after deploying (poll the health endpoint — Chapter 11 formalizes this).

3. **Deploy credentials in the workflow file, or in repo secrets when they should be environment secrets.** The hook URL *is* a credential. Repo-level secrets are exposed to every workflow, including ones a future PR modifies. Correction: production credentials live in the `production` Environment, whose rules can also restrict which branches may use it. Rotate any credential that ever landed in a file.

4. **Rebuilding the artifact at each stage.** Test job builds an image, deploy job builds *again* — different timestamps, possibly different dependency resolutions; staging verified a sibling, not the twin. Correction: build once, push tagged-by-SHA, deploy by tag. If two stages need the artifact, they *share* it (registry, or Actions' upload/download-artifact), never recreate it.

5. **First rollback attempted during the first incident.** You're reading platform docs about rollback while production is down — worst possible timing. Correction: practice the rollback within an hour of setting up any deploy pipeline, while everything is healthy (Exercise 4). An unpracticed rollback plan is a rumor, not a plan.

6. **Deploying enormous, rare batches "to be safe."** Fifty commits ship together; something breaks; which commit? The whole batch rolls back, including five good features. This is the fear-driven anti-pattern the entire chapter argues against. Correction: merge small, deploy every merge. When each deploy is one small change, cause and effect are adjacent, and rollback costs almost nothing.

7. **Auto-deploy fighting your pipeline.** Platform auto-deploys on push *and* your workflow triggers a deploy — double deploys, races, confusion about which build is live. Correction: pick one owner of the deploy decision. If your pipeline triggers deploys, turn the platform's auto-deploy off (or set it to wait on CI checks if the platform supports it). One doorway into production.

## Practice Exercises

1. **Vocabulary precision.** Write, from memory, a paragraph a junior teammate could learn from distinguishing CI, continuous delivery, and continuous deployment — including which one a "required reviewer" gate on the production environment gives you. Check against the definitions and patch your gaps.

2. **Build the gate.** Extend a project that already has Chapter 7 CI with a `deploy` job stub gated by `needs: test` and `environment: production` — have it merely `echo "would deploy $GITHUB_SHA here"` for now. Verify the gate: push a branch with a failing test, open a PR, merge is blocked; fix, merge, and confirm the deploy job ran on main only (not on the PR).

3. **Immutable artifact drill.** Add the GHCR build-and-push job. After two or three merges, list the image tags on your repo's Packages page, pick the *previous* SHA, and `docker run` it locally. Write one sentence on why answering "what was live yesterday?" just took ten seconds.

4. **Rollback fire drill (while healthy).** With any deployed service (Project 1's static site qualifies; Project 5 ideally): deploy a visible change, then roll it back using the platform's mechanism, timing yourself. Then do a fix-forward (`git revert`, push, pipeline redeploys). Record both durations and when you'd choose each.

5. **Break the deploy honestly.** Sabotage the deploy step (corrupt the hook URL secret) and merge a harmless change. Observe: does your pipeline actually report failure, or lie green? Fix the reporting if it lies (that's pitfall 2 live), then restore the secret. Note what alerting you'd want so a failed deploy can't go unnoticed (foreshadowing Chapter 11).

6. **Map a real pipeline.** Find the deploy workflow of an open-source app that ships from Actions (search for `deploy.yml` in .github/workflows of projects you know, or GitHub's code search). Diagram it: triggers, jobs, gates, where secrets enter, what the artifact is, how they'd roll back. Identify one thing they do that this chapter didn't cover, and find out what it's for.
