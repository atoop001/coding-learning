# Chapter 11: Monitoring, Logging & Debugging Production

## Overview

On your laptop, debugging is luxurious: breakpoints, a REPL, `console.log` and re-run, the whole app frozen mid-thought while you poke at variables. Production revokes all of it. The process is on a machine you can't (or shouldn't) attach a debugger to, the bug happened forty minutes ago to a user you'll never meet, and the only witness is whatever your app wrote down at the time. That's the discipline shift this chapter teaches: **in production, you debug the past, using evidence your past self chose to record.** The tools are logs (done properly: leveled, structured, greppable), health checks (letting machines ask "are you okay?"), uptime monitoring (finding out you're down before your users tweet it), and error tracking (Sentry-style aggregation that turns ten thousand crashes into one ranked list). None of this is optional at real jobs — "check the logs" is the first sentence of every production investigation, and juniors who can navigate logs calmly stand out immediately.

## Definitions & Explanations

**Observability** — the umbrella term: how well you can infer what's happening inside a system from what it emits. The classic three pillars are **logs** (discrete events: "this happened"), **metrics** (numbers over time: requests/sec, memory), and **traces** (one request's journey across services). At your scale, logs plus a health check plus error tracking cover 95% of needs; metrics and traces are recognize-the-word material for now.

**Log** — a timestamped record of an event, written to **stdout/stderr** in containerized and PaaS deployments — *not* to files. The platform captures the stream, timestamps it, stores and rotates it, and serves it back via dashboard and CLI (`docker logs`, `render logs`, `fly logs` are all the same idea). This is the 12-factor logging principle: the app's job is to *emit*, the platform's job is to *collect*. Apps that write their own logfiles inside containers lose them on every restart (ephemeral filesystems — Chapter 4) and fight the platform instead of using it.

**Log levels** — the severity ladder, so you can dial verbosity without redeploying: **`debug`** (chatty internals, off in prod by default), **`info`** (normal notable events: startup, config loaded, request served), **`warn`** (something's off but handled: retry succeeded, deprecated call, slow query), **`error`** (an operation failed and someone should eventually look: unhandled exception, upstream 500). The level threshold is *config* (`LOG_LEVEL=info` — Chapter 3's contract again): production runs at `info`, and when you're actively hunting a bug you temporarily lower it to `debug`.

**Structured logging** — writing logs as machine-parseable key-value records (usually JSON lines) instead of freeform prose: `{"level":"error","msg":"payment failed","order_id":"842","err":"card_declined"}` rather than `"Payment for order 842 failed!!"`. Why it wins: you can *filter* ("all errors for order 842"), *count* ("card_declined per hour"), and *correlate*. Libraries: pino (Node), structlog (Python) — thin wrappers, big payoff. A related pro habit: attach a **request ID** to every log line within one request, so a user's bug report ("it failed around 3pm") becomes a thread you can pull.

**Health check (endpoint)** — a route, conventionally `/health` or `/healthz`, that answers "is this instance able to serve?" — HTTP 200 when yes. Two grades: **liveness** ("the process responds at all" — return 200, done) and **readiness** ("and my dependencies work" — e.g., ping the DB before saying yes). Machines are the audience: the platform polls it to decide whether to restart your app or route traffic to a new deploy (this is how deploy-on-green *verifies* the green), and your uptime monitor polls it from outside. Design rule: fast, unauthenticated, cheap, and honest — a health check that returns 200 while the DB is unreachable is worse than none.

**Uptime monitoring** — an external service (UptimeRobot and similar have free tiers; platforms often build it in) that requests your URL every 1–5 minutes and alerts you (email/webhook) on failure. External is the operative word: it exercises the whole public path — DNS, TLS, platform, app — the same way a user does. This is the answer to the quietly embarrassing question "how long was I down before I noticed?"

