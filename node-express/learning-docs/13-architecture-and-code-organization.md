# Chapter 13: Architecture & Code Organization

## Overview

Every Express app you have built so far probably lives in one or two files, and that has been fine — for learning, small files are honest and visible. But real backends grow. A single `server.js` holding routes, validation, business rules, and SQL becomes impossible to test, painful to change, and scary to touch. This chapter teaches the standard cure: **layering**. You will learn the router → controller → service → data-layer structure that most professional Node backends use in some form, why each layer exists, how to keep Express itself out of your business logic, and how to refactor a messy single-file app step by step. You will also learn when layering is *overkill*, because adding architecture an app doesn't need is its own kind of mess. This chapter is the difference between "I can make an endpoint work" and "I can work on a backend with other people."

## Definitions & Explanations

- **Architecture** — the deliberate arrangement of code into parts with defined responsibilities and defined relationships. Not a framework feature; a set of decisions you make.

- **Layer** — a group of modules that all do the same *kind* of job. In a typical Express backend, the layers are: routing (HTTP wiring), controllers (translate HTTP ↔ plain data), services (business rules), and the data layer (talk to the database). Each layer only talks to the layer directly below it.

- **Router** — an `express.Router()` module that maps URLs and HTTP methods to controller functions. It knows *paths*; it should know almost nothing else. One router per resource (`users.router.js`, `bookmarks.router.js`) is the usual split.

- **Controller** (also called a **handler**) — the function that receives `(req, res, next)`. Its job is translation: pull data out of the request (params, body, query), call a service with plain JavaScript values, and turn the service's answer into an HTTP response (status code + JSON). Controllers should be thin — if a controller has an `if` about business rules, that logic is in the wrong place.

- **Service** — a module of plain functions that implement your **business logic**: the rules that would still exist if you rewrote the app as a CLI or a desktop app. "A user can't bookmark the same URL twice," "an order over $100 ships free" — that's service territory. Crucially, services never see `req` or `res`. They take plain arguments and return plain values (or throw).

- **Data layer** (also **repository** or **data access layer / DAL**) — the only code allowed to touch the database. It exposes intention-revealing functions like `findUserByEmail(email)` and hides the SQL inside. If you ever switch from SQLite to Postgres, this is the only layer that changes.

- **Separation of concerns** — the principle behind all of this: each piece of code should have one reason to change. HTTP details change? Touch controllers. Business rule changes? Touch a service. Schema changes? Touch the data layer.

- **Dependency direction** — the rule that dependencies point *downward only*: routers import controllers, controllers import services, services import the data layer. The data layer imports nothing above it. A service that imports a controller, or a data-layer file that imports Express, is an architecture bug even if the app runs.

- **Fat controller** — an anti-pattern: a route handler that does everything (parse, validate, query, compute, respond). It works, but nothing in it can be reused or tested without spinning up HTTP.

- **Business logic** — the rules of your problem domain, as opposed to plumbing. The test: would this rule survive a total change of delivery mechanism (HTTP → CLI → queue worker)? If yes, it's business logic and belongs in a service.

## Code Examples

### The mess we start from

A realistic "everything in one place" endpoint. It works. It is also untestable without HTTP, welded to SQLite, and mixes four jobs in one function:

```js
// server.js — the "before" picture (condensed)
import express from "express";
import Database from "better-sqlite3";

const db = new Database("app.db");
const app = express();
app.use(express.json());

app.post("/api/bookmarks", (req, res) => {
  // 1. validation
  const { url, title } = req.body;
  if (!url || !/^https?:\/\//.test(url)) {
    return res.status(400).json({ error: "url must start with http(s)://" });
  }
  // 2. business rule: no duplicate URLs per user
  const existing = db
    .prepare("SELECT id FROM bookmarks WHERE user_id = ? AND url = ?")
    .get(req.userId, url);
  if (existing) {
    return res.status(409).json({ error: "already bookmarked" });
  }
  // 3. data access
  const info = db
    .prepare("INSERT INTO bookmarks (user_id, url, title) VALUES (?, ?, ?)")
    .run(req.userId, url, title ?? url);
  // 4. HTTP response shaping
  res.status(201).json({ id: info.lastInsertRowid, url, title: title ?? url });
});

app.listen(3000);
```

