# Project 7: Capstone — Full-Stack App Backend

## Description

Build the complete, production-quality backend for a real application of your choosing — the kind of codebase you'd be comfortable handing to another developer, pointing a React frontend at, and deploying to a real host. This is the portfolio piece of the track: authentication, a relational database behind a layered architecture, validation on every input, a real test suite, structured logging, health checks, graceful shutdown, and a README good enough that a stranger can run it in five minutes.

**Pick an app you'd actually use.** Suggestions — but the choice is yours:

- **Tutoring-session scheduler** — tutors, students, availability, bookings; rich rules (no double-booking) and a natural fit if tutoring is your world
- **Workout logger** — exercises, workouts, sets/reps over time; simple entities, satisfying progress queries
- **Recipe box** — recipes, ingredients, tags, search; pairs directly with the JavaScript track's capstone frontend
- **Reading tracker** — books, reading sessions, status, notes, yearly goals

Whatever you choose needs: at least two related resources (beyond users) with a real relationship between them, per-user ownership, and at least one business rule that isn't just CRUD (a booking conflict, a uniqueness rule, a status transition).

This backend is designed to pair with a frontend from your React track (CORS configured for a local dev server, clean JSON contract, API docs a frontend dev could build from) and to be deployed later in the deployment-devops track (env-driven config, health endpoint, graceful shutdown, `npm start`).

## Difficulty & Estimated Effort

Advanced — 25–40 hours. Treat it as a multi-week build with milestones, not a weekend sprint.

## Chapters Used

All of them — chapters 1–14. This is the track's proof of completion.

## Requirements

**Milestone 0 — Design on paper (before any code)**

- [ ] A short design doc (`DESIGN.md`): the app in one paragraph, the entities and their relationships (a sketch or text diagram), and your named business rules
- [ ] The full endpoint table: every route with method, path, auth requirement, request body summary, success status, and error statuses — reviewed against Chapter 7 before implementation
- [ ] Database schema drafted as `schema.sql` with foreign keys, constraints backing your business rules where possible, and your SQLite-vs-Postgres choice stated in one sentence (SQLite is fully acceptable; choose Postgres only if you want the extra setup as practice)

**Milestone 1 — Skeleton that deploys nothing but works**

- [ ] Layered structure (routes/controllers/services/repositories), app/server split for testability, config module reading all env vars, `.env.example` committed with placeholder values only
- [ ] `GET /api/health` returning status, uptime, and a database connectivity check
- [ ] pino logging: one line per request (with status and duration) and structured error logs; log level from env; no `console.log` in the finished codebase
- [ ] Centralized error handler with the standard error shape; JSON 404 for unknown routes; malformed-JSON handling
- [ ] Graceful shutdown: on SIGTERM/SIGINT (and Windows equivalents — note what you learned about signal support), stop accepting connections, let in-flight requests finish, close the database, exit cleanly — with a log line proving the path ran

**Milestone 2 — Auth**

- [ ] Register/login/logout/me endpoints with bcrypt, your chosen session mechanism (sessions or JWT — reuse or revise your Project 5 justification), validation, and enumeration-safe failures
- [ ] Auth middleware protecting everything except register/login/health
- [ ] Login rate-limited; helmet applied; secrets only from env
- [ ] Auth flow covered by supertest tests before moving to Milestone 3

**Milestone 3 — The domain**

- [ ] Full CRUD for your resources, validated (zod or equivalent) at every write, all queries parameterized, all data scoped to the owning user in the repository layer
- [ ] Your non-CRUD business rule(s) implemented in the service layer with tests that prove them (e.g., the double-booking attempt fails with a meaningful `409`; the invalid status transition is rejected)
- [ ] Relationship endpoints designed RESTfully (e.g., nested or filtered routes for "sessions belonging to a booking" — whatever fits your domain and your endpoint table)
- [ ] Pagination on every list endpoint that can grow; filtering/sorting where your frontend would obviously need it (allowlisted sort columns)
- [ ] Seed script so a fresh clone can have demo data (`npm run db:seed`)

**Milestone 4 — Quality gate**

- [ ] Test suite (vitest + supertest, isolated test DB) covering: happy paths for every endpoint, validation failures, 404s, auth boundaries (401s and cross-user isolation), and every business rule — passing with `npm test`
- [ ] Security audit note as in Project 6, updated for this app
- [ ] Coverage generated and a paragraph of honest interpretation in the README

**Milestone 5 — Frontend & deployment readiness**

- [ ] CORS configured (the `cors` middleware) to allow your local React dev server origin with credentials if your auth needs them — origins from env, not hard-coded
- [ ] `API.md` (or a README section): every endpoint documented with an example request and response body — written well enough that a frontend developer who has never seen the code could integrate against it
- [ ] Statically serving a built frontend from Express is either implemented (a placeholder `dist/` is fine) or explicitly documented as a deployment decision — either way, show you know how the pieces would connect
- [ ] `npm start` runs the app in production mode; the README states every env var required and what happens when each is missing
- [ ] README: what the app is, setup from clean clone (install → db:init → seed → dev) verified by actually doing it in a fresh folder, test instructions, and a short architecture overview
- [ ] Git history shows the milestones — small commits throughout, not one "capstone done" commit

## Hints

- Scope is the boss fight. Two resources with one solid business rule, finished and tested, beats five resources half-built. Write a "not doing" list in `DESIGN.md` and defend it.
- Do Milestone 0 honestly. Every hour on the endpoint table and schema saves several downstream; when a frontend question comes up later ("what does the error body look like?"), the answer should already be written down.
- The business rule is where the service layer finally earns its keep — it's a rule about your *domain*, not about HTTP, so it belongs where Express can't see it. Test it through the API (the `409` response) and, if the logic is intricate, directly at the service level too.
- Graceful shutdown on Windows behaves differently than on Unix (Ctrl+C delivery, SIGTERM support) — Chapter 14 covers it; verify what actually happens in PowerShell rather than assuming the tutorial-standard Unix behavior.
- For CORS with cookie auth, three things must line up: the origin allowlist, the credentials flag on the server, and the frontend's fetch including credentials. If you're on JWTs, decide where the token lives before writing `API.md`, because it changes what you document.
- Write `API.md` *as you build each endpoint*, not at the end. Stale API docs are worse than none, and end-of-project documentation always gets rushed.
- When stuck architecturally, the question from Chapter 13 usually resolves it: "which layer would have to change if this requirement changed?" If the answer is "several," the responsibility is in the wrong place.
- Estimate → build → compare. Keep a rough hours log per milestone; the gap between estimate and reality is real professional data about yourself.

## Stretch Goals

- Deploy it for real once you reach the deployment-devops track — this project is deliberately shaped to be that track's first deployable artifact.
- Build (or adapt from your React track) a minimal frontend against `API.md` *without reading the backend code* — every place you're forced to peek is a documentation bug; fix it.
- Add OpenAPI: describe your API in an OpenAPI document and serve interactive docs from an endpoint — then compare it honestly with your hand-written `API.md`.
- Add one "operational" feature: request IDs flowing through logs, a `/api/health/ready` vs `/live` split, or basic metrics (request counts/durations) exposed for scraping.
- Swap SQLite for Postgres behind the repository layer (or vice versa) and write up exactly which files had to change — the punchline of Chapter 13 measured in a diff.
