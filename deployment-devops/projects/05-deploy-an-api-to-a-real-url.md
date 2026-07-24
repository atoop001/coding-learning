# Project 5: Deploy an API to a Real URL

## Description

This is the track's centerpiece: take a backend project you've already built — from the node-express or python track — and run it in production, for real, indefinitely. A PaaS hosts the service; a managed Postgres holds its data; secrets live in the platform's environment store and nowhere else; merging to main deploys automatically *only when tests pass*; a health endpoint reports honestly; an external monitor watches it; and you can answer, with evidence, the question every operations conversation starts with: "what's running right now, and how do you know?" When this project is done you have the artifact that matters most to the job hunt — a URL in your portfolio backed by a pipeline you built and can explain line by line.

## Difficulty & Estimated Effort

Advanced — 8–12 hours, sensibly spread over one to two weeks. Deploy early, then layer the pipeline, database discipline, and monitoring on top of a working baseline.

## Chapters Used

- Chapter 2: Servers & SSH (what the PaaS is doing for you)
- Chapter 3: Configuration & Secrets (the platform env store)
- Chapter 8: Continuous Delivery & Deployment (core)
- Chapter 9: Cloud Platforms (core)
- Chapter 10: Databases in Production (core)
- Chapter 11: Monitoring, Logging & Debugging (core)

## Requirements

- [ ] Choose a backend of yours with a database dependency and at least a handful of tests (your Project 3/4 project is ideal — it arrives with CI and compose already). Confirm it honors the full config contract: platform-assigned port, all-interfaces binding, everything tunable via env vars, startup validation that names missing keys.
- [ ] Pick a PaaS (Render, Railway, or Fly.io) *after* reading its current pricing page; record in your notes which free-tier gotchas (sleep-on-idle, database expiry, credit burn) apply to you and what you'll do about each.
- [ ] Provision a managed Postgres on the platform, same region as the service will use. Identify its internal vs external connection URLs and write one sentence on where each will be used.
- [ ] Deploy a first working version — dashboard-driven, auto-deploy on, no pipeline yet. Milestone: your API answers requests at the platform URL. Do not proceed until it does; everything after builds on a working baseline.
- [ ] Configure all secrets and config through the platform's environment store. The repo must contain zero production values — audit it (`git log` included) and state the result.
- [ ] Make schema changes flow through migrations, run automatically as part of each deploy (the platform has a hook for exactly this — find it). Verify by shipping one real migration and confirming from deploy logs that it applied before the new code served traffic.
- [ ] Now take ownership of the deploy decision: disable the platform's blind auto-deploy (or configure it to wait on CI) and wire deploy-on-green from GitHub Actions — a deploy job gated by `needs:` on your test job, authenticated via a secret stored in an Actions **environment**, triggering the platform by hook or CLI. A red test run must provably prevent deployment; demonstrate it.
- [ ] Ensure the deploy step can fail loudly: a bad hook URL or failed platform deploy must turn the workflow red, not lie green. Test this deliberately, then fix the sabotage.
- [ ] Add `/health` (liveness) and a readiness variant that actually probes the database; point the platform's health-check setting at it. Return honest status codes and leak no internals.
- [ ] Set up structured, leveled logging to stdout with `LOG_LEVEL` as config; find where the platform surfaces your logs and practice filtering them (capture one example of finding a specific request's error).
- [ ] Add an external uptime monitor (free tier) on the health endpoint with alerts to your email. Verify the alert path fires by breaking something on purpose, briefly.
- [ ] Run the rollback fire drill while healthy: deploy a visible change, roll it back via the platform mechanism, time it; then fix-forward via revert + pipeline. Record both durations and when you'd choose each.
- [ ] Run the backup drill: dump the production database, restore into a scratch local container, verify with row counts. Note your worst-case data-loss window given the platform's automated backup terms on your tier.
- [ ] Write `OPERATIONS.md` in the repo — your runbook: the URL, architecture sketch, where config lives, how a change reaches production (every gate in order), how to read logs, how to roll back, how to restore data, and the monitor/alert setup. Written so that a competent stranger could operate the service from it.

## Hints

- Sequence is strategy here: working baseline first (deploy by dashboard, day one), *then* replace conveniences with your own pipeline piece by piece. Attempting the full pipeline before the app runs on the platform multiplies every unknown.
- If the first deploy fails, the platform's build/deploy log is the whole story — read bottom-up. The three usual suspects, in order: port/binding (Chapter 9 pitfall 1), missing env var (your startup validation should name it — that's why you wrote it), region/URL mismatch on the database.
- The internal-vs-external URL distinction bites during the backup drill: your laptop is outside the platform's network. Chapter 10's examples used the external URL there for a reason.
- One doorway into production: if deploys ever seem to happen twice or from nowhere, Chapter 8's auto-deploy-vs-pipeline pitfall is the checklist.
- Free-tier sleep will make your uptime monitor's numbers weird (each probe may wake the service, or record slow responses). Deciding how to handle that tension — accept, pay, or tune probe interval — is a genuinely instructive judgment call; write down what you chose.
- The rollback and backup drills feel skippable because everything works. They are the difference between this being a deployment and being *operations experience* — which is the phrase your résumé wants.
- Budget honesty: this can be $0 on free tiers with the documented gotchas, or a few dollars monthly to remove them. Either is fine; knowing *why* you chose is the lesson.

## Stretch Goals

- Attach a custom domain (Chapter 12) with apex + www, canonical redirect, and the four-layer verification battery; update the monitor to watch the real domain.
- Add error tracking (Sentry-style) with release tagging tied to your Git SHA, and prove a thrown test error arrives grouped, with the correct release attributed.
- Add a staging environment: a second service instance + second database deploying from a `staging` branch, with production promotion happening only via PR from staging to main — a real two-environment pipeline.
- Ship the containerized version: make the pipeline build and push your Docker image (SHA-tagged, from Project 2's Dockerfile) and have the platform run the image instead of building from source; write two sentences on what this buys.
- Do a load sanity-check: point a simple load tool at the free-tier service, find where it degrades, and record what the platform's paid tier would change — capacity planning in miniature.
