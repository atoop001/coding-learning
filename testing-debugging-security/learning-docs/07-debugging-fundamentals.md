# Chapter 7: Debugging Fundamentals

## Overview

Debugging is not staring at code until inspiration strikes — it's a learnable, repeatable process. Professionals debug faster than beginners not because they're smarter but because they follow a method: read the error carefully, reproduce the bug reliably, narrow down where it lives, form hypotheses, and test them one at a time. This chapter covers reading stack traces in Python and JavaScript, reproducing bugs, binary-search debugging, rubber-duck debugging, and the honest tradeoffs between print debugging and a real debugger (which Chapter 8 covers in depth).

## Definitions & Explanations

**Bug** — a difference between what the program does and what it should do. Note the two halves: many "bugs" turn out to be wrong expectations. Always establish what *should* happen before hunting why it doesn't.

**Stack trace (traceback)** — the report printed when a program crashes: the error type, the message, and the chain of function calls that led to the failure. It is the single most information-dense debugging artifact you get for free, and most beginners don't read it. Rules for reading:

- *Python*: read **bottom-up**. The last line is the error type and message; the lines above it are the call chain, with the **most recent call last** — so the failure site is the last file/line listed before the error.
- *JavaScript (Node/browser)*: read **top-down**. The first line is the error; the frame directly under it is where it was thrown.
- In both: skim past frames inside libraries/frameworks and find the *first frame that's your code*. That's where to look.
- The error *message* often contains the exact broken value: `KeyError: 'email'` tells you which key; `TypeError: Cannot read properties of undefined (reading 'name')` tells you *something* was `undefined` and you asked for `.name` on it.

**Reproducing a bug** — making it happen on demand. An unreproducible bug is nearly unfixable; a reliably reproducible one is half-fixed. Record the exact input, steps, and environment. If it's intermittent, hunt for the varying factor: timing, ordering, uninitialized state, randomness, time of day, data-dependent paths.

**Binary-search debugging (bisecting)** — when the failure could be anywhere in a long pipeline, don't inspect everything: check the *middle*. Is the data still correct halfway through? If yes, the bug is in the second half; if no, the first. Repeat. Ten steps of code take at most ~4 probes; a thousand-line pipeline takes ~10. The same idea applies across *time* with `git bisect`: binary-search your commit history for the commit that introduced a regression.

**Hypothesis-driven debugging** — the scientific method applied to code: (1) observe the failure, (2) form ONE specific hypothesis ("`total` is wrong because `price` is a string here"), (3) design the cheapest experiment that could falsify it (print `type(price)`), (4) run it, (5) conclude and repeat. The discipline is changing/testing *one thing at a time* — shotgun edits that "fix" a bug leave you not knowing what the bug was, which means it's probably still there.

**Rubber-duck debugging** — explaining the problem, line by line, out loud, to an inanimate object (or a patient friend, or a text file). Verbalizing forces you to move from "I know what this code does" to "here is what it actually says" — and the mismatch you've been blind to for an hour often surfaces mid-sentence.

**Print debugging vs debugger** — adding `print()`/`console.log()` statements vs stepping through with breakpoints. Print debugging is fast to start, works everywhere (including logs from servers), and shows a *timeline* of many values across a run. A debugger shows *everything* at one frozen moment and lets you explore without editing code. Rough guide: prints for "what happened over 500 iterations?"; debugger for "what exactly is the state right here?" Neither is amateurish; using *only* one of them is.

## Code Examples

### Reading a Python traceback

```python
# report.py
def average(scores):
    return sum(scores) / len(scores)

def summarize(records):
    return {name: average(scores) for name, scores in records.items()}

def main():
    data = {"ada": [90, 95], "grace": []}
    print(summarize(data))

main()
```

Running it produces:

```
Traceback (most recent call last):
  File "report.py", line 11, in <module>
    main()
  File "report.py", line 9, in main
    print(summarize(data))
  File "report.py", line 5, in summarize
    return {name: average(scores) for name, scores in records.items()}
  File "report.py", line 2, in average
    return sum(scores) / len(scores)
ZeroDivisionError: division by zero
```

How to read it: bottom line → the error (`ZeroDivisionError`). One up → the failing line (`report.py`, line 2, in `average`). The chain above shows *how we got there*: `main → summarize → average`. Combined with the data, the diagnosis writes itself: some `scores` list was empty (`grace`), so `len(scores)` was 0. The *fix* is a design decision (skip empty lists? return `None`? raise a clear `ValueError`?) — and per Chapter 4, whichever you choose gets a test.

### Reading a JavaScript stack trace

