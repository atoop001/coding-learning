# Project 6: Tested & Hardened API

## Description

This project adds no new features. Instead, you take an API you've already built — Project 4 (bookmarks + SQLite) or Project 5 (authenticated notes; harder but richer) — and make it *trustworthy*: a full automated test suite with vitest and supertest, tests running against an isolated test database, a security-middleware audit, error handling verified by tests rather than by hope, and configuration cleanly split by environment.

The mindset shift is the deliverable. Until now, "it works" meant you poked it with a REST client and the responses looked right. After this project, "it works" means `npm test` proves it — every endpoint's happy path, its validation failures, its 404s, and (if you chose Project 5) its authorization boundaries. When a test suite like this exists, refactoring stops being scary, and that's precisely what employers mean when they list "testing" in a job ad.

Where this project says **test-first**: for any behavior you have to *add or fix*, write the failing test before the code. For behavior that already exists, you're writing characterization tests — and you should expect a few of them to fail and expose real bugs. Fixing those is part of the project, and each fix starts from the failing test you just wrote.

## Difficulty & Estimated Effort

Advanced — 8–12 hours (on top of a completed Project 4 or 5).

## Chapters Used

- Chapter 8: Validation & error handling
- Chapter 11: Security & configuration
- Chapter 12: Testing APIs

## Requirements

**Make the app testable**

- [ ] The Express `app` is created and exported separately from the code that calls `listen` — supertest needs the app without a running server
- [ ] Database location comes from configuration, so tests can point at their own database; no code path hard-codes the dev database file
- [ ] vitest and supertest installed as devDependencies; `npm test` runs the suite; `npm run test:watch` exists for development

**Test database isolation**

- [ ] Tests run against a dedicated test database (a separate file or in-memory SQLite) — running the suite never touches your dev data (prove it: note a dev record, run the suite, confirm it's untouched)
- [ ] Each test starts from a known state: schema applied and data reset between tests (per-test cleanup or fresh setup in hooks — your choice, but deliberate)
- [ ] The full suite passes when run twice in a row and when a single test file is run alone — no ordering dependence

**Coverage of behavior (per resource)**

- [ ] Happy paths: every endpoint has at least one test asserting status code *and* response body shape
- [ ] Validation failures: for each write endpoint, tests for missing required fields, wrong types, and at least one boundary case (empty string, over-length, malformed URL/email as applicable) — asserting both the 4xx status and your standard error shape
- [ ] Not-found paths: GET/PUT-or-PATCH/DELETE against a nonexistent ID each return `404` in the standard shape
- [ ] Contract details: `201` + `Location` on create, `204` + empty body on delete, unknown routes return your JSON 404 (not HTML), malformed JSON bodies return a clean `400`
- [ ] If Project 5: auth rules under test — protected routes return `401` without credentials; user A cannot read, update, or delete user B's data (the `404` isolation tests); register/login/logout flow works end to end through supertest
- [ ] At least one test proves the centralized error handler catches an *unexpected* thrown error and responds with a `500` in the standard shape, with no stack trace in the body

**Security hardening audit**

- [ ] helmet applied (if it wasn't already) and a test asserts at least one expected security header is present on responses
- [ ] Rate limiting configured via env so it can be effectively disabled in tests without changing code
- [ ] Written audit note (README or `SECURITY.md`): walk Chapter 11's checklist against this app — what's covered (parameterized queries, validation, error hygiene, headers, rate limits, secrets handling), what's consciously out of scope, and one thing you fixed as a result of the audit
- [ ] The injection check from Project 4 is now a permanent automated test (hostile-looking input is stored/returned literally, table still intact)

**Environment configuration**

- [ ] One config module reads all env vars; `NODE_ENV` distinguishes development/test/production behavior where behavior must differ (logging verbosity, rate limits, database path)
- [ ] `.env.example` updated to document every variable including test-relevant ones
- [ ] Tests set their environment programmatically or via vitest config — running `npm test` needs no manual env setup

**Coverage look**

- [ ] Generate a coverage report (vitest's coverage flag), read it, and write three sentences in the README: where coverage is thin, whether that gap matters, and what you'd test next — the goal is judgment, not a percentage target

## Hints

- The app/server split is Chapter 12's first move and everything else depends on it. If your `app.listen` call is tangled into the same file that builds routes, do that refactor first, verify the dev server still runs, then start testing.
- Decide early how a test gets a database: a fresh in-memory SQLite per test file is fast and perfectly isolated, but your schema-apply step has to run in test setup. If your Project 4 `db:init` logic is a script rather than an importable function, extract the function — a small refactor that pays off immediately.
- Characterization tests will surprise you. Common finds: malformed JSON produces an HTML error page, an async handler bypasses the error middleware, delete of a missing ID returns `204`. Don't patch the test to match the bug — the test is right; fix the app (test-first: you already have the failing test).
- Supertest chains read like the HTTP they perform: request the app, perform a method on a path, send a body, expect a status. For auth flows, remember each supertest request is independent — you'll need to carry the cookie or token between calls explicitly. Chapter 12 shows the pattern; work from what it taught rather than copy-pasting.
- "What to test in an API" (Chapter 12) is your scope control. You're testing through HTTP — status, shape, rules — not asserting on internal function calls. If you're reaching for mocks of your own repository, reread the chapter's mocking-vs-real-dependencies section; with SQLite there's rarely a reason.
- Rate limiting in tests: the clean approach is config, not code branches — limits high enough (or the limiter absent) under test env. But keep at least the *login* limiter testable if you're on Project 5 and want a stretch test for it.
- If the suite is slow or flaky, suspect shared state between tests before anything else — usually a shared database file or a module-level singleton holding the old connection.

## Stretch Goals

- Add a GitHub Actions workflow that runs `npm test` on every push — your first CI pipeline, and a preview of the deployment-devops track.
- Test the rate limiter itself: with tight test-configured limits, assert the N+1th login attempt returns `429`.
- Add mutation-level confidence the cheap way: comment out one validation rule, run the suite, and confirm at least one test fails; if nothing fails, you found a coverage gap — close it. Repeat for two more rules.
- Introduce a deliberately breaking refactor (rename a service function, restructure a repository) and let the suite guide the fix — write one paragraph on what the tests caught and what they couldn't have.
- Measure suite runtime, then make it faster (in-memory DB, parallel test files) without losing isolation — document the before/after numbers.
