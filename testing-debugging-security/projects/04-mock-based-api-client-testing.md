# Project 4: Mock-Based Testing of an API Client

## Description

Build and thoroughly test a **weather dashboard client** — a module that talks to an external HTTP API — *without your test suite ever touching the network*. This is the test-doubles project: stubs for canned responses, mocks for interaction checks, fakes for a cache, and fake timers for retry/expiry logic. You'll design for injection (Chapter 3) so most tests need no patching at all, and you'll finish with a thin optional integration layer against a real free API to see exactly what unit tests with doubles can and cannot catch. One language, your choice — the JS version pairs naturally with `vi.useFakeTimers()`; the Python version exercises `unittest.mock` deeply. 

The client module (`weatherclient`) spec:

1. `get_current(city)` — fetch current weather, returning a clean dict/object: `{city, temp_c, condition, fetched_at}` (translate the raw API shape; don't leak it).
2. Error mapping — HTTP 404 → `CityNotFoundError`; 401 → `AuthError` (bad API key); 429 or 5xx → `TemporarilyUnavailableError`; malformed/missing JSON fields → `BadResponseError`. No raw HTTP exceptions escape the module.
3. Retry — on 429/5xx only, retry up to 3 times with waits of 1s, 2s, 4s between attempts; then give up with `TemporarilyUnavailableError`. Never retry 404/401.
4. Cache — successful results cached per city for 10 minutes; within that window no network call is made; after it, a fresh fetch. Cache must be per-client-instance, not global.
5. `get_forecast_summary(city, days)` — fetches a forecast list and computes min/max/mean temps; `days` must be 1–7.
6. API key — read from an environment variable at client construction; missing key raises immediately with a helpful message (Chapter 12 habit).

## Difficulty

**Intermediate-to-advanced.** Estimated effort: 8–12 hours.

## Chapters used

- Chapter 3 — Designing Testable Code (inject the HTTP function and the clock)
- Chapter 5 — Test Doubles (the core of the project)
- Chapter 4 — Edge Cases & Test Design (error-path tables)
- Chapter 6 — Integration & E2E (the thin real-API layer at the end)

## Requirements checklist

- [ ] Client accepts its HTTP function and clock/sleep as constructor parameters with real defaults — most tests inject doubles with **zero patching**
- [ ] A reusable stub-response builder in the test helpers (status + JSON body in, response double out)
- [ ] Happy path tested: raw API JSON fixture → clean output shape, including field translation
- [ ] Error mapping tested with a parameterized table covering 404, 401, 429, 500, 503, and malformed-JSON — each asserting the *specific* custom error type
- [ ] Retry tested with a scripted double ("fail 500, fail 500, succeed") asserting: final success, exact call count, and the waits requested (1, 2, 4) — via fake timers or an injected recorded `sleep`; the suite itself runs in milliseconds, never actually sleeping
- [ ] "Never retry 404/401" has explicit tests asserting call count == 1
- [ ] Cache behavior tested with a controllable clock: hit within 10 min (network called once), miss after 10 min (called twice), and per-city independence
- [ ] Two-instances test proving the cache isn't shared global state
- [ ] Interaction test: the correct URL and API key are sent (assert on the double's recorded call args)
- [ ] Missing-API-key construction failure tested
- [ ] All custom exceptions defined in the module; no `requests`/`fetch` errors leak (test this)
- [ ] Optional-but-graded honestly: 2–3 integration tests against a real free API (e.g., Open-Meteo, no key needed), clearly separated/markable so the unit suite runs offline; note in comments one thing these caught or *could* catch that your stubs cannot
- [ ] `DESIGN.md` (half a page): which tests are state-based vs interaction-based, and why each interaction assertion earns its place

## Hints

- Design the seams first: if `get_current` receives `http_get` and the cache receives `now()`, nearly every requirement above becomes a plain function call with hand-built doubles — the project is dramatically harder if you patch your way through instead.
- Script sequenced doubles with `side_effect` (a list, in Python) or `mockResolvedValueOnce` chains (Vitest) for the retry choreography.
- For retry waits, the cleanest assertion is an injected `sleep` double that just records its arguments: `assert sleeps == [1, 2, 4]`. Fake timers are the alternative; pick one and be consistent.
- Build one realistic raw-API JSON fixture early and reuse it — drifting hand-typed fixtures across tests is how stub-vs-reality rot (Chapter 5's pitfall) starts inside a single project.
- The over-mocking check from Chapter 5 applies: after the suite is green, refactor an internal detail (e.g., how the cache stores entries) — behavior-level tests should survive untouched. If several break, tighten your assertions' altitude, not the code.

## Stretch goals

- Add a `FakeWeatherAPI` fake (in-memory, settable conditions per city) and rewrite three of your stub-heavy tests against it; compare readability.
- Add rate-limiting on your side (max 30 calls/minute per instance) tested entirely with the fake clock.
- Record a real API response to a JSON file and build a "replay" double from it, keeping fixtures honest by construction.
