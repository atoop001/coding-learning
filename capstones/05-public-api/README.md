# Capstone 5: Public API with a Public Face

Build a themed, documented, hosted REST API that a total stranger could discover, sign up for,
and start calling within five minutes — without ever talking to you.

The product here is not an app. The product is the API itself: its design, its docs, its keys,
its limits, its uptime.

## Why This Project

Walk through a hundred junior developer portfolios and you will see a hundred apps: todo lists,
weather dashboards, clones of well-known sites. What you will almost never see is a well-designed,
well-documented, actually-operated API — one with real-world concerns handled: API keys, rate
limiting, versioning, error contracts, and documentation a stranger can follow.

That gap is your opportunity, because backend interviews are built almost entirely out of those
concerns:

- "How would you design the endpoints for X?" — you did that, on paper, before writing code.
- "How do you handle authentication for an API?" — you shipped self-serve key signup and stored
  keys safely.
- "What happens when a client hammers your API?" — you built rate limiting and can explain your
  429 responses and headers from memory.
- "How do you evolve an API without breaking clients?" — you versioned yours from day one and can
  say why.
- "How do you know your service is up?" — you have health checks, monitoring, and an uptime badge
  to point at.

An app shows you can build features. A public API shows you can design contracts, operate a
service, and think about the developers on the other end. That is exactly what backend and
full-stack roles hire for.

There is also a credibility multiplier: at the end of this project, real humans (at least two)
will sign up and make calls against your API. "People I don't live with hold API keys to my
service" is a sentence very few junior candidates can say.

## Difficulty & Estimated Effort

**Difficulty:** Advanced

**Estimated effort:** 25–40 hours

This assumes you have completed the prerequisite tracks and are not learning Express or SQL for
the first time here. The hours are spread unevenly:

- The core API (Phase 2) goes fast if your Phase 1 design is honest and complete.
- Productizing (Phase 3) takes longer than most people expect — keys and rate limits have sharp
  edges.
- Documentation (Phase 4) deserves real hours. Docs are half the product; budget accordingly.
- Operating (Phase 5) is mostly waiting and fixing, but the fixing is where the learning lives.

## Prerequisites

Complete these before starting. This capstone assumes the skills, not just exposure to them.

- **node-express** — the entire track. Routing, middleware, error handling, request validation,
  project structure.
- **sql** — schema design, joins, indexes, and using a relational database from Node.
- **testing-debugging-security** — writing API tests, input validation habits, secure secret
  handling, hashing.
- **deployment-devops** — through the CI/CD and cloud deployment chapters. You will deploy this
  with a managed database and a pipeline, not by dragging files onto a server.

If any of those feel shaky, go shore them up first. This project punishes gaps — a weak
validation habit or a fuzzy understanding of middleware will cost you hours in Phase 3.

## Phased Milestones

Work the phases in order. Each phase produces something real. Do not start the next phase until
the current one's boxes are checked.

### Phase 1: Design on Paper

No code in this phase. The single biggest predictor of how smoothly Phases 2–5 go is how honest
and complete this phase is.

**First, pick your theme.** Given your tutoring business, a **practice-problem generator API**
(math drills by topic and difficulty, vocabulary questions) or a **flashcard deck API** has a
genuine tie-in — you could eventually use it yourself, and "I built this for my tutoring
business" is a great interview story. But it is your project. Other solid options:

- A local-trivia API — questions about your city or region, organized by category.
- A workout-programs API — exercises, programs, progressions.

Pick the one you can imagine explaining with enthusiasm in an interview. Enthusiasm survives
40 hours of work; obligation does not.

- [ ] Choose your theme and write one paragraph stating who the imaginary consumer of this API is
      and what they would build with it
- [ ] Define your resources (the nouns of your API — e.g. `problems`, `topics`, `decks`, `cards`)
      and how they relate to each other
- [ ] Write a full API spec document **before any code**: every endpoint, its method, URL, query
      parameters, request body shape, success response shape, every status code it can return,
      and the error shape for each failure mode
- [ ] Design one consistent error format used by every endpoint, and document every error code
      your API can produce
- [ ] Design the data model: tables, columns, types, relationships, and which columns need
      indexes
- [ ] Write down explicitly what v1 will NOT include — features you are deliberately cutting.
      This list is a design artifact, not an apology

For the error format, something in this spirit (design your own, but keep it consistent):

```json
{
  "error": {
    "code": "INVALID_PARAM",
    "message": "difficulty must be an integer from 1 to 5"
  }
}
```

Review your spec by role-playing the consumer: for each thing they would want to do, can you
point at the exact endpoint and say precisely what comes back? If you catch yourself saying
"hmm, I guess they'd...", the spec is not done.

### Phase 2: Core API

Now build exactly what the spec says. When you are tempted to deviate, update the spec first,
then the code — keep the two in lockstep for the whole project.

- [ ] Set up the Express project with a clean structure (routes, middleware, and data access
      separated) and a SQL database built from your Phase 1 schema
