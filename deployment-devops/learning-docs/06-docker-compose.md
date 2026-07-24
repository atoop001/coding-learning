# Chapter 6: Docker Compose

## Overview

Real applications are rarely one process. Even a modest web app is an app server *plus* a database, and often a cache or an admin tool besides. You could manage that with a pile of `docker run` commands — remembering the right flags, starting things in order, wiring up networking by hand — but that pile becomes unreadable by the third flag and unshareable immediately. **Docker Compose** replaces the pile with one declarative file, `compose.yaml`, that describes your whole stack: which containers, built from what, connected how, storing data where, configured with which variables. Then `docker compose up` brings the entire system to life, and `docker compose down` removes it, every time, identically, on any machine. This is transformative for development teams — "clone the repo, run `docker compose up`" is the modern replacement for a week of environment-setup wiki pages — and it's the conceptual gateway to how multi-service systems are described everywhere (CI services, Kubernetes manifests, PaaS blueprints all rhyme with it).

## Definitions & Explanations

**Docker Compose** — a tool (bundled with Docker Desktop; invoked as `docker compose`, a subcommand — the old separate `docker-compose` binary with a hyphen is the legacy V1 form) that reads a YAML file describing multiple containers and manages them as a unit: create, start, stop, rebuild, view aggregate logs.

**`compose.yaml`** — the stack description file, in the project root, committed to the repo. (Docker also accepts the older name `docker-compose.yml`; both work, `compose.yaml` is the modern preference.) Like the Dockerfile, it turns environment knowledge into versioned code.

**Service** — Compose's central noun: a named definition of a container — its image (or build instructions), env vars, ports, volumes, dependencies. "Service" rather than "container" because the definition could scale to several identical containers; for you, one service = one container is the right mental model.

**YAML** — the config language Compose uses. Indentation-is-structure (spaces only, never tabs), `key: value` maps, `- item` lists. You've met it in passing; Chapter 7's GitHub Actions files are YAML too, so fluency pays double. Its classic trap: unquoted values that YAML "helpfully" retypes (`no` becomes boolean false; `3.10` becomes the number 3.1) — quote strings when in doubt.

**Compose network** — Compose automatically creates a private network for the stack and attaches every service. On it, **service names are hostnames**: the app reaches the database at `db:5432`, not `localhost:5432` — because from inside the app's container, `localhost` is *that container*, not the database. This one line of understanding resolves the most common Compose confusion in existence.

**`depends_on`** — declares start *ordering* ("start `db` before `app`"). Crucially, plain `depends_on` waits only for the container to be *started*, not for the software inside to be *ready* — Postgres takes a few seconds to accept connections after its container starts. The fix is a **healthcheck** (a command Compose runs repeatedly inside a service to probe readiness) plus `depends_on` with `condition: service_healthy`.

**Named volume (in Compose)** — same concept as Chapter 4, declared in a top-level `volumes:` block and referenced by services. This is what makes `docker compose down` followed by `up` preserve your database. The destructive variant `down -v` deletes volumes too — respect the flag.

**Bind mount (in Compose)** — mapping project source into a container (`./src:/app/src`) so code edits on your machine appear instantly inside — the backbone of the **compose dev environment** pattern: run the app in a container (identical environment for everyone) while editing files natively with hot reload.

**`env_file` and variable interpolation** — services can load env vars from a file (`env_file: .env`), and the compose file itself can interpolate host-environment values with `${VAR}` syntax (with defaults: `${PORT:-3000}`). Same Chapter 3 rules apply: the `.env` holding real values stays gitignored; the compose file, which references but doesn't contain secrets, is committed.

**Project name & isolation** — Compose namespaces everything (containers, network, volumes) with a project name derived from the folder. Two different projects' stacks coexist without collision — `myblog-db-1` and `shopapp-db-1` are strangers to each other.

**`docker compose run`** — start a *one-off* container from a service definition to run a single command, with the service's env vars, network, and volumes already wired: `docker compose run --rm app npx prisma migrate dev`. The difference from `exec`: `exec` enters an *already-running* container; `run` creates a fresh temporary one — which works even when the stack is down, and is the idiomatic way to run migrations, seeds, and maintenance scripts against your compose stack.

**Restart policies in Compose** — the same `restart:` values as `docker run` (`no`, `always`, `unless-stopped`, `on-failure`), set per service. For dev stacks, `unless-stopped` means your database quietly returns when Docker Desktop starts — one less morning ritual.

