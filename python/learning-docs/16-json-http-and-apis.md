# Chapter 16: Working with JSON, HTTP Requests & APIs

## Overview

Modern software talks over the web. Your future back-end apps will *serve* data to browsers and mobile apps, and *consume* data from other services — weather providers, payment processors, databases-as-a-service. The near-universal format for this data is **JSON**, and the protocol is **HTTP**. This chapter teaches both from the consumer side: parsing and producing JSON with the standard `json` module, and making HTTP requests with the beloved third-party `requests` library.

Everything here is directly reused in Chapter 18 (Flask — the *server* side of the same conversation) and the weather-app project.

## Definitions & Explanations

**JSON (JavaScript Object Notation)** — a text format for structured data. It maps almost 1:1 onto Python types:

| JSON | Python |
|---|---|
| object `{"a": 1}` | dict |
| array `[1, 2]` | list |
| string `"hi"` (double quotes only!) | str |
| number `3` / `3.5` | int / float |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

JSON has **no** tuples, sets, comments, or trailing commas; keys must be strings.

**The `json` module** — four functions, named by a scheme: `s` = string, no-`s` = file:

- `json.dumps(obj)` — Python → JSON **s**tring ("dump string")
- `json.loads(text)` — JSON string → Python ("load string")
- `json.dump(obj, file)` — Python → open file
- `json.load(file)` — open file → Python

`json.dumps(obj, indent=2)` pretty-prints — use it for human-readable saved files and debugging. Malformed JSON raises `json.JSONDecodeError` (a subclass of `ValueError`).

**HTTP in one paragraph** — a client sends a **request** to a URL with a **method** (`GET` = fetch data, `POST` = send/create data, `PUT/PATCH` = update, `DELETE` = remove); the server sends back a **response** containing a **status code** and a **body**. Status codes to know: `200` OK, `201` Created, `301/302` redirect, `400` bad request, `401` unauthorized, `403` forbidden, `404` not found, `429` too many requests (slow down!), `500` server error. Rule of thumb: 2xx you succeeded, 4xx *you* messed up, 5xx *they* messed up.

**API (Application Programming Interface)** — here meaning a *web API*: a set of URLs (**endpoints**) a server exposes for programs. A **REST**-style API maps methods+URLs onto resources: `GET /users/42` fetches user 42; `POST /users` creates one. Responses are almost always JSON.

**Query parameters** — key=value pairs after `?` in a URL (`?city=London&units=metric`) — the standard way to pass options to a `GET` endpoint.

**API keys** — many APIs require a token identifying you, passed as a query parameter or a header. Treat keys like passwords: don't hardcode them into shared code — read them from an environment variable (`os.environ`) or an untracked config file.

**The `requests` library** — the de-facto standard HTTP client (`pip install requests` — in a venv, per Chapter 10!):

- `r = requests.get(url, params={...}, timeout=10)`
- `r.status_code`, `r.ok` (True for < 400), `r.raise_for_status()` (raise on 4xx/5xx)
- `r.json()` — parse the JSON body straight into Python objects
- `r.text` — the raw body as a string
- `requests.post(url, json={...})` — send a JSON body
- Network failures raise `requests.exceptions.RequestException` (base for `ConnectionError`, `Timeout`, ...)

**Always pass `timeout=`** — without it a hung server hangs your program forever.

## Code Examples

### JSON round-trips

```python
import json

profile = {
    "name": "Ada",
    "skills": ["python", "sql"],
    "years": 3,
    "remote": True,
    "manager": None,
}

text = json.dumps(profile)                 # Python -> JSON string
print(text)                                # {"name": "Ada", ..., "manager": null}
print(type(text))                          # <class 'str'>

restored = json.loads(text)                # JSON string -> Python
print(restored["skills"][0])               # python
print(restored == profile)                 # True — a faithful round-trip

print(json.dumps(profile, indent=2))       # pretty, for humans
```

### Saving and loading app data (the persistence pattern)

```python
import json
from pathlib import Path

DATA_FILE = Path(__file__).parent / "contacts.json"

def load_contacts():
    """Return the saved list, or [] on first run / corrupt file."""
    try:
        with open(DATA_FILE, encoding="utf-8") as f:
            return json.load(f)
    except FileNotFoundError:
        return []
    except json.JSONDecodeError:
        print("Warning: contacts.json is corrupt — starting fresh.")
        return []

def save_contacts(contacts):
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(contacts, f, indent=2)

contacts = load_contacts()
contacts.append({"name": "Grace", "email": "grace@navy.mil"})
save_contacts(contacts)
print(f"{len(contacts)} contact(s) on disk")
```

This load/modify/save trio is the backbone of several track projects.

### A first real request

```python
# pip install requests   (inside your venv!)
import requests

# GitHub's public API — no key needed
r = requests.get("https://api.github.com/users/octocat", timeout=10)

print(r.status_code)               # 200
data = r.json()                    # dict, straight from the JSON body
print(data["name"])                # The Octocat
print(data["public_repos"])        # e.g. 8
print(sorted(data.keys())[:5])     # explore what fields exist
```

