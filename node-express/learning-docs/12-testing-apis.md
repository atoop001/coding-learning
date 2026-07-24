# Chapter 12: Testing APIs

## Overview

Until now you've tested your APIs by hand: start the server, poke it with a REST client, eyeball the response. That works for one endpoint; it collapses at twenty, because every change means re-poking everything you might have broken. Automated tests turn that manual ritual into a command that runs in seconds and fails loudly when behavior regresses. This chapter covers testing Express APIs specifically: **vitest** as the test runner (with Node's built-in `node:test` as the dependency-free alternative), **supertest** for making real HTTP requests against your app *without* starting a listening server, setting up **test databases** so tests are fast and isolated, and the honest version of the mocking-vs-real-dependencies debate. Just as importantly, you'll learn *what* to test in an API — status codes, response shapes, validation, auth — and what's a waste of time. Tests are also a hiring signal: a portfolio project with a real test suite stands out immediately, and TDD-ish habits are expected on the job.

## Definitions & Explanations

- **Automated test** — code that exercises your code and asserts on the outcome. Run it after every change; a green suite means the behaviors you encoded still hold.
- **Test runner** — the tool that finds test files, executes them, and reports results. **vitest** is a popular, fast runner with a great watch mode; Node 22 also ships a built-in runner (**`node:test`** with `node --test`) that needs zero dependencies. This track teaches vitest and notes the built-in where it differs — the concepts transfer completely.
- **Assertion** — a single checked claim, e.g. `expect(res.status).toBe(201)`. If it's false, the test fails with a message showing expected vs actual.
- **`describe` / `it` (or `test`)** — the standard structure: `describe` groups related tests; each `it` is one behavior stated as a sentence ("it returns 404 for a missing bookmark").
- **Lifecycle hooks** — `beforeEach`, `afterEach`, `beforeAll`, `afterAll`: setup/teardown that runs around tests. Fresh state per test lives here.
- **supertest** — a library that takes your Express `app` object and makes real HTTP requests to it *in-process* — no `app.listen()`, no port, no separate server to start. You get actual status codes, headers, and JSON bodies, exercising the full middleware stack.
- **End-to-end (route-level) test** — a test that enters through HTTP and exits through HTTP, crossing routing, middleware, validation, handlers, and the database. For APIs this is the highest-value test type: it checks what clients actually experience.
- **Unit test** — a test of one function in isolation (e.g. a pure service function). Useful for tricky logic; less useful for glue code that mostly calls Express and SQL.
- **Test database** — a separate database used only by tests, rebuilt to a known state for each test or suite, so tests never touch development data and never depend on leftovers from previous runs. With SQLite this is gloriously easy: `:memory:` gives you a fresh in-RAM database in microseconds.
- **Fixture / seed data** — known rows inserted before a test so it has something to act on ("given a user with 2 bookmarks...").
- **Test isolation** — no test depends on another test having run first, or on shared mutable state. Isolated tests can run in any order and fail for exactly one reason.
- **Mock (test double)** — a stand-in replacing a real dependency, with canned behavior you control. Mocks make sense for things that are slow, non-deterministic, or external (a payment API, an email sender). Mocking your *own database layer* is usually a mistake when a real in-memory database is this cheap — see below.
- **Watch mode** — the runner re-runs affected tests on every file save (`vitest` does this by default when run interactively). This is the tight feedback loop that makes tests pleasant instead of a chore.

## Code Examples

### Setup

```powershell
npm install --save-dev vitest supertest
```

```json
// package.json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest"
  }
}
```

`vitest run` executes once and exits (CI-friendly); bare `vitest` stays in watch mode. (Built-in alternative: `node --test` runs files named `*.test.js` with zero installs; assertions come from `node:assert`.)

### Make your app testable: export it

supertest needs your `app` *without* a listening server, so split creation from listening:

```js
// ❌ Naive: app.js both builds the app and listens. Importing it in a test
// starts a real server on a real port — slow, and two test files collide.
const app = express();
/* ...routes... */
app.listen(3000);
```

```js
// ✅ app.js — builds and exports the app. No listen() here.
import express from 'express';
import { bookmarksRouter } from './routes/bookmarks.js';
import { errorHandler } from './middleware/error-handler.js';

export function createApp(db) {          // dependency injection: pass the db in —
  const app = express();                 // tests pass a test db, prod passes the real one
  app.use(express.json());
  app.use('/api/bookmarks', bookmarksRouter(db));
  app.use(errorHandler);
  return app;
}
```

