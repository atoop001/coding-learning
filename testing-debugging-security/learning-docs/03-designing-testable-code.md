# Chapter 3: Designing Testable Code

## Overview

Sometimes you sit down to test a function and the test flows out in five lines. Other times you can't even figure out how to *call* the thing without a database, a network connection, and today's date being a Tuesday. The difference is not testing skill — it is *code design*. This chapter teaches the handful of design habits that make code easy to test: pure functions, separating logic from side effects, passing dependencies in instead of reaching out for them ("dependency injection lite"), and creating *seams* where tests can take control. These habits also happen to make code easier to reuse, debug, and reason about — testability is a quality signal, not just a convenience.

## Definitions & Explanations

**Pure function** — a function whose output depends *only* on its inputs, and which changes nothing outside itself. `slugify("Hello World")` always returns `"hello-world"`; it doesn't read files, print, mutate globals, or care what time it is. Pure functions are trivially testable: arrange inputs, act, assert on the return value. Done.

**Side effect** — anything a function does besides computing its return value: writing files, printing, sending HTTP requests, mutating a global or an argument, reading the clock or random numbers. Side effects aren't bad — a program with no side effects does nothing useful — but they should be *pushed to the edges* of your program, leaving a pure core.

**Functional core, imperative shell** — the architecture that falls out of that idea: a core of pure logic (heavily unit tested) wrapped by a thin shell that does I/O (lightly tested, or covered by integration tests). The shell gathers inputs, calls the core, and acts on the result.

**Dependency** — anything a function needs that isn't a plain argument: the current time, a random generator, a database, an HTTP client, a config file. Hidden dependencies (reached via imports or globals) are what make code untestable.

**Dependency injection (DI), lite version** — instead of a function *reaching out* for its dependency, the caller *passes it in* — often just as a parameter with a sensible default. No frameworks needed. If `is_expired(deadline)` receives `now` as a parameter, tests can pass any time they like; if it calls `datetime.now()` internally, tests are at the mercy of the wall clock.

**Seam** — a place where you can change a program's behavior without editing the code itself. A parameter with a default is a seam. So is a constructor argument, or a function stored where a test can swap it. Testable code is code with seams in the right places.

**Coupling** — how entangled two pieces of code are. A function that computes a report *and* prints it *and* emails it is coupled to the console and the mail server; you can't test the computation without triggering the rest.

## Code Examples

### Untestable → testable: separating logic from I/O (Python)

```python
# BEFORE: logic, input, and output are welded together.
# To test this you'd have to fake keyboard input and capture stdout. Painful.
def grade_report():
    scores = []
    while True:
        raw = input("score (blank to finish): ")
        if not raw:
            break
        scores.append(float(raw))
    avg = sum(scores) / len(scores)
    if avg >= 90:
        print("Grade: A")
    elif avg >= 80:
        print("Grade: B")
    else:
        print("Grade: C or below")
```

```python
# AFTER: a pure core anyone can test in one line, plus a thin shell.

def average(scores):                 # pure
    if not scores:
        raise ValueError("no scores given")
    return sum(scores) / len(scores)

def letter_grade(avg):               # pure
    if avg >= 90:
        return "A"
    if avg >= 80:
        return "B"
    return "C or below"

def grade_report():                  # imperative shell — thin, boring, low-risk
    scores = []
    while True:
        raw = input("score (blank to finish): ")
        if not raw:
            break
        scores.append(float(raw))
    print(f"Grade: {letter_grade(average(scores))}")
```

```python
# test_grades.py — the core is now trivially testable
import pytest
from grades import average, letter_grade

def test_average_of_two_scores():
    assert average([80, 100]) == 90

def test_average_rejects_empty_list():
    with pytest.raises(ValueError):
        average([])

def test_ninety_is_an_A_boundary():
    assert letter_grade(90) == "A"

def test_just_below_ninety_is_a_B():
    assert letter_grade(89.99) == "B"
```

### Injecting time (JavaScript)

```javascript
// BEFORE: hidden dependency on the real clock — untestable without tricks.
export function isExpired(deadlineIso) {
  return new Date() > new Date(deadlineIso);
}

// AFTER: the clock is a parameter with a default. Production code calls it
// exactly as before; tests pass a fixed date. This default-parameter trick
// is "dependency injection lite" — no framework, one seam.
export function isExpired(deadlineIso, now = new Date()) {
  return now > new Date(deadlineIso);
}
```

