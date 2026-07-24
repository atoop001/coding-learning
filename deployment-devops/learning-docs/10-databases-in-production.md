# Chapter 10: Databases in Production

## Overview

Everything else in your deployment is replaceable in minutes: containers are cattle, app servers redeploy from Git, the whole platform could be swapped by Friday. The database is different. It is the one component that *remembers* — and therefore the one place where a mistake can be permanent. Drop a table in production and no amount of redeploying brings back your users' data; only a backup does, and only if you actually have one that restores. This chapter covers the operational half of databases that no SQL tutorial teaches: what a managed database provides, connection strings as the crown-jewel secret, how schema changes (migrations) travel through deploys without breaking running code, why backups don't exist until you've *restored* one, and the iron rule that every environment gets its own database. The material is less glamorous than Docker and pipelines. It is also the chapter most likely to save you from the kind of mistake you'd still wince about years later.

## Definitions & Explanations

**Managed database** — a database server operated by your platform (Render/Railway Postgres, AWS RDS): they handle installation, patching, disk management, automated backups, and monitoring; you get a connection string and administrative peace. Chapter 9's managed-service trade at its most worthwhile — self-hosting a production database responsibly means owning backups, tuning, upgrades, and 3 a.m. disk-full pages. Postgres note: it's the default choice in this ecosystem for good reasons (rock-solid, feature-rich, universally supported); when in doubt, Postgres.

**Connection string** — one URL carrying everything needed to reach and authenticate to a database: `postgres://user:password@host:5432/dbname?sslmode=require`. Read its anatomy: scheme, credentials, host, port, database name, options. Because the password is *embedded*, the whole string is a secret of the highest tier — it *is* your data. It travels only via Chapter 3's channels (env var `DATABASE_URL`, platform-injected), appears in no log, no error message, no commit, ever. The `sslmode=require` suffix matters off-platform: managed DBs are reached over networks you don't control; TLS is non-optional.

**Internal vs external connection URLs** — managed databases typically issue two strings: an *internal* one (reachable only from services inside the platform's network — faster, safer; what your deployed app uses) and an *external* one (reachable from the public internet — what your laptop's tooling uses, and what backups/restores from home run against). Mixing them up produces the classic "works deployed, times out locally" (or vice versa).

**Migration** — a versioned, ordered script describing one schema change: "add table `posts`", "add column `users.last_login`". Migrations live in the repo alongside the code that needs them; the migration *tool* (Prisma Migrate, Knex, Alembic, Django migrations — every stack has one) tracks which have been applied to a given database in a bookkeeping table, and `migrate` applies exactly the pending ones, in order, once. This turns "the schema" from a fact about a server into *code with history* — reviewable, reproducible on a fresh database, and applied identically in dev, CI, and prod. Hand-typed `ALTER TABLE` against production is the anti-pattern migrations exist to end.

**Running migrations during deploys** — the sequencing question every deploy pipeline must answer: *when does the schema change happen relative to the new code starting?* Standard answer: migrations run as a deploy step after the artifact is built and before (or as) the new version boots — platforms provide hooks for exactly this (Render's Pre-Deploy Command; a `release_command` in Fly's config; a dedicated pipeline step on your own infra). The subtle discipline underneath is **expand/contract (backward-compatible migrations)**: during a deploy, old code and new schema briefly coexist (and rollbacks make old code + new schema last longer). So: *expand* first (add the new nullable column; new code writes both), *contract* later (drop the old column in a future deploy once nothing reads it). Renaming a column in one shot breaks the old code the instant the migration lands — the expand/contract dance is how teams change schemas with zero downtime.

**Rollbacks vs migrations** — Chapter 8's warning, expanded: rolling back *code* is one click; rolling back a *migration* is not symmetrical. Down-migrations exist but can destroy data (un-adding a column deletes whatever was written into it), and restoring a backup loses everything written since. Consequence: treat schema changes as more permanent than code changes, which is one more argument for small, additive, expand-style migrations.