```js
// server.js — the ONLY file that listens. Tests never import this.
import { createApp } from './app.js';
import { openDb } from './data/db.js';
import { config } from './config.js';

const app = createApp(openDb(config.databasePath));
app.listen(config.port, () => console.log(`listening on ${config.port}`));
```

This split (Chapter 13 formalizes it) is the single change that makes everything below possible.

### First route tests with supertest

```js
// tests/bookmarks.test.js
import { describe, it, expect, beforeEach } from 'vitest';
import request from 'supertest';
import { createApp } from '../app.js';
import { openDb, migrate } from '../data/db.js';

let app;

beforeEach(() => {
  const db = openDb(':memory:');   // brand-new in-memory SQLite per test
  migrate(db);                     // create tables (your schema function)
  app = createApp(db);
});

describe('POST /api/bookmarks', () => {
  it('creates a bookmark and returns 201 with the new resource', async () => {
    const res = await request(app)
      .post('/api/bookmarks')
      .send({ title: 'MDN', url: 'https://developer.mozilla.org' });

    expect(res.status).toBe(201);
    expect(res.body).toMatchObject({ title: 'MDN', url: 'https://developer.mozilla.org' });
    expect(res.body.id).toBeDefined();
  });

  it('rejects a missing url with 400 and does not create anything', async () => {
    const res = await request(app)
      .post('/api/bookmarks')
      .send({ title: 'no url' });

    expect(res.status).toBe(400);
    expect(res.body.error).toBeDefined();       // your consistent error shape (Ch. 8)

    const list = await request(app).get('/api/bookmarks');
    expect(list.body).toHaveLength(0);          // the invalid write really was rejected
  });
});

describe('GET /api/bookmarks/:id', () => {
  it('returns 404 for an id that does not exist', async () => {
    const res = await request(app).get('/api/bookmarks/9999');
    expect(res.status).toBe(404);
  });
});
```

Run it:

```powershell
npm test          # once
npm run test:watch  # re-runs on save — leave this running while you code
```

Note what these tests never do: import route handlers directly, inspect internal variables, or care *how* the handler is written. They test the contract — request in, response out — which means you can refactor internals freely and the tests still guard behavior.

### Testing authenticated routes

Session cookies flow through supertest if you hold onto them — `request.agent()` does it automatically, like a browser:

```js
import request from 'supertest';

it('lets a logged-in user read only their own bookmarks', async () => {
  const agent = request.agent(app);   // an agent remembers cookies between calls

  await agent.post('/api/auth/register')
    .send({ email: 'a@example.com', password: 'long-enough-password' });
  await agent.post('/api/auth/login')
    .send({ email: 'a@example.com', password: 'long-enough-password' });

  await agent.post('/api/bookmarks').send({ title: 'mine', url: 'https://example.com' });

  const res = await agent.get('/api/bookmarks');
  expect(res.status).toBe(200);
  expect(res.body).toHaveLength(1);

  // And without a session? Plain request(app) has no cookie:
  const anon = await request(app).get('/api/bookmarks');
  expect(anon.status).toBe(401);
});
```

Tip: signing up through your own endpoints in `beforeEach` (or a small `loginAs(agent, email)` helper) beats inserting session rows by hand — it tests the real flow and survives refactors.

### Mocking vs real dependencies — the honest version

```js
// Option A: mock the data layer.
// You now maintain a fake database. The test passes even if your real SQL
// has a typo, because the SQL never runs. What exactly did you verify?
vi.mock('../data/bookmarks.js', () => ({
  list: vi.fn(() => [{ id: 1, title: 'fake' }]),
}));
```

```js
// Option B: real in-memory SQLite (what we did above).
// Runs in ~1ms, exercises your actual SQL, schema, and constraints.
const db = openDb(':memory:');
```

