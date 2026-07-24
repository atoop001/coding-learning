# Chapter 4: Docker Fundamentals

## Overview

"Works on my machine" is the oldest joke in software, and it isn't funny when it's your deploy failing at midnight because the server has Node 18 while you developed on Node 22, or because a teammate's install is missing a system library yours happens to have. Docker attacks the problem at the root: instead of shipping code and a prayer that the destination machine is set up right, you ship a **container image** — your app *plus* its entire environment (runtime, libraries, OS userland) — that runs identically on your laptop, your teammate's laptop, CI, and production. Docker is on effectively every backend job posting for a reason: it's the common packaging format of the industry. This chapter is hands-on with *using* containers — running, inspecting, port-mapping, persisting data, and getting shells inside them — plus the Windows-specific realities of Docker Desktop and WSL2. Writing your own images comes next chapter; you can't write good Dockerfiles until running containers feels boring.

## Definitions & Explanations

**Container** — a process (or small group of processes) running on your machine's Linux kernel, but *isolated*: it sees its own filesystem, its own network interfaces, its own process list, its own hostname. It is **not** a virtual machine — no second OS is booting. It's ordinary processes wearing very good blinders, courtesy of Linux kernel features (namespaces for isolation, cgroups for resource limits). That's why containers start in milliseconds and you can run dozens where you could run a handful of VMs.

**Image** — the frozen, immutable template a container is created from: a snapshot of a filesystem (Ubuntu-ish userland + Node + your app, say) plus metadata like which command to run at start. The relationship is exactly class-to-instance from programming: **image : container :: class : instance**. One image, many containers; containers are disposable, images are the artifact you keep, version, and ship.

**Layer** — images are built as stacks of read-only layers, each recording a filesystem change. Layers are shared between images (fifty images based on the same Node base share one copy of it on disk) and cached (Chapter 5 exploits this heavily for fast builds).

**Base image** — the image another image is built on top of (Chapter 5 makes this concrete). When you pull `postgres:16`, you're getting Postgres already installed on a slim Debian userland — someone else's layers, doing your setup work.

**Official image** — Docker Hub images maintained by the software's own team or Docker (unprefixed names: `postgres`, `nginx`, `node`, `redis`). Prefer them; a random user's `coolguy/postgres-fast` is an unaudited stranger's software you'd be running with real privileges.

**Registry** — a server that stores and distributes images. **Docker Hub** is the default public one; GitHub's is **GHCR**; clouds run private ones. `docker pull` downloads from a registry, `docker push` uploads. Think "GitHub, but for images."

**Tag** — a human-readable label on an image, written `name:tag` — `node:22-alpine`, `postgres:16`. A tag is a *movable pointer*, not a version guarantee: `node:latest` points somewhere new constantly. `latest` is also just a default tag name, not necessarily the newest release — one more reason to always specify tags explicitly.

**Docker Engine / daemon (`dockerd`)** — the background service that actually builds, runs, and manages everything. The `docker` CLI is just a client sending it requests. The error `cannot connect to the Docker daemon` means the engine isn't running — on Windows, that Docker Desktop isn't started.

**Docker Desktop** — the Windows/macOS application bundling the engine, CLI, and a management GUI. Crucial Windows fact: **containers are Linux processes and need a Linux kernel**, which Windows doesn't have — so Docker Desktop runs the engine inside **WSL2** (Windows Subsystem for Linux 2), a lightweight, Microsoft-blessed Linux VM integrated into Windows. You type `docker run` in PowerShell; the container actually executes in WSL2's Linux environment. Almost all of the time this is invisible; the visible edges are file performance and memory use (see Pitfalls).

**Port mapping (publishing)** — containers have isolated networking, so a server listening on port 3000 *inside* a container is unreachable until you map it: `-p 8080:3000` means "my machine's port 8080 forwards to the container's 3000." Order is always `host:container`. This is the #1 beginner stumbling block, so tattoo the order somewhere.

**Volume** — persistent storage that outlives containers. A container's own writable filesystem is deleted with it; anything you care about (database files above all) must live in a **named volume** (Docker-managed storage, best for databases) or a **bind mount** (a host folder mapped in, best for live-editing code in development).

**`docker exec`** — run an additional command *inside* an already-running container; `docker exec -it <name> sh` gets you an interactive shell in it. This is your flashlight for "what does the world look like from inside?"

**Restart policy** — a per-container setting telling the engine what to do when the process dies: `--restart unless-stopped` (restart on crash and on machine reboot, unless you stopped it yourself) is the sane default for anything long-running. This is the container world's answer to Chapter 2's "who restarts my app at 3 a.m.?"

**Bind mount vs named volume, the decision rule** — named volume when *Docker* should own the data and you just need it to persist (databases); bind mount when *you* need to see and edit the files from Windows (source code during development). Mixing them up isn't dangerous, just awkward — database files strewn through your project folder, or source code you can't conveniently edit.

