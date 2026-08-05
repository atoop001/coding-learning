# Chapter 6: Middleware

## Overview

Middleware is the single most important idea in Express — the framework is honestly little more than a router plus a middleware pipeline. Everything you'll bolt onto your APIs from here on (body parsing, logging, authentication, CORS, rate limiting, error handling) is middleware. This chapter builds the mental model properly: requests flow through an ordered pipeline of functions, each of which can inspect, modify, respond, or pass the request along by calling `next()`. Once this clicks, Express stops being a bag of tricks and becomes one simple rule applied everywhere. You'll use the built-in middleware, write your own, learn why registration order can silently break an app, and adopt your first third-party middleware (morgan and cors) — including how to judge whether a package deserves a place in your dependency tree.

## Definitions & Explanations

**Middleware** — a function with the signature `(req, res, next)` that Express runs as part of handling a request. It can: (a) run any code, (b) read or modify `req` and `res`, (c) end the request by sending a response, or (d) call `next()` to hand off to the next function in the pipeline. Every middleware must do (c) or (d) — doing neither hangs the request; doing both causes errors.

**The pipeline (middleware stack)** — the ordered list of every middleware and route you registered, in registration order. A request enters at the top and flows down until something sends a response. A route handler is just middleware that usually doesn't call `next()`.

**`next`** — the third argument Express passes to middleware: a function that means "I'm done, continue to the next matching middleware/route." Calling `next(err)` *with an argument* instead means "an error occurred, skip ahead to error-handling middleware" (Chapter 8 builds on this).

**`app.use(fn)`** — registers middleware for *all* methods and paths. `app.use('/api', fn)` restricts it to paths starting with `/api`. Contrast with `app.get(path, fn)`, which only matches GET requests to that exact pattern.

**Built-in middleware** — ships with Express: **`express.json()`** parses JSON request bodies into `req.body`; **`express.urlencoded({ extended: true })`** parses classic HTML-form bodies; **`express.static('public')`** serves files from a folder (an `index.html`, CSS, images) without you writing routes for them. None of these parse **`multipart/form-data`** — the encoding a browser uses to upload a *file* — which needs its own middleware (below).

**File uploads (recognition level)** — a form or fetch call that includes a file sends `Content-Type: multipart/form-data`, not JSON. `express.json()`/`express.urlencoded()` silently do nothing with it — `req.body` stays empty and the file goes nowhere. Handling it needs dedicated middleware, most commonly **multer**, which parses the multipart body, writes the file somewhere (disk or memory), and populates `req.file`/`req.files` plus `req.body` for the other form fields. You don't need to master this now — just recognize the shape when you see it.

**Application-level vs router-level middleware** — the same concept, different attachment point. `app.use(...)` affects the whole app; `router.use(...)` affects only requests that reach that router. Use router-level middleware for concerns that belong to one resource (e.g., auth on `/api/admin` routes only).

**Third-party middleware** — npm packages that export middleware. **morgan** logs each request (method, path, status, response time). **cors** sets the CORS headers browsers require before allowing a frontend on one origin to call your API on another (the full CORS story is in Chapter 11 — here you just need it working for local dev).

**Cross-cutting concern** — a need that applies to many/all routes (logging, auth, parsing). Middleware exists precisely so cross-cutting concerns are written once, not copy-pasted into every handler.

## Code Examples

### The mental model, made visible

```js
import express from 'express';

const app = express();

// Middleware #1 — runs for every request.
app.use((req, res, next) => {
  console.log('A: before');
  next(); // hand off downward
  console.log('A: after'); // runs AFTER everything below finishes (like layers of an onion)
});

// Middleware #2
app.use((req, res, next) => {
  console.log('B');
  next();
});

// A route is just middleware that ends the request instead of calling next().
app.get('/', (req, res) => {
  console.log('C: route handler');
  res.json({ ok: true });
});

app.listen(3000);
// One GET / prints: A: before, B, C: route handler, A: after
```

Internalize that output order. Requests go *down* the stack; anything after your `next()` call runs on the way back *up*.

