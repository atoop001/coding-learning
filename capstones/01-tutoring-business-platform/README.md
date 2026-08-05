# Capstone 1: Tutoring Business Platform

A full-stack web application that runs a real tutoring business: session scheduling with a
calendar, accounts with roles, per-student session notes, and a public-facing information page.

You are not building a demo. You are building software you will actually operate for TNT
Tutoring, with yourself as the first user and your students and their parents as the second.
This is the flagship capstone of the workspace — everything you have learned across the tracks
(HTML/CSS, JavaScript, TypeScript, React, Node/Express, SQL, testing, deployment) converges
here into one shipped product.

**What v1 includes:**

- Session scheduling and booking with a calendar view
- Accounts with roles: admin/tutor on one side, student/parent on the other
- Session notes attached to each student, written by the tutor
- A public info page for the business (no login required)

**What v1 deliberately excludes** (these live in Stretch Goals): invoicing and payments,
email/SMS reminders, recurring sessions, self-service availability management.

## Why This Project

Most portfolio projects are clones: a todo app, a weather dashboard, a Netflix knockoff.
Interviewers have seen a thousand of them, and they all share the same fatal weakness — nobody
uses them, so nothing in them was ever a real decision.

This project is different, and the difference is unfakeable:

- **It has real users.** You, your students, and their parents. When the booking flow is
  confusing, an actual parent gets confused. That feedback loop teaches you things no tutorial
  can.
- **It has real data.** Real names, real schedules, real session histories. That forces you to
  take authorization, validation, and backups seriously, because the consequences of getting
  them wrong are real.
- **It operates in production.** Not "deployed once for the screenshot" — running week after
  week, with you on the hook when it breaks. Operating software is a skill employers pay for
  and almost no junior candidate can demonstrate.
- **It gives you the best possible interview story.** "I built and run the scheduling platform
  for my tutoring business" beats any clone. You can talk about why you made each decision,
  what broke, what you changed after two weeks of real use, and what you would do differently.

When an interviewer asks "tell me about a project," you will have a problem you genuinely had,
constraints you genuinely faced, and outcomes you can genuinely measure. That is a senior-shaped
conversation coming from a junior candidate, and it is worth more than five polished clones.

## Difficulty & Estimated Effort

**Advanced.** Expect roughly **40–80 hours spread across several weeks.**

This is deliberately not a weekend marathon. The project is structured in five phases, and each
phase ends with something real:

1. A written design
2. A tested API
3. A working UI
4. A live deployment
5. An operated, iterated product

Ship each checkpoint before moving to the next. If you try to build all of it before showing
anything to a user (even when the user is you), you will drown in half-finished features.
Small, finished, shipped increments — every time.

If a phase is taking far longer than expected, that is a signal to cut scope (see Hints), not
to push harder on an oversized plan.

## Prerequisites

Complete these before starting. The capstone assumes them; it does not reteach them.

- **html-css** — full track. You will hand-build the public page and need solid layout skills.
- **javascript** — full track. Everything else stands on this.
- **typescript** — full track. The frontend (and ideally the backend) is typed.
- **react** — full track, including the guided project. Components, hooks, forms, client-side
  routing, data fetching.
- **node-express** — through the authentication and testing chapters at minimum. You need
  sessions or tokens, password hashing, middleware, and API testing.
- **sql** — full track. You will design a relational schema with real foreign keys and write
  real queries.
- **testing-debugging-security** — full track. The backend ships with tests, and this app holds
  data about minors — security is not optional.
- **git-github** — you should already be committing in small, well-messaged increments by
  habit.
- **deployment-devops** — needed by Phase 4. You can start Phases 1–3 while still finishing
  this track.

If any of these feel shaky, go back and finish that track's guided project first. Struggling
with fundamentals in the middle of a capstone is the slowest possible way to learn them.

## Phased Milestones

### Phase 1: Requirements & Data Model — on paper, before any code

