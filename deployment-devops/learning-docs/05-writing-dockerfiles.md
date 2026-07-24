# Chapter 5: Writing Dockerfiles

## Overview

Last chapter you ran other people's images; now you'll produce your own. A **Dockerfile** is a short script of instructions that assembles an image: start from a base, copy code in, install dependencies, declare how to launch. Simple in principle — and yet the gap between a naive Dockerfile and a good one is enormous in practice: the naive one rebuilds five minutes of `npm install` every time you change a comment, weighs 1.5 GB, runs as root, and ships your `.env` file to anyone who pulls the image. The good one rebuilds in seconds, weighs 150 MB, and leaks nothing. The difference comes down to three ideas you'll learn here: **layer caching** (order instructions from least- to most-frequently-changing), **build context hygiene** (`.dockerignore`), and **multi-stage builds** (build with the kitchen sink, ship only the dishes). You'll containerize both a Node app and a Python app step by step — after this chapter, "containerize an arbitrary project" stops being a research task and becomes a ten-minute chore.

## Definitions & Explanations

**Dockerfile** — a text file (conventionally named exactly `Dockerfile`, no extension) of instructions executed top-to-bottom by `docker build` to produce an image. It is the *recipe*, committed to the repo right next to the code it packages — which means your environment setup is now versioned, reviewable, and reproducible, exactly like code.

**`FROM`** — the first real instruction: which existing image to build on top of. `FROM node:22-alpine` means "start with a filesystem that already contains Node 22 on Alpine Linux." Base image choice is your biggest single size/compatibility lever:
- **Full images** (`node:22`, `python:3.12`) — Debian-based, everything included, large (~1 GB).
- **`-slim`** — Debian minus most extras; a good default for Python.
- **`-alpine`** — built on tiny Alpine Linux (~5 MB base); great for Node; occasionally causes friction for Python packages with compiled C extensions (Alpine uses musl instead of glibc, so prebuilt wheels may not exist).

**`WORKDIR`** — sets the working directory inside the image for subsequent instructions (creating it if needed). Convention: `/app`.

**`COPY`** — copies files from the **build context** (see below) into the image. `COPY . .` means "everything from the context into the current WORKDIR."

**`RUN`** — executes a command *during the build* and bakes the result into a new layer: `RUN npm ci`, `RUN pip install -r requirements.txt`.

**`CMD`** — declares the default command executed *when a container starts* (not during build). One per Dockerfile. Prefer the JSON-array "exec form": `CMD ["node", "server.js"]`. (Its sibling `ENTRYPOINT` fixes a command that `CMD` then supplies arguments to — file it away; `CMD` alone covers you for now.)

**`EXPOSE`** — documentation metadata saying "the app inside listens on this port." It does **not** publish the port — you still need `-p` at run time. Harmless, useful for humans and tools, frequently misunderstood.

**Build context** — the set of files sent to the Docker engine when you run `docker build .` (the `.` names the context directory). `COPY` can only reach files in the context. A bloated context (with `node_modules`, `.git`, gigabytes of junk) makes every build slow *before it even starts* — hence:

