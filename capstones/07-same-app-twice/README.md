# Capstone 7: Same App, Twice

Re-implement the backend of one of your existing Node/Express capstones in Java (Spring Boot) or C# (ASP.NET Core) — against the **identical** API contract. Same endpoints, same status codes, same JSON shapes, same auth behavior.

The proof that you pulled it off: **one language-agnostic, black-box test suite** that talks to a running server over HTTP and passes **unchanged** against both implementations. Only the base URL changes between runs.

Almost every junior developer *says* "concepts transfer; the language is just syntax." Almost none of them can point at anything that proves it. This project is that proof.

By the end, the capstone directory looks roughly like this:

```
07-same-app-twice/
  CONTRACT.md            # the frozen API contract — the law
  DIFFERENCES.md         # what the second stack pushed back on
  contract-tests/        # one black-box HTTP suite, base URL via env var
  node-app/              # the original implementation (or a link to it)
  java-app/  OR  csharp-app/
  docker-compose.yml     # both apps + databases, one command
  README.md              # how to run everything
```

## Why This Project

**It demonstrates language-independence of engineering skill.**

Anyone can follow a Spring Boot tutorial. What's rare is showing that you understand HTTP APIs, auth, validation, error handling, and data modeling well enough to express the *same design* in two ecosystems with very different opinions. When both implementations pass the same test suite, the argument makes itself — an interviewer doesn't have to take your word for it, they can watch CI go green twice.

**The contract-test approach mirrors real professional work.**

When companies rewrite a service — Node to Go, a monolith module to a microservice, Rails to Java — the central engineering question is always "how do we know the new one behaves like the old one?" The industry answer is exactly what you'll build here:

1. Pin down the contract.
2. Encode it as black-box tests.
3. Run those tests against both the old and new systems until both are green.

Doing this once, end to end, teaches you more about migrations than any blog post about them. It is also the core mechanic behind "strangler fig" rewrites, API versioning discipline, and consumer-driven contract testing — all things enterprise teams talk about in interviews.

**It makes you conversant in the enterprise stack.**

A huge share of web-dev jobs — especially stable, well-paying ones — run on Spring Boot or ASP.NET Core. Having a real project in one of them, one that solves a problem you already understood deeply from the Node side, is far more convincing than a tutorial-shaped to-do app. You'll be able to talk about dependency injection, static typing at the API boundary, and framework conventions from experience rather than from a course.

## Difficulty & Estimated Effort

**Difficulty: Advanced.**

This is deliberately the **last capstone to attempt**. It has hard prerequisites that the others don't: you need a *finished, working* Node/Express backend to re-implement, and you need the Java or C# track behind you. Attempting this before both are true turns one hard project into three simultaneous ones — learning a language, learning its web framework, and reverse-engineering your own API at the same time.

**Estimated effort: 30–50 hours**, roughly:

- Freezing the contract and building the test suite: 8–14 hours
- The second implementation: 15–25 hours
- Parity debugging, Docker, CI, and the write-up: 7–11 hours

The wide range is honest. If your source API is small (the public API from Capstone 5), you'll land near the bottom. If you're re-implementing the tutoring platform's full API with auth and roles, expect the top end. Budget calendar time accordingly — this is a multi-week project done in evening sessions.

## Prerequisites

All of these are hard prerequisites, not suggestions:

- **node-express track complete**, with a finished API project to re-implement. The suggested candidates:
  - The backend of **Capstone 1** (tutoring business platform) — bigger, with auth and roles; the more impressive choice.
  - **Capstone 5** (public API) — smaller and cleaner; the faster choice.
  - Any other nontrivial API of yours also works. "Nontrivial" means: multiple resources, auth of some kind, validation, and real error handling.
- **java track OR csharp track complete.** Pick the one you finished. You do not need both; the second implementation is in exactly one of them.
- **testing-debugging-security track complete.** The contract test suite is the spine of this project — you need to be comfortable writing HTTP-level tests and reasoning about auth flows and edge cases.
- **Docker chapters of deployment-devops complete.** Docker Compose is what makes running two backends plus a database side by side on one Windows machine sane instead of miserable.

What you do **not** need: production experience in Spring or ASP.NET, both enterprise languages, or a frontend. This capstone is backend-only on purpose — any frontend you built for the source capstone keeps talking to the Node app and is out of scope here.

**Which second stack?** Use whichever track you completed — that decision was already made when you chose Java or C# earlier in the learning path. If you somehow finished both, pick based on the jobs you're actually applying to locally: Spring Boot dominates at banks, insurers, and large legacy enterprises; ASP.NET Core is strong at Microsoft-shop companies and much of the mid-market. There is no wrong answer; the skills mirror each other almost exactly, which is rather the point of this project.

