# Chapter 14: Production Readiness

## Overview

Everything so far has run on `localhost`, restarted by hand, watched by you in a terminal. Production is a different world: nobody is watching the console, the process must survive (or die cleanly) without you, other systems need to ask "are you alive?", and your API may also have to serve the React frontend you built in the other track. This chapter covers the gap between "works on my machine" and "deployable": structured logging with pino, graceful shutdown, health-check endpoints, `NODE_ENV`, and serving a built frontend from Express. It ends with a pre-deployment checklist and hands you off to the deployment-devops track, where you'll actually put an app on the internet. None of this is glamorous, and all of it is exactly what separates hobby code from professional code in an interviewer's eyes.

## Definitions & Explanations

- **Production** — the environment where real users hit your app. Contrasted with **development** (your machine) and often **staging** (a production-like rehearsal environment). Code paths can legitimately differ between them — but only in controlled, explicit ways.

- **Structured logging** — writing logs as machine-parseable JSON (one object per line) instead of free-form text. `{"level":30,"time":1721800000000,"msg":"user created","userId":42}` can be filtered, counted, and alerted on by log tools; `User 42 created!!` cannot. In production, logs are *data*, not prose.

- **pino** — the standard fast JSON logger for Node. It logs structured JSON by default, supports child loggers (loggers that automatically attach context like a request id), and has an Express integration, **pino-http**, that logs every request/response automatically.

- **Log level** — a severity attached to every log line: `fatal`, `error`, `warn`, `info`, `debug`, `trace`. You set a threshold per environment: production typically logs `info` and above; while debugging you drop to `debug`. Levels let you turn detail up and down *without editing code*.

- **pino-pretty** — a development-only formatter that turns pino's JSON into colored, human-readable lines. The critical word is *development-only*: production wants raw JSON.

- **Graceful shutdown** — when the process is told to stop, it: stops accepting new connections, finishes requests already in flight, closes the database, and only then exits. The opposite — the process just vanishing — can drop in-flight requests and corrupt state.

- **SIGTERM / SIGINT** — signals an operating system sends a process. `SIGINT` is Ctrl+C; `SIGTERM` is the polite "please shut down" that deployment platforms (Docker, Kubernetes, most PaaS hosts) send before force-killing you (`SIGKILL`, which cannot be caught) a few seconds later. Your app should treat both as "begin graceful shutdown." Windows note: PowerShell's Ctrl+C delivers `SIGINT` to Node normally, but Windows has no true `SIGTERM` from the console — Node emulates these events well enough for local testing, and your deployed app will almost certainly run on Linux where they behave canonically. Write the handlers; test with Ctrl+C locally.

- **Health check** — an endpoint (conventionally `/health` or `/healthz`) that infrastructure calls to ask "is this instance okay?". A **liveness** check answers "is the process running?" (restart me if not); a **readiness** check answers "can I serve traffic right now?" (e.g., is the database reachable?) — send traffic elsewhere if not. Small apps often use a single endpoint; the distinction matters once an orchestrator manages you.

- **`NODE_ENV`** — the conventional environment variable declaring which environment you're in, usually `development`, `test`, or `production`. Express itself reads it: with `NODE_ENV=production`, Express disables some debugging behavior and stops leaking stack traces in default error pages. Your code reads it to pick log formats, error verbosity, and so on.

- **SPA fallback** — when Express serves a single-page app (your built React frontend), any URL the API doesn't recognize should return `index.html` so client-side routing can take over. Without it, refreshing the browser on `/dashboard` returns a 404.

- **Process manager** — software that keeps your Node process alive and restarts it on crash: `pm2` on a bare VM, or more commonly today the platform itself (Docker restart policies, systemd, Kubernetes, a PaaS). You'll meet these properly in the deployment-devops track; the takeaway now is that **your app should crash loudly on unrecoverable errors and let the manager restart it**, not limp along in a broken state.

