# Project 6: CLI Weather App

## Description

Build a command-line weather tool that fetches **live data** from a real, free API and presents it beautifully in the terminal: current conditions for any city, a multi-day forecast, and a comparison view for multiple cities. Recent lookups are cached so repeat requests within a few minutes don't hit the network at all.

Using it should feel like a proper CLI utility: `python weather.py London` just works, output is tidy and readable at a glance, and when the network is down or a city doesn't exist, the app says so plainly instead of vomiting a traceback.

Recommended API: **Open-Meteo** (https://open-meteo.com) — free, no API key, includes a geocoding endpoint to turn city names into coordinates. (Any weather API is acceptable; if yours needs a key, it must come from an environment variable.)

## Difficulty & Effort

**Difficulty:** Intermediate
**Estimated effort:** 6–9 hours

## Chapters Used

- `16-json-http-and-apis.md` — the core: `requests`, query params, JSON, error handling
- `10-modules-packages-pip-venv.md` — venv + `pip install requests`, `requirements.txt`, `sys.argv`, module layout
- `12-error-handling-and-exceptions.md` — network failure handling
- `11-file-io-and-paths.md` — the cache file
- `08-dictionaries-and-sets.md` / `09-comprehensions.md` — digging through API responses
- `06-functions.md` — clean separation of fetch / transform / display

## Requirements Checklist

### Setup & structure

- [ ] The project has its own venv and a `requirements.txt` containing `requests`
- [ ] Code is split into modules along seams like: API access, cache, formatting/display, and a `main` entry — with no network code in the display module and no `print` in the API module
- [ ] City name comes from the command line (`python weather.py Tokyo`); with no argument, the app prompts interactively

### Core features

- [ ] City names are resolved to latitude/longitude via a geocoding request; ambiguous names (multiple matches) show the top few with country, letting the user pick
- [ ] **Current conditions**: temperature, wind speed, and a human-readable description of the weather code, displayed with units
- [ ] **Forecast**: the next 5 days with date, min/max temperature, and precipitation, in aligned columns
- [ ] **Compare mode**: `python weather.py London Paris Cairo` (or a menu option) prints a one-line summary per city, sorted warmest first

### Robustness

- [ ] Every request passes a `timeout`; timeouts, connection errors, and HTTP error statuses each produce a *distinct*, friendly message and a non-zero exit code
- [ ] An unknown city ("Atlantis") is a clean "couldn't find that place" — not an IndexError from an empty results list
- [ ] Responses missing expected fields are handled with `.get()`/defaults rather than crashing

### Caching

- [ ] Successful lookups are cached to a JSON file keyed by city, storing the data plus a timestamp
- [ ] A request for a city cached within the last 10 minutes uses the cache (and says so, e.g. "(cached 3 min ago)") instead of calling the API
- [ ] A corrupt cache file is discarded gracefully, never fatal

## Hints

- Explore before you build: in the REPL, make one geocoding call and one forecast call, and `json.dumps(data, indent=2)` the responses. Write your field-picking code against what you *see*, not what you assume.
- Open-Meteo shape: geocode at `https://geocoding-api.open-meteo.com/v1/search?name=London&count=5`; then forecast at `https://api.open-meteo.com/v1/forecast` with `latitude`, `longitude`, and parameters like `current_weather=true` and `daily=temperature_2m_max,temperature_2m_min,precipitation_sum` plus `timezone=auto`. The daily data arrives as *parallel lists* — `zip` is your friend.
- Weather codes are integers; build a dict mapping code → description (the docs list them). `.get(code, "unknown")` covers the ones you didn't map.
- Timestamps for the cache: `time.time()` (seconds since epoch) makes "age in minutes" a subtraction and a division — no date parsing needed.
- The fetch-with-fallback shape from Chapter 16 (`fetch_json` returning `None` on failure) keeps `main` readable: fetch, check, display.
- Design the display functions to take plain dicts/lists your *own* code produced, not raw API responses — a "transform" layer between fetch and display isolates you from API weirdness and makes compare-mode trivial.

## Stretch Goals

- A `--units imperial` flag converting to °F/mph at display time
- ASCII art or emoji per weather condition next to the description
- Hourly view for today, as a compact temperature sparkline built from characters like `▁▂▃▅▇`
- "Favorites" stored in a config JSON: running with no arguments shows all favorites at once
- Wrap it as a proper installed command using a `pyproject.toml` entry point so `weather London` works without typing `python` (research required — that's the point)