## Phased Milestones

Work the phases in order. Each phase's output is the input to the next — skipping ahead (especially into Phase 3) is the classic way this project fails.

The shape of the whole project in one line each:

1. **Freeze the contract** — write down exactly what the existing API does.
2. **Contract test suite** — turn that document into black-box HTTP tests, proven against Node.
3. **Second implementation** — build the Java/C# app until those tests go green.
4. **Parity & polish** — both green, differences documented.
5. **Ship the comparison** — Compose, CI matrix, write-up.

### Phase 1: Freeze the Contract

Document the existing API completely, then treat that document as law.

- [ ] Choose the source app (Capstone 1's API, Capstone 5, or another nontrivial API of yours) and confirm it still runs and passes its own tests
- [ ] Create `CONTRACT.md` and list **every endpoint**: method, path, path/query parameters, request body shape, response body shape, and every status code each endpoint can return
- [ ] Document the error shape exactly — field names, types, structure of validation errors. Copy real responses from the running app; don't write it from memory
- [ ] Document the full auth flow: how tokens/sessions are issued, where they're sent (header name, cookie name), expiry behavior, and what an unauthenticated vs. unauthorized request each get back (401 vs. 403, and the body of each)
- [ ] Document edge behavior for every endpoint that has it:
  - Nonexistent ID vs. malformed ID (are they both 404, or 404 and 400?)
  - Empty request body, and a body with unknown extra fields (ignored or rejected?)
  - Missing pagination params (what are the defaults?) and out-of-range ones (clamped or 400?)
  - Duplicate creation (409, 400, or silently allowed?)
- [ ] Wherever the Node code is ambiguous or accidental, **decide** what the correct behavior is and write the decision down
- [ ] Declare the contract frozen: from this point the document is law for *both* implementations — including the original

A contract entry doesn't need to be fancy. Something like this per endpoint is enough:

```
GET /api/students/:id
  Auth: Bearer token required
  200: { "id": number, "name": string, "email": string, "createdAt": ISO-8601 string }
  401: standard error body (no/invalid token)
  404: standard error body (unknown id)
  400: standard error body (id not a number)
```

Your Node app almost certainly has behavior you never consciously chose — one route returns `200` with an empty array where another returns `404`; one error body says `message`, another says `error`. Phase 1 is where you choose, once, on purpose. Expect to find at least a handful of inconsistencies; discovering them is a feature of this phase, not a distraction from it.

### Phase 2: Contract Test Suite

Encode the contract as executable, black-box HTTP tests.

- [ ] Pick a test stack and create the suite as its own directory (e.g., `contract-tests/`) — it must know *nothing* about either implementation's internals, only HTTP
- [ ] Make the base URL configurable via an environment variable (e.g., `API_BASE_URL`) so the identical suite can point at any running server
- [ ] Verify the switch actually works from PowerShell — e.g., `$env:API_BASE_URL = "http://localhost:3000"` for one run, then the other port for the next — and document both commands in the suite's README
- [ ] Write happy-path tests for every endpoint in `CONTRACT.md`: correct status code, correct response shape, correct headers where they matter (e.g., `Content-Type`, `Location` on 201s)
- [ ] Write error-case tests: every documented 400/401/403/404/409 path, asserting the error body shape, not just the status code
- [ ] Write auth-flow tests: register/login (or key issuance) works; authenticated requests succeed; missing credentials get 401; wrong-user or wrong-role access gets the documented response
- [ ] Write edge-case tests for everything in the edge-behavior section of the contract
- [ ] Solve test data setup in a black-box way: either a documented seed script both implementations will share, or tests that create their own data through the API itself
- [ ] Run the whole suite against the **existing Node implementation** and iterate until green

That last checkbox has a rule attached: when a test fails against Node, exactly one of three things is wrong — the test, the contract doc, or the Node app. Decide which, fix that one thing, and re-run. The Node app (as amended by your Phase 1 decisions) is the reference implementation.

Do **not** start Phase 3 until this suite is green against Node. An unverified test suite would have you debugging two unknowns at once later — "is my Java code wrong, or was the test always wrong?" is a miserable question at hour 30.

### Phase 3: Second Implementation

Build the same API in Spring Boot or ASP.NET Core, using the contract suite as your progress meter.

- [ ] Scaffold the project (Spring Initializr or `dotnet new webapi`), wire up configuration, and get a health/root endpoint answering over HTTP
- [ ] Point the new app at the **same database schema** — identical table names, column names, types, and constraints as the Node version. Same schema, separate database instance
- [ ] Set up configuration for secrets and connection strings using environment variables — placeholders like `<your-db-password-here>` in any committed example files, never real values
- [ ] Implement the auth flow first, since most endpoints depend on it; get the auth section of the contract suite green
- [ ] Implement endpoints one resource at a time, running the contract suite (pointed at the new app) after each one — watch the green count climb
- [ ] Match the error shape exactly: this usually means a global exception handler / middleware that maps framework errors into *your* documented error body instead of Spring's or ASP.NET's default problem-details format
- [ ] Match validation behavior: the same bad inputs must be rejected with the same status codes and error shapes as Node
- [ ] Match serialization details: JSON property casing, date formats, null-field handling — configure the framework's serializer to obey the contract

Resist every urge to "do it the proper Spring/ASP.NET way" when that way changes the wire format. Internally, absolutely follow the framework's conventions — DI, layering, its ORM, its project structure. Externally, the bytes on the wire follow the contract. Holding that line is the entire discipline of this project.

### Phase 4: Parity & Polish

Close the gap to 100% and capture what you learned while it's fresh.

- [ ] All contract tests green against the Node implementation
- [ ] All contract tests green against the Java/C# implementation — the same suite, unmodified, only the base URL differing
- [ ] Create `DIFFERENCES.md` and log every place the second stack pushed back:
  - Framework conventions you had to override (default error formats, JSON key casing, trailing-slash handling, default 404 bodies)
  - Things the type system caught that Node let slide
  - How dependency injection changed the code's structure compared to your Node modules
  - Anything that genuinely surprised you, in either direction
- [ ] Clean up both codebases: consistent structure, no dead code, sensible names — this project will be *read* by interviewers
- [ ] Write a short README in each implementation's directory: how to run it, how to run the contract suite against it

The differences log is not busywork. It is the raw material for both the Phase 5 write-up and your interview answers, and it evaporates from memory within two weeks if you don't write it down now. One honest paragraph per friction point beats ten vague bullet points.

### Phase 5: Ship the Comparison

Make the whole thing runnable by a stranger, and tell the story.

- [ ] One `docker-compose.yml` at the capstone root that brings up **both** implementations (on different ports) plus their databases with a single command
- [ ] Verify a cold start on your own machine: delete local volumes, `docker-compose up`, run the suite against each port — this is the exact experience a reviewer will have
- [ ] CI (GitHub Actions) with a matrix job: one leg starts the Node app and runs the contract suite against it; the other leg does the same for the Java/C# app; both must pass for the build to be green
- [ ] A top-level README explaining the project, the contract-test idea, and exactly how to run everything (compose up, run the suite against port A, run it against port B)
- [ ] A written comparison of the two implementations, with two rules:
  - **Developer-experience comparison is required.** What each stack made easy, hard, verbose, or safe — with concrete examples pulled from your differences log, not vibes.
  - **Performance comparison is optional, and must be caveated if included.** Dev-mode servers on a laptop under Docker is not a benchmark. If you publish numbers, say exactly what they do and don't mean.

## Definition of Shipped

This capstone is done when every box below is checked — not before.

- [ ] One contract test suite is green against **both** implementations in CI, with the runs visible in the Actions history
- [ ] `docker-compose up` brings up both apps side by side, and the README's instructions for pointing the suite at each one actually work on a clean machine
- [ ] `DIFFERENCES.md` is written, specific, and honest
- [ ] The comparison write-up is published — post it on the blog you built in Capstone 2 and link it from this repo's README
- [ ] The top-level README lets someone who has never seen this project run everything in under ten minutes
- [ ] No secrets committed anywhere — connection strings and signing keys come from environment variables, with `<placeholder>` values in every committed example file

If a checkbox is at 90%, it is unchecked. "The CI matrix mostly passes" and "compose works if you also run this one manual step" are the difference between a portfolio piece and a repo you have to apologize for in an interview.

## Hints

**Freeze the contract *before* reading Spring or ASP.NET docs.**

Once you see how the new framework "wants" to do pagination or error bodies, the temptation to improve the API mid-rewrite becomes overwhelming — and every improvement breaks parity and invalidates your tests. Improvements go on a list for *after* both implementations are green. The discipline of not touching the contract is most of what this project teaches, and it's the part hiring managers recognize.

**The test suite's language doesn't matter — pick what's fast for you.**

JavaScript with `fetch` or supertest-style helpers pointed at a URL is completely fine. Because the tests are black-box HTTP, writing them in JS doesn't favor the Node implementation in any way; the tests can't see inside either server. Don't burn hours agonizing over a "neutral" language — neutrality comes from the black-box boundary, not the test runner.

**Date/time and JSON serialization are where parity breaks first.**

Expect fights over:

- ISO-8601 formatting: timezone suffix (`Z` vs. `+00:00`), milliseconds present or absent, precision
- Numbers serialized as `1` vs. `1.0`, and big IDs as numbers vs. strings
- `null` fields included vs. omitted entirely
- Property-name casing — Jackson and System.Text.Json each ship with their own default opinions, and neither defaults to what Express happened to send

Write contract tests that assert exact serialized values for these early, so the second implementation hits the mismatch on day one instead of day ten.

**Auth parity needs deliberate attention.**

If your Node app issues JWTs, the second implementation must accept and issue tokens the contract suite treats identically: same header name, same scheme (`Authorization: Bearer ...`), same expiry behavior, same 401/403 split. You do *not* need byte-identical tokens — the suite logs in through each app's own login endpoint and uses whatever token it gets back — but the *observable behavior* around tokens must match exactly. Keep signing secrets in environment variables on both sides (`<your-jwt-secret-here>` in committed examples), and test the expired-token and tampered-token paths explicitly; they are the classic silent divergence.

**Keep both database schemas literally identical.**

Same table names, same column names, same types, same constraints. If the ORM on the new side wants different naming conventions, configure the ORM to match the schema — never the reverse. Divergent schemas quietly produce divergent behavior (default ordering, uniqueness errors, nullability) that surfaces later as baffling contract failures.

**When a contract test fails on the new app, read the raw HTTP first.**

Log or capture the actual response body and headers before diving into framework code. Nine times out of ten, the diff between "what Node sent" and "what the new app sent" tells you exactly which serializer or middleware default to override.

**Run the suite constantly, not at the end.**

The green-test count is your progress bar. If you build all endpoints and then run the suite, you'll face fifty failures with tangled causes instead of two failures with obvious ones.

**Scope control: match, don't grow.**

Everything you're tempted to add to the second implementation — an extra endpoint, better pagination, nicer error messages — is out of scope by definition, because the contract is frozen. If a genuinely good idea surfaces, add it to a "post-parity improvements" list at the bottom of `DIFFERENCES.md`. When both apps are green, you can implement it in *both* (updating the contract and the tests first), which is itself a realistic exercise in versioning a live API.

## Stretch Goals

- **A third implementation.** Python (Flask or FastAPI) or Rust (Actix or Axum) against the same contract. The third one goes dramatically faster than the second — which is itself the lesson, and a great closing line for the write-up.
- **An honest load-test comparison.** Production builds, identical hardware, warmed-up servers, a real tool (k6, oha), and a write-up that states every caveat plainly. The honesty is both the hard part and the valuable part — a hedged, careful benchmark is far more credible than a dramatic one.
- **Extract the contract suite into its own repository**, with its own README describing the contract it enforces and how to point it at any candidate server. This is exactly the shape of a real conformance suite, and it makes the "one suite, N implementations" idea visible at a glance.
- **A blog series, one post per phase.** "Freezing an API contract," "Testing a rewrite before writing it," "What C# taught me about my JavaScript" — each phase of this project is a genuinely good post on its own.

## Telling the Story

**On your resume**, lead with the mechanism, not the languages:

> Rebuilt a production-style REST API (Node/Express) in [Spring Boot / ASP.NET Core] against a frozen API contract; verified behavioral parity with a language-agnostic black-box suite of N HTTP tests running against both implementations in a CI matrix.

Make N a real number, and make sure the linked CI history backs it up. A second bullet can carry the infrastructure: one Docker Compose file running both stacks side by side, matrix CI, published comparison write-up.

**In interviews**, this project gives you two unusually strong angles:

1. **What the second language's constraints taught you about the first.** Talk about specific things the type system or DI container forced you to make explicit that Node had let you leave implicit — and whether, in hindsight, the Node version was hiding a bug or just being pleasantly concise. Concrete examples from your `DIFFERENCES.md` turn "I know two stacks" into "I understand the trade-offs between them," which is a senior-sounding sentence backed by evidence a junior can actually produce.

2. **How contract tests de-risk migrations.** You can now describe, from experience, the workflow real teams use for rewrites: pin the behavior, encode it as tests the old system passes, build the new system against those tests, ship when both are green. If an interviewer mentions a migration, rewrite, or "strangler fig" project — and at enterprise shops, someone will — you have a firsthand story instead of a definition.

A third, smaller angle worth having ready: **the tests as documentation.** When asked "how do you approach an unfamiliar codebase," you can point out that your contract suite is a complete, executable description of the API's behavior — anyone could rebuild either implementation from the tests alone. That reframes testing from "chore after coding" to "the most durable artifact in the repo," which is exactly how strong teams think about it.

The quiet flex of this capstone is the discipline it demonstrates: you resisted rewriting the API "better," matched behavior byte for byte, and proved it mechanically. That is the temperament teams are actually hiring for.