To unit-test the duplicate rule, you'd have to boot Express and a database and send real HTTP. That's the smell.

### The refactored file tree

```text
src/
  app.js                     # builds the Express app (no listen) — importable by tests
  server.js                  # imports app, calls listen — the only "run me" file
  db.js                      # opens the database connection, exports it
  routes/
    bookmarks.router.js      # URL wiring only
  controllers/
    bookmarks.controller.js  # req/res translation only
  services/
    bookmarks.service.js     # business rules; no Express, no SQL
  repositories/
    bookmarks.repo.js        # SQL only
  middleware/
    error-handler.js
    validate.js
```

Splitting `app.js` from `server.js` matters for Chapter 12's supertest work: tests import the app without opening a port.

### The same endpoint, layered

```js
// repositories/bookmarks.repo.js — SQL lives here and nowhere else
import { db } from "../db.js";

export function findByUserAndUrl(userId, url) {
  return db
    .prepare("SELECT * FROM bookmarks WHERE user_id = ? AND url = ?")
    .get(userId, url);
}

export function insert(userId, url, title) {
  const info = db
    .prepare("INSERT INTO bookmarks (user_id, url, title) VALUES (?, ?, ?)")
    .run(userId, url, title);
  return { id: info.lastInsertRowid, url, title };
}
```

```js
// services/bookmarks.service.js — plain functions, plain values.
// Note: no req, no res, no SQL. This file would work in a CLI app unchanged.
import * as repo from "../repositories/bookmarks.repo.js";

export class DuplicateBookmarkError extends Error {}

export function createBookmark(userId, { url, title }) {
  if (repo.findByUserAndUrl(userId, url)) {
    // The service reports the *fact* (duplicate); it does not choose a
    // status code — 409 is an HTTP concept, so that decision stays upstairs.
    throw new DuplicateBookmarkError("already bookmarked");
  }
  return repo.insert(userId, url, title ?? url);
}
```

```js
// controllers/bookmarks.controller.js — translation only
import * as service from "../services/bookmarks.service.js";
import { DuplicateBookmarkError } from "../services/bookmarks.service.js";

export function create(req, res, next) {
  try {
    const bookmark = service.createBookmark(req.userId, req.body);
    res.status(201).json(bookmark);
  } catch (err) {
    if (err instanceof DuplicateBookmarkError) {
      return res.status(409).json({ error: err.message });
    }
    next(err); // unknown errors go to the central error handler (Chapter 8)
  }
}
```

```js
// routes/bookmarks.router.js — wiring only
import { Router } from "express";
import * as controller from "../controllers/bookmarks.controller.js";
import { validateBookmark } from "../middleware/validate.js";

export const bookmarksRouter = Router();
bookmarksRouter.post("/", validateBookmark, controller.create);
```

```js
// app.js — assembly point
import express from "express";
import { bookmarksRouter } from "./routes/bookmarks.router.js";
import { errorHandler } from "./middleware/error-handler.js";

export const app = express();
app.use(express.json());
app.use("/api/bookmarks", bookmarksRouter);
app.use(errorHandler);
```

More files, more imports — and each file is now boring, which is the goal. `createBookmark` can be unit-tested with a test database and zero HTTP. Swapping SQLite for Postgres touches `db.js` and the `repositories/` folder only.

### Where validation lives

Two layers validate, for two different reasons:

- **Shape validation** (is `url` a string? is the body JSON?) belongs at the HTTP edge — middleware or controller, e.g. a zod schema from Chapter 8. Reject garbage before it travels deeper.
- **Business validation** (is this URL already bookmarked? does this user own this resource?) belongs in the service, because it needs domain knowledge and often the database.