**Error tracking** — a service (Sentry is the archetype; GlitchTip is an open-source cousin) whose SDK catches unhandled exceptions in your app and ships them — with stack trace, request context, user agent, release version — to a dashboard that **groups** identical errors, **counts** them, marks **regressions** ("this came back in v1.4.2"), and alerts on new ones. The conceptual leap over logs: logs are a chronological river; error tracking is a *ranked inventory of distinct problems* with frequencies. "TypeError in checkout, 412 occurrences, started with yesterday's deploy, here's the exact line" is a workday-changing sentence. Free tiers cover hobby projects comfortably.

**Request ID / correlation ID (mechanics)** — a random ID generated at the top of each request (or accepted from an incoming header like `x-request-id`, so IDs survive across services) and attached to every log line that request produces. The practical payoff: one filter turns an interleaved log river — hundreds of concurrent requests sharing one stream — back into per-request stories. Most web frameworks have middleware for this; it's ten lines by hand.

**Uptime percentages, translated** — "99.9% uptime" sounds abstract until you do the arithmetic: 99% = ~7 hours down per month; 99.9% = ~43 minutes; 99.99% = ~4 minutes. Knowing the table makes platform SLAs and job-posting reliability talk concrete — and explains why each additional nine costs so much engineering.

**Alerting discipline** — the meta-skill: alerts must be *actionable and rare*. Alert on "site down" and "new error type," not on every warning — because the failure mode of noisy alerting is a human who's learned to ignore alerts (**alert fatigue**), which is worse than no alerting at all.

**Postmortem / incident notes** — the professional habit after anything breaks: a short blameless write-up — what happened, timeline, root cause, what detection/prevention changes. Even solo, five minutes of this converts each incident into permanent skill (and interview stories).

