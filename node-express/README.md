# Node.js + Express Learning Track

A self-paced path from "I know JavaScript" to "I can build, secure, test, and ship a real backend." Everything you learned in the JavaScript track still applies — same language, new environment: instead of running inside a browser and manipulating a page, your code runs directly on the machine (via **Node.js**) and answers HTTP requests (via **Express**).

**Why this matters for jobs:** almost every web application has a backend, and JavaScript backends are one of the most-hired stacks because teams can use one language end to end. "Full-stack JavaScript" (React on the front, Node/Express on the back) is a standard junior-to-mid job profile. This track is the back half of that profile; your React track is the front half, and the deployment-devops track takes what you build here and puts it on the internet.

- **`learning-docs/`** — 14 chapters of primary study material. Read in order; each assumes only the chapters before it. Do the practice exercises — they're where the learning actually happens.
- **`projects/`** — 7 guided project specs, easiest to hardest, with requirement checklists but **no solution code**. Each bundles several chapters; build them at the checkpoints below.

## Prerequisites

- **JavaScript track (required)** — all 18 chapters, especially functions, error handling, async/await, fetch, and modules. This track assumes you are fluent in the language and never re-teaches it.
- **SQL track (strongly recommended)** — chapters 9 onward store data in a real database. You can start this track without SQL and pause to learn it before chapter 9, but don't skip it entirely.
- **Windows note:** examples use PowerShell. Almost everything is identical on macOS/Linux; the chapters call out the few places it differs (installing Node, environment variables, `curl`).

You do **not** need the React track first — but if you've started it, the capstone here is designed to become the backend for a React frontend.

## Chapters (read in order)

| # | File | Topic |
|---|------|-------|
| 01 | `learning-docs/01-node-and-the-runtime.md` | What Node is, V8, browser vs server JS, installing with nvm-windows, REPL, `process`, CLI args |
| 02 | `learning-docs/02-npm-and-modules.md` | package.json, dependencies, semver, lockfiles, npm scripts, ES modules vs CommonJS |
| 03 | `learning-docs/03-async-node-and-the-event-loop.md` | The event loop on a server, callbacks → promises → async/await, fs/promises, why blocking kills servers |
| 04 | `learning-docs/04-http-server-from-scratch.md` | The `http` module: requests, responses, routing, and body parsing by hand — so Express isn't magic |
| 05 | `learning-docs/05-express-fundamentals.md` | App setup, routing, route params, query strings, sending responses, project structure |
| 06 | `learning-docs/06-middleware.md` | The req/res/next pipeline, built-in and custom middleware, why order matters, morgan, cors |
| 07 | `learning-docs/07-rest-api-design.md` | Resources, HTTP methods and status codes done right, URL design, versioning, idempotency, designing on paper |
| 08 | `learning-docs/08-validation-and-error-handling.md` | Validating input (manual + zod), centralized error middleware, async errors, consistent error shapes |
| 09 | `learning-docs/09-databases-from-node.md` | better-sqlite3 (and pg), parameterized queries always, the data-access layer, migrations conceptually |
| 10 | `learning-docs/10-authentication-and-sessions.md` | bcrypt password hashing, register/login, sessions vs JWTs honestly, auth middleware, logout |
| 11 | `learning-docs/11-security-and-configuration.md` | Env vars and dotenv, helmet, CORS properly, rate limiting, the OWASP mindset for APIs |
| 12 | `learning-docs/12-testing-apis.md` | vitest + supertest, end-to-end route tests, test databases, mocking vs real dependencies |
| 13 | `learning-docs/13-architecture-and-code-organization.md` | Routers/controllers/services/data layers, keeping Express out of business logic, refactoring |
| 14 | `learning-docs/14-production-readiness.md` | Logging with pino, graceful shutdown, health checks, NODE_ENV, serving a frontend, deploy prep |

## Projects (build in order)

| # | File | Project | Difficulty |
|---|------|---------|-----------|
| 1 | `projects/01-cli-toolbox.md` | CLI toolbox — small command-line utilities | Beginner |
| 2 | `projects/02-http-server-from-scratch.md` | Multi-route HTTP server, no Express | Beginner+ |
| 3 | `projects/03-rest-api-in-memory.md` | Bookmarks REST API (in-memory, full CRUD + validation) | Intermediate |
| 4 | `projects/04-rest-api-with-database.md` | Bookmarks API with SQLite and a layered structure | Intermediate+ |
| 5 | `projects/05-authenticated-api.md` | Authenticated notes API — users, bcrypt, protected routes | Advanced− |
| 6 | `projects/06-tested-and-hardened-api.md` | Full test suite + security hardening on project 4/5 | Advanced |
| 7 | `projects/07-capstone-fullstack-backend.md` | Capstone: production-quality backend for an app of your choice | Advanced |

## Suggested cadence

Assuming ~5–8 focused hours per week (adjust freely — the checkpoints matter, not the calendar):

| Phase | Study | Then build | Rough time |
|-------|-------|-----------|------------|
| 1. Node itself | Chapters 01–03 | **Project 1** — CLI toolbox | Weeks 1–3 |
| 2. HTTP by hand | Chapter 04 | **Project 2** — HTTP server from scratch | Week 4 |
| 3. Express core | Chapters 05–08 | **Project 3** — In-memory REST API | Weeks 5–8 |
| 4. Real data | Chapter 09, then skim 13 | **Project 4** — API with SQLite | Weeks 9–10 |
| 5. Users & security | Chapters 10–11 | **Project 5** — Authenticated API | Weeks 11–13 |
| 6. Confidence | Chapter 12 (re-read 08, 11) | **Project 6** — Tested & hardened API | Weeks 14–15 |
| 7. Ship shape | Chapters 13–14 in full | **Project 7** — Capstone backend | Weeks 16–19 |

## How to work

- **Type every example.** Reading code teaches recognition; typing it teaches recall. Break examples on purpose and read the errors — Node's stack traces are your new best friend.
- **Keep a terminal open constantly.** Backend development lives in the terminal far more than frontend does. Get comfortable with two panes: one running the server, one firing requests at it.
- **Exercises before projects.** Each project assumes you did the exercises of its chapters.
- **Projects are specs, not tutorials.** Getting stuck is the point; the Hints sections nudge without spoiling. Budget being stuck ~20–30 minutes before looking anything up.
- **Use git from Project 1 onward,** and never commit `node_modules` or `.env` files — chapter 2 and chapter 11 explain why.
- **Finish the capstone properly** (README, API docs, tests passing). A deployed full-stack app — this backend plus a React frontend — is the strongest single portfolio piece a junior can have.

## After the track

Pair the capstone backend with a frontend from your React track, then take both into the deployment-devops track to put them on the internet. From there: TypeScript on the backend, PostgreSQL in place of SQLite, and reading real open-source Express codebases to see these patterns at production scale.