**Profiles / overrides (awareness level)** — Compose can vary the stack per context: `--profile` flags to include optional services (like an admin tool), and an override file (`compose.override.yaml`, auto-merged when present) traditionally holds dev-only additions like bind mounts. Know these exist; reach for them when a real need appears.

## Code Examples

The archetypal two-service stack — an app you build, a database you pull:

```yaml
# compose.yaml
services:
  app:
    build: .                    # build the image from ./Dockerfile (Chapter 5's!)
    ports:
      - "3000:3000"             # host:container — same rule as docker run -p. Quote these:
                                # unquoted times like 08:00 are a classic YAML mis-parse.
    environment:
      DATABASE_URL: postgres://postgres:localdevonly@db:5432/myapp
      #                        hostname is the SERVICE NAME ^^ — not localhost!
      NODE_ENV: development
    depends_on:
      db:
        condition: service_healthy   # wait for real readiness, not mere start
    restart: unless-stopped

  db:
    image: postgres:16          # pulled, not built; version pinned
    environment:
      POSTGRES_PASSWORD: localdevonly     # fine for LOCAL DEV only — prod uses injected secrets
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data   # named volume = data survives down/up
    healthcheck:                          # how Compose knows the DB is truly ready:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 2s
      timeout: 2s
      retries: 15
    # NOTE: no ports! The app reaches db over the internal network without any
    # published port. Publish "5432:5432" only if a tool ON YOUR HOST needs in.

volumes:
  pgdata:                       # declaring the named volume
```

The daily commands, in PowerShell (identical in bash):

```powershell
docker compose up               # build if needed, start everything, stream all logs (Ctrl+C stops stack)
docker compose up -d            # same, detached
docker compose up -d --build    # force image rebuild — REQUIRED after Dockerfile/code changes,
                                # because `up` alone happily reuses the stale image

docker compose ps               # stack status, health included
docker compose logs -f app      # follow one service's logs (or omit the name for all, interleaved)
docker compose exec app sh      # shell inside a running service
docker compose exec db psql -U postgres myapp    # database prompt, zero client installs

docker compose down             # stop and remove containers + network; VOLUMES SURVIVE
docker compose down -v          # ...and volumes too. This deletes your data. On purpose only.

docker compose config           # print the fully-resolved file — first stop for YAML/interpolation bugs
```

Verifying the networking model — worth doing once with your own hands:

```powershell
docker compose exec app sh
# now inside the app container:
#   ping db          -> resolves! Compose DNS maps service names to container IPs
#   exit
```

The dev-environment pattern — hot reload inside a container via bind mounts:

```yaml
# excerpt: a dev-flavored app service
  app:
    build: .
    command: npm run dev        # override the image's production CMD for dev
    ports:
      - "3000:3000"
    volumes:
      - ./src:/app/src          # your edits appear in the container instantly
      - /app/node_modules       # anonymous volume trick: keeps the image's Linux-built
                                # node_modules from being shadowed by your Windows one
    environment:
      NODE_ENV: development
```

Reading a stack's wiring at a glance — the questions to ask of any compose file you open (your own in six months, or a stranger's in Project 6):

```text
For each service:   built or pulled? (build: vs image:)
                    reachable from the host? (ports: present?)
                    what does it store, and does a volume protect it?
                    what config does it expect, and where do values come from?
For the stack:      who depends on whom, and is readiness (not just start) gated?
                    what would `down -v` destroy?
Answering these six questions IS understanding the stack. Practice until it
takes under two minutes.
```

(Windows note: file-change events across the bind-mount boundary are sometimes missed; if hot reload doesn't fire, most dev servers accept a polling option, e.g. Vite's `usePolling` — or keep the project under the WSL2 filesystem as Chapter 4 discussed.)

Adding an optional admin tool — a taste of a third service:

```yaml
  adminer:                      # lightweight web UI for databases
    image: adminer
    ports:
      - "8081:8080"
    depends_on:
      - db
# browse to http://localhost:8081, server: db, user: postgres — the service-name
# lesson again: Adminer connects to "db", because Adminer is ALSO on the compose network.
```

One-off commands against the stack — the `run` pattern that carries migrations, seeds, and scripts:

```powershell
# A temporary container from the app service, correct env and network included,
# removed when done (--rm). The stack doesn't even need to be up:
docker compose run --rm app npx prisma migrate dev
docker compose run --rm app node scripts/seed.js

# Contrast with exec (requires the service to be RUNNING):
docker compose exec app node scripts/seed.js
```

The end-of-day and start-of-day rhythm, honestly stated — this is what living with Compose feels like:

```powershell
# Morning:
docker compose up -d          # seconds; volumes mean yesterday's data is present
docker compose ps             # everything healthy?

# After changing app code that's baked into the image:
docker compose up -d --build app     # rebuild JUST the app service

# After changing only compose.yaml (env var, port):
docker compose up -d          # compose diffs desired vs actual and recreates only what changed

# Evening (or never — it's fine to leave it running):
docker compose down           # or just let Docker Desktop sleep with it
```

That "diffs desired vs actual" behavior is worth noticing: the compose file is a *declaration* of the state you want, and the tool reconciles reality toward it. You'll meet this exact philosophy again, scaled up, in every infrastructure tool that matters (Kubernetes, Terraform) — Compose is where the mental model gets installed.

## Common Pitfalls

1. **Connecting to the database at `localhost` from the app service.** The single most common Compose error: `ECONNREFUSED 127.0.0.1:5432` while the DB runs perfectly. Inside the app container, localhost is the app container. Correction: service name as hostname (`db:5432`) in any URL used *by containers*; `localhost:5432` only in tools running on your actual host — and only if you published the port.

2. **Trusting bare `depends_on` for readiness.** App starts, DB container is "up" but Postgres is still initializing, app crashes on first connect — "works when I restart it," the telltale symptom. Correction: healthcheck on the DB + `condition: service_healthy`; optionally make the app retry its initial connection anyway (production networks hiccup too).

3. **Editing code or the Dockerfile and running plain `up` again.** Compose reuses the existing image; your changes aren't in it; confusion follows. Correction: `docker compose up -d --build` after any change to code baked into the image (dev bind mounts exist precisely to escape this loop for source files).

4. **`down -v` as a reflexive cleanup.** The `-v` deletes named volumes — your database — permanently. Correction: plain `down` for daily use; `down -v` is a deliberate "reset the world" action you type slowly. If data matters beyond an experiment, know how you'd back it up first (Chapter 10).

5. **YAML indentation and typing errors.** A service indented one level too far, a tab character, an unquoted `8080:80` mis-parsed — Compose errors can be cryptic. Correction: two-space indentation, quote port mappings and anything ambiguous, and use `docker compose config` to see what Compose actually understood — it validates and prints the resolved file.

6. **Publishing every port out of habit.** Exposing the database on `5432:5432` puts it one bad firewall away from the internet and invites conflicts with a natively-installed Postgres. Correction: publish only what the *host* genuinely needs (usually just the app); inter-service traffic never needs published ports. If host port 5432 is taken, map `"5433:5432"` — host side is your choice.

7. **Real secrets committed inside `compose.yaml`.** `POSTGRES_PASSWORD: localdevonly` is acceptable for a local dev database holding fake data; production credentials in a committed file are not. Correction: interpolate from the environment (`${DB_PASSWORD}`) or `env_file: .env` (gitignored) the moment a value is actually secret — and remember compose files are for dev/staging conveniences; production platforms have their own injection (Chapter 8–9).

## Practice Exercises

1. **Assemble the archetype.** Take your containerized app from Chapter 5's exercises and write a compose file pairing it with Postgres: named volume, healthcheck-gated `depends_on`, env-var wiring, no published DB port. One `docker compose up -d` must yield a working app that talks to the DB. Then run the full survival test: `down`, `up`, data intact; `down -v`, `up`, data gone. Record both observations.

2. **Networking autopsy.** With the stack up: exec into the app container and reach the DB by service name; then try to reach the DB from PowerShell on your host and observe the failure (no published port). Publish it, retry, succeed. Write three sentences distinguishing the two network vantage points.

3. **Break readiness on purpose.** Remove the healthcheck and `condition`, replace with bare `depends_on`, and `down`/`up` repeatedly until you catch the race (add an artificial delay to Postgres startup if needed — or just watch the logs on a cold volume init). Reinstate the fix. Keep the failing log excerpt in your notes as future pattern-recognition material.

4. **Third service.** Add Adminer (or pgAdmin) to the stack, use it through the browser to inspect your tables, and explain in one sentence per service where its config values come from and which of them are secrets.

5. **Dev-mode variant.** Create a dev flavor of your app service (bind-mounted source, dev command, the node_modules volume trick if Node) and demonstrate an edit on your machine hot-reloading inside the container. Document any Windows file-watching friction you hit and what fixed it.

6. **Read a stranger's stack.** Find any open-source project with a compose file of 3+ services (search GitHub for `compose.yaml` in projects you recognize). Without running it, write a plain-English narration: each service's role, what talks to what over which hostname, what persists where, and what you'd type to bring it up. This is Project 6 training in miniature.
