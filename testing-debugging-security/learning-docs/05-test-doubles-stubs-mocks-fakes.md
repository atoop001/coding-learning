# Chapter 5: Test Doubles — Stubs, Mocks & Fakes

## Overview

Real code talks to messy things: web APIs that rate-limit you, clocks that keep moving, databases, email servers. You cannot (and should not) hit the real GitHub API 400 times every time you run your unit tests. **Test doubles** are stand-ins — like stunt doubles — that replace those messy collaborators during a test. This chapter covers the three doubles you'll actually use (stubs, mocks, fakes), how to mock HTTP calls and time in both pytest and Vitest, and — just as important — how to recognize when mocking has gone too far and is testing nothing but your own imagination.

## Definitions & Explanations

**Test double** — umbrella term for any object that replaces a real dependency in a test. The names below describe *what job* the double does; frameworks blur them, but the concepts keep your thinking straight.

**Stub** — a double that *returns canned answers*. You use it to feed your code the input it needs: "when asked for the weather, say 'sunny'." Stubs answer questions; you then assert on *your code's output*.

**Mock** — a double that *records how it was called* so you can assert on the interaction itself: "verify that `send_email` was called exactly once, with this address." Mocks answer the question "did my code talk to its collaborator correctly?" Use them when the *call itself is the point* (sending an email, charging a card) — there's no return value to check.

**Fake** — a *working but simplified* implementation: an in-memory dictionary posing as a database, a fake filesystem in RAM. Fakes behave realistically across many calls (you can save then load), which makes them great for tests that involve sequences of operations.

**Spy** — a real object (or thin wrapper) that additionally records calls. Vitest's `vi.spyOn` and `unittest.mock.patch` produce spies/mocks; the distinction rarely matters in practice.

**Patching / monkeypatching** — temporarily replacing an attribute (a function, a class, `requests.get`) for the duration of one test, then restoring it. This is how you install a double when you *can't* pass it as a parameter. Chapter 3's lesson still stands: designing for injection is better than patching — patching is the fallback for code at the edges or code you don't control.

**State-based vs interaction-based testing** — asserting on *results* (state) vs asserting on *calls made* (interaction). Prefer state-based: it survives refactoring. Interaction assertions lock in *how* the code works, so they should be reserved for interactions that are the actual requirement.

**The over-mocking trap** — if a test mocks everything the code touches, the test just confirms that the code calls the mocks the way the test says it does — a tautology. It will stay green while the real system burns. Symptoms and cures appear in the pitfalls section.

## Code Examples

### Stubbing an HTTP API in Python

```python
# weather.py — the code under test
import requests

def outfit_advice(city):
    """Suggest an outfit based on live weather. Talks to a real API."""
    resp = requests.get(f"https://api.example.com/weather/{city}", timeout=5)
    resp.raise_for_status()
    temp_c = resp.json()["temp_c"]
    if temp_c < 5:
        return "heavy coat"
    if temp_c < 18:
        return "light jacket"
    return "t-shirt"
```

```python
# test_weather.py — no network involved. We patch requests.get with a stub.
from unittest.mock import patch, Mock
import pytest
import requests
from weather import outfit_advice


def make_response(json_body, status=200):
    """Build a stub response object with just the attributes our code uses."""
    resp = Mock()
    resp.json.return_value = json_body
    if status >= 400:
        resp.raise_for_status.side_effect = requests.HTTPError(f"{status}")
    else:
        resp.raise_for_status.return_value = None
    return resp


# patch swaps weather.requests.get for a Mock during this test only.
# IMPORTANT: patch where the name is USED ("weather.requests.get"),
# not where it is defined ("requests.get") — the #1 patching mistake.
@patch("weather.requests.get")
def test_freezing_weather_means_heavy_coat(fake_get):
    fake_get.return_value = make_response({"temp_c": -3})   # canned answer

    assert outfit_advice("oslo") == "heavy coat"            # state-based assert


@patch("weather.requests.get")
def test_api_failure_propagates_as_http_error(fake_get):
    fake_get.return_value = make_response({}, status=500)

    with pytest.raises(requests.HTTPError):                 # error path!
        outfit_advice("atlantis")
```

### Mocking (interaction-based) when the call is the point

```python
# alerts.py
def check_and_alert(temp_c, send_sms):
    """send_sms is injected (Chapter 3!) — no patching needed."""
    if temp_c > 40:
        send_sms("Heat warning: stay hydrated")

# test_alerts.py
from unittest.mock import Mock
from alerts import check_and_alert

def test_extreme_heat_sends_exactly_one_sms():
    send_sms = Mock()

    check_and_alert(45, send_sms)

    # The SMS *is* the behavior — so we assert on the interaction.
    send_sms.assert_called_once_with("Heat warning: stay hydrated")

def test_mild_weather_sends_nothing():
    send_sms = Mock()
    check_and_alert(20, send_sms)
    send_sms.assert_not_called()          # asserting absence matters too
```