**Backup** — a copy of the data from which you can rebuild the database. Two families: **logical** (a dump file of SQL/statements — `pg_dump`; portable, human-inspectable, slower at scale) and **physical/snapshot** (platform-managed disk-level copies on a schedule, often with **point-in-time recovery** — restore to "Tuesday 14:03," bounded by the retention window). Managed platforms give you the second kind automatically (check the retention period and whether the free tier includes it — often it doesn't); `pg_dump` gives you the first kind on demand, including right before anything scary.

**Restore drill** — actually performing a restore, into a scratch database, on a calendar schedule and before every risky operation. The operations proverb this chapter most wants you to keep: **nobody needs backups; everybody needs restores.** An untested backup is a hope. The drill also measures your **RPO/RTO** informally — how much data you'd lose (time since last backup) and how long recovery takes — two terms worth recognizing in job conversations.

**One database per environment** — the iron rule: dev, staging (if any), CI, and production each get their *own* database. The moment two environments share one, every "safe local experiment" is a production incident with extra steps: test rows in real queries, a dev-machine `DELETE` wiping customers, a migration test corrupting live schema. CI deserves emphasis: tests that truncate tables are *normal test hygiene* — pointed at a shared database they're a catastrophe. The rule costs nothing (local Postgres is a Chapter 4 one-liner; CI can spin a throwaway service container) and prevents an entire genre of disaster.

**Connection pooling** — production apps don't open a fresh database connection per request (connections are expensive to establish and databases cap how many they'll hold — small managed instances often allow only ~20–100). A **pool** keeps a handful of connections open and lends them out per query. Client libraries build this in (`pg.Pool`, SQLAlchemy's pool) — your job is knowing the cap exists: the error `too many connections` in production means your pool size × instance count exceeded the database's limit, a bit of arithmetic that surprises everyone exactly once.

**Read replica (awareness tier)** — a continuously-synced read-only copy of the database, used to spread read load and as a warm standby. You won't need one; you'll hear the term in every scaling conversation and should be able to say what it is and its catch (replication lag — a just-written row may take a moment to appear on the replica).

**Seed data** — scripts that populate a fresh database with plausible fake data for development. Pairs with migrations: `migrate` builds the empty structure, `seed` fills it — together they make "blow away my dev DB and rebuild" a 30-second, zero-fear operation, which is exactly the posture you want.

## Code Examples

Reading and using connection strings safely, locally and in prod:

```powershell
# Local dev: DATABASE_URL lives in .env (gitignored), pointing at your Docker Postgres:
#   DATABASE_URL=postgres://postgres:localdevonly@localhost:5432/myapp_dev
# Production: the platform injects DATABASE_URL (internal URL). Same code reads both:
```

```js
// Node — the app knows only "my environment provides DATABASE_URL":
const { Pool } = require("pg");
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
// NEVER: console.log("connecting to", process.env.DATABASE_URL)  <- password into logs
```

Talking to databases from PowerShell without installing anything — Docker as a client toolbox (Chapter 4's borrowed-runtime trick, now earning rent):

```powershell
# psql against your local compose Postgres:
docker compose exec db psql -U postgres myapp

# psql against a REMOTE managed DB (external URL), no local install:
docker run -it --rm postgres:16 psql "<external-connection-url>"
```

Migrations in practice — Prisma spelling shown; every tool has these same three verbs:

```powershell
npx prisma migrate dev --name add-posts-table   # DEV: generate a new migration from schema changes + apply
npx prisma migrate deploy                        # PROD/CI: apply pending committed migrations, nothing else
npx prisma migrate status                        # what's applied here vs. what's in the repo?
# (Alembic: revision --autogenerate / upgrade head / current. Knex: migrate:make / migrate:latest / migrate:status.)
```

```text
Where `migrate deploy` runs in a Render deploy:
  service → Settings → Pre-Deploy Command: npx prisma migrate deploy
Sequence per deploy: build succeeds → pre-deploy runs migrations → new code boots.
If the migration fails, the deploy STOPS and the old version keeps serving — exactly
the failure containment you want.
```

Expand/contract, as a concrete two-deploy rename (the discipline, in miniature):

```text
Goal: rename users.name -> users.full_name without downtime.
  Deploy 1 (expand):  migration ADDS full_name (nullable); code writes BOTH, reads name.
  Backfill:           one-off script copies name -> full_name for old rows.
  Deploy 2:           code reads/writes full_name only.
  Deploy 3 (contract): migration DROPS name — days later, once you'd never roll back past 2.
One-shot RENAME instead: old code errors the moment the migration applies. That's the trap.
```

Guardrails worth building once you have a real production database — cheap paranoia that works:

```powershell
# 1. Make production visually LOUD wherever you connect to it. psql supports a
#    custom prompt; GUI tools (TablePlus, pgAdmin) support per-connection colors —
#    make prod's red. Your eyes are a real safety system; feed them.

# 2. Never keep a prod connection string in your shell history. If you must
#    connect ad hoc, paste it into the command via an env var that dies with
#    the window, and prefer read-only credentials when the platform offers them.

# 3. Before ANY manual prod operation (rare, but real): take a dump first.
docker run --rm postgres:16 pg_dump "<external-url>" -Fc -f /dev/stdout > pre-change.dump
#    Thirty seconds of insurance against the worst afternoon of your year.
```

Backup and restore with `pg_dump` — the drill itself:

```powershell
# Dump a remote managed DB to a local file (custom format: compressed, flexible):
docker run --rm -v ${PWD}:/backup postgres:16 `
  pg_dump "<external-connection-url>" -Fc -f /backup/myapp-$(Get-Date -Format yyyy-MM-dd).dump

# Restore INTO A FRESH LOCAL CONTAINER — never test restores against anything real:
docker run -d --name restore-test -e POSTGRES_PASSWORD=scratch -p 5433:5432 postgres:16
docker cp .\myapp-2026-07-24.dump restore-test:/tmp/
docker exec restore-test pg_restore -U postgres -d postgres --create /tmp/myapp-2026-07-24.dump

# Verify like you mean it — row counts and a few real records, not just "it didn't error":
docker exec -it restore-test psql -U postgres -d myapp -c "SELECT count(*) FROM users;"

docker rm -f restore-test    # drill complete; note how long it took — that's your informal RTO
```

The pool, visible in code — and the arithmetic that goes with it:

```js
const { Pool } = require("pg");
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 10,                    // this app instance holds AT MOST 10 connections
});
// The production math: max (10) × app instances (2) = 20 connections needed.
// Free-tier Postgres allowing 22 total: fine today, an outage the day you scale
// to 3 instances. Write the arithmetic down wherever you set `max`.
```

Seeding, so a fresh environment is usable in one command — the dev/prod split matters:

```js
// scripts/seed.js — plausible FAKE data for dev and staging. Guard it:
if (process.env.NODE_ENV === "production") {
  console.error("Refusing to seed production.");   // yes, someone will try. Maybe you.
  process.exit(1);
}
// ...insert demo users, sample posts...
```

```powershell
# The fresh-database ritual this chapter has been building toward — structure,
# then data, each step idempotent and scripted:
docker compose up -d db
docker compose run --rm app npx prisma migrate deploy
docker compose run --rm app node scripts/seed.js
# Total fear associated with "blow away my dev DB": zero. That's the goal state.
```

CI's throwaway database — the per-environment rule, GitHub Actions spelling:

```yaml
# excerpt from ci.yml — a Postgres that exists only for this run, then evaporates:
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: ci-only }
        ports: ["5432:5432"]
        options: >-
          --health-cmd "pg_isready -U postgres" --health-interval 5s --health-retries 10
    steps:
      # ...checkout/setup...
      - run: npx prisma migrate deploy && npm test
        env:
          DATABASE_URL: postgres://postgres:ci-only@localhost:5432/postgres
