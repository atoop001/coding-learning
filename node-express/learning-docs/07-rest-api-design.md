# Chapter 7: REST API Design

## Overview

So far you have made Express respond to requests — but *which* requests should an API accept, and what should the responses look like? That is a design question, and it matters more than any code you'll write. A well-designed API is predictable: someone who has never read your docs can guess that `GET /api/books/42` fetches book 42 and that deleting it returns `204`. A badly designed one (`POST /getBookData?realDelete=true`) forces every consumer — including future you, and the React frontend you'll build later — to memorize its quirks. This chapter teaches REST as it's practiced in real jobs: resources, correct HTTP methods, correct status codes, sane URLs, versioning, and idempotency. Most importantly, it teaches you to design the whole API on paper *before* writing a line of Express — a habit that separates professional backend work from improvisation. Nothing in this chapter is Express-specific; it's the shared vocabulary of nearly every backend team you might join.

## Definitions & Explanations

**REST (Representational State Transfer)** is an architectural style for APIs built on the mechanics of HTTP itself. Instead of inventing custom commands ("run the addBook operation"), a REST API exposes **resources** (things) at URLs and manipulates them with HTTP's standard methods. You transfer *representations* of resources — usually JSON — back and forth. Real-world APIs are rarely 100% pure REST; the goal is *consistency and predictability*, not ideological purity.

A **resource** is any "thing" your API manages: a book, a user, an order, a comment. Each resource has an identity (usually an `id`) and lives at a URL. A **collection** is the set of all resources of one type (`/api/books`); an **item** (or document) is one member of it (`/api/books/42`). Almost every endpoint you design is an operation on either a collection or an item.

**HTTP method semantics** — the method says *what kind* of operation this is:

- **GET** — read. Never changes anything on the server. Safe to repeat, cache, or prefetch.
- **POST** — create a new resource in a collection (server picks the id), or trigger an operation that doesn't fit CRUD. Not idempotent: two POSTs create two things.
- **PUT** — *replace* an item entirely with the body you send. Fields you omit are gone. Idempotent: sending the same PUT twice leaves the same result.
- **PATCH** — *partially update* an item; only the fields present in the body change.
- **DELETE** — remove an item. Idempotent in effect: after one or five DELETEs, the resource is equally gone.

A method is **safe** if it causes no server-side changes (GET, HEAD). A method is **idempotent** if performing it N times has the same effect as performing it once (GET, PUT, DELETE — but not POST). Idempotency matters practically: networks fail mid-request, and clients (and proxies) will *retry*. A retried PUT is harmless; a retried POST can double-charge a credit card. That's why "create" operations deserve extra care.

**Status codes** are the machine-readable summary of what happened. The ones a working backend developer uses constantly:

- **200 OK** — success, and here's a body (typical for GET, PATCH, PUT).
- **201 Created** — a new resource exists; conventionally you return the created resource and a `Location` header pointing at it.
- **204 No Content** — success, nothing to say (typical for DELETE).
- **400 Bad Request** — the client sent something malformed (unparseable JSON, missing required field). "Fix your request."
- **401 Unauthorized** — misleadingly named; it really means *unauthenticated* — "I don't know who you are; log in."
- **403 Forbidden** — authenticated, but not allowed — "I know who you are, and you can't do this."
- **404 Not Found** — the resource doesn't exist (also commonly used to *hide* resources the caller may not know about).
- **409 Conflict** — the request is valid but collides with current state (duplicate email on registration, editing a stale version).
- **422 Unprocessable Content** — syntactically fine, semantically invalid (JSON parsed, but `email` isn't an email). Many teams use 400 for this too; pick one convention and stay consistent.
- **500 Internal Server Error** — *your* code broke. Never return details of the crash to the client.

The families matter as much as the numbers: **2xx** = success, **4xx** = the client's fault (they can fix it and retry), **5xx** = the server's fault.

**Query parameters** (`?status=read&sort=title&page=2`) modify *how* a collection is read — filtering, sorting, **pagination** (returning results in pages rather than all at once). They belong on GET collection endpoints; the resource identity stays in the path.

**API versioning** means marking your API's contract (`/api/v1/...`) so you can later change response shapes without breaking existing clients. URL-path versioning is the most common and most visible approach; header-based versioning (`Accept: application/vnd.myapp.v2+json`) is cleaner in theory but harder to debug. For your projects, `/api/v1` in the path is plenty — the point is that *once clients depend on a shape, you can't change it in place*.

**An API contract** is the full agreement: endpoints, methods, request shapes, response shapes, status codes, error format. Designing the contract first — on paper — is the professional workflow, because changing paper is free and changing deployed APIs is expensive.

**OpenAPI (formerly Swagger)** is the industry-standard way to write that contract down in a format both humans and tools can read — a YAML or JSON file (conventionally `openapi.yaml`) listing every endpoint, its parameters, request/response shapes, and status codes. Tools then generate interactive docs, client SDKs, and even test stubs from it, so the contract stays a single source of truth instead of a stale wiki page. You don't need to master the full spec now — even hand-writing an `openapi.yaml` for three endpoints teaches the vocabulary (`paths`, `schemas`, `responses`) that shows up in nearly every mid-to-senior backend job posting. The capstone project's stretch goals give you a chance to write one for real.

**REST isn't the only API style.** **GraphQL** is the other one you'll meet in job postings: instead of many fixed endpoints, a GraphQL API exposes a single endpoint where the *client* specifies exactly which fields it wants in a query, and the server returns exactly that shape — no more, no less. This solves REST's two classic annoyances: **over-fetching** (a REST endpoint returns a whole object when you needed one field) and **under-fetching** (you need three REST calls to assemble one screen).

The tradeoff: a single endpoint means HTTP-level caching and status codes stop doing much work, the server must guard against arbitrarily expensive client-constructed queries, and the tooling/mental-model investment is real. Most backend teams still default to REST for simple CRUD and reach for GraphQL when clients (especially mobile, with expensive round trips) need to shape their own responses. Recognition level for now — REST is what this track builds.

## Code Examples

Less code in this chapter, by design — the deliverable of API design is a *table*, not code. Here is the worked example: designing a bookmarks API on paper, then a thin slice of it in Express.

**Step 1: name the resources.** A bookmarks manager has one obvious resource: `bookmark` (fields: `id`, `url`, `title`, `tags`, `createdAt`). Resources are nouns. If you catch yourself naming a resource "createBookmark", you're naming an *operation*, not a thing.

**Step 2: write the endpoint table before any code:**

```text
Method  Path                     Purpose                      Success        Failures
------  -----------------------  ---------------------------  -------------  ------------------
GET     /api/v1/bookmarks        List bookmarks (filterable)  200 + array    —
GET     /api/v1/bookmarks/:id    Fetch one bookmark           200 + object   404
POST    /api/v1/bookmarks        Create a bookmark            201 + object   400/422 (bad body)
PATCH   /api/v1/bookmarks/:id    Update some fields           200 + object   404, 400/422
DELETE  /api/v1/bookmarks/:id    Delete a bookmark            204, no body   404
```

**Step 3: decide the shapes.** Example response for `GET /api/v1/bookmarks/42`:

```json
{
  "id": 42,
  "url": "https://developer.mozilla.org",
  "title": "MDN Web Docs",
  "tags": ["reference", "javascript"],
  "createdAt": "2026-07-01T09:30:00.000Z"
}
```

And a list endpoint that supports pagination wraps the array so there's room for metadata:

```json
{
  "data": [ { "id": 42, "title": "MDN Web Docs" } ],
  "page": 1,
  "pageSize": 20,
  "total": 137
}
```

Returning a bare array works until the day you need to add `total` — then it's a breaking change. Wrapping from day one costs nothing.

**Naive vs. better URL design.** The naive version invents verbs; the better version lets the HTTP method be the verb:

```text
# Naive — verbs in URLs, method ignored (everything is POST)
POST /api/getBookmarks
POST /api/createBookmark
POST /api/deleteBookmark?id=42

# Better — nouns in URLs, methods carry the meaning
GET    /api/v1/bookmarks
POST   /api/v1/bookmarks
DELETE /api/v1/bookmarks/42
```

The naive style forces consumers to read docs for every single call; the better style is guessable, cacheable (GETs can be cached; POSTs can't), and plays correctly with retries.

**Filtering, sorting, pagination via query strings** — identity in the path, refinement in the query:

```text
GET /api/v1/bookmarks?tag=javascript          # filter
GET /api/v1/bookmarks?sort=createdAt&order=desc
GET /api/v1/bookmarks?page=2&pageSize=20      # pagination
GET /api/v1/bookmarks?tag=reference&sort=title&page=1&pageSize=10   # combined
```

**Nested routes for ownership — one level deep, then stop:**

```text
GET /api/v1/users/7/bookmarks        # fine: bookmarks belonging to user 7
GET /api/v1/users/7/bookmarks/42/tags/3/edits   # too deep — hard to build, hard to read
GET /api/v1/tags/3/edits             # better: promote the nested thing to a top-level resource
```

**Step 4 (only now): code.** A slice of the table implemented in Express — notice how the design decisions (201 + Location, 204, query params) translate directly:

```js
// routes shown assume app.use(express.json()) is registered (Chapter 6)
app.get('/api/v1/bookmarks', (req, res) => {
  const { tag, page = '1', pageSize = '20' } = req.query;
  let results = bookmarks;
  if (tag) results = results.filter((b) => b.tags.includes(tag));

  const p = Number(page);
  const size = Number(pageSize);
  const start = (p - 1) * size;
  res.json({
    data: results.slice(start, start + size),
    page: p,
    pageSize: size,
    total: results.length, // total of the FILTERED set — clients need this to render page controls
  });
});

app.post('/api/v1/bookmarks', (req, res) => {
  const bookmark = createBookmark(req.body); // validation comes in Chapter 8
  res.status(201)
    .location(`/api/v1/bookmarks/${bookmark.id}`) // tell the client where it lives
    .json(bookmark);
});

app.delete('/api/v1/bookmarks/:id', (req, res) => {
  const existed = deleteBookmark(req.params.id);
  if (!existed) return res.status(404).json({ error: 'Bookmark not found' });
  res.status(204).end(); // 204 promises "no body" — .json() here would be a contract violation
});
```

**Setting status codes in Express — the small mechanics.** A few method details that trip people up when translating a design table into code:

```js
res.json(book);                  // 200 implied — fine for GET/PATCH success
res.status(201).json(book);      // set the code, then the body (chainable)
res.status(204).end();           // 204 means "no body" — don't attach one
res.sendStatus(404);             // shortcut: sets 404 AND sends "Not Found" as a text body
                                 // — usually NOT what a JSON API wants; prefer
res.status(404).json({ error: { message: 'Bookmark not found' } });
```

`res.sendStatus(n)` and `res.status(n)` look interchangeable and aren't: `sendStatus` *ends the response* with a plain-text body, while `status` only sets the code and waits for you to send JSON. In a JSON API, use `status(...).json(...)` so error responses keep the same shape as everything else (Chapter 8 makes that shape official).

**When an action isn't CRUD.** Some operations don't map to create/read/update/delete — "publish this draft", "retry this payment". The common pragmatic pattern is a sub-resource or action noun under the item, still using POST:

```text
POST /api/v1/drafts/42/publish
POST /api/v1/payments/97/retries
```

This is technically un-pure REST. That's fine. Consistency and clarity beat purity.

## Common Pitfalls

1. **Verbs in URLs** (`/api/createBookmark`, `/api/bookmarks/delete/42`). The HTTP method already says what you're doing; the URL should only say *to what*. Correction: nouns in paths, methods for actions — `POST /api/v1/bookmarks`, `DELETE /api/v1/bookmarks/42`.

2. **Returning 200 for everything**, with success buried in the body (`{ "ok": false, "error": "not found" }` with status 200). This breaks every HTTP-aware tool: caches, monitoring, `fetch`'s `response.ok`, test assertions. Correction: the status code must reflect the outcome — 404 for missing, 400/422 for invalid, 201 for created.

3. **Using GET for state changes** (`GET /api/bookmarks/42/delete`). GET must be safe: browsers prefetch links, proxies cache them, crawlers follow them — a crawler once deleted an entire site's content this way. Correction: anything that mutates goes through POST/PUT/PATCH/DELETE.

4. **Confusing 401 with 403.** 401 = "I don't know who you are" (missing/invalid credentials — the fix is to authenticate). 403 = "I know exactly who you are, and the answer is no." Returning 403 for a missing token sends clients down the wrong debugging path. Correction: no/bad credentials → 401; valid credentials, insufficient rights → 403.

5. **Confusing PUT with PATCH.** Treating PUT as a partial update means a client that PUTs `{ "title": "New" }` silently wipes `url` and `tags` — or worse, your handler makes PUT partial and violates the expectations of anyone who knows HTTP. Correction: PUT replaces the whole resource; PATCH changes only the provided fields. If you only implement one, implement PATCH.

6. **Unbounded collection endpoints.** `GET /api/bookmarks` returning *all* rows works with 50 test records and melts with 500,000 real ones (slow queries, huge payloads, frozen clients). Correction: paginate from day one with `page`/`pageSize` (and a maximum `pageSize` the server enforces), and return `total` so clients can render page controls.

7. **Designing in the code editor.** Adding endpoints one at a time as needs pop up yields inconsistencies — `/bookmarks` here, `/bookmark/list` there, plural here, singular there, three different error shapes. Correction: write the full endpoint table (method, path, request shape, response shape, status codes) *before* implementing, and hold every new endpoint to the same conventions.

## Practice Exercises

1. **Critique and repair.** Here is a real-looking bad API. For each line, write down everything wrong (method choice, URL, status code) and the corrected version: `GET /api/getUsers` → 200; `POST /api/user/new` → 200 `{ "success": true }`; `GET /api/deleteUser?id=5` → 200; `POST /api/user/5/update` → 200; `GET /api/users/999` (nonexistent) → 200 `{ "error": "no user" }`.

2. **Design on paper: a to-do list API.** Resources: lists, and tasks belonging to lists. Produce the full endpoint table (method, path, purpose, success code, failure codes) covering CRUD for both resources, including fetching all tasks in one list and marking a task complete. Decide: is "complete" a PATCH of a field or a POST action? Justify your choice in a sentence.

3. **Design the query language.** For `GET /api/v1/tasks`, specify every query parameter you'd support for filtering (by status, by due date range), sorting, and pagination — including defaults and the maximum `pageSize`. Write out five example URLs and the exact JSON shape of the paginated response.

4. **Status-code drill.** For each scenario, name the single most correct status code: (a) client POSTs a new task successfully; (b) client PATCHes a task that doesn't exist; (c) client sends `pageSize=-3`; (d) client registers with an email that's already taken; (e) client DELETEs a task successfully; (f) your database connection throws during a GET; (g) a logged-in user tries to delete someone else's list.

5. **Idempotency audit.** For each endpoint in your exercise-2 table, mark it safe/unsafe and idempotent/non-idempotent. Then answer: if the network drops after the server processes `POST /api/v1/lists` but before the client gets the response, and the client retries — what happens? Sketch (in prose, no code) one strategy an API could use to make that retry harmless.

6. **Version-break scenario.** You shipped `GET /api/v1/bookmarks` returning a bare JSON array, and a mobile app depends on it. Now you must add pagination metadata. Write a short plan (in prose) for introducing the wrapped `{ data, page, total }` shape without breaking the mobile app, and state when — if ever — you could remove the old shape.
