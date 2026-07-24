# Project 2: Containerize a Past Project

## Description

Pick one of your existing Node or Python projects — an Express API, a Flask app, any long-running service you've built — and containerize it properly: a well-ordered Dockerfile, a `.dockerignore` that actually guards the build context, a small final image, env-var configuration honored at run time, and a README section that lets a stranger run your project with two commands and zero questions. "Properly" is the entire point: anyone can get a container to start once; this project is about producing the image you wouldn't be embarrassed to hand a team. When you finish, "does it run on your machine?" becomes a question you never have to answer again — the machine ships with the app.

## Difficulty & Estimated Effort

Beginner-to-intermediate — 3–5 hours, assuming Docker Desktop is already installed and behaving (add an hour of WSL2 patience if not).

## Chapters Used

- Chapter 4: Docker Fundamentals
- Chapter 5: Writing Dockerfiles

## Requirements

- [ ] Choose a past project that runs a server process (Node or Python). It must currently work when run natively; fix it first if not — containerizing a broken app just gives you a broken container.
- [ ] Audit its configuration against Chapter 3: any hardcoded ports, URLs, or secrets move to environment variables (with a `.env.example` documenting the contract) *before* you write a line of Dockerfile.
- [ ] Write the `.dockerignore` **first**. At minimum it must exclude dependency folders, VCS internals, env files, and build junk. Justify each line with a comment.
- [ ] Write a `Dockerfile` that: pins an explicit base image tag, sets a working directory, exploits layer caching correctly (dependency manifests copied and installed before source), uses the reproducible install command for your ecosystem, binds the app so it's reachable from outside the container, and declares a sensible default command.
- [ ] Build the image with a real tag (not `latest`-only). Record the image size.
- [ ] Run the container with a published port and env-var configuration passed at run time; verify the app responds from a browser or curl **on the host**.
- [ ] Prove the caching works: make a trivial source-code change, rebuild, and confirm the dependency-install layer reports as cached. Capture the build output showing it.
- [ ] Prove config injection works: run two containers from the *same image* with different env values (e.g., different greeting text, log level, or port) and demonstrate the different behavior.
- [ ] Get the image *smaller*: try at least one deliberate size reduction (slimmer base image, pruned dev dependencies, multi-stage build if your project has a build step) and record before/after sizes with `docker images` and `docker history` evidence of what shrank.
- [ ] Inspect your final image for leaks: confirm no `.env`, no secrets in `ENV` layers, no `.git` directory inside. Show the commands you used to check.
- [ ] Exec into the running container and verify the world inside looks like you intended: the right files present, the excluded files absent, the process running as the user you expect.
- [ ] Add a **"Run with Docker"** section to the project README: prerequisites, the exact build command, the exact run command (including which env vars are required and what they do), how to see logs, and how to stop/remove. Test it by following it verbatim in a fresh terminal.
- [ ] Handle the lifecycle honestly: document what data (if any) the app writes, and whether a volume is needed. If the app writes nothing durable, say so explicitly in the README.

## Hints

- The Chapter 5 "blind-spec drill" is the warm-up for this project. If you can write the canonical Dockerfile shape from memory, this project is mostly finishing work.
- If the container starts and immediately exits, you know the two-command reflex: `docker ps -a`, then `docker logs`. The answer is nearly always printed there.
- If the app runs but nothing answers on the mapped port, there are exactly two usual suspects, and Chapters 4 and 5 each named one: the direction of your `-p` mapping, and which interface the app binds inside the container.
- Python people: think carefully about `-slim` vs `-alpine` before chasing the smallest number — Chapter 5 explains which packages make Alpine expensive. Node people: `-alpine` is usually a free win.
- For the two-containers-one-image requirement, remember tags are cheap and containers are cheaper — this whole demo is four commands.
- A one-line fragment like `COPY package*.json ./` is the *kind* of thing worth checking against the chapter; resist copying any full Dockerfile from the internet — this one has to be yours, because interviewers ask "walk me through your Dockerfile" and it shows immediately when the answer is "I pasted it."
- `Measure-Command { docker build ... }` in PowerShell gives you honest numbers for the caching claims.

## Stretch Goals

- Add a `HEALTHCHECK` instruction to the Dockerfile and watch `docker ps` report health status (foreshadows Chapter 11).
- Run the container as a non-root user and prove it with a command executed inside the container.
- Push the tagged image to a registry (Docker Hub free tier or GHCR) and pull-and-run it on a different machine — or in a fresh WSL2 distro — to complete the "ships anywhere" claim.
- Containerize a *second* project from the other ecosystem (if you did Node, do Python) and note which lessons transferred unchanged.
- Get the final image under a target you set in advance (e.g., under 200 MB for Node, under 150 MB for Python) and document every step of the diet with `docker history` receipts.
