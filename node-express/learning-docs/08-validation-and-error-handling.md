# Chapter 8: Validation & Error Handling

## Overview

Every request that reaches your API was assembled by something you don't control. Maybe it's your own frontend behaving well; maybe it's a curious user with `curl`, a broken script, or an attacker probing for weaknesses. The server cannot tell the difference, so it must treat *every* input as untrusted — this is the single most important mindset shift in backend work. This chapter covers the two halves of handling bad input and bad luck: **validation** (rejecting malformed or malicious data at the door, both by hand and with the `zod` library) and **error handling** (making sure that when things go wrong — invalid input, missing resources, thrown exceptions, rejected promises — your API responds with a *consistent, safe, correct* error instead of crashing, hanging, or leaking internals). You'll build the centralized error-handling middleware pattern that nearly every production Express app uses, and learn the async error gotcha that has bitten every Express developer at least once.

## Definitions & Explanations

**Validation** is checking that incoming data has the right shape, types, and values *before* your business logic touches it. It answers: is `title` present? Is it a string? Is it under 200 characters? Is `email` actually an email? Validation belongs at the **boundary** — the moment data enters your system — so everything past that point can trust the data's shape.

**"Never trust the client"** means exactly that: client-side validation (HTML `required` attributes, JavaScript form checks) is a *convenience for honest users*, not a defense. Anyone can bypass your frontend entirely and send raw HTTP to your API. Server-side validation is the only validation that counts for correctness and security; client-side validation is just better UX layered on top.

A **schema** is a declarative description of the shape data must have — "an object with a `url` string that parses as a URL, an optional `title` string, and a `tags` array of strings." **Schema validation libraries** like **zod** let you write the schema once and get validation, precise error messages, and (in TypeScript) static types from the same declaration. The alternative — **manual validation** — is a pile of `if` statements: workable for two fields, unmaintainable for twenty.

**Parsing vs. validating:** zod's `schema.parse(data)` returns the validated (and cleaned) data or *throws*; `schema.safeParse(data)` never throws — it returns `{ success: true, data }` or `{ success: false, error }`. In request handlers, prefer `safeParse` so *you* decide what the failure response looks like.

An **error-handling middleware** in Express is any middleware with exactly **four parameters**: `(err, req, res, next)`. The arity is how Express recognizes it — remove one parameter and it silently becomes ordinary middleware that never sees errors. When any handler calls `next(err)` or throws, Express skips all remaining normal middleware and jumps to the error handlers. Registering **one centralized error handler last** gives every error in your app a single exit point: one place to log, one place to shape the response, one place to hide stack traces.

**Async error propagation** is where Express versions differ, and it matters:
- **Express 5** (current; what you get with `npm install express` on Node 22): if an `async` handler's promise rejects — including any `throw` inside it — Express automatically forwards the error to your error-handling middleware. `async` handlers "just work."
- **Express 4** (still everywhere in tutorials, Stack Overflow, and older codebases): a rejected promise in an `async` handler is *not* caught. The error vanishes, the client's request hangs until timeout, and you may see an unhandled-rejection warning in the console. Express 4 code needs `try/catch` + `next(err)` in every async handler, or a wrapper utility (often called `asyncHandler` or the `express-async-handler` package).

You should know both because you will read both. Check your installed major version with `npm ls express` before trusting either behavior.

A **custom error class** extends `Error` with the extra fields your error handler needs — most importantly an HTTP `status`. Throwing `new HttpError(404, 'Bookmark not found')` anywhere in your app and letting the central handler translate it into a response keeps status-code decisions out of your business logic's plumbing.

An **operational error** is an expected failure of normal operation: invalid input, missing resource, duplicate email. These map to 4xx responses and are *not bugs*. A **programmer error** is a bug: `undefined is not a function`, a typo'd property. These map to a generic 500 — and critically, the client must receive *no details*, because stack traces and internal messages are reconnaissance gifts to attackers (they reveal file paths, library choices, and query structure).

A **consistent error response shape** means every error your API ever returns has the same JSON structure, so clients can write one error-handling code path. Pick a shape (this track uses `{ "error": { "message": ..., "details": [...] } }`) and never deviate.

**400 vs. 422:** 400 Bad Request = the request was malformed at the protocol/syntax level (unparseable JSON, wrong content type). 422 Unprocessable Content = syntax fine, semantics wrong (parsed cleanly, but `email` fails validation). Plenty of respected APIs use 400 for both — either convention is defensible; *mixing them randomly* is not. This track uses 422 for validation failures.

## Code Examples