- [ ] Implement every v1 endpoint from your spec, with full CRUD where the spec calls for it
- [ ] Implement the consistent error format from Phase 1 in one place (error-handling
      middleware), so no endpoint can accidentally return a differently-shaped error
- [ ] Validate every input — path params, query params, request bodies — and reject bad input
      with your documented 400 error shape, never a stack trace or a 500
- [ ] Add pagination to every list endpoint (limit/offset or cursor — your choice, but document
      it and apply it consistently), with sane defaults and an enforced maximum page size
- [ ] Seed the database with enough realistic sample data that responses look real, not like
      test fixtures
- [ ] Write thorough automated tests: happy paths, validation failures, not-found cases, and
      pagination edge cases for every endpoint

The bar for this phase: someone hitting your API with curl and deliberately malformed requests
should never see anything your spec did not predict.

### Phase 3: Productize

This is what separates "an Express app" from "a public API." Strangers will hold keys to your
service; act like it.

- [ ] Build a self-serve signup endpoint (or minimal page) that issues an API key: the user
      provides an email, receives a key, and the key is shown **exactly once**
- [ ] Store keys the way you store passwords: hash them, so the raw key never touches your
      database, and authenticate incoming requests by comparing hashes
- [ ] Require a valid key on all data endpoints via middleware; return your documented 401 error
      shape when the key is missing or invalid
- [ ] Implement per-key rate limiting: choose a window and limit (e.g. N requests per minute),
      return a proper `429 Too Many Requests` with your standard error shape when exceeded
- [ ] Include rate-limit headers on responses (`RateLimit-Limit`, `RateLimit-Remaining`,
      `RateLimit-Reset`, or the `X-RateLimit-*` variants) so clients can throttle themselves
- [ ] Track usage per key (request counts, at minimum daily totals) so you can answer "how much
      is key X using?" with a query
- [ ] Move all endpoints under a versioned path prefix (`/v1/...`) and treat v1 as frozen: from
      now on, breaking changes mean `/v2`, not edits to `/v1`
- [ ] Extend your test suite to cover signup, key-auth failures, rate-limit trips (including the
      headers), and rejection of revoked or invalid keys

In every document and example, use obviously fake placeholder keys such as
`demo_key_REPLACE_ME`. Never paste a real key into the repo, the docs, or a test fixture.

### Phase 4: Documentation Site

Docs are part of the product. A brilliant API with bad docs is, to a stranger, a bad API. Hold
this phase to the same bar as your code.

- [ ] Build a human-readable docs page (a static site is fine) with a clear landing: what the
      API is, what you can build with it, and a prominent link to get a key
- [ ] Write a quickstart guide: sign up, get a key, make your first call — with a
      copy-pasteable curl command using the `demo_key_REPLACE_ME` placeholder
- [ ] Document every endpoint: method, URL, parameters, an example request (copy-pasteable curl)
      and an example response (real JSON produced from your seeded data)
- [ ] Write an error-code reference: every error code your API returns, what it means, and what
      the caller should do about it
- [ ] Document authentication (exactly where the key goes in a request) and rate limits (the
      numbers, the headers, what a 429 means, and how to back off)
- [ ] Proofread the whole site by following it yourself, literally copy-pasting each command
      into a fresh terminal — every command must work exactly as written

A useful test for each page: could a developer who has never heard of you succeed using only
what is on the screen? If a step lives in your head instead of on the page, it does not exist.

### Phase 5: Ship & Operate

An API that runs on your laptop is a demo. Ship it, keep it up, and put it in front of real
people.

- [ ] Deploy the API to a cloud host with a **managed** database (not SQLite on the app server,
      not a database you hand-administer over SSH)
- [ ] Set up CI/CD: every push runs the full test suite; deploys only happen on green
- [ ] Add a health-check endpoint (e.g. `/v1/health`) that verifies the database connection, and
      wire it into your host's health monitoring
- [ ] Set up basic monitoring/alerting so you find out the API is down before a user tells you
- [ ] Add a status or uptime badge to the docs site
- [ ] Confirm no secrets are in the repo — database URLs, hashing secrets, and admin credentials
      all live in environment configuration, and none appear anywhere in git history
- [ ] Get at least 2 real humans (not you) to sign up, get keys, and make successful calls using
      only your docs site
- [ ] Watch where they get confused, then fix the docs or the API accordingly — their confusion
      is your bug list, and closing it is part of this phase

## Definition of Shipped

This capstone is done when every box below is checked. Not before.

- [ ] The API is live at a public base URL and stays up without you babysitting it
- [ ] The docs site is live and covers quickstart, every endpoint, authentication, rate limits,
      and error codes
- [ ] A stranger can go from landing on the docs page to their first successful API call in
      **under 5 minutes**, with zero help from you — verified with a real stranger
- [ ] The full test suite runs in CI and is green on the currently deployed commit
- [ ] Rate limiting demonstrably works: you can show a request being 429'd, with correct
      headers, against the live deployment
