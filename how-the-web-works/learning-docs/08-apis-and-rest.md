# Chapter 8: APIs & REST — The Web for Programs

## Overview

So far the "client" has mostly meant a browser fetching pages for humans. But an enormous share of web traffic is programs talking to programs: your weather app calling a forecast service, JavaScript on a page fetching fresh data, one company's servers querying another's. The interface these conversations go through is an **API** — and on the web, APIs are overwhelmingly **HTTP endpoints exchanging JSON**, most of them organized by a set of conventions called **REST**.

This chapter covers what an API actually is, how JSON works, the REST conventions (resources, methods-as-verbs, status codes, statelessness), API versioning, and the practical craft of *exploring* an unfamiliar API with curl and `fetch` — a skill you'll use weekly for the rest of your programming life. Everything from Chapters 4–7 applies directly: an API call is just an HTTP request wearing work clothes.

## Definitions & Explanations

### What an API is

**API (Application Programming Interface)**: a defined surface through which one program uses another. The term is general (Python's `list.sort()` is an API), but on the web it means: **URLs you request to get or change data, instead of pages**.

The response difference is the whole story:

```
Human-facing:   GET /products/42      -> HTML: layout, images, styles... for eyes
Program-facing: GET /api/products/42  -> {"id": 42, "name": "Boots",
                                          "price": 89.95}         ... for code
```

Same server skills, same HTTP, but the API returns *pure data* and leaves presentation to the caller — a website, a mobile app, a script, anyone.

### JSON in ninety seconds

**JSON (JavaScript Object Notation)** is the lingua franca of web APIs — text representing data with six building blocks:

```json
{
  "name": "Ada",                       // string
  "age": 36,                           // number
  "active": true,                      // boolean (true/false)
  "nickname": null,                    // null
  "languages": ["python", "js"],       // array
  "address": {"city": "London"}        // object (nested)
}
```

Rules worth tattooing: keys are double-quoted strings; no trailing commas; no comments (the `//` above are illustration only). It maps 1:1 onto Python dicts/lists (`json.loads`) and JavaScript objects (`response.json()`), which is precisely why it won.

The response header that announces it: `Content-Type: application/json`.

### REST: conventions, not magic

**REST** (Representational State Transfer) is a style for arranging an API so that HTTP's existing machinery does the heavy lifting. The core conventions:

1. **Everything is a resource with a URL.** Nouns, usually plural: `/users`, `/users/42`, `/users/42/orders`.
2. **The HTTP method supplies the verb.** No `/getUser` or `/deleteUser` paths — the resource stays put and the method changes:

```
GET    /users        -> list users
POST   /users        -> create a user (body: the new user's data)
GET    /users/42     -> fetch user 42
PUT    /users/42     -> replace user 42
PATCH  /users/42     -> update part of user 42
DELETE /users/42     -> remove user 42
```

3. **Status codes carry the outcome.** `200` fetched, `201` created, `204` deleted-nothing-to-say, `400` your JSON was malformed, `401/403` identity problems, `404` no such resource, `422` well-formed but invalid data, `429` rate limited, `500` our bug.
4. **Statelessness.** Each request carries everything needed (auth token, parameters); the server keeps no per-client conversation state — Chapter 4's principle, now a design virtue enabling any server to handle any request.

Common supporting conventions you'll meet immediately in the wild:

- **Query parameters for filtering/paging**: `GET /users?role=admin&page=2&per_page=50`. Large collections are always **paginated** — served in chunks with links or counts to fetch more.
- **Auth via headers**: `Authorization: Bearer <token>` or an `X-Api-Key` (Chapter 7). Public read-only APIs may need none.
- **Rate limiting**: servers cap your requests-per-hour and report status in headers like `X-RateLimit-Remaining`; exceed it and you get `429`.
- **Versioning**: APIs evolve, but old clients must not break, so versions are pinned — most visibly in the path (`/v1/users`, `/v2/users`), sometimes in headers (`Accept: application/vnd.github+json` plus a version header). Rule of thumb: adding a field is safe; renaming or removing one breaks clients, hence a new version.

"RESTful" is a spectrum, not a certification — real APIs follow these conventions to varying degrees. The conventions matter because they make APIs *guessable*: once you know an API has `/articles`, you can often predict the rest.

### The two clients you'll use

- **curl** — exploration and debugging from the terminal. You already speak it.
- **`fetch`** — JavaScript's built-in HTTP client, how web pages call APIs:

```js
const resp = await fetch("https://api.example.com/users/42");
const data = await resp.json();   // parsed JSON -> a JS object
```

One browser-only wrinkle to file away: **CORS** (Cross-Origin Resource Sharing). Browsers block page JavaScript from reading responses from a *different* site unless that site's response headers (`Access-Control-Allow-Origin`) permit it. It protects users' logged-in sessions from malicious pages. curl and Python are unaffected — CORS is enforced *by browsers only*. When a fetch fails with a CORS error but curl works, nothing is "down"; the API simply doesn't allow browser pages from your origin.

## Hands-On Examples

All of these use free, no-key public APIs.

### 1. First API call

```powershell
curl.exe -s https://api.github.com/users/octocat
```

Expected: a JSON object — `"login": "octocat"`, `"public_repos": ...`, and note the fields that are *URLs to other resources* (`followers_url`, `repos_url`): the API telling you where to go next. Add `-i` and confirm `content-type: application/json`.

### 2. Read REST in the URL structure

```powershell
curl.exe -s https://api.github.com/users/octocat/repos?per_page=3
```

Expected: a JSON *array* of 3 repo objects. Dissect the URL like a sentence: collection (`users`) → item (`octocat`) → sub-collection (`repos`) → paging parameter. Now guess-and-check the pattern on a repo: `https://api.github.com/repos/microsoft/vscode` — guessability is REST working as intended.

### 3. Meet status codes and rate limits

```powershell
curl.exe -i -s https://api.github.com/users/this-user-hopefully-does-not-exist-12345 | Select-Object -First 15
```

Expected: `404` with a JSON error body (`"message": "Not Found"`) — errors are data too. Then inspect your rate budget:

```powershell
curl.exe -s -i https://api.github.com/zen | Select-String "ratelimit"
```

Expected: headers like `x-ratelimit-limit: 60`, `x-ratelimit-remaining: 57` — unauthenticated callers get 60/hour. Exhaust it and you'd see `403`/`429` responses: rate limiting in person.

### 4. A different API, same grammar

The US National Weather Service (no key needed; requires a User-Agent):

```powershell
curl.exe -s -H "User-Agent: learning-track (you@example.com)" "https://api.weather.gov/points/40.7128,-74.006"
```

Expected: JSON about that location including a `forecast` URL inside `properties`. Follow it (copy the URL from the response) to get an actual forecast — an API designed as linked resources. Notice you learned this API's shape in two requests without reading a manual.

### 5. Same API from JavaScript

On any page (or a local page of yours), open the DevTools Console:

```js
const r = await fetch("https://api.github.com/users/octocat");
console.log(r.status);            // 200
const user = await r.json();
console.log(user.name, user.public_repos);
```

Then trigger CORS deliberately: from the console on `https://example.com`, run `await fetch("https://www.google.com")` — expected: a CORS error in red. Same request from curl succeeds. Write down the lesson: the *browser* refused to share the response with the page, the network worked fine.

### 6. POST JSON to an API

```powershell
curl.exe -s -X POST https://httpbin.org/post -H "Content-Type: application/json" -d "{\"title\":\"hello\",\"done\":false}"
```

Expected: your object echoed under `"json"`. This is the exact shape of every "create" call you'll ever make; only the URL and the auth header change.

## Common Misconceptions

- **"APIs are a separate technology from the web."** A REST API call is a plain HTTP request — same DNS, TCP, TLS, methods, headers, status codes. If you did Chapters 2–5, you already know how APIs travel.
- **"REST is a protocol / standard I must implement exactly."** REST is a set of conventions of varying strictness. HTTP is the protocol; REST is table manners.
- **"JSON is JavaScript."** JSON is a language-independent text format that merely *borrowed* JS syntax. Python, Rust, and Excel parse it equally happily; conversely, valid JS object literals (single quotes, trailing commas) are often *invalid* JSON.
- **"A 404/500 from an API means my code crashed."** Error responses are normal, designed API behavior — read the status code and the error body; well-built APIs tell you exactly what went wrong. Robust client code *expects* them.
- **"CORS errors mean the API is down or blocking me personally."** CORS is a browser-only policy protecting users; the request may even have succeeded server-side. Test with curl to separate network reality from browser policy.
- **"If I can see data in the browser, I may hammer the API for it."** Public ≠ unlimited. Rate limits, API keys, and terms of service exist; polite clients identify themselves (User-Agent), cache responses, and back off on 429.

## Practice Exercises

1. **API safari.** Pick any two no-key public APIs (candidates: `api.github.com`, `api.weather.gov`, `pokeapi.co`, `restcountries.com`, `open-meteo.com`). For each, make at least four distinct GET requests with curl, saving commands and trimmed responses. Identify: base URL, how resources nest, pagination mechanism, and content-type.
2. **REST design on paper.** Design (URLs + methods + status codes only, no code) a REST API for a recipe-box app: recipes, ingredients per recipe, and starring a recipe. Cover list/fetch/create/update/delete, one filtered listing, and what status each operation returns on success and on its most likely failure.
3. **Error atlas.** Using curl against real APIs, deliberately provoke and capture four different failure classes: a 404 (bad resource), a 400 or 422 (bad request body — try malformed JSON at `httpbin.org/post` variants or a real API), a 401/403 (call an endpoint that needs auth without it — e.g. `api.github.com/user`), and a rate-limit view (show the `x-ratelimit-*` headers counting down). For each: the exact command, status line, and error body.
4. **fetch mini-dashboard.** Build a local HTML page (your JS track skills) that fetches a GitHub user's profile with `fetch` when you type a username, and displays name, avatar (`avatar_url` in an `<img>`), and repo count. Handle the 404 case with a friendly message — check `resp.status` before parsing.
5. **Versioning field notes.** Investigate how two real APIs handle versioning (look at GitHub's `X-GitHub-Api-Version` header docs vs any `/v1/`-in-path API, e.g. `pokeapi.co/api/v2/`). Write a short comparison: where the version lives, what happens if you omit it, and one advantage of each placement.