**What about work that shouldn't block a request?** (recognition level) Sending a welcome email, resizing an uploaded image, generating a report — none of that should make the client wait, and a slow third-party API in your request handler is a request your event loop can't use for anything else (Chapter 3). The general pattern is *respond now, do the slow thing async*: handle the request, kick off the work, and return a response immediately. How you "kick off the work" scales with need: `setImmediate`/`setTimeout(fn, 0)` is the naive version (fire it after the current response, no durability — it's gone if the process crashes first); a **jobs table** in your database plus an interval worker that polls for pending rows is the honest middle ground (durable, no new infrastructure, good enough for most side projects); and real **queues** like **BullMQ** (backed by Redis) are what production systems reach for once volume or reliability guarantees matter — retries, backoff, concurrency control, and visibility into what's stuck. You don't need to implement any of this now; just recognize the pattern name and know it exists for the day a request handler needs to do something slow.

## Code Examples

### From console.log to pino

```powershell
npm install pino pino-http
npm install --save-dev pino-pretty
```

```js
// src/logger.js — one logger for the whole app
import pino from "pino";

const isProd = process.env.NODE_ENV === "production";

export const logger = pino({
  level: process.env.LOG_LEVEL ?? (isProd ? "info" : "debug"),
  // Pretty output is a DEV convenience only. In production we want raw
  // JSON lines that log collectors can parse — never pino-pretty.
  transport: isProd ? undefined : { target: "pino-pretty" },
});
```

```js
// Using it — note we log OBJECTS with context, then a message.
// Searchable fields beat sentence-mashing:
logger.info({ userId: 42, plan: "free" }, "user registered");
logger.error({ err }, "payment webhook failed");   // pino serializes err properly

// ❌ the habit to break:
console.log("User " + userId + " registered on plan " + plan);
```

```js
// src/app.js — automatic per-request logging
import pinoHttp from "pino-http";
import { logger } from "./logger.js";

app.use(pinoHttp({ logger }));
// Every request now logs method, url, status, and response time.
// Inside any handler, req.log is a child logger tied to this request:
app.get("/api/things", (req, res) => {
  req.log.info("listing things");   // automatically includes the request id
  res.json([]);
});
```

**Tying the request id to your logs.** Chapter 6 had you attach `req.id` to every request with a small middleware. Do that *before* `pinoHttp` and it's no longer just a header — `pino-http` picks it up automatically and gives every request a **child logger** (`req.log`) that has the id baked into every line it writes, so one request's log lines all correlate even when a hundred requests are interleaved in production:

```js
import crypto from "node:crypto";

app.use((req, res, next) => {
  req.id = crypto.randomUUID();          // Chapter 6's request-id middleware
  next();
});
app.use(pinoHttp({ logger, genReqId: (req) => req.id })); // reuse it, don't generate a second id

app.post("/api/bookmarks", (req, res) => {
  req.log.info({ url: req.body.url }, "creating bookmark"); // this line and every
  // other req.log line for THIS request carry the same req.id automatically
  res.status(201).json({ ok: true });
});
```

Without `genReqId`, pino-http generates its own id and you end up with two unrelated ids per request — wire them together instead.

### Graceful shutdown

```js
// src/server.js
import { app } from "./app.js";
import { logger } from "./logger.js";
import { db } from "./db.js";

const server = app.listen(process.env.PORT ?? 3000, () => {
  logger.info(`listening on ${server.address().port}`);
});

function shutdown(signal) {
  logger.info({ signal }, "shutting down");
  // 1. Stop accepting new connections; the callback fires when
  //    in-flight requests have finished.
  server.close(() => {
    db.close();          // 2. then release the database
    logger.info("clean exit");
    process.exit(0);     // 3. then actually exit
  });
  // Safety net: if something hangs (a stuck connection), force-exit.
  // Platforms send SIGKILL after ~10-30s anyway; beat them to it cleanly.
  setTimeout(() => {
    logger.error("forced exit after timeout");
    process.exit(1);
  }, 10_000).unref();    // unref: don't let this timer keep the process alive
}

process.on("SIGTERM", () => shutdown("SIGTERM"));
process.on("SIGINT", () => shutdown("SIGINT"));   // Ctrl+C in PowerShell
```

Test it locally: start the server, make a request, press Ctrl+C, and watch the log order — "shutting down" then "clean exit". On Windows, closing the PowerShell window kills the process without ceremony, so use Ctrl+C when you want to observe the graceful path.

### Health checks

```js
// Liveness: cheap, no dependencies — "the event loop is turning".
app.get("/health", (req, res) => {
  res.json({ status: "ok", uptime: process.uptime() });
});

// Readiness: verifies the app can actually do its job.
app.get("/health/ready", (req, res) => {
  try {
    db.prepare("SELECT 1").get();   // is the database reachable?
    res.json({ status: "ready" });
  } catch {
    // 503 tells the load balancer: alive, but don't send traffic yet.
    res.status(503).json({ status: "not ready" });
  }
});
```

Keep health checks out of auth middleware (infrastructure has no login) and out of request logging if they get noisy — pino-http accepts an `autoLogging.ignore` option for exactly this.

### NODE_ENV in practice

```powershell
# PowerShell — set for the current session:
$env:NODE_ENV = "production"
node src/server.js

# bash equivalent (for reference — inline prefixes don't work in PowerShell):
# NODE_ENV=production node src/server.js
```

```js
// Typical environment-dependent choices:
const isProd = process.env.NODE_ENV === "production";

// Error handler from Chapter 8, environment-aware:
app.use((err, req, res, next) => {
  req.log.error({ err }, "unhandled error");
  res.status(err.status ?? 500).json({
    error: isProd ? "Internal server error" : err.message,
    // Never leak stack traces to production clients:
    ...(isProd ? {} : { stack: err.stack }),
  });
});
```

Cross-platform tip: npm scripts that set env vars the bash way break in PowerShell. Either use the `cross-env` package in scripts (`"start:prod": "cross-env NODE_ENV=production node src/server.js"`), or rely on the deployment platform to set `NODE_ENV=production` — which real hosts do for you.

### Serving a built frontend

Your React track produces a `dist/` folder of static files. One common deployment shape is Express serving both the API and that frontend:

```js
import path from "node:path";
import { fileURLToPath } from "node:url";

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const distDir = path.join(__dirname, "../client/dist");

// Order matters (Chapter 6): API routes first, then static files,
// then the SPA fallback for everything else.
app.use("/api/bookmarks", bookmarksRouter);
app.use(express.static(distDir));
// Express 5 note: bare "*" is no longer a valid path — use a named splat.
app.get("/*splat", (req, res) => {
  res.sendFile(path.join(distDir, "index.html"));
});
```

Now `/api/...` returns JSON, `/assets/index-abc123.js` comes from disk, and `/dashboard` (or any client route, even on refresh) gets `index.html`. The fallback must come *after* every API route, or your API's 404s turn into mysterious HTML responses.

## Common Pitfalls

1. **Shipping `pino-pretty` (or any pretty-printing) to production.** Pretty output is for eyeballs; log pipelines want JSON. Correction: gate the transport on `NODE_ENV` as shown, and keep `pino-pretty` in `devDependencies`.

2. **Logging secrets.** `logger.info({ body: req.body })` on a login route writes passwords into your logs — a real breach category. Correction: log identifiers, never credentials or tokens; pino's `redact` option (`redact: ["req.headers.authorization", "*.password"]`) is cheap insurance.

3. **`process.exit(0)` immediately in the signal handler.** It "works" — by killing in-flight requests mid-response. Correction: `server.close()` first, exit in its callback, with a forced-exit timer as backstop.

4. **Forgetting that `server.close()` waits forever on open keep-alive connections.** Your shutdown can hang on an idle browser connection. Correction: the `.unref()`ed timeout backstop; on Node 18.2+, `server.closeIdleConnections()` right after `server.close()` speeds this up.

5. **A readiness check that's too expensive or too honest.** Pinging the DB with a heavy query every 5 seconds, or failing readiness because a *non-critical* dependency is down, causes platforms to pull healthy instances. Correction: keep checks under ~50ms and fail readiness only for dependencies you genuinely cannot serve without.

6. **Comparing `NODE_ENV` sloppily.** `if (process.env.NODE_ENV == "prod")` fails because the value is unset locally, `"production"` on the host, or capitalized weirdly. Correction: compare against exact conventional values, default sensibly (`?? "development"`), and set it explicitly in every environment.

7. **Putting the SPA fallback before the API routes.** Every `/api/whatever` request suddenly returns `index.html` with a 200, which frontends "handle" by failing to parse HTML as JSON. Correction: API routers first, static second, fallback last — and keep a real 404 handler *inside* the `/api` scope so bad API paths still return JSON errors.

## Practice Exercises

1. Take your Project 4 or 5 API and replace every `console.log`/`console.error` with pino: a shared `logger.js`, pino-http for requests, and at least three log lines that attach structured context objects. Run it once with `NODE_ENV=production` and once without, and confirm the output format differs.

2. Implement graceful shutdown, then prove it works: add a temporary route that takes 5 seconds to respond (`setTimeout` before responding), start a request to it, press Ctrl+C during those 5 seconds, and verify from the logs that the response completed before the process exited.

3. Add `/health` and `/health/ready` endpoints. Then stop your database (or rename the SQLite file while the server runs) and confirm readiness returns 503 while liveness still returns 200. Restore and confirm recovery without a restart.

4. Audit your error handler and your logs for leaks: with `NODE_ENV=production`, trigger a thrown error and confirm the client response contains no stack trace and no internal message, while the *log* contains the full error. List anything else your API currently reveals that production shouldn't.

5. Build your React track project (`npm run build`), copy or point Express at its `dist/`, and wire up static serving plus the SPA fallback. Verify all three behaviors: an API route returns JSON, a client-side route survives a hard refresh, and an unknown `/api/...` path returns a JSON 404 — not HTML.

6. Write your own pre-deployment checklist in the project README — at minimum: secrets out of code and in env vars, `NODE_ENV=production` set, logs structured, shutdown graceful, health endpoint live, error responses sanitized, tests passing. This checklist is your entry ticket to the deployment-devops track, where you'll put this app on a real server.