**`.dockerignore`** — the context's bouncer, same syntax as `.gitignore`: files listed here are never sent to the engine and can never be COPY'd — even by `COPY . .`. Minimum viable contents for any project: `node_modules` (or `venv`/`__pycache__`), `.git`, `.env`, `dist`. This is simultaneously a speed feature *and a security feature* (it's what keeps `.env` out of your shipped image).

**Layer caching** — every instruction produces a layer; when rebuilding, Docker reuses cached layers *until it hits the first instruction whose inputs changed* — and everything after that point rebuilds. For `COPY`, "inputs" means the copied files' contents. Consequence, and the single most important Dockerfile idiom: **copy your dependency manifest and install dependencies *before* copying the rest of your code.** Code changes daily; dependencies change weekly. Order the recipe so the expensive install layer stays cached across code-only changes.

**Multi-stage build** — multiple `FROM` blocks in one Dockerfile; later stages can `COPY --from=<stage>` selected files out of earlier ones, and only the final stage becomes the shipped image. The pattern: a "builder" stage with compilers, dev dependencies, and source; a lean "runtime" stage that copies in only the finished output. This is how a React app's image is nginx + 2 MB of static files instead of Node + 400 MB of `node_modules`.

**Image tagging (at build time)** — `docker build -t myapp:1.2.0 .` names your image. Untagged builds get IDs like `f3a91c...` and become "which one was that?" mysteries. A useful discipline even solo: tag with something traceable (a version, or later, the Git commit SHA — Chapter 8 automates this). Retagging is free: `docker tag myapp:1.2.0 myapp:latest` — tags are pointers, remember.

**Non-root user** — processes in containers run as root by default; if the app is compromised, the attacker is root *in the container*, which is a better position than you want to give them. Production-grade Dockerfiles switch: `USER node` (the Node images ship a `node` user) or create one. Good habit to see now, adopt as you go.

## Code Examples

Containerizing a Node/Express app — the canonical form, with the reasoning inline:

```dockerfile
# Dockerfile
FROM node:22-alpine

WORKDIR /app

# Dependency manifests FIRST — this is the caching idiom.
# These files change rarely, so the npm ci layer below usually stays cached.
COPY package*.json ./

# npm ci (not install): exact, clean install from package-lock.json — reproducible builds.
# --omit=dev: no test frameworks/linters in the production image.
RUN npm ci --omit=dev

# NOW the code — changes here no longer invalidate the install layer above.
COPY . .

# The env-var contract from Chapter 3, honored at image level:
ENV NODE_ENV=production

EXPOSE 3000

# Drop root before the app starts (node user is built into the official images):
USER node

CMD ["node", "server.js"]
```

```text
# .dockerignore — write this BEFORE the first build, not after the first 900 MB context upload
node_modules
.git
.env
dist
npm-debug.log
Dockerfile
```

```powershell
# Build (from the project root; "." = build context) and tag:
docker build -t myapp:0.1.0 .

# Run it — image ENV came along, but secrets/config are injected at RUN time, never baked in:
docker run -d --name myapp -p 3000:3000 -e DATABASE_URL="postgres://..." myapp:0.1.0

# The payoff check — edit any .js file, rebuild, and watch the output:
docker build -t myapp:0.1.1 .
# Steps through npm ci should say CACHED. If they don't, your COPY ordering is wrong.
```

The Python/Flask equivalent — same skeleton, different dialect:

```dockerfile
FROM python:3.12-slim
# -slim over -alpine for Python: prebuilt wheels for packages with C extensions
# (psycopg2, numpy, ...) target glibc; Alpine's musl can force slow source compiles.

WORKDIR /app

# Same caching idiom: manifest first, install, then code.
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
#               ^ don't keep pip's download cache as dead weight in the layer

COPY . .

EXPOSE 8000

# Dev server (flask run / python app.py) is not production-grade.
# gunicorn is the standard production server for Flask (put it in requirements.txt):
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
#                          ^ 0.0.0.0, not 127.0.0.1 — inside a container, localhost-only
#                            binding is unreachable from outside. Classic gotcha.
```

Multi-stage: shipping a built React app without shipping Node —

```dockerfile
# ---- Stage 1: builder (big, disposable) ----
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci                 # dev deps included — the build needs them
COPY . .
RUN npm run build          # produces /app/dist

# ---- Stage 2: runtime (tiny, shipped) ----
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
# nginx:alpine's default CMD already starts nginx — inheriting it is fine.
```

```powershell
docker build -t mysite:1.0 .
docker images mysite        # note the size — then mentally compare against node:22 + node_modules
docker run -d -p 8080:80 mysite:1.0
```

Two spelling details that separate working Dockerfiles from confusing ones:

```dockerfile
# CMD exec form vs shell form — prefer exec form:
CMD ["node", "server.js"]      # exec form: node IS process 1, receives stop signals,
                               # shuts down cleanly when the platform says stop
CMD node server.js             # shell form: a shell wraps your app; signals often
                               # don't reach it -> 10-second force-kill on every deploy

# ARG vs ENV — build-time vs run-time values:
ARG NODE_VERSION=22            # exists ONLY during build (usable in FROM, RUN)
ENV NODE_ENV=production        # baked into the image, present at run time
# Corollary you already know: neither is for secrets — ARG values are visible
# in `docker history` too.
```

Size forensics, for when an image is mysteriously huge:

```powershell
docker history myapp:0.1.0   # per-layer sizes — the offending RUN/COPY jumps right out
```

And the debugging move for builds that "succeed but the app won't start" — override the CMD and go look:

```powershell
# Start a shell INSTEAD of the app, in the exact image you just built:
docker run -it --rm myapp:0.1.0 sh

# Inside: is your code actually where you think? did .dockerignore exclude too much?
#   ls -la            # what did COPY actually bring in?
#   node server.js    # run the CMD by hand and read the error with your own eyes
# This beats ten blind rebuild-and-pray cycles every single time.
```

## Common Pitfalls

1. **`COPY . .` before installing dependencies.** The one-line mistake that makes every build take five minutes: any code change invalidates the copy layer, which invalidates the install layer after it. Correction: manifests → install → `COPY . .`, always. Verify by rebuilding after a trivial code edit and confirming the install step reports CACHED.

2. **No `.dockerignore`, so `node_modules`, `.git`, and `.env` ride along.** Three distinct wounds: a giant slow context, your host's platform-specific `node_modules` stomping the image's clean install, and *your secrets embedded in a distributable artifact*. Correction: write `.dockerignore` before the first build; when in doubt, mirror your `.gitignore` and add `.git` itself.

3. **Baking secrets in with `ENV` or `COPY .env`.** `docker history` and `docker inspect` show ENV values to anyone holding the image — an image is a publication, like a Git repo. Correction: images carry *code and defaults*; secrets arrive only at run time (`-e`, `--env-file`, platform injection). If a secret ever got baked in: rotate it (Chapter 3's playbook), don't just rebuild.

4. **Binding the app to `127.0.0.1` inside the container.** Flask's default, notoriously — the container runs, `docker ps` looks perfect, and nothing answers on the mapped port, because localhost inside the container *is the container*. Correction: bind to `0.0.0.0` in containerized apps; that means "all interfaces," which is what port mapping needs to reach.

5. **`EXPOSE` treated as publishing the port.** You wrote `EXPOSE 3000` and skipped `-p`, and the app is unreachable. Correction: `EXPOSE` is documentation; `-p host:container` at run time does the actual wiring.

6. **Using `npm install` instead of `npm ci` in images.** `install` may resolve versions differently over time and mutate the lockfile; builds stop being reproducible. Correction: in any automated/build context, `npm ci` — it installs exactly the lockfile or fails loudly. (Python's analog: pin versions in `requirements.txt` rather than installing loose latest.)

7. **Chasing the smallest possible image before the correct one.** Alpine-everything, aggressive layer golf, distroless — and then a package fails to compile against musl and eats an afternoon. Correction: sane defaults first (`node:*-alpine` is safe; `python:*-slim` over alpine), multi-stage where there's a build step, and only optimize further with `docker history` evidence of an actual problem.

## Practice Exercises

1. **Containerize your Node project.** Take a real Express (or any Node) project of yours: `.dockerignore` first, then the canonical Dockerfile, build tagged as `:0.1.0`, run with published ports and env-var config, verify with curl. Then prove the caching: touch one source file, rebuild, and screenshot/copy the CACHED lines into your notes.

2. **Containerize your Python project.** Same exercise for a Flask/FastAPI (or script-based) Python project, using `python:3.12-slim`, a pinned `requirements.txt`, and a production server (gunicorn/uvicorn) as the CMD. Confirm the `0.0.0.0` binding lesson by *first* deliberately binding to 127.0.0.1 and observing the symptom from outside.

3. **Cache-order experiment.** Make a copy of one Dockerfile with the COPY/install order deliberately wrong. Time three consecutive builds (code tweak between each) for both variants (`Measure-Command { docker build ... }` in PowerShell). Record the numbers — this is the chapter's thesis, measured.

4. **Multi-stage a React app.** Apply the builder→nginx pattern to any Vite/React project. Report three numbers: final image size, `node:22-alpine` base size, and what a single-stage image (build + serve from Node) would weigh. `docker history` the winner and identify its two biggest layers.

5. **Image audit.** Run `docker history` and `docker inspect` on an image you built for exercise 1 or 2. Answer in writing: What ENV values does it expose? What's its CMD? Its user — root or not? Anything in it you wouldn't want a stranger pulling? Fix what you find.

6. **Blind-spec drill (interview simulation).** Without referring to this chapter, write from memory a Dockerfile for a hypothetical Node API (deps, caching idiom, non-root, port, CMD) and a companion `.dockerignore`. Then diff against the chapter's version and list every discrepancy. Repeat in a week; the goal is zero diffs from memory.