```
TypeError: Cannot read properties of undefined (reading 'toFixed')
    at formatPrice (D:\shop\format.js:3:22)
    at D:\shop\cart.js:12:19
    at Array.map (<anonymous>)
    at cartSummary (D:\shop\cart.js:12:9)
```

Top line: something was `undefined` and we called `.toFixed` on it, inside `formatPrice` at `format.js` line 3. The frames below say it happened during a `.map` in `cartSummary`. Hypothesis: some cart item has no price. Cheapest experiment: `console.log(items.filter(i => i.price === undefined))` just before the map.

### Binary-search debugging a pipeline

```python
def process(raw_text):
    lines = raw_text.splitlines()          # step 1
    rows = [l.split(",") for l in lines]   # step 2
    rows = [r for r in rows if len(r) == 3]# step 3
    prices = [float(r[2]) for r in rows]   # step 4
    return sum(prices)                     # step 5

# Symptom: returns 0 for a file that clearly has prices.
# Don't read all 5 steps. Probe the MIDDLE:
#   print(len(rows)) after step 3  ->  0        <- data already gone!
# Bug is in steps 1-3. Probe between 2 and 3:
#   print(rows[0]) after step 2    ->  ['12', ' 3', ' 4.50']
# Aha: whitespace after commas — len is fine... look again: the filter keeps
# len == 3, but the header row also has 3 columns and float() would crash...
# and OUR file uses semicolons on some rows. Two probes localized everything.
```

### Rubber-ducking, transcript style

> "This function takes the list of users… it filters active ones… then for each it builds… wait. I filter `users` but then I loop over `user` — singular — which is the leftover variable from the loop above. I never use the filtered list."

That is the typical rubber-duck outcome: the bug is spoken before the sentence ends.

### Print debugging that doesn't make a mess

```javascript
// Label every output — five bare `console.log(x)` lines are unreadable.
console.log("cartSummary: items =", items.length, "first =", items[0]);

// console.table is superb for arrays of objects:
console.table(items);

// Make temporary prints greppable so cleanup is one search away:
console.log("DBG cart:", JSON.stringify(cart, null, 2));   // search "DBG" later
```

```python
# Python: f-strings with = show the expression AND its value:
print(f"DBG {len(rows)=} {rows[:2]=}")
# -> DBG len(rows)=0 rows[:2]=[]
```

## Common Pitfalls

- **Not reading the error message.** Half of all beginner debugging sessions end with "oh, it literally said that." Correction: read the message word by word, and identify the first frame in *your* code before touching anything.
- **Changing code before reproducing the bug.** If you can't trigger it on demand, you can't know your fix worked. Correction: reproduce first, ideally as a failing test (Chapter 9).
- **Changing several things at once.** The bug disappears and you've learned nothing — one of your three edits fixed it (or masked it). Correction: one hypothesis, one change, one re-run; revert changes that didn't help.
- **Debugging by superstition** ("maybe it's caching, let me restart everything"). Occasionally right, never informative. Correction: restarts are fine as a *first* quick check, but after that, form hypotheses your experiments can actually falsify.
- **Assuming the bug is where the error appears.** The crash site is where bad data was *noticed*, not necessarily where it was *created* — the `undefined` price came from a parsing bug three functions earlier. Correction: trace the bad value backward to its origin before fixing.
- **Leaving debug prints in committed code.** Noise for everyone forever. Correction: tag them (`DBG`) and sweep before committing — or graduate to proper logging (Chapter 9) for anything worth keeping.

## Practice Exercises

1. Paste this into a Python file, run it, and — before fixing anything — write down: the error type, the failing file/line, the call chain in order, and your one-sentence diagnosis:
   ```python
   def lookup(d, keys):
       return [d[k] for k in keys]
   def report(config):
       return lookup(config, ["host", "port", "user"])
   print(report({"host": "x", "port": 8080}))
   ```
2. Write a 6-step data pipeline (any theme) and have a friend — or your future self — secretly break one middle step. Find the broken step using at most 3 probes, and log each probe and conclusion as you go.
3. Take a bug you fixed in a past project. Write the "reproduction recipe" you *wish* you'd had: exact input, steps, expected vs actual. Then write the failing test that encodes it.
4. Practice rubber-ducking: pick a function you wrote over a month ago and explain it aloud, line by line, to an actual object on your desk. Note anything you say that the code doesn't actually do.
5. Trigger and read three different JavaScript errors on purpose (`TypeError` via `undefined.prop`, `ReferenceError` via a typo, `RangeError` via infinite recursion). For each, write one sentence on how the *message alone* points to the cause.