If you find shape-checking inside a service or SQL inside validation middleware, a boundary has leaked.

### When layering is overkill

Honesty time: a four-endpoint webhook receiver does not need five folders. Architecture has a cost — indirection, file-hopping, boilerplate — and the payoff only arrives when the app is big enough or lives long enough. Reasonable rule of thumb:

- **One file** is fine up to roughly a handful of routes with trivial logic.
- **Split routers out** as soon as you have two resources.
- **Add services** the moment a business rule appears in a second place, or the first time you *want* to test logic without HTTP.
- **Add a repository layer** when SQL starts appearing in more than one file, or when you write your first data-layer test.

Growing into architecture beats starting with an empty cathedral. But know the shape in advance so you refactor *toward* something.

## Common Pitfalls

1. **Passing `req` or `res` into a service.** The moment a service signature includes `req`, the layer boundary is gone — the service can now secretly read anything, and testing it requires faking an entire request. Correction: controllers extract plain values (`req.params.id`, `req.body`) and pass those.

2. **Services choosing HTTP status codes.** `res.status(404)` inside a service couples business logic to HTTP. Correction: services throw typed errors (`NotFoundError`, `DuplicateBookmarkError`); the controller or error-handling middleware maps error type → status code.

3. **Skipping the service layer "just this once."** A controller that calls the repository directly for a "simple" endpoint starts the rot — the next developer copies the pattern, and soon rules live in random controllers. Correction: even a one-line pass-through service keeps the seam; or consciously decide the whole app is small enough not to layer, and be consistent about *that*.

4. **Circular imports between layers.** A service importing from a controller (usually to reuse some helper) creates a cycle that ES modules resolve confusingly — you get `undefined` exports at runtime. Correction: dependencies point down only; shared helpers move to a `utils/` or domain module both can import.

5. **Organizing by layer when the app is large, and never by feature.** `controllers/` with 40 files means every feature change touches four distant folders. Correction: at larger scale, group by feature (`src/bookmarks/` containing its own router, service, repo). For this track's project sizes, layer-first folders are fine — just know the alternative exists.

6. **A `utils.js` junk drawer.** Anything that doesn't obviously fit gets tossed into `utils`, which grows into an unstructured second app. Correction: name modules by what they contain (`slug.js`, `dates.js`); if you can't name it, you don't understand it yet.

7. **Refactoring everything at once with no tests.** A big-bang restructure of a working app, without tests, reliably breaks it in quiet ways. Correction: refactor one route at a time — extract repo, extract service, re-run your requests (or the test suite from Chapter 12) after each step. The app should work after every commit.

## Practice Exercises

1. Take any single-file Express app you built in Projects 3 or 4 and, **without changing any behavior**, refactor exactly one resource into the full layered structure (router → controller → service → repo). Verify every endpoint still responds identically before and after (save sample responses first).

2. Find every place in that app where a status code is chosen. List each one and classify it: HTTP-edge concern (fine in controller/middleware) or business rule leaking into HTTP code (should be a typed error). Fix one leak.

3. Write down, in plain English, three business rules from your project 4 API (e.g., "titles must be unique"). For each, state which file enforces it now and which layer *should* enforce it under the rules in this chapter.

4. Deliberately create a dependency-direction violation: make a repository import a service. Observe what happens (it may even work!). Then explain in a comment why it's still wrong — what future change does it sabotage?

5. Split your app into `app.js` (builds and exports the app) and `server.js` (imports and listens). Confirm `node src/server.js` still works, then write a three-line script that imports `app` from `app.js` without the server starting — proof the split succeeded.

6. Sketch (on paper or in a markdown file) the file tree you would use for an API with three resources — users, projects, tasks — where tasks belong to projects and projects belong to users. Decide: layer-first or feature-first folders? Write two sentences defending the choice for *this* size of app.