### Writing useful custom middleware

```js
import express from 'express';
import crypto from 'node:crypto';

const app = express();

// 1) Request ID — tag every request so log lines can be correlated.
app.use((req, res, next) => {
  req.id = crypto.randomUUID(); // attaching data to req is THE way
  res.set('X-Request-Id', req.id); // middleware passes info to later handlers
  next();
});

// 2) Timing logger — measures the whole downstream pipeline.
app.use((req, res, next) => {
  const start = performance.now();
  // 'finish' fires when the response has been sent, no matter which route sent it.
  res.on('finish', () => {
    const ms = (performance.now() - start).toFixed(1);
    console.log(`[${req.id}] ${req.method} ${req.originalUrl} -> ${res.statusCode} (${ms}ms)`);
  });
  next();
});

// 3) A gatekeeper: auth stub. Real authentication comes in Chapter 10;
//    the SHAPE is what matters — check something, then either reject or next().
function requireApiKey(req, res, next) {
  const key = req.get('X-Api-Key'); // req.get reads a request header
  if (key !== process.env.API_KEY) {
    return res.status(401).json({ error: 'Missing or invalid API key' });
    // No next() — the pipeline stops here for unauthorized requests.
  }
  next();
}

app.get('/api/public', (req, res) => res.json({ anyone: 'can see this' }));

// Per-route middleware: pass it before the handler. Only this route is guarded.
app.get('/api/secret', requireApiKey, (req, res) => {
  res.json({ secret: 'only with a key' });
});

app.listen(3000, () => console.log('http://localhost:3000'));
```

Run with the key set for this session (PowerShell — bash would use `API_KEY=... node server.js` inline, which PowerShell does not support):

```powershell
$env:API_KEY = 'example-not-a-real-secret'
node --watch server.js

# In another tab:
Invoke-RestMethod http://localhost:3000/api/secret -Headers @{ 'X-Api-Key' = 'example-not-a-real-secret' }
```

### Order matters: broken → fixed

```js
// ❌ BROKEN ordering
const app = express();

app.post('/api/notes', (req, res) => {
  // req.body is undefined here — the JSON parser is registered LATER,
  // so it sits BELOW this route in the pipeline and never runs first.
  res.status(201).json({ title: req.body.title }); // TypeError!
});

app.use(express.json());          // too late for the route above
app.use((req, res) => res.status(404).json({ error: 'Not found' }));
```

```js
// ✅ FIXED ordering — a reliable template for the top of every app
const app = express();

app.use(morgan('dev'));           // 1. logging first, so even failures get logged
app.use(cors());                  // 2. CORS headers before any route can answer
app.use(express.json());          // 3. body parsing before routes need req.body
app.use(express.static('public'));// 4. static files (returns early on a hit)

app.post('/api/notes', (req, res) => {         // 5. routes
  res.status(201).json({ title: req.body.title }); // req.body exists now
});

app.use((req, res) => res.status(404).json({ error: 'Not found' })); // 6. 404 LAST
// (7. error-handling middleware goes after even this — Chapter 8.)
```

The 404 handler placement is the same rule in reverse: it matches everything, so if you register it early, it swallows requests meant for real routes below it.

### Third-party middleware: morgan and cors

```powershell
npm install morgan cors
```

```js
import express from 'express';
import morgan from 'morgan';
import cors from 'cors';

const app = express();

app.use(morgan('dev'));  // logs: "GET /api/books 200 3.214 ms - 87"
app.use(cors());         // dev-friendly: allows ALL origins. Fine locally;
                         // in production you restrict it (Chapter 11).
app.use(express.json());

app.get('/api/books', (req, res) => res.json([{ id: 1, title: 'Dune' }]));

app.listen(3000);
```

Before adding *any* third-party middleware, apply a quick sniff test: Is it actively maintained? Weekly downloads in the millions or a well-known maintainer? Does it do one thing you actually need — or could three lines of your own middleware do it? (morgan and cors pass; plenty of abandoned middleware on npm does not.) Every dependency is code you now ship without having read.