### Query parameters and drilling into nested JSON

```python
import requests

# Open-Meteo: free weather API, no key required
params = {
    "latitude": 51.51,          # London
    "longitude": -0.13,
    "current_weather": True,
}
r = requests.get("https://api.open-meteo.com/v1/forecast",
                 params=params, timeout=10)
r.raise_for_status()             # explode clearly on 4xx/5xx

data = r.json()
current = data["current_weather"]            # nested dict
print(f"Temp: {current['temperature']}°C, wind {current['windspeed']} km/h")
```

`params=` builds and encodes the query string for you — never glue URLs together with f-strings and hope the escaping works out.

### Robust request handling (the production shape)

```python
import requests

def fetch_json(url, params=None):
    """GET JSON with proper error handling. Returns None on failure."""
    try:
        r = requests.get(url, params=params, timeout=10)
        r.raise_for_status()
        return r.json()
    except requests.exceptions.Timeout:
        print("The server took too long to respond.")
    except requests.exceptions.ConnectionError:
        print("Network problem — are you online?")
    except requests.exceptions.HTTPError as e:
        print(f"Server said no: {e.response.status_code}")
    except ValueError:                      # .json() on a non-JSON body
        print("Response wasn't valid JSON.")
    return None

data = fetch_json("https://api.github.com/users/octocat")
if data:
    print(data["name"])
```

### POSTing JSON

```python
import requests

# httpbin.org echoes whatever you send — perfect for practice
payload = {"task": "learn APIs", "done": False}
r = requests.post("https://httpbin.org/post", json=payload, timeout=10)
#                                              ^^^^ serializes + sets Content-Type header

echoed = r.json()
print(echoed["json"])              # {'task': 'learn APIs', 'done': False}
```

### API keys done safely

```python
import os
import requests

API_KEY = os.environ.get("WEATHER_API_KEY")     # set in your shell, not in code
if not API_KEY:
    raise SystemExit("Set WEATHER_API_KEY first (e.g. $env:WEATHER_API_KEY='...' in PowerShell)")

r = requests.get("https://api.example.com/data",
                 params={"appid": API_KEY, "q": "London"}, timeout=10)
```

## Common Pitfalls

**1. Confusing `dumps/loads` direction** — mnemonic: **dump** your Python objects *out* to JSON; **load** JSON *in* to Python. The trailing `s` means you're working with a **s**tring instead of a file.

**2. JSON isn't Python** — single quotes, `True/False/None`, trailing commas, and comments are all invalid JSON. If you hand-write a `.json` file and get `JSONDecodeError`, check for exactly these. Conversely `str(my_dict)` is *not* JSON — use `json.dumps`.

**3. Not checking the status code**

```python
data = requests.get(url, timeout=10).json()    # a 404 error page may parse as JSON
                                               # ... or crash .json() entirely
r = requests.get(url, timeout=10)              # RIGHT
r.raise_for_status()
data = r.json()
```

**4. No timeout** — `requests.get(url)` can hang forever on a dead server. Always `timeout=10` (or similar).

**5. Assuming response shape** — APIs change, fields go missing, error responses have different shapes than success responses. Use `.get()` with defaults for optional fields, and print the raw `r.json()` while developing to *see* the actual structure instead of guessing.

**6. Hardcoding secrets** — an API key pasted in code ends up in screenshots and Git history forever. Environment variables from day one.

**7. Hammering an API in a loop** — a tight loop of requests gets you rate-limited (`429`) or banned. Cache what you can, and `time.sleep()` between calls when looping.

**8. Serializing non-JSON types** — `json.dumps({1, 2, 3})` or dumping a `datetime`/custom class raises `TypeError`. Convert first (set→list, datetime→`isoformat()` string).

## Practice Exercises

1. **Round-trip drill.** Build a nested Python structure (dict containing a list of dicts) describing three books. Dump it to a pretty-printed string, write that to `books.json` using file-mode `dump`, read it back with `load`, and verify equality with `==`. Then hand-edit the file to break the JSON and confirm your loader's error handling catches it.
2. **Settings module.** Write `settings.py` exposing `get(key, default)` and `set(key, value)` backed by a `settings.json` file (auto-created on first use). Reuse the load/modify/save pattern; import and exercise it from another script.
3. **GitHub explorer.** Ask the user for a GitHub username and print their name, bio, follower count, and five most recently updated repos (endpoint: `/users/<name>/repos`, params `sort=updated&per_page=5`). Handle unknown users (404) with a clean message, not a traceback.
4. **Currency check.** Use the free `https://open.er-api.com/v6/latest/USD` endpoint to fetch exchange rates. Let the user type a currency code and an amount, and print the conversion — handling unknown codes and network failures gracefully.
5. **Response autopsy.** Request `https://httpbin.org/status/404`, `https://httpbin.org/delay/3` (with `timeout=1`), and an obviously bogus domain. For each, write down which exception (or status) you predicted vs. what actually happened, and make one `fetch` function that survives all three.
