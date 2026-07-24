# Project 4: Compose a Full Stack

## Description

Turn a multi-piece application — your app plus a real database, plus optionally a database admin tool — into a stack that one command brings fully to life on any machine with Docker: `docker compose up`. Data survives restarts because you put it in volumes on purpose; services find each other by name on the compose network; configuration flows in through environment variables exactly as Chapter 3's contract demands; and the app doesn't crash-race the database at startup because you gated it on real readiness. This is the project where the pieces of the track's first half snap together into a system — and the deliverable ("clone, one command, working stack") is precisely what senior engineers mean when they say a repo has a good developer experience.

## Difficulty & Estimated Effort

Intermediate — 4–6 hours.

## Chapters Used

- Chapter 6: Docker Compose (core)
- Chapter 3: Configuration & Secrets (core — the env-var wiring is half the project)
- Chapters 4–5 (assumed: you'll reuse your Project 2 image/Dockerfile)

## Requirements

- [ ] Start from a project of yours that talks (or will now talk) to a real database — your Project 2 app is the natural choice; add database usage to it if it has none (a single table it reads/writes is enough).
- [ ] Write a compose file defining at minimum: your app service (built from your Dockerfile) and a pinned-version database service (Postgres recommended).
- [ ] Wire the app to the database *by service name* over the compose network. The database URL must arrive in the app via environment variables — never hardcoded.
- [ ] Do not publish the database port to the host in the default setup. Document (in a compose-file comment) exactly when and why you'd temporarily publish it.
- [ ] Give the database a named volume. Prove persistence with the full survival test: create real data → `down` → `up` → data intact; then `down -v` → `up` → data gone. Record both results and the exact commands.
- [ ] Add a healthcheck to the database service and gate the app's startup on `service_healthy`. Demonstrate you understand what it prevents: describe (or capture) the failure mode with bare `depends_on`.
- [ ] Route all tunable configuration through environment variables with interpolation or an `env_file` — the compose file itself must contain no value you'd mind committing. Ship a `.env.example`; gitignore the real `.env`.
- [ ] Make the whole stack come up, from a fresh clone in a clean folder, with a single `docker compose up -d` (plus at most copying `.env.example` to `.env`). Test this literally: clone your own repo to a new directory and follow your own README.
- [ ] Ensure the app initializes its schema reproducibly on a fresh database (migration tool, init script, or the database image's init hooks — your choice, but it must be automatic or a single documented command, not hand-typed SQL).
- [ ] Demonstrate the operational toolkit on your stack, capturing each: `compose ps` (with health states), `compose logs` for one service, `compose exec` into the app, `compose exec` a database prompt, and `compose config` resolving your interpolations.
- [ ] Write the README section: prerequisites, first-run steps, the one command, how to see logs, how to reset the database deliberately, and a warning about which command destroys data.
- [ ] (Optional but recommended) Add a third service — Adminer or pgAdmin — and use it through the browser to inspect your data. If you add it, it too follows the rules: config via env, sensible ports, documented.

## Hints

- The chapter's archetype compose file is the correct skeleton. Your work is the wiring: *your* image, *your* env contract, *your* schema init — and making the fresh-clone test actually pass, which is where the honest surprises live.
- When the app can't reach the database, Chapter 6 pitfall 1 is the culprit until proven otherwise. Check what hostname your connection string actually uses before debugging anything else.
- `ECONNREFUSED` at first boot but fine on restart is the readiness race wearing its usual disguise — that's your healthcheck requirement earning its place.
- If `up` doesn't reflect your latest code changes, remember which command rebuilds images and which happily reuses stale ones.
- `docker compose config` is the best YAML debugger you own: it shows what Compose *understood*, resolved interpolations included, before anything runs.
- For schema init, smallest thing that works: Postgres images execute scripts they find in a special init directory — but only on a *fresh* volume, which interacts with your survival-test requirement in a way worth understanding rather than fighting.
- The fresh-clone test fails for almost everyone the first time (a file that existed only locally, an undocumented step, a gitignored necessity). That failure is the most valuable moment in the project — it's Project 6's core skill arriving early.

## Stretch Goals

- Add a dev-mode variant (override file or profile) with bind-mounted source and hot reload inside the container — plus notes on any Windows file-watching friction and its fix.
- Add a fourth service your app genuinely uses (Redis for caching/sessions is the classic) and wire it with the same discipline.
- Write a `Makefile`-style task runner file (or PowerShell script) exposing `up`, `down`, `logs`, `reset-db` as memorable one-worders for your future self.
- Add a healthcheck to your *app* service too, and make `compose ps` show the whole stack healthy.
- Run your test suite against the composed stack (tests execute inside a container or against the published app port) — one command, full-stack verification, foreshadowing CI service containers from Chapter 10.