**Manual validation first — so you know what the library saves you from.** Validating one endpoint by hand:

```js
app.post('/api/v1/bookmarks', (req, res) => {
  const { url, title, tags } = req.body ?? {};
  const details = [];

  if (typeof url !== 'string' || url.length === 0) {
    details.push({ field: 'url', message: 'url is required and must be a string' });
  } else {
    // URL.canParse (Node 18.17+) beats regexes for URL checking
    if (!URL.canParse(url)) details.push({ field: 'url', message: 'url must be a valid URL' });
  }
  if (title !== undefined && (typeof title !== 'string' || title.length > 200)) {
    details.push({ field: 'title', message: 'title must be a string of at most 200 chars' });
  }
  if (tags !== undefined && (!Array.isArray(tags) || !tags.every((t) => typeof t === 'string'))) {
    details.push({ field: 'tags', message: 'tags must be an array of strings' });
  }

  if (details.length > 0) {
    return res.status(422).json({ error: { message: 'Validation failed', details } });
  }
  // ...create the bookmark
});
```

Three fields, ~20 lines, and it still misses things (extra unknown fields pass straight through). Now the same contract with **zod**:

```powershell
npm install zod
```

```js
import { z } from 'zod';

const createBookmarkSchema = z.object({
  url: z.string().url(),                       // required, must be a valid URL
  title: z.string().max(200).optional(),
  tags: z.array(z.string()).max(20).default([]),
}).strict(); // reject unknown fields instead of silently ignoring them

app.post('/api/v1/bookmarks', (req, res) => {
  const result = createBookmarkSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        message: 'Validation failed',
        details: result.error.issues.map((i) => ({
          field: i.path.join('.'),
          message: i.message,
        })),
      },
    });
  }
  const input = result.data; // cleaned: defaults applied, unknown fields gone
  // ...create the bookmark from input, never from req.body
});
```

The crucial habit: after validation, use `result.data`, **never `req.body`** — `result.data` is the cleaned version with defaults applied and unknown fields stripped.

**Params and query need validation too** — they're just as client-controlled as the body, and they always arrive as strings:

```js
const idSchema = z.coerce.number().int().positive();       // "42" -> 42, "abc" -> error
const listQuerySchema = z.object({
  tag: z.string().optional(),
  page: z.coerce.number().int().min(1).default(1),
  pageSize: z.coerce.number().int().min(1).max(100).default(20), // server-enforced cap
});
```

`z.coerce` handles the everything-is-a-string reality of URLs. The `max(100)` on `pageSize` is a small security decision: without it, `?pageSize=10000000` is a free denial-of-service lever.

**The centralized error handler.** First, a custom error class:

```js
// errors.js
export class HttpError extends Error {
  constructor(status, message, details = undefined) {
    super(message);
    this.status = status;
    this.details = details;
  }
}
```

Then the handler, registered **after every route** in `app.js`:

```js
import { HttpError } from './errors.js';

// 404 catch-all: only reached if no route matched
app.use((req, res, next) => {
  next(new HttpError(404, `No route for ${req.method} ${req.path}`));
});

// THE error handler: four parameters, or Express won't treat it as one.
app.use((err, req, res, next) => {
  if (err instanceof HttpError) {
    // Operational error we threw on purpose: safe to show its message
    return res.status(err.status).json({
      error: { message: err.message, ...(err.details && { details: err.details }) },
    });
  }
  // Programmer error / unknown: log everything, reveal nothing
  console.error(err); // in Chapter 14 this becomes a real logger
  res.status(500).json({ error: { message: 'Internal server error' } });
});
```

Now any handler anywhere can simply throw, and the response comes out correct and consistent:

```js
app.get('/api/v1/bookmarks/:id', async (req, res) => {
  const id = idSchema.parse(req.params.id); // ZodError -> 500 for now; exercise 4 fixes that
  const bookmark = await findBookmark(id);
  if (!bookmark) throw new HttpError(404, 'Bookmark not found');
  res.json(bookmark);
});
```

**The Express 4 vs 5 async difference, concretely.** The handler above throws inside an `async` function:

```js
// Express 5: rejected promise is forwarded to the error handler automatically. Done.

// Express 4: the rejection is LOST. The client hangs. You needed:
app.get('/api/v1/bookmarks/:id', async (req, res, next) => {
  try {
    const bookmark = await findBookmark(req.params.id);
    if (!bookmark) throw new HttpError(404, 'Bookmark not found');
    res.json(bookmark);
  } catch (err) {
    next(err); // hand it to the error middleware manually
  }
});

// Express 4, the less repetitive fix — a wrapper:
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);
app.get('/api/v1/bookmarks/:id', asyncHandler(async (req, res) => { /* ... */ }));
```

