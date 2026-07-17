# Project 6: Weather Dashboard (Fetch + Async)

## Description

Build a weather dashboard powered by a real API. The user types a city name; the app looks up its coordinates, fetches current weather and a multi-day forecast, and renders a clean dashboard: current temperature, conditions, wind, plus a row of forecast cards (day, high/low, weather icon/emoji). Recent searches appear as clickable chips for one-click re-checks.

Use **Open-Meteo** (`https://open-meteo.com`) — it's free, needs no API key, and allows browser requests. You'll chain two endpoints: the geocoding API (`https://geocoding-api.open-meteo.com/v1/search?name=...`) to turn "Berlin" into latitude/longitude, then the forecast API (`https://api.open-meteo.com/v1/forecast?latitude=...&longitude=...&current_weather=true&daily=temperature_2m_max,temperature_2m_min,weathercode&timezone=auto`).

The heart of this project is handling the *three states of every fetch* — loading, success, error — so the app feels trustworthy: a spinner while fetching, a helpful message for unknown cities, and graceful recovery from network failure (test by going offline!).

## Difficulty & Effort

- **Difficulty:** Intermediate+
- **Estimated effort:** 7–10 hours

## Chapters Used

- `11-events.md` — search form handling
- `12-error-handling.md` — HTTP and not-found errors, `try/catch`
- `15-asynchronous-javascript.md` — `async/await`, sequencing vs. parallel
- `16-fetch-and-apis.md` — the whole chapter, applied
- `07-arrays-and-array-methods.md` + `08-objects.md` — reshaping API responses
- `10-dom-manipulation.md` — rendering the dashboard

## Requirements Checklist

- [ ] Search form (input + submit, `preventDefault`) that kicks off the lookup; empty/whitespace queries are ignored
- [ ] Two-step async flow with `async/await`: geocode the city name, then fetch the forecast with the returned coordinates (this is inherently sequential — the second call needs the first's result)
- [ ] Every fetch checks `response.ok` and throws on failure; all await-ing code sits inside `try/catch`
- [ ] "City not found" (geocoding returns no results) is treated as an *expected* outcome with its own friendly message — distinct from network/server errors
- [ ] A visible **loading state** (spinner or "Loading…") appears during fetches and always clears afterwards — use `finally`
- [ ] Current weather panel: city name (as returned by the API, with country), temperature, wind speed, and a text/emoji description mapped from the numeric `weathercode`
- [ ] Forecast row: at least 5 day-cards with weekday name, high, low, and condition emoji — built by `map`ping the API's parallel arrays into an array of day objects first
- [ ] API data is reshaped into your own clean objects (e.g., `{ city, current: {...}, days: [...] }`) in a function separate from rendering — rendering never digs into raw API paths
- [ ] Recent searches (last 5, no duplicates) rendered as clickable chips that re-run the search; stored in an array
- [ ] Simulated failure test: temporarily break the URL, confirm the user sees a readable error (not a blank page or console-only stack trace), then restore it
- [ ] While a request is in flight, submitting again doesn't fire a duplicate request (disable the button or guard with a flag)

## Hints

- Explore the API in the browser first: paste the geocoding URL with `?name=berlin` into a tab and read the JSON shape *before* writing code. Note that results live in `data.results` — which is **absent** (not an empty array) for unknown cities; `?.` and `??` are your friends.
- Open-Meteo's daily forecast comes as parallel arrays (`daily.time[]`, `daily.temperature_2m_max[]`...). `daily.time.map((date, i) => ({ date, max: daily.temperature_2m_max[i], ... }))` zips them into sane objects.
- Weathercode → description: build a plain object lookup (`{ 0: ["Clear", "☀️"], 61: ["Rain", "🌧️"], ... }`); the docs list the codes. A `??`-fallback covers unmapped codes.
- Weekday names from dates: `new Date(dateString).toLocaleDateString(undefined, { weekday: "short" })`.
- `URL` + `searchParams` (Chapter 16, example 6) keeps those long forecast URLs readable and correctly encoded.
- Write `getWeatherFor(city)` as a pure async data function returning your clean object (throwing on failure), and keep DOM work in the caller — you can then test the data layer straight from the console with `await getWeatherFor("Tokyo")`.

## Stretch Goals

- **Multiple matches:** when geocoding returns several cities (Springfield!), show a chooser list instead of picking the first blindly.
- **Unit toggle:** °C/°F switch that re-renders without re-fetching.
- **Geolocation:** a "use my location" button via `navigator.geolocation`, feeding coordinates straight to the forecast call.
- **Persistent recents:** keep recent searches in `localStorage`.
- **Parallel dashboards:** let the user pin 2–3 cities and load them concurrently with `Promise.allSettled`, so one failing city doesn't break the rest.