The most valuable phase, and the one most people skip. You have a rare advantage here: you can
interview the business owner (you) and the users (your students' parents) directly. Use it.
Every hour spent here saves several hours of rework later, because changing a diagram is free
and changing a deployed schema is not.

- [ ] Write 10–15 user stories drawn from how TNT Tutoring actually runs today, in the form
  "As a [parent / student / tutor / admin], I want to [action] so that [outcome]." Include the
  unglamorous ones: cancellations, reschedules, no-shows.
- [ ] Explicitly list what is **out of scope** for v1 (payments, reminders, recurring
  sessions). Write it down — scope creep is the #1 killer of this project, and a written
  boundary is much harder to drift past.
- [ ] Identify your entities and sketch an entity-relationship diagram on paper or a
  whiteboard tool. At minimum: users, students, sessions, session notes.
- [ ] Answer the modeling questions your real business raises. Is a "student" the same thing
  as a "user account," or can one parent account be linked to multiple students? (In a real
  tutoring business, it usually can.) Can a session ever have more than one student? Decide
  based on how you actually tutor, and write down the decision.
- [ ] Define each relationship and its cardinality: one parent to many students, one session
  to one tutor, and so on. Mark which foreign keys are required and which are optional.
- [ ] Decide what "cancelled" means in your data. Do you delete the session row or keep it
  with a status? (Keep it — you will want the history. Notice how a real business answers
  design questions a tutorial never asks.)
- [ ] Sketch the API on paper: list each endpoint, its method, URL, who is allowed to call it,
  and an example request/response shape. Something like
  `GET /api/sessions?from=2026-09-01&to=2026-09-07` returning
  `{ "sessions": [ { "id": 1, "startsAt": "...", "studentId": 3, "status": "booked" } ] }`
  is enough detail.
- [ ] Decide your roles (e.g., `admin`, `tutor`, `parent`, `student` — or collapse admin/tutor
  into one for v1, since that is you) and write a one-line permission rule for each entity:
  who can create, read, update, delete.
- [ ] Review the whole design against your user stories: can every story be satisfied by the
  schema and the API? Fix mismatches now, on paper, where changes are free.

### Phase 2: Backend — Express + SQL, auth with roles, tested

Build the API exactly as designed in Phase 1, and let the tests grow alongside the code rather
than after it.

- [ ] Set up the Express project with TypeScript, environment-based config (`.env` locally),
  and a connected SQL database. Put `.env` in `.gitignore` in your **first** commit, before
  any secret exists to leak.
- [ ] Implement the schema from Phase 1 as migrations — versioned and re-runnable — not ad-hoc
  `CREATE TABLE` statements run by hand. You should be able to rebuild the database from
  scratch with one command.
- [ ] Write a seed script that fills your local database with fake students and sessions, so
  development does not require manually re-creating data after every schema change.
- [ ] Build authentication: login, password hashing with a proper algorithm (bcrypt or
  argon2), and session or token handling. Decide whether accounts self-register or the admin
  creates them — for a tutoring business, admin-created accounts are simpler and safer, and
  that is a defensible v1 choice.
- [ ] Build authorization middleware that enforces roles: a parent can see only their own
  students' sessions and notes; the admin/tutor can see everything. Return 403 for forbidden,
  401 for unauthenticated — know the difference.
- [ ] Implement the sessions API: create, list (filterable by date range and student), update
  (reschedule, change status), cancel. Enforce business rules server-side: no double-booking
  the tutor, no booking in the past, no overlapping sessions.
- [ ] Implement session notes: create/edit notes attached to a session and student. Decide
  deliberately whether parents can read notes about their own child, and enforce whatever you
  decide.
- [ ] Validate all input at the API boundary and return a consistent JSON error shape (status
  code, machine-readable code, human-readable message) so the frontend can handle failures
  predictably.
- [ ] Write automated tests covering: auth (login success and failure), authorization (role A
  cannot read role B's data — this is the suite that matters most), booking conflict rules,
  and at least one full create-read-update flow per resource.
- [ ] Run the whole suite clean, then commit. This tested API is the checkpoint — the frontend
  builds on it without guessing.

### Phase 3: Frontend — React + TypeScript, booking calendar, role-based views

- [ ] Set up the React + TypeScript app with client-side routing and a typed API client
  layer — one module that owns all `fetch` calls and response types. Components never call
  the API directly.
- [ ] Build the auth flow: login page, persisted session, protected routes, logout, and
  graceful handling of expired sessions (redirect to login with a message, not a white screen
  or a silent failure).
- [ ] Build the calendar view of sessions — week view at minimum. You may use a calendar
  library or build a simple CSS grid yourself; either is defensible, but be able to explain
  the choice in one sentence. Booked, available, and cancelled slots should be visually
  distinct at a glance.
- [ ] Build the booking flow: pick a slot, confirm, see it appear on the calendar. Handle the
  failure case where the slot was taken between page load and submission — show a clear
  message and refresh availability instead of pretending it worked.
- [ ] Build role-based views: the admin/tutor sees all students and sessions with management
  controls (reschedule, cancel, add notes); a parent sees only their students, upcoming
  sessions, and permitted notes. The UI hides what a role cannot do — but remember the server
  is the real enforcer.
- [ ] Build the session notes UI for the tutor: fast to write immediately after a session,
  viewable per student as a running history. Optimize this screen for your real workflow —
  you will use it several times a week, and you will feel every extra click.
- [ ] Build the public-facing info page (no login): what TNT Tutoring offers, subjects, how to
  get in touch. This is the page you can put on a business card, so make it look like you
  meant it.
- [ ] Show loading and error states everywhere data is fetched. Real users on real connections
  will see them; a spinner and a retry beat a frozen screen.
- [ ] Make it work on a phone. Parents will book from their phones — walk the entire booking
  flow at a small viewport before calling this phase done.

### Phase 4: Ship It — Docker, CI, deployed, custom domain, HTTPS

- [ ] Containerize the application with Docker so it runs identically on your machine and in
  production. A single `docker compose up` should bring up the app and database locally.
- [ ] Set up CI (GitHub Actions) that installs, lints, builds, and runs the full test suite on
  every push. A red build blocks deployment — treat that as a rule, not a suggestion.
- [ ] Deploy to a PaaS with a genuinely free tier. As of mid-2026 that means Render's free
  web service paired with a free Neon Postgres — Railway and Fly.io no longer have true
  free tiers, and Render's own free Postgres expires after 30 days, so verify current
  terms before committing. Free services spin down when idle and cold-start in 30–60
  seconds; that's acceptable here — just note it in your README. Verify that data
  survives a redeploy before you trust it with real records.
- [ ] Manage all secrets through the platform's environment configuration — database URL,
  session secret, everything. Use placeholders like `<your-session-secret>` in any
  documentation or example files.
- [ ] Audit your git history for leaked secrets. If one was ever committed, rotate it —
  deleting the file in a later commit does not un-leak it.
- [ ] Point a custom domain at the deployment and confirm HTTPS works end to end. Most PaaS
  platforms provision certificates automatically — verify, don't assume.
- [ ] Set up automated database backups (or a scheduled dump you can restore from) and perform
  one **test restore**. A backup you have never restored is a hope, not a backup.
- [ ] Do a production smoke test: create a real account, book a session, write a note, then
  log in as the other role and confirm you see exactly what that role should see — and
  nothing more.

### Phase 5: Operate & Iterate — the phase that makes this capstone special

This phase produces no new architecture, and it is the most valuable one on your resume.
Almost nobody at your level can talk about operating software; after this, you can.

- [ ] Use the platform to run your actual tutoring schedule for **at least two weeks**. All
  real bookings and session notes go through it. No shadow spreadsheet, no fallback texting —
  if the app makes something painful, that pain is the data you came for.
- [ ] Keep an operations log (a simple markdown file in the repo is fine): every pain point,
  confusing moment, bug, workaround, and feature wish — yours and your users'. Date each
  entry.
- [ ] Invite at least one real parent or student to book or view sessions, and record what
  confused them **without explaining anything**. Watching a real user get lost in your UI is
  humbling and irreplaceable.
- [ ] From the log, pick the three highest-impact improvements and ship them one at a time,
  each through the full pipeline: branch, tests, CI green, deploy. Resist the urge to batch
  them — three small shipped fixes teach you more than one big one.
- [ ] Write a short retrospective in the project README: what real usage taught you that
  building did not. This paragraph will do more for you in interviews than any feature.

## Definition of Shipped

The capstone is done when every box below is checked — not before, and nothing extra required:

- [ ] The app is live at a custom domain over HTTPS, and has stayed up through normal use.
- [ ] The repository README includes screenshots, a description of the architecture (what
  talks to what, and why you chose each piece), and setup instructions good enough that a
  stranger could run it locally.
- [ ] The test suite runs in CI on every push and is green on the main branch.
- [ ] No secrets exist anywhere in the git history; configuration is entirely
  environment-based.
- [ ] At least one **real** booking — a genuine TNT Tutoring session — has flowed through the
  system end to end: booked, held, and noted.

## Hints

Nudges for the parts that reliably hurt. No solutions here — the struggle is where the
learning is — but these will keep you struggling with the right things.

**Timezones will hurt you if you improvise.** Store every timestamp in UTC in the database,
always. Convert to the user's local time only at the display layer. Never store "3:00 PM" as a
bare string — a bare local time is a bug waiting for daylight saving to trigger it. Test what
happens to a weekly schedule across a DST transition; that boundary is where naive scheduling
logic quietly breaks. If your business operates in a single timezone you can simplify the UI,
but the storage rule still holds.

**Authentication and authorization are different problems — treat them that way.**
Authentication answers "who are you?" (login, sessions, passwords). Authorization answers
"what are you allowed to do?" (a parent reading another family's session notes must get a
403). The most common security hole in apps like this is an endpoint like
`/api/students/42/notes` that checks you are logged in but never checks that student 42 is
*yours*. Write a test for exactly that case on every resource. UI hiding is courtesy;
server-side checks are security. And remember whose data this is — children's names and
learning records deserve your most paranoid code.

**Keep business logic out of route handlers.** A route handler should parse and validate
input, call a function that does the real work, and shape the response. The rule "a session
cannot overlap another session for the same tutor" belongs in a plain, independently testable
function — not inline in an Express callback tangled up with `req` and `res`. You will feel
the payoff the first time you test conflict rules without spinning up a server, and again in
the interview when you can explain *why* your code is layered this way.

**The double-booking check belongs in the database transaction, not just in JavaScript.**
Checking availability and then inserting are two steps, and two requests can interleave
between them. Understand the race, then close it with a transaction or a database constraint.
Even if collisions are unlikely at your scale, being able to explain this race condition and
your fix is interview gold.

**Let the database enforce what the database can enforce.** Foreign keys, NOT NULL, unique
constraints — turn them on. Application code has bugs; constraints are the last line of
defense that keeps a bug from becoming corrupt data. When a constraint violation surfaces
during development, it is telling you about a logic error earlier in the flow. Listen to it.

**When you are stuck or behind, simplify scope — never quality.** A calendar that only shows
one week, with the tutor's availability hardcoded in a config table, fully tested and
deployed, beats a half-built drag-and-drop scheduler every single time. Cut features
ruthlessly; never cut tests, auth checks, or backups. Phase 5 exists precisely so features can
arrive later, driven by real need instead of guesswork.

**Commit small and write messages your future self can read.** This project will span weeks.
"Fix stuff" tells you nothing in month two; "Reject bookings that overlap an existing session
for the same tutor" is documentation for free. Your git history is part of the portfolio —
interviewers do look.

## Stretch Goals

Only after the Definition of Shipped is fully met. Each of these is a meaningful project on
its own — pick the one your real usage log says you need most, not the one that sounds
coolest.

- **Email reminders** — automated "session tomorrow at 4 PM" emails to parents via a
  transactional email service. This is your first scheduled/background job, which is a
  genuinely new architectural skill.
- **Invoicing and payments** — generate invoices per student per month; integrate Stripe
  **in test mode**, using only Stripe's published test keys and test card numbers. Do not
  touch live payment credentials until the flow is genuinely hardened and you understand the
  liability you are taking on.
- **Recurring sessions** — "every Tuesday at 4 PM" is deceptively hard: exceptions, cancelling
  one occurrence versus the whole series, and DST all bite. Research how established calendar
  systems model recurrence before designing your own.
- **Availability management** — let the tutor define open hours inside the app instead of a
  config table, so parents can only book genuinely open slots.
- **SMS notifications** — booking confirmations or reminders via an SMS API. Compare cost and
  deliverability against email, and be ready to justify whichever you choose.

## Telling the Story

You built and operate real software for a real business. Do not undersell that — most
candidates would trade their entire portfolio for this one project.

**On your resume**, lead with the outcome and the fact of operation, not the tech list:

> Built and operate a full-stack scheduling platform (React, TypeScript, Node, PostgreSQL)
> for a tutoring business; handles all real session bookings and student records, deployed
> with Docker and CI to a custom domain.

The tech stack is the parenthetical. "Built and operate," "real," and a concrete business
function are the load-bearing words. Avoid vague verbs like "worked on" or "helped with" —
you built it, you run it, say so.

**In interviews**, structure the narrative as problem → constraints → decisions → operating
lessons:

1. **Problem.** "I run a tutoring business and scheduling lived in text messages and a
   spreadsheet. Double-bookings and lost session notes were costing me real time every week."
2. **Constraints.** Real users including non-technical parents; data about minors that made
   authorization non-negotiable; a solo developer with limited hours; a business that could
   not pause while I built.
3. **Decisions — and why.** Why a relational database (the data is genuinely relational:
   parents to students to sessions to notes). Why role checks live server-side. Why business
   rules sit outside route handlers. Why UTC everywhere. Why recurring sessions were cut from
   v1. Every decision has a "because" rooted in a real constraint — this is exactly what
   separates you from clone-builders, who can only recite *what* they built.
4. **Operating lessons.** What two-plus weeks of real usage exposed that building never did.
   What a real parent found confusing. What you changed because of it, and how the CI
   pipeline let you ship those changes safely. Almost no junior candidate can speak to this —
   it is your strongest material, so do not let modesty edit it out.

**Metrics to mention** — small honest numbers beat vague big claims:

- Sessions booked through the platform to date
- Weeks in continuous operation
- Number of active users (even "4 families" is a real number, and its honesty is the point)
- Test count, and the fact that CI blocks every deploy
- Improvements shipped post-launch, driven by logged user feedback

"It has handled every booking for my business since September" is a stronger sentence than any
feature list you could recite.

One final note: keep the operations log and the retrospective safe. Six months from now they
are your interview prep — written by the only person who was there.