If you're on Express 5 (you are, on a fresh install), you don't need the wrapper — but you *will* meet it in older codebases, so recognize it.

**Leaky vs. safe 500s** — why the "reveal nothing" rule exists:

```js
// ❌ Leaky: hands the attacker your file paths, driver, and query structure
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message, stack: err.stack });
});

// ✅ Safe: full detail to your logs, generic message to the client
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: { message: 'Internal server error' } });
});
```

## Common Pitfalls

1. **Relying on frontend validation.** "The form already checks that" protects you against nobody who matters — `curl -X POST` skips your form entirely. Correction: the server validates everything, every time; the frontend's checks are UX only.

2. **Writing the error handler with three parameters.** `(err, req, res)` or `(req, res, next)` — either way, Express no longer recognizes it as error middleware, and errors fall through to the default HTML error page. Correction: exactly four parameters, `(err, req, res, next)`, even though you rarely use `next` inside it.

3. **Registering the error handler before the routes.** Middleware order is registration order (Chapter 6); an error handler mounted first never sees errors from routes mounted after it in the way you expect, and your 404 catch-all will shadow real routes if it's early. Correction: routes first, then the 404 catch-all, then the error handler — always last.

4. **Forgetting `return` before an early error response.** `if (!valid) res.status(422).json(...)` without `return` keeps executing the handler, leading to the classic crash `Cannot set headers after they are sent to the client`. Correction: `return res.status(422).json(...)` — make the early exit actually exit.

5. **Validating the body but not params and query.** `req.params.id` and `req.query.page` are attacker-controlled strings like everything else; `parseInt(req.params.id)` yielding `NaN` and getting handed to your database is a whole genre of bug. Correction: schema-validate all three inputs; use `z.coerce` for the string-to-number reality of URLs.

6. **Trusting `req.body` after validation.** You called `safeParse`, it succeeded — then you wrote `createBookmark(req.body)`. The raw body still contains unknown fields and lacks defaults. Correction: always use the *output* of validation (`result.data`); treat `req.body` as radioactive after the parse.

7. **One-off error shapes.** `{ "error": "..." }` here, `{ "message": "..." }` there, a bare string from a third place — every inconsistency is a special case some client must handle. Correction: define one error envelope, produce it *only* in the centralized handler, and make handlers throw rather than build their own error responses.

## Practice Exercises

1. **Manual-to-zod migration.** Write manual validation (plain `if` statements) for a `POST /api/v1/tasks` body: `title` (required string, 1–120 chars), `dueDate` (optional ISO date string), `priority` (optional, one of `"low" | "medium" | "high"`, default `"medium"`). Then rewrite it as a zod schema with `.strict()`. Compare line counts and list two bad inputs your manual version accepted that zod rejects.

2. **Build the full error pipeline.** In a small Express app, implement: the `HttpError` class, a 404 catch-all, and the centralized error handler with the `{ error: { message, details? } }` shape. Verify with `curl` (or `Invoke-RestMethod`) that (a) an unknown route returns 404 in the standard shape, (b) a route that throws `new Error('boom')` returns a bodyless-of-detail 500, and (c) the boom's stack trace appears in your console but *not* in the response.

3. **Prove the async behavior of your Express version.** Write a route that does `throw new Error('async boom')` inside an `async` handler. Request it. Does your centralized handler catch it? Check `npm ls express`, state your major version, and write two sentences explaining what would happen on the *other* major version and why.

4. **Translate ZodError centrally.** In the Code Examples, `idSchema.parse()` throwing a `ZodError` would fall into the generic 500 branch. Extend your error handler to detect `err instanceof ZodError` and translate it into a 422 with the standard details array — so handlers can use bare `.parse()` and stay clean. Then simplify one route to rely on it.

5. **Validate all three inputs.** For `GET /api/v1/tasks/:id/comments?page=&pageSize=`, write zod schemas for params and query (with coercion, defaults, and a `pageSize` cap), wire them into the route, and test the edge cases: `id=abc`, `page=0`, `pageSize=9999`, and no query string at all. Every failure must return your standard error shape.

6. **Error-shape audit.** Take the in-memory API you'll have from Project 3 (or any Express app you've written so far) and list every distinct error response shape it can produce, including ones produced by Express itself (bad JSON body → what does `express.json()` do?). Refactor until the list has exactly one entry.