**Debugging without a debugger — the method.** What replaces breakpoints is a loop:
1. **Observe** precisely (error tracker entry, log excerpt, monitor alert — *what*, *when*, *how often*, *who's affected*).
2. **Correlate with change** — "what deployed/changed right before?" answers a shocking majority of production bugs (deploy history + `git log` is your first tool, not your last).
3. **Hypothesize and check the evidence** you already have (logs around the timestamp, request IDs, platform status pages for *their* outages).
4. **Reproduce locally** if possible — same inputs, prod-like config (Docker shines here).
5. **If evidence is insufficient, add logging and redeploy** — this *is* the production equivalent of adding a breakpoint, and it's a normal, non-shameful move teams make daily.
6. Fix, deploy, **verify against the original signal** (error count drops to zero, monitor goes green), write the note.

## Code Examples

Leveled, structured logging in Node with pino — note how little ceremony:

```js
// logger.js
const pino = require("pino");
module.exports = pino({ level: process.env.LOG_LEVEL || "info" });
```

```js
// usage — key-value context, not string soup:
const log = require("./logger");
log.info({ port }, "server started");
log.warn({ upstream: "payments", attempt: 2 }, "retrying upstream call");
log.error({ err, orderId }, "order creation failed");
log.debug({ query, ms: elapsed }, "db query");   // invisible in prod until LOG_LEVEL=debug
// Output is JSON lines on stdout — exactly what platforms want to collect.
```

Python's equivalent shape with the standard library (structlog if you want richer):

```python
import logging, os
logging.basicConfig(level=os.environ.get("LOG_LEVEL", "INFO"))
log = logging.getLogger("app")
log.info("server started port=%s", port)
log.error("order creation failed order_id=%s err=%s", order_id, exc)
```

Request-ID middleware — the ten lines that make production logs navigable:

```js
const crypto = require("crypto");

app.use((req, res, next) => {
  // honor an incoming ID (caller may be another service); otherwise mint one
  req.id = req.headers["x-request-id"] || crypto.randomUUID();
  res.setHeader("x-request-id", req.id);          // echo it so users/support can quote it
  req.log = log.child({ reqId: req.id });         // pino: a logger that stamps every line
  next();
});

// Everywhere else in the app, use the stamped logger:
app.post("/orders", async (req, res) => {
  req.log.info({ userId: req.user?.id }, "order started");
  // ...
  req.log.error({ err }, "payment declined");
  // Both lines carry the same reqId. One filter reconstructs this request's story.
});
```

A health check with both grades:

```js
// Liveness — "the process is up." Cheap, always:
app.get("/health", (req, res) => res.status(200).json({ status: "ok" }));

// Readiness — "and I can actually do my job":
app.get("/health/ready", async (req, res) => {
  try {
    await pool.query("SELECT 1");                 // dependency probe: the DB answers
    res.status(200).json({ status: "ok", db: "up" });
  } catch (e) {
    res.status(503).json({ status: "degraded", db: "down" });   // honest 503, and
    // log the cause — but never leak internals (connection strings!) in the response body
  }
});
```

Reading logs like a practitioner — the grammar is identical across platforms:

```powershell
# Local container:
docker logs --since 15m myapp                    # recent window
docker logs -f myapp                             # follow live
docker logs myapp 2>&1 | Select-String '"level":"error"'    # errors only (PowerShell grep)

# PaaS (Render CLI / Fly.io respectively — dashboards show the same stream):
render logs -r <service-id> --tail
fly logs

# The universal investigation moves, whatever the spelling:
#   1. narrow to the TIME WINDOW of the incident
#   2. filter by level=error, then widen to warn
#   3. find the FIRST error, not the loudest — cascades bury root causes under symptoms
#   4. pull the request ID from the first error and re-filter on it for the full story
```

What good logs look like when something breaks — a worked contrast, because "log with context" is abstract until you see the difference:

```text
BAD (what beginners' logs show at 3 a.m.):
  Error: something went wrong
  Error: something went wrong
  Error: something went wrong
Three failures. Of what? For whom? Same cause or three causes? No idea.

GOOD (structured, leveled, contextual):
  {"level":"error","time":"2026-07-24T14:32:07Z","reqId":"9f2c","userId":184,
   "route":"POST /orders","err":"connect ETIMEDOUT db:5432","msg":"order failed"}
  {"level":"warn","time":"2026-07-24T14:32:08Z","msg":"db pool exhausted, waiting"}
  {"level":"error","time":"2026-07-24T14:32:09Z","reqId":"a01d","userId":92,
   "route":"POST /orders","err":"connect ETIMEDOUT db:5432","msg":"order failed"}
Same three failures — but now: it's the DATABASE (not the code path), it started
at 14:32, it affects all users on /orders, and the pool warning between the
errors points at the mechanism. The investigation is half done before you've
opened an editor. The difference was authored months earlier, by you, in calm.
```

Wiring an uptime monitor and error tracking — checklist form, ten minutes total:

```text
Uptime: uptimerobot.com (free) → New Monitor → HTTPS → your /health URL →
  interval 5 min → alert contact = your email. Done. You now find out before users do.

Error tracking: sentry.io (free hobby tier) → create project (Node/Python) →
  SDK gives you two lines to add:
      require("@sentry/node").init({ dsn: process.env.SENTRY_DSN });
  DSN goes in env vars (it's config; treat it as mildly sensitive) →
  deploy → throw a test error → watch it appear, grouped, with a stack trace.
  Then set ONE alert rule: "notify me on new issue types." Rare and actionable.
```

The correlate-with-change move, made concrete:

```powershell
# Error tracker says: "TypeError: cannot read 'email' — first seen 14:32 today."
gh run list --workflow=deploy.yml --limit 5      # what deployed around 14:32?
git log --oneline -5                              # what was IN that deploy?
# Nine times out of ten you are now staring at the culprit commit.
```

## Common Pitfalls

1. **Logging nothing but errors — or logging everything at `info`.** Silent-until-catastrophe apps give you no trail to follow backward from a failure; firehose apps bury the one line that matters. Correction: `info` for lifecycle landmarks (started, connected, listening), `warn` for handled weirdness, `error` for failures with context (IDs! the error object!), `debug` for the chatty stuff behind a config gate. Test: could you reconstruct "what was the app doing at 14:30?" from `info` alone?

2. **Logging secrets and personal data.** The request logger that prints headers (Authorization tokens), the config dump at startup (`DATABASE_URL`...), the error that echoes a password field. Logs are stored, copied to third-party services, and read by many eyes — a leak into logs is a real leak (Chapter 3 playbook applies, rotation included). Correction: log *identifiers* (user ID, order ID), never credentials or raw request bodies; most logging libraries support redaction lists — configure one.

3. **Writing logs to files inside containers.** `app.log` in the container dies with the container, fills the writable layer meanwhile, and never reaches the platform's log system. Correction: stdout/stderr, full stop. The platform owns storage and rotation; your app owns emission.

4. **The lying health check.** Returns 200 unconditionally — so the platform happily routes traffic to an instance whose DB connection died, and your uptime monitor glows green through a total outage. Correction: readiness checks probe what serving actually requires; and the inverse discipline too — don't make `/health` so heavy (five dependency calls, invoked every 10s by three systems) that the check itself becomes load. Cheap and honest.

5. **No external monitor because "the platform would tell me."** Platform dashboards show *their* view from *inside*; DNS misconfiguration, expired domains, TLS problems, and platform-edge outages are all invisible from in there. Correction: one free external monitor on the public URL. The whole path, tested the way users travel it, every five minutes.

6. **Treating recurring errors as ambient noise.** The tracker shows the same exception 300 times a day; it's "known," so it's ignored — until it's page-one. Meanwhile real new errors drown in it. Correction: every distinct error gets a decision — fix it, or explicitly resolve/mute it with a reason. A tracker where everything is "unresolved, seen 4,000 times" has stopped being a tool.

7. **Debugging production by vibes and redeploys.** Change something plausible, redeploy, "seems fine?", repeat — each cycle contaminating the evidence and possibly adding regressions. Correction: the six-step loop, in order — observe, correlate with change, hypothesize *from evidence*, reproduce, instrument if blind, verify against the original signal. Slower per step, dramatically faster to actual resolution.

## Practice Exercises

1. **Retrofit real logging.** Replace ad-hoc `console.log`s in one of your backends with a leveled, structured logger: lifecycle events at `info`, an operation-failure path at `error` with contextual IDs, something chatty at `debug`. Demonstrate `LOG_LEVEL=debug` versus `info` changing the output *without code changes*, and confirm everything goes to stdout (run it in Docker and read it purely via `docker logs`).

2. **Two-grade health check.** Add `/health` (liveness) and `/health/ready` (DB-probing readiness) to that app. Prove honesty: stop the database container and show ready flipping to 503 while the process stays up; restart the DB and watch it recover. Wire the platform's health-check setting to it if deployed.

3. **Log forensics under time pressure.** Have the app log steadily for ten minutes while you trigger a handful of errors at known times (note them down). Then, using only log filtering — no memory, no source diving — reconstruct the incident timeline: first error, frequency, affected request IDs. Time yourself; this is the skill interviews mean by "comfortable with logs."

4. **Full monitoring rig on a live deploy.** On any deployed project (Project 5's API is ideal): external uptime monitor on the health endpoint + Sentry (or similar) SDK with a deliberate test error thrown in production. Verify both alert paths fire — actually receive the down-alert (temporarily break the app or suspend the service) and the new-error email. Screenshot both for your notes.

5. **Root-cause a planted bug like it's production.** Take a small app of yours, have a friend (or your future self via a script) introduce a subtle bug on a branch, deploy it locally in Docker with `info` logging, and diagnose using only logs, the health endpoint, and `git log` — no debugger, no reading the diff. If the logs weren't enough, do the professional move: add targeted logging, redeploy, catch it. Write the postmortem paragraph.

6. **Alert design audit.** List every notification your setup can now send (uptime down, Sentry new-issue, CI failure, deploy failure...). For each: is it actionable? How rare? What would you do on receiving it at 9 a.m.? Kill or downgrade anything that fails the test, and write your one-line alerting philosophy at the top of the list.