**`docker inspect`** — the everything-dump for any container, image, volume, or network: full JSON of config, env vars, mounts, network addresses, restart policy. When a flag "didn't take" or you're unsure what a container was started with, inspect answers instead of guessing.

**Ephemeral by design** — the core cultural point: containers are cattle, not pets. You never repair a container; you delete it and start a fresh one from the image. All durable state goes in volumes or external services. Internalizing this now makes every later chapter (and Kubernetes, someday) make sense.

## Code Examples

Setup check, in PowerShell, with Docker Desktop installed and running (install it from docker.com; let it use the WSL2 backend, which is the default):

```powershell
docker --version          # client exists?
docker info               # can it reach the engine? (errors here = Desktop not running)
wsl --status              # WSL2 present and default version 2
```

The traditional first contact:

```powershell
docker run hello-world
# What just happened, in order:
# 1. Engine looked for image "hello-world" locally — not found
# 2. Pulled it from Docker Hub
# 3. Created a container from it, ran it; it printed and exited
```

A real service — Postgres in one command, no installer, no Windows service, nothing on your actual machine:

```powershell
docker run -d `
  --name pg-practice `
  -e POSTGRES_PASSWORD=localdevonly `
  -p 5432:5432 `
  -v pgdata:/var/lib/postgresql/data `
  postgres:16
# -d            run detached (in the background)
# --name        a handle so we don't have to use generated IDs
# -e            env var INTO the container — Chapter 3's contract, in action
# -p 5432:5432  host port 5432 -> container port 5432
# -v            named volume "pgdata" mounted where Postgres keeps its files
# postgres:16   image name : explicit tag
# NOTE: the backtick ` is PowerShell's line continuation; in bash use \
```

The daily driver commands:

```powershell
docker ps                   # running containers
docker ps -a                # including stopped ones (crashed containers hide here!)
docker logs pg-practice     # everything it printed — your first stop when anything misbehaves
docker logs -f pg-practice  # ...streamed live (Ctrl+C stops watching, not the container)

docker exec -it pg-practice psql -U postgres    # a Postgres prompt INSIDE the container
docker exec -it pg-practice sh                  # a plain shell inside; look around; `exit` to leave

docker stop pg-practice     # graceful stop
docker start pg-practice    # ...and back, same container, data intact
docker rm pg-practice       # delete the (stopped) container — the pgdata VOLUME survives
docker run -d --name pg-practice -e POSTGRES_PASSWORD=localdevonly -p 5432:5432 -v pgdata:/var/lib/postgresql/data postgres:16
# fresh container, old volume: your data is still there. This is the whole volume lesson in one move.
```

Decoding `docker ps` output — you'll read this table thousands of times, so learn its columns once:

```text
CONTAINER ID   IMAGE         COMMAND                  STATUS                    PORTS                    NAMES
f3a91c2b1d4e   postgres:16   "docker-entrypoint.s…"   Up 2 hours (healthy)      0.0.0.0:5432->5432/tcp   pg-practice
a1b2c3d4e5f6   nginx:alpine  "/docker-entrypoint.…"   Exited (1) 5 minutes ago                           web

STATUS column, decoded:
  Up 2 hours              running fine
  Up 2 hours (healthy)    running AND its healthcheck passes (Chapter 6 adds these)
  Exited (0) ...          finished normally — fine for one-shot containers
  Exited (1) ...          CRASHED. Non-zero exit code = the process died with an
                          error. `docker logs <name>` tells you why. Always.
  Restarting (1) ...      crash-looping under a restart policy — logs, again.

PORTS column: 0.0.0.0:5432->5432/tcp reads as "host port 5432 forwards to
container port 5432". No PORTS entry = nothing published = unreachable from Windows.
```

Proving the port-mapping model with nginx:

```powershell
docker run -d --name web -p 8080:80 nginx:alpine
# nginx listens on 80 inside; we chose 8080 outside.
curl http://localhost:8080        # nginx welcome page
curl http://localhost:80          # nothing here — 80 is the CONTAINER's port, not yours
```

Housekeeping (images are big; disk fills fast):

```powershell
docker images               # what's stored locally, with sizes
docker volume ls            # volumes
docker system df            # total disk usage by images/containers/volumes
docker system prune         # delete stopped containers + unused networks/dangling images (asks first)
docker rm web; docker volume rm pgdata    # targeted cleanup when done practicing
```

Bind mounts and file traffic between host and container — the development-facing side of storage:

```powershell
# Serve YOUR local folder with nginx — a bind mount maps a host path into the container.
# ${PWD} is PowerShell's current directory (bash would use $(pwd)):
docker run -d --name mysite -p 8080:80 -v ${PWD}:/usr/share/nginx/html nginx:alpine

# Edit index.html in VS Code on Windows, refresh the browser: the change is live.
# The container is reading your actual files — no rebuild, no copy.

# One-off file copies, either direction, no mount needed:
docker cp .\notes.txt mysite:/tmp/notes.txt
docker cp mysite:/etc/nginx/nginx.conf .\nginx-default.conf   # great for studying defaults
```

Inspection and the restart policy, in practice:

```powershell
# What EXACTLY is this container running with? (env, mounts, ports, policy — full JSON)
docker inspect pg-practice