### File uploads, the shape of it (recognition level)

```js
import multer from 'multer';
const upload = multer({ dest: 'uploads/' }); // writes files to disk; a real project validates type/size (Project stretch goal)

app.post('/api/avatar', upload.single('avatar'), (req, res) => {
  res.json({ filename: req.file.filename }); // req.file exists only because multer ran first
});
```

That's the whole shape: `multer(...)` builds middleware, `upload.single('fieldName')` guards one route, and `req.file` shows up only downstream of it — same pipeline model as everything else in this chapter.

### Router-level middleware

```js
// routes/admin.js
import { Router } from 'express';

const router = Router();

// Applies to every route in THIS router only — the rest of the app is untouched.
router.use(requireApiKey);

router.get('/stats', (req, res) => res.json({ users: 42 }));
router.delete('/cache', (req, res) => res.sendStatus(204));

export default router;

// server.js:  app.use('/api/admin', adminRouter);
```

## Common Pitfalls

1. **Never calling `next()` and never responding.** The request hangs silently until the client gives up. If a route "does nothing" when hit, check that every branch of your middleware either responds or calls `next()`.
2. **Calling `next()` after already sending a response.** Execution continues into later routes, which may respond again → `ERR_HTTP_HEADERS_SENT`. If you responded, `return`; if you didn't, `next()` — never both for the same request path.
3. **Registering body parsing or CORS after the routes that need them.** Symptom for JSON: `req.body` is `undefined` on some routes and fine on others. The pipeline runs strictly in registration order — put app-wide middleware above all routes.
4. **Putting the 404 catch-all anywhere but last.** `app.use((req,res) => 404...)` matches every path, so anything registered after it is unreachable. If a route "doesn't exist" even though you wrote it, look for an early catch-all.
5. **Forgetting the parentheses: `app.use(express.json)` instead of `express.json()`.** `express.json` is a *factory* that returns middleware; passing the factory itself fails in confusing ways. Same trap with `cors` vs `cors()` and `morgan` vs `morgan('dev')`.
6. **Doing per-route work in app-level middleware.** Example: checking auth in a global middleware, then whitelisting `/login`, `/register`, `/health`, `/docs`... with a growing `if` list. Invert it: leave the app open, and attach `requireAuth` to the routers/routes that need protecting.
7. **Modifying `req` with clashing names.** Attaching `req.user`, `req.id`, etc. is idiomatic, but overwriting Express's own properties (`req.query`, `req.params`) or another middleware's attachment causes spooky bugs. Pick distinctive names and be consistent across the app.

## Practice Exercises

1. Write a middleware that logs `METHOD path -> status (ms)` using `res.on('finish', ...)`, and register it first. Compare its output with `morgan('dev')` side by side, then keep whichever you prefer — but be able to explain what morgan gives you that yours doesn't.
2. Predict-then-run: build an app with three `app.use` middlewares that each `console.log` before *and* after their `next()` call, plus one route. Write down the exact expected console order for one request, then run it and check. Repeat with the second middleware sending a response instead of calling `next()`.
3. Write `requireJson` middleware that rejects any POST/PUT request whose `Content-Type` header isn't `application/json` with a `415 Unsupported Media Type`. Attach it only to routes with bodies. Test the happy and sad paths from PowerShell.
4. Build a tiny in-memory rate limiter middleware: track request counts per IP (`req.ip`) in a `Map`, allow 10 requests per minute, and respond `429 Too Many Requests` beyond that. (A production-grade one arrives in Chapter 11 — the point here is the middleware shape and state.)
5. Take your movies API from Chapter 5 and deliberately sabotage the middleware order three ways (json after routes; 404 first; static after the 404). For each, write down the symptom you observe *before* fixing it — you're building a symptom→cause lookup table for future debugging.
6. Create an `admin` router protected by router-level API-key middleware (key read from an environment variable — never hard-coded), mounted at `/api/admin`, with two routes. Prove the rest of the app still works without a key and that both admin routes reject a missing/wrong key with 401.