### Stubbing fetch and controlling time in Vitest

```javascript
// session.js — code under test
export async function fetchUser(id, fetchFn = fetch) {
  const resp = await fetchFn(`https://api.example.com/users/${id}`);
  if (!resp.ok) throw new Error(`HTTP ${resp.status}`);
  return resp.json();
}

export function sessionAgeMinutes(startedAtMs, nowMs = Date.now()) {
  return Math.floor((nowMs - startedAtMs) / 60000);
}
```

```javascript
// session.test.js
import { describe, it, expect, vi, afterEach } from "vitest";
import { fetchUser, sessionAgeMinutes } from "./session.js";

describe("fetchUser", () => {
  it("returns parsed user data on success", async () => {
    // vi.fn() creates a mock function; mockResolvedValue = async canned answer
    const fakeFetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ id: 7, name: "Ada" }),
    });

    const user = await fetchUser(7, fakeFetch);

    expect(user.name).toBe("Ada");
    expect(fakeFetch).toHaveBeenCalledWith("https://api.example.com/users/7");
  });

  it("throws a useful error on a 404", async () => {
    const fakeFetch = vi.fn().mockResolvedValue({ ok: false, status: 404 });

    await expect(fetchUser(999, fakeFetch)).rejects.toThrow("HTTP 404");
  });
});

describe("sessionAgeMinutes with fake timers", () => {
  afterEach(() => vi.useRealTimers());    // ALWAYS restore — see pitfalls

  it("computes age against a frozen clock", () => {
    vi.useFakeTimers();
    vi.setSystemTime(new Date("2026-01-01T10:30:00Z"));   // freeze "now"

    const started = new Date("2026-01-01T10:00:00Z").getTime();

    expect(sessionAgeMinutes(started)).toBe(30);
  });
});
```

### A fake: in-memory repository

```python
# A fake is a real, working implementation — just simplified.
class FakeUserRepo:
    """Stands in for a database table. Supports the same operations."""
    def __init__(self):
        self._rows = {}
    def save(self, user_id, data):
        self._rows[user_id] = dict(data)
    def load(self, user_id):
        if user_id not in self._rows:
            raise KeyError(user_id)
        return dict(self._rows[user_id])

def test_renaming_a_user_persists(  ):
    repo = FakeUserRepo()
    repo.save(1, {"name": "Ada"})

    user = repo.load(1)
    user["name"] = "Ada Lovelace"
    repo.save(1, user)

    assert repo.load(1)["name"] == "Ada Lovelace"   # multi-step realism
```

## Common Pitfalls

- **Patching where a name is defined instead of where it's used.** `@patch("requests.get")` often silently fails to intercept `weather.py`'s call. Correction: patch `"weather.requests.get"` — the lookup path *your module* uses.
- **Over-mocking.** If your test mocks five collaborators and asserts six specific calls, refactoring the code (without changing behavior!) breaks the test. That's a test testing implementation, not behavior. Correction: mock only true boundaries (network, time, randomness, hardware); use real objects or fakes for your own classes.
- **Mocking the thing you're supposed to be testing.** If `outfit_advice` is mocked, the test proves nothing. Correction: doubles replace *dependencies*, never the unit under test.
- **Forgetting to restore fake timers or patches.** A leaked fake clock makes later tests fail mysteriously. Correction: use decorators/context managers (auto-restore), or `afterEach(() => vi.useRealTimers())` / `vi.restoreAllMocks()`.
- **Stubs drifting from reality.** The real API changed its field from `temp_c` to `tempC`; your stub still says `temp_c`; unit tests stay green while production breaks. Correction: keep a small number of integration tests against the real thing (Chapter 6), and update stubs from real recorded responses.
- **Asserting interactions when a state assertion exists.** `assert repo.save was called` is weaker than `assert repo.load(1) == expected`. Correction: prefer checking outcomes; check calls only when the call *is* the outcome.

## Practice Exercises

1. Write a Python function `get_top_story(fetch_json)` that calls an injected function to retrieve `{"stories": [...]}` and returns the title of the highest-scored story. Test it with stubs for: normal data, empty story list, and the fetch function raising a network error.
2. Rewrite the `outfit_advice` example so `requests.get` is injected as a parameter instead of patched, and convert the tests. Compare both versions: which test file is shorter and clearer?
3. In Vitest, write and test a `retry(fn, times)` helper that retries a failing async function. Use `vi.fn()` with `mockRejectedValueOnce`/`mockResolvedValueOnce` to script "fail, fail, succeed," and assert the call count.
4. Build a `FakeClock` class (Python or JS) with `now()` and `advance(seconds)`, then use it to test a rate-limiter that allows at most 3 calls per 60 seconds.
5. Deliberately write an *over-mocked* test for one of your own functions — mock every collaborator, assert every call. Then refactor the function's internals without changing its behavior, and observe the test break. Rewrite the test state-based so it survives the same refactor. Write two sentences on what you learned.
