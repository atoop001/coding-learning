# Project 4: API Field Guide

## Description

Choose one public, no-authentication (or free-key) REST API and become its cartographer. Through systematic exploration with curl — and later `fetch` — you'll map its resources, conventions, error behavior, pagination, and rate limits, then write the "field guide" you wish its documentation was: a practical, evidence-backed reference with working examples for every claim. You'll finish by building a small HTML+JS page that consumes the API live, proving your guide is accurate enough to build from.

Good candidate APIs (pick one, or propose your own): **PokéAPI** (`pokeapi.co`), **REST Countries** (`restcountries.com`), **Open-Meteo** weather (`open-meteo.com`), **GitHub's REST API** (`api.github.com`), **NASA APIs** (free key), or the **US National Weather Service** (`api.weather.gov`). Choose one whose subject matter you'll enjoy staring at for a few hours.

## Difficulty

**Intermediate–Advanced.** Estimated effort: 5–8 hours.

## Chapters Used

- Chapter 4 (HTTP methods, status codes, headers, curl fluency)
- Chapter 7 (auth headers and tokens — even if your API needs none, you must document its stance)
- Chapter 8 (REST conventions, JSON, pagination, versioning, rate limits, fetch, CORS)
- Chapter 6 (caching headers on API responses — often overlooked and revealing)

## Requirements Checklist

Produce a field guide (`api-field-guide.md`) containing:

- [ ] **Overview**: what the API serves, its base URL, protocol/format (confirm `application/json` with evidence), and where its version lives (path? header? nowhere? — show how you checked)
- [ ] **Resource map**: a diagram or tree of at least five distinct resource types/endpoints and how they nest or link to each other, each with a working example URL
- [ ] **Conventions observed**: plural vs singular nouns, ID formats, casing of JSON keys, how relationships are expressed (nested objects? URLs to follow?) — with one quoted response snippet as evidence for each claim
- [ ] **Query parameters**: at least three documented-by-you parameters (filtering, searching, or shaping), each with a with/without comparison showing the effect
- [ ] **Pagination**: how large collections are chunked (page numbers? offsets? cursors? `Link` headers?), demonstrated by actually walking at least three pages and showing how you knew where the next one was
- [ ] **Error atlas**: real captured responses for at least four distinct failure cases (nonexistent resource, malformed ID, invalid parameter, and one more of your choosing) — status code plus error body for each, and a verdict on how helpful the errors are
- [ ] **Rate limits & politeness**: any rate-limit headers or documented limits, what a 429 would look like, and what identifying headers (User-Agent etc.) the API asks of clients
- [ ] **Caching stance**: `cache-control`/`etag`/`age` headers on a typical response, and what they imply about how aggressively you may re-request
- [ ] **Auth stance**: none / key / token — and, whichever it is, one paragraph on how Chapter 7's models map onto it
- [ ] **The proof-of-guide app**: a local HTML+JS page that uses `fetch` against at least two endpoints from your map, renders real data into the DOM, and handles at least one error case gracefully (bad user input → friendly message, driven by the status code)
- [ ] **CORS note**: whether the API allowed your page's fetch (check the `access-control-allow-origin` header), and what you'd conclude if it hadn't

*(No solution code beyond your own app is expected anywhere — the guide documents the API, not answers.)*

## Hints

- Explore breadth-first: hit the base URL and root endpoints before drilling in — many APIs are self-describing and hand you their own map (follow URLs embedded in responses).
- Keep a running `commands.md` as you go; the guide assembles itself from a good lab notebook.
- Piping curl output to a file (`curl.exe -s ... > out.json`) then pretty-printing with Python (`python -m json.tool out.json`) keeps large responses readable.
- For pagination, deliberately request tiny pages (e.g. `?limit=3` or `per_page=3`) so the mechanics are visible in small responses.
- Provoking good errors is an art: try an ID of `0`, `-1`, `999999999`, `abc`, and an endpoint name that's *almost* right — different wrongness often yields different status codes.
- In your app, check `resp.ok` / `resp.status` before `resp.json()` — building the error path first makes the happy path trivial.
- If your fetch hits a CORS wall, that's a *finding*, not a failure: document it, and fall back to a Node/Python fetch or a different API for the app portion.

## Stretch Goals

- Add a **write-operations appendix**: using `httpbin.org` as a stand-in (since your API is likely read-only), document what POST/PUT/DELETE against your resource map *would* look like — URLs, bodies, expected statuses — following your API's observed conventions.
- Compare your API's design against a second API on three axes (error quality, pagination style, self-describability) in a half-page "design review."
- Measure and chart (even a text table) response times for the same endpoint called 10 times — is the API's caching visible in the timings or `age` headers?
- Package your proof-of-guide app to also work offline-ish: cache the last successful response in `localStorage` and show it (marked as stale) when a fetch fails.