- [ ] No secrets are committed anywhere in the repository history

## Hints

Nudges, not answers. Reach for these when you are stuck on the *approach* — the implementation
is yours.

**API keys are passwords.** Treat them with exactly the same care. The raw key exists in two
places only: the response body at creation time, and the user's hands. Your database stores a
hash. If your database leaks, attackers should get nothing usable. Corollary: you cannot show
the user their key again later — that is not a limitation, it is the design. This is also why
real services like Stripe and GitHub show keys exactly once.

**Rate-limit storage is a real decision — make it consciously.** You need to count requests per
key per time window. Where does that counter live? In-process memory is simple but resets on
every deploy and breaks the moment you run two instances. The database is durable but adds a
write on every request. A dedicated store like Redis is the industry answer but adds a moving
part to deploy and pay for. For this project, any of the three can be right — what matters is
that you can articulate the trade-off you chose and what would force you to outgrow it. That
articulation is, word for word, an interview answer.

**Design errors for the developer on the other end.** When your API rejects a request, the
caller is a stressed programmer staring at a terminal. `{"error": "Bad Request"}` tells them
nothing. A good error names the specific problem, points at the offending field, and ideally
says what valid input looks like. Test your errors the way you test your docs: seeing only the
response body, would *you* know what to fix?

**Content APIs need content.** A practice-problem or trivia API with nine rows in the database
feels dead on arrival. Think about generation early: math drills can be generated from templates
with randomized values, which gives you effectively infinite content for free. Vocabulary and
trivia need curation, so either budget real time for it or scope the topic list down. Fifty
solid items across three topics beats five hundred junk items across twenty.

**Freezing v1 is a discipline, not a technicality.** The moment your first outside user gets a
key, changing a response shape in `/v1` breaks someone else's code. Practice the discipline now:
additive changes (new optional fields, new endpoints) are fine within v1; renames, removals, and
shape changes are not — they wait for v2. You will feel the itch to "just fix" a field name you
regret. Write it on the v2 list instead. Living with a small design regret inside a frozen
version is the most authentic API-maintainer experience this project offers.

**Test your rate limiter like an attacker, not like a demo.** One loop of polite sequential
requests is the easy case. What happens at the exact boundary of the limit? What happens to a
*different* user's key while one key is being throttled? Does the window actually reset when the
headers say it will?

## Stretch Goals

Only after "Definition of Shipped" is fully checked:

- **OpenAPI specification** — describe your API in an OpenAPI (Swagger) document and generate
  interactive reference docs from it. Keep your hand-written quickstart; let the generated docs
  handle the per-endpoint reference.
- **Tiny demo frontend** — a small page that consumes your own API (e.g. renders a practice quiz
  from the problems endpoint). It proves the API is pleasant to build against and gives
  non-technical viewers something to look at.
- **Webhooks** — let users register a URL and notify it on some event (new content published, a
  usage threshold crossed). Delivery, retries, and request signing make this a serious backend
  deep-dive.
- **Usage dashboard** — a page where a key holder can see their own request counts over time.
  You already collected the data in Phase 3.
- **SDK snippet examples in two languages** — for each endpoint, show the call in JavaScript
  (fetch) and one other language (Python is a natural pick) alongside the curl example. Real API
  docs do this, and it reads as very polished.

## Telling the Story

**On your resume.** Lead with what it is, and prove it operated:

> Designed, built, and operate a public REST API (Node/Express, PostgreSQL) with self-serve
> API-key signup, per-key rate limiting, versioned endpoints, and a hosted documentation site;
> deployed via CI/CD with health checks and uptime monitoring.

If you have real numbers, use them — they transform the bullet: "12 signups," "4,000 requests
served," "99.9% uptime over 3 months." Even small numbers beat none, because any real number
proves real operation. Keep a running note of your stats; future-you writing applications will
thank present-you.

**In interviews.** This project generates unusually good stories. Prepare these two in
particular:

- **A design decision you reversed.** Somewhere between Phase 1 and Phase 5, you will change
  your mind — an endpoint shape, a pagination style, an error code, something you froze into v1
  and came to regret. That story ("here is what I designed, here is what reality taught me, here
  is what I did about it, and here is what v2 will do instead") demonstrates judgment and
  honesty, which interviewers rate far above never-made-a-mistake polish.
- **How you chose your rate-limiting approach.** Walk through the options you considered
  (memory vs. database vs. dedicated store; fixed vs. sliding window), the trade-offs, what you
  picked for your scale, and — crucially — what would force you to change it. This is a
  systems-design interview answer in miniature, backed by something you actually shipped.

Also be ready for the follow-up every interviewer asks: "what would you do differently?" Your
v2 list from the freezing discipline in Phase 3 *is* that answer. Bring it.

Finally: the two real humans who signed up are part of the story. What confused them, and what
you changed because of it, is user-driven iteration — say so, in exactly those terms.
