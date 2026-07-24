# Project 2: HTTP Server from Scratch

## Description

Build a working multi-route JSON API using **only Node's built-in `node:http` module** — no Express, no frameworks, no npm dependencies at all. Your server will expose a small "quotes" API (or another simple collection of your choosing): several GET routes serving JSON, a POST route that accepts a JSON body and adds to an in-memory array, correct status codes and headers throughout, and one hand-served static HTML page.

This is the project where HTTP stops being magic. You will personally parse URLs, collect request bodies from a stream, set `Content-Type` headers, and route by method + path with plain `if`/`switch` logic. It will feel laborious — that's the point. When you meet Express in Chapter 5, you'll know exactly what work it's doing for you, because you'll have done that work yourself.

## Difficulty & Estimated Effort

**Beginner+ — 4–6 hours.**

## Chapters Used

- Chapter 3: Async Node & the Event Loop (streams and events under the hood, not blocking the loop)
- Chapter 4: Building an HTTP Server from Scratch (the `http` module, manual routing, parsing bodies)

## Requirements

Work top to bottom.

### Foundation
- [ ] A single `server.js` (split into modules later if you like) that creates a server with `node:http` and listens on a port taken from an environment variable, defaulting to 3000.
- [ ] The server logs one line per request: method, URL, and the status code you responded with.
- [ ] An in-memory array of at least 5 seed objects, each with an `id` and 2–3 other fields — e.g. `{ "id": 1, "text": "...", "author": "..." }`. (Data resets on restart; that's expected here.)

### GET routes
- [ ] `GET /api/quotes` returns the full collection as JSON with status 200 and `Content-Type: application/json`.
- [ ] `GET /api/quotes/<id>` returns the matching single object, or a JSON error body with status 404 if no such id exists.
- [ ] `GET /api/quotes/random` returns one random item. Decide deliberately: does this route need to be checked before or after the `<id>` route in your routing logic, and why?
- [ ] `GET /api/quotes?author=<name>` filters the collection by author using a parsed query string — the same path as the unfiltered list, behavior switched by query parameter.

### POST route
- [ ] `POST /api/quotes` reads the request body (collected from the stream, chunk by chunk), parses it as JSON, assigns the next id, stores it, and responds 201 with the created object.
- [ ] A body that isn't valid JSON gets a 400 with a JSON error message — the server must **not** crash.
- [ ] A body missing required fields gets a 400 explaining which field is missing.
- [ ] A request with a `Content-Type` other than `application/json` gets a 415 (Unsupported Media Type).

### Errors & edge cases
- [ ] Any unknown path returns a JSON 404 body — never an empty response or a hang.
- [ ] A known path with the wrong method (e.g., `DELETE /api/quotes`) returns 405 with an `Allow` header listing the methods that *are* supported.
- [ ] Every error response uses the same JSON shape, e.g. `{ "error": "message here" }` — pick one shape and use it everywhere.

### Static page
- [ ] `GET /` serves a small hand-written HTML page from disk (read the file, don't inline the string) with `Content-Type: text/html`, briefly describing your API's routes.

### Verification
- [ ] Test every route and every error case from PowerShell and save the commands in your README. Note: in PowerShell, `curl` is an alias for `Invoke-WebRequest` — use `Invoke-RestMethod` for JSON, or `curl.exe` for real curl. Example of the kind of call you'll need:
  `Invoke-RestMethod -Uri http://localhost:3000/api/quotes -Method Post -ContentType "application/json" -Body '{"text":"...","author":"..."}'`
- [ ] Confirm the 404, 405, 400, and 415 cases return the right codes (with `Invoke-WebRequest`, non-2xx throws — `-SkipHttpErrorCheck` shows you the raw response instead).

## Hints

- The request body is not a property — it's a **stream** that arrives in pieces. Chapter 4 covered the event pattern for collecting chunks and knowing when the body is complete. What could go wrong if the JSON.parse happens before the last chunk arrives?
- The WHATWG `URL` class (`new URL(req.url, ...)`) splits path from query string and gives you `searchParams` for free. `req.url` alone is a raw string — resist parsing it manually.
- Routing without a framework is just data: method + pathname → handler. Whether you use a chain of `if`s or an object lookup, keep the *decision* in one place. For `/api/quotes/<id>`, you'll need to extract the id yourself — string methods or a small regex both work.
- Think about route-matching **order** early: `/api/quotes/random` and `/api/quotes/17` look identical to a naive pattern. Whichever check runs first wins.
- Wrap `JSON.parse` thoughtfully — it's the one line in this project guaranteed to throw on user input. Where does that try/catch belong so the server always answers instead of dying?
- Write yourself a tiny `sendJson(res, statusCode, data)` helper the second you notice the same three lines repeating. That helper is a preview of what Express's `res.json()` does.
- If a request seems to hang forever in your client, check: did every code path actually call `res.end()`?

## Stretch Goals

- Add `DELETE /api/quotes/<id>` (204 on success, 404 if missing) and `PUT /api/quotes/<id>` for full CRUD.
- Persist the collection to a JSON file with `fs/promises` so data survives restarts — and think about what happens if two requests write at nearly the same time.
- Serve an entire small static folder (HTML + CSS + one image) with correct `Content-Type` per file extension, and defend against `..` path traversal in the URL.
- Add a `?limit=N&offset=M` pagination option to the list route, with validation and sensible defaults.
- Record how long each request took (from first seeing it to response finished) and include it in your per-request log line.