# Tests can truncate every table with abandon. Nothing real is reachable. That's the point.
```

## Common Pitfalls

1. **Connection string in a commit, a log line, or an error page.** It happens via innocence: a debug `console.log` of config, an ORM error that echoes the URL, a committed `.env`. Any of these = full data breach potential. Correction: Chapter 3's rules at maximum strictness — env-only, log config *keys* never values, and if it leaks anywhere: rotate the database password immediately (platforms have a reset button; then update the env var everywhere).

2. **Two environments, one database.** "I'll just point local dev at the prod DB to debug with real data" — and one test run or stray migration later you're explaining data loss. Correction: iron rule, no exceptions; debug with a *restored backup* locally (the drill doubles as your safe real-data supply — mind privacy if data is sensitive). If you catch a `DATABASE_URL` on your laptop pointing at production, treat it like a smoke alarm.

3. **Hand-editing production schema in a GUI/psql "just this once."** Now prod disagrees with the migration history; the next `migrate deploy` fails or, worse, half-applies. Correction: every schema change is a migration in the repo, even the one-liner, even solo. If prod has already drifted: reconcile deliberately (most tools can mark migrations as applied) rather than piling on more manual edits.

4. **Believing in backups you've never restored.** Corrupt dumps, wrong flags, missing `--create`, incompatible versions, seven-day retention you thought was thirty — all discovered at the worst possible moment, none discoverable without a drill. Correction: restore-to-scratch quarterly and before every risky operation; record the date and duration in your notes. Also learn your platform's restore *UI* before you need it under adrenaline.

5. **The one-shot destructive migration.** Renaming or dropping a column in a single deploy — old code (still serving during rollout, or after a rollback) explodes on contact with the new schema. Correction: expand/contract. Additive changes freely; destructive changes only in a later deploy, after nothing references the old shape and you'd no longer roll back past the boundary.

6. **Migrations that run "sometimes."** Applied by hand when someone remembers, so prod is three migrations behind, and the deploy that finally runs them applies a semester of changes at once. Correction: migrations are a *pipeline step*, every deploy, mechanically. `migrate status` against prod should always answer "up to date."

7. **Free-tier amnesia (Chapter 9's warning, now with your data on it).** Free managed Postgres that expires, or excludes automated backups, treated as if it were durable. Correction: know your tier's backup and lifespan terms *the day you create the database*; put `pg_dump` on a schedule for anything you'd mind losing; calendar the expiry.

## Practice Exercises

1. **Anatomy and blast radius.** Dissect a (fake) connection string into its six parts, then write the incident timeline if it appeared in a public repo: what an attacker does first, what you must rotate, in what order, and which of Chapter 3's playbook steps apply unchanged.

2. **Migration lifecycle, end to end.** In a project with a database (or a fresh scratch project), set up a migration tool and ship three sequential migrations: create a table, add a column, add an index. Run them against a *brand-new* empty database in one command; inspect the tool's bookkeeping table; run `status` before and after. Write down what the bookkeeping table is for in one sentence.

3. **Expand/contract rehearsal.** Execute the two-deploy rename from this chapter against your local stack, *with the old app version actually running* between steps: prove old code keeps working after the expand migration, then complete the transition. Document the exact moment a one-shot rename would have broken it.

4. **The restore drill, for real.** Take a `pg_dump` of any database you have (local compose counts), restore into a fresh scratch container, and verify with row counts plus three spot-checked records. Time the whole thing. Then answer: if this were production and the last dump were from Sunday, what's your data-loss window (RPO)? What would shrink it?

5. **CI gets its own database.** Add a Postgres service container to an existing CI workflow (Chapter 7's project), run migrations + tests against it, and include one test that truncates a table — the kind that makes shared databases lethal. Confirm in your notes why this test is safe here and exactly where it wouldn't be.

6. **Platform backup audit.** For the managed database from Chapter 9's exercises (or the docs of the platform you'll use in Project 5): find the automated backup schedule, retention window, restore procedure, and whether your tier includes any of it. Then do one manual restore through the platform's own mechanism into a new instance. Write the two numbers that summarize your safety: worst-case data loss, and measured time-to-restored.