```javascript
// isExpired.test.js
import { it, expect } from "vitest";
import { isExpired } from "./expiry.js";

it("is not expired one second before the deadline", () => {
  const deadline = "2026-01-01T00:00:00Z";
  const now = new Date("2025-12-31T23:59:59Z");   // we control time
  expect(isExpired(deadline, now)).toBe(false);
});

it("is expired one second after the deadline", () => {
  const deadline = "2026-01-01T00:00:00Z";
  const now = new Date("2026-01-01T00:00:01Z");
  expect(isExpired(deadline, now)).toBe(true);
});
```

### Injecting a collaborator (Python)

```python
# BEFORE: the function constructs its own dependency internally.
import smtplib

def notify_overdue(user):
    server = smtplib.SMTP("smtp.example.com")     # real mail server — yikes
    server.sendmail("app@example.com", user.email, "Your book is overdue!")

# AFTER: the sender is passed in. Production passes the real one;
# tests pass a fake that just records calls (details in Chapter 5).
def notify_overdue(user, send_mail):
    send_mail(to=user.email, body="Your book is overdue!")
```

```python
def test_notify_overdue_emails_the_user():
    sent = []                                      # our recording fake
    fake_send = lambda to, body: sent.append((to, body))

    notify_overdue(user=FakeUser(email="a@b.com"), send_mail=fake_send)

    assert sent == [("a@b.com", "Your book is overdue!")]
```

### Return data, don't print it

```javascript
// BEFORE: asserting on console output is awkward and brittle.
function showTopScores(scores) {
  const top = [...scores].sort((a, b) => b - a).slice(0, 3);
  console.log(`Top: ${top.join(", ")}`);
}

// AFTER: compute the string, return it; let the caller print.
export function formatTopScores(scores) {
  const top = [...scores].sort((a, b) => b - a).slice(0, 3);
  return `Top: ${top.join(", ")}`;
}
// somewhere at the program's edge:
// console.log(formatTopScores(scores));
```

Notice a theme: in every example, "making it testable" meant giving the function honest inputs and outputs. That's the whole trick.

## Common Pitfalls

- **Reaching for `random`/`Date.now()`/`datetime.now()` deep inside logic.** The test can never predict the output. Correction: accept the value (or the generator function) as a parameter with a default.
- **Functions that both compute and perform I/O.** You're forced to trigger the I/O to test the computation. Correction: split into a pure function plus a thin caller.
- **Over-engineering in the name of testability.** You do not need interfaces, factories, and a DI container for a 200-line script. Correction: default parameters and constructor arguments cover 95% of cases at this scale.
- **Mutating input arguments.** `sort()` in place surprises callers and makes tests order-dependent. Correction: prefer returning new values (`sorted(xs)`, `[...xs].sort()`); mutate only when performance demands it, and document it.
- **Global mutable state as a communication channel.** Two functions that talk via a module-level variable can't be tested independently, and tests pollute each other. Correction: pass the state explicitly, or wrap it in an object created fresh per test.
- **Testing private internals because the public function is untestable.** If you're tempted to test a helper by poking at underscored names, the design is telling you the helper wants to be a proper, separately-tested pure function. Correction: extract it.

## Practice Exercises

1. Find (or write) a function that mixes computation with `print`/`console.log`. Refactor it into a pure core plus a thin shell, then write four unit tests for the core.
2. Write `generate_username(first, last, rng)` that produces e.g. `"jsmith42"` where `42` comes from the injected random source. Write tests that pass a fixed-value stand-in for `rng` and assert the exact output.
3. Write a JS function `nextBackupTime(schedule, now = new Date())` that returns the next 02:00 local time after `now`. Test it for: before 02:00, after 02:00, and exactly at 02:00 — all without touching the real clock.
4. Take the `notify_overdue` pattern and apply it to something of yours: find code that constructs its own dependency (file handle, HTTP client, DB connection) and refactor so the dependency is passed in. Write one test using a hand-made fake.
5. Audit one of your past projects: list every function that (a) is pure, (b) has side effects, (c) mixes both. For one function in category (c), sketch — on paper, without coding — how you would split it.