Rule of thumb: **use the real thing when the real thing is cheap; mock what is expensive, external, or nondeterministic.** In-memory SQLite is cheap — use it. A third-party email API is external — mock it (you don't want tests emailing people). `Date.now()` in expiry logic is nondeterministic — control it with vitest's fake timers (`vi.useFakeTimers()`). Over-mocked suites are a known trap: everything is green, nothing is tested. If you later use Postgres (Chapter 9), the same principle holds but setup is heavier — a real test database with per-test truncation, or accepting SQLite-in-tests with the small dialect-difference risk. Both are defensible; know which you chose and why.

### What to test in an API — a working checklist

For each endpoint, in priority order:

1. **The happy path** — correct status code *and* response shape for a valid request.
2. **Validation rejections** — bad/missing input → 400 with your standard error shape, and *no state change*.
3. **Not-found paths** — missing ids → 404, not 500 and not an empty 200.
4. **Auth** — no session/token → 401; wrong user's resource → 403/404 (whichever you chose in Chapter 10 — test that it's consistent!).
5. **Edge cases with teeth** — duplicate email on register → 409; empty list → `[]` not error.

And what *not* to test: Express itself (routing works, `express.json` parses — not your job), exact human-readable error message wording (assert an `error` field exists, not its prose, or every copy tweak breaks tests), and private helper functions already covered through the routes that use them.

## Common Pitfalls

1. **Testing against the development database.** Tests wipe or pollute your real dev data, and pass/fail depends on what was left there yesterday. Correction: `:memory:` SQLite created in `beforeEach` — every test starts from a known, empty world.
2. **Tests that depend on each other** ("test 2 uses the bookmark test 1 created"). They pass in order, then fail mysteriously when run alone or in parallel. Correction: each test creates its own data; if setup is shared, it lives in `beforeEach`, which runs fresh every time.
3. **Forgetting to `await` supertest calls.** The test finishes before the request does and reports a false green — vitest may warn, but not always in a way beginners notice. Correction: every `request(app)...` is awaited; make the test function `async`.
4. **Calling `app.listen()` in the file tests import.** Ports collide, tests hang, watch mode leaks servers. Correction: the export-app/`server.js`-listens split shown above; only `server.js` listens, and no test imports it.
5. **Mocking so much that nothing real runs.** A suite where routes, database, and validation are all mocked verifies only that your mocks call your mocks. Correction: default to route-level tests with a real in-memory DB; mock only external/expensive/nondeterministic things.
6. **Asserting too little** — checking `res.status` but never the body, so an endpoint returning the wrong data (or a password hash! Chapter 10) stays green. Correction: assert status *and* shape (`toMatchObject`, `toHaveLength`), and for auth endpoints assert sensitive fields are absent: `expect(res.body.password_hash).toBeUndefined()`.
7. **Only testing happy paths.** The bugs that hurt live in the sad paths — bad input, missing rows, wrong user. Correction: the checklist above; a decent heuristic is at least one sad-path test per happy-path test.

## Practice Exercises

1. Refactor your bookmarks API into the `createApp(db)` + `server.js` split, verify it still runs normally with `npm run dev`, then write your first two supertest tests: `GET /api/bookmarks` returns 200 and `[]` on an empty database, and `POST` then `GET` returns the created bookmark. Leave `npm run test:watch` running for the rest of these exercises.
2. Work through the checklist for your `POST /api/bookmarks` endpoint: happy path, each validation failure your Zod schema can produce (missing field, wrong type, invalid URL), and confirmation that failed requests create no rows. Aim for 5–7 focused `it` blocks with sentence-style names.
3. Cover the auth flows from Chapter 10 with `request.agent()`: register → login → access a protected route; access without login → 401; user A cannot read user B's bookmark (create both users in the test). Extract a `registerAndLogin(app, email)` helper when the duplication annoys you.
4. Write one test that would have caught a real bug: temporarily reintroduce a mistake you actually made earlier in this track (e.g. returning 200 instead of 201, or forgetting `requireAuth` on one route), run the suite, and confirm a test fails. Revert the bug. (If your suite stayed green, you found a coverage hole — fill it.)
5. Test time-dependent behavior: if your sessions or tokens expire, use `vi.useFakeTimers()` and `vi.setSystemTime()` to test that an expired credential yields 401 — without your test actually waiting. Read the vitest fake-timers docs first.
6. Rewrite any three of your vitest tests using Node's built-in runner (`node --test`, `node:assert`) in a separate scratch folder, then write 3–4 sentences in `NOTES.md`: what did vitest give you that the built-in didn't, and when might the zero-dependency option be the right call?
