# Deployment & DevOps Learning Track

A self-paced track that takes you from "my project only runs on my laptop" to shipping real software: a project of yours live at a real URL, with automated tests running on every push, a container image anyone can run, and the confidence to walk into a codebase you've never seen and get it deployed. Designed for Windows 11 with PowerShell, Docker Desktop, and WSL2.

**Why this track exists.** In 2026, entry-level job postings treat three things as baseline, not bonus: Docker, one CI/CD pipeline, and one cloud deployment path. "Built a React app" is table stakes; "shipped a React app to a URL with tests gating every merge" is what separates candidates. And the #1 complaint hiring managers voice about junior developers isn't a missing framework — it's that they can't work inside code they didn't write. This track ends with a capstone built specifically to fix that.

**Prerequisites.** Finish (or be far along in) the `git-github/`, `command-line/`, and `how-the-web-works/` tracks first. You'll also need at least one real project of your own — anything from the JavaScript, Python, React, or Node/Express tracks — because most projects here operate on *your existing code*, not toy examples.

## Structure

- `learning-docs/` — 12 chapters, the primary study material. Each has an overview, plain-English definitions, runnable commands and configs with commentary, common pitfalls with corrections, and practice exercises.
- `projects/` — 6 project specs, difficulty-ordered. Specs only — no answer keys, no paste-able Dockerfiles or workflows. The struggle is the curriculum.

## Chapters (in order)

| # | File | Covers |
|---|------|--------|
| 1 | `learning-docs/01-from-localhost-to-the-world.md` | What deployment actually is; environments (dev/staging/prod); builds vs source; static hosting vs servers vs serverless |
| 2 | `learning-docs/02-servers-and-ssh.md` | What a Linux server is; SSH keys and connecting; server navigation; why beginners should start on a PaaS — and what the PaaS hides |
| 3 | `learning-docs/03-configuration-and-secrets.md` | Environment variables as the deployment contract; .env vs injected config; 12-factor config; secret managers; rotating leaked secrets |
| 4 | `learning-docs/04-docker-fundamentals.md` | The problem containers solve; images vs containers; Docker Desktop on Windows/WSL2; run, ports, volumes, exec |
| 5 | `learning-docs/05-writing-dockerfiles.md` | Containerizing Node and Python apps; layers and caching; .dockerignore; multi-stage builds; small images; tagging |
| 6 | `learning-docs/06-docker-compose.md` | Multi-container apps (app + database); compose files; container networks; volumes for persistence; dev environments |
| 7 | `learning-docs/07-continuous-integration.md` | Why teams refuse to work without CI; GitHub Actions anatomy; a lint+test workflow; reading failed runs; branch protection |
| 8 | `learning-docs/08-continuous-delivery-and-deployment.md` | CI vs CD; build artifacts; deploy on merge to main; environment secrets in Actions; rollbacks; deploy-on-green |
| 9 | `learning-docs/09-cloud-platforms.md` | PaaS (Render/Railway/Fly.io) vs IaaS (AWS/Azure/GCP), honestly; managed services; free tiers; core AWS vocabulary |
| 10 | `learning-docs/10-databases-in-production.md` | Managed databases; connection strings as secrets; migrations during deploys; backups and restore drills; one DB per environment |
| 11 | `learning-docs/11-monitoring-logging-debugging.md` | Production logs and log levels; health checks; uptime monitoring; error tracking; debugging without a debugger |
| 12 | `learning-docs/12-domains-dns-https.md` | Buying a domain; A/CNAME records in practice; HTTPS via Let's Encrypt; redirects; custom domains on deployed projects |

## Projects (easiest → hardest)

| # | File | Ships | After chapters |
|---|------|-------|----------------|
| 1 | `projects/01-ship-a-static-site.md` | An HTML/CSS or built React project on GitHub Pages AND Netlify/Vercel, with a custom 404 | 1 (12 helps) |
| 2 | `projects/02-containerize-a-past-project.md` | A Dockerfile for one of your existing projects, small image, documented run instructions | 4–5 |
| 3 | `projects/03-ci-pipeline.md` | Lint + test on every push/PR, status badge, branch protection that blocks red merges | 7 |
| 4 | `projects/04-compose-a-full-stack.md` | App + database (+ optional admin tool) up with one command, persistent data, env-based config | 3, 6 |
| 5 | `projects/05-deploy-an-api-to-a-real-url.md` | A backend of yours on a PaaS with managed DB, platform secrets, auto-deploy on merge, monitored health check | 2–3, 8–11 |
| 6 | `projects/06-capstone-adopt-an-unfamiliar-codebase.md` | The "first week at a job" simulator: run, containerize, CI, and fix a real issue in an open-source project you did not write | everything |

## Suggested cadence

At **4–6 hours/week**, the track takes roughly **10–12 weeks**:

- **Week 1:** Chapter 1, then Project 1. Ship something in week one — the psychological unlock of a real URL matters.
- **Weeks 2–3:** Chapters 2–3. No project yet; these are the mental models everything else leans on.
- **Weeks 4–5:** Chapters 4–5, then Project 2. Expect Docker Desktop/WSL2 setup friction — budget a session for it.
- **Week 6:** Chapter 6, then Project 4 can start (finish it after you're comfortable). Chapter 7, then Project 3 — do Project 3 promptly; you'll keep that pipeline forever. (Project 4 starting before Project 3 finishes is deliberate overlap, not a contradiction of the difficulty ordering below — Project 3's CI pipeline can bake in the background while you start Project 4's compose work.)
- **Weeks 7–8:** Chapters 8–9, then start Project 5. This is the track's centerpiece deliverable: your API at a real URL.
- **Week 9:** Chapters 10–11 while Project 5 is in flight (it needs both).
- **Week 10:** Chapter 12; put a custom domain on Project 5 or Project 1 if you bought one.
- **Weeks 11–12:** The capstone, Project 6. Do not rush it; it is the thing you'll talk about in interviews.

Rules of thumb:
- Read the chapter's pitfalls section *before* the exercises, not after something breaks.
- $0 is genuinely achievable (as of mid-2026: Render's free web service + a free Neon or Turso database), but the free-tier landscape has narrowed — Railway and Fly.io no longer have true free tiers, and free tiers change constantly regardless. Chapters flag where they bite (cold starts, expiring databases); capstones/README.md's "Keeping It Free" section is the maintained summary if you want current specifics without re-deriving them.
- When a deploy fails, the logs are the exercise. Resist the urge to delete and retry blindly.
- Keep a running `DEPLOY-NOTES.md` in each project you ship: every error you hit and how you fixed it. That file becomes interview material.

## Done when

You can: explain what happens between `git push` and a user seeing your change; containerize an arbitrary Node or Python app without a tutorial; read a failed GitHub Actions run and fix it; deploy a backend + database to a PaaS from scratch in under an hour; and — the real graduation bar — clone a stranger's repo, get it running, and ship a fix. That last one is the skill hiring managers say juniors lack. You won't be one of them.