# Just one field, using a Go template filter — the grep-able version:
docker inspect -f "{{ .HostConfig.RestartPolicy.Name }}" pg-practice
docker inspect -f "{{ .NetworkSettings.Ports }}" pg-practice

# A container that survives crashes AND Docker Desktop restarts:
docker run -d --name keeper --restart unless-stopped -p 8081:80 nginx:alpine
# Reboot Windows, reopen Docker Desktop: `docker ps` shows keeper came back on its own.
```

Taming WSL2's appetite — the config file to know about *before* your laptop starts swapping:

```text
# File: C:\Users\<you>\.wslconfig   (create it if absent; restart WSL to apply: wsl --shutdown)
[wsl2]
memory=6GB        # cap WSL2 (and thus Docker) RAM — default can grow to most of your machine
processors=4      # optional CPU cap
```

## Common Pitfalls

1. **Reversing the port mapping.** `-p 3000:8080` when you meant `-p 8080:3000`, then "the container is broken" when it's fine. Correction: the order is always **host:container** — mnemonic: it reads left-to-right the same direction traffic flows, from your machine into the container. When unsure, `docker port <name>` shows current mappings.

2. **Expecting data to survive container deletion without a volume.** Run Postgres without `-v`, load data for a week, `docker rm`, everything's gone. Correction: containers are disposable *by design*; any state you'd miss goes in a named volume from the very first `docker run`. (Note the asymmetry: `stop`/`start` keeps container state; `rm` destroys it.)

3. **A container that "immediately exits" being treated as a Docker bug.** A container lives exactly as long as its main process; if that process crashes (bad env var, missing config) or finishes instantly, the container stops. Correction: `docker ps -a` to see it exited, then `docker logs <name>` — the reason is virtually always printed there. This two-command reflex will serve you for years.

4. **Running `latest` and getting surprise breakage.** `postgres:latest` is a different database version this month than last month. Correction: always pin at least the major version (`postgres:16`, `node:22-alpine`). Treat `latest` as "unspecified," because that's what it is.

5. **Keeping project code on the Windows filesystem while working on it from WSL2 contexts.** Docker Desktop bridges Windows paths into the Linux VM, but crossing that bridge on every file access is slow — bind-mounted `node_modules` on `C:\` can make installs and dev servers crawl. Correction: for casual use it's fine; when bind-mount performance hurts, keep the project inside the WSL2 filesystem (e.g., `\\wsl$\Ubuntu\home\you\project`) or skip bind mounts and rebuild images instead. Also know that WSL2 can eat RAM; a `.wslconfig` file in your Windows home directory can cap it.

6. **Confusing being *inside* the container with being on your machine.** You `docker exec` in, install a tool, edit a file — and it all evaporates with the container, or worse, you think you edited your project but touched a copy baked into the image. Correction: treat exec-shells as read-only inspection posts. Durable changes belong in the image's build recipe (Chapter 5) or in volumes.

7. **Ignoring disk usage until Docker Desktop consumes 60 GB.** Every pulled image, every stopped container, every orphaned volume persists. Correction: a monthly `docker system df` and a considered `docker system prune` (add `--volumes` only when you're sure no wanted data is in unused volumes — that flag deletes data).

## Practice Exercises

1. **Lifecycle drill.** Run `nginx:alpine` detached with a port mapping of your choice, curl it, view its logs, exec in and find the HTML it's serving (hint: look under `/usr/share/nginx/`), stop it, start it, remove it. Narrate each state transition with `docker ps -a` output in your notes.

2. **Volume proof.** Run Postgres with a named volume, exec in with `psql`, create a table and insert a row. Remove the container entirely, start a brand-new one attached to the same volume, and show the row survived. Then do the identical sequence *without* a volume and document the difference.

3. **Two webs, one host.** Run two nginx containers simultaneously on host ports 8081 and 8082. Explain in one sentence why the *containers* can both use port 80 without conflict while the *host* ports must differ.

4. **Crash forensics.** Start a Postgres container but deliberately omit `POSTGRES_PASSWORD` (and its alternatives). Use the `ps -a` + `logs` reflex to find out what happened and quote the decisive log line in your notes.

5. **Image safari.** Pull `python:3.12-alpine` and `python:3.12` and compare their sizes in `docker images`. Run `docker run -it python:3.12-alpine sh` and poke around: what shell is it? Is `bash` there? Is `apt`? Write three observed differences — this materializes next chapter's slim-image tradeoffs.

6. **Dockerized one-off tools.** Use a container as a disposable tool without installing anything: run `docker run --rm -it node:22-alpine node` to get a Node REPL (`--rm` auto-deletes the container on exit), and then figure out how to run a one-off Python script from your current folder using a bind mount (`-v ${PWD}:/work` in PowerShell) and `python:3.12-alpine`. This pattern — borrow a runtime for five minutes, leave no trace — is one of Docker's best everyday tricks.
