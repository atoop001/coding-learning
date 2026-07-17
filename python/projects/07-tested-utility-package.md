# Project 7: `handytools` — a Tested Utility Package

## Description

Build a small, well-organized utility **package** — a collection of genuinely reusable helper functions for text, numbers, dates, and files — developed with a full **pytest** suite. The deliverable is not an app but a *library*: something another program (or another person) could import and rely on, because the tests prove it works, including at the ugly edges.

This project flips your usual workflow: for at least half the functions you must write the **tests first**, watch them fail, then implement until green. Working this way — and being able to talk about it — is directly relevant to job interviews.

## Difficulty & Effort

**Difficulty:** Intermediate–Advanced
**Estimated effort:** 6–10 hours

## Chapters Used

- `17-testing-with-pytest.md` — the core
- `10-modules-packages-pip-venv.md` — package layout, venv, `pip install pytest`
- `06-functions.md` — clean signatures, docstrings, defaults
- `12-error-handling-and-exceptions.md` — defined failure behavior
- `14-decorators-and-closures.md` — one decorator utility
- `15-generators-and-iterators.md` — one generator utility
- `11-file-io-and-paths.md` — file helpers tested with `tmp_path`

## Requirements Checklist

### Package structure

- [ ] Layout: a `handytools/` package (`__init__.py`, `text.py`, `numbers.py`, `dates.py`, `files.py`, `decorators.py`) with a sibling `tests/` folder mirroring it (`test_text.py`, ...)
- [ ] A venv with `pytest` installed and a `requirements.txt`
- [ ] Every public function has a docstring stating what it does, its parameters, its return, and what it raises
- [ ] `pytest` from the project root runs the whole suite green; there are **at least 40 tests** overall

### Required functions (minimum set — add your own)

- [ ] `text.slugify(s)` — `"Hello, World! "` → `"hello-world"` (lowercase, alphanumerics kept, runs of other characters become single hyphens, no leading/trailing hyphens)
- [ ] `text.truncate(s, max_len, suffix="...")` — shortens to at most `max_len` *including* the suffix; never cuts below the suffix; raises `ValueError` if `max_len` is too small to fit it
- [ ] `text.pluralize(word, count)` — naive English rules: `+s`, `-y → -ies`, `-s/-x/-ch/-sh → +es`; returns e.g. `"3 boxes"`
- [ ] `numbers.clamp(value, low, high)` — pins a value into a range; raises `ValueError` when `low > high`
- [ ] `numbers.mean(values)` and `numbers.median(values)` — with *defined, tested* behavior for empty input (your choice: raise or return None — the docstring and tests must agree)
- [ ] `numbers.percent_change(old, new)` — with defined behavior when `old` is 0
- [ ] `dates.humanize_delta(seconds)` — `4000` → `"1 hour, 6 minutes"` (largest two units, correct singular/plural)
- [ ] `dates.workdays_between(start, end)` — count of Mon–Fri days between two dates
- [ ] `files.count_lines(path)` — line count for a text file
- [ ] `files.unique_lines(path)` — a *generator* yielding each distinct stripped line in first-seen order
- [ ] `decorators.retry(times)` — decorator factory re-invoking a failing function, re-raising after the final attempt, preserving name/docstring via `functools.wraps`

### Testing requirements

- [ ] At least half the functions were built test-first; mark those test files with a comment noting it
- [ ] Every function has happy-path tests **and** edge/error tests (`pytest.raises` where applicable)
- [ ] At least three functions use `@pytest.mark.parametrize` with 4+ cases each
- [ ] File helpers are tested exclusively through `tmp_path` — the suite creates no files in the project tree
- [ ] Float comparisons use `pytest.approx`
- [ ] The `retry` decorator's tests prove: success passes through the return value; it retries the right number of times (hint below); the final exception propagates

## Hints

- Test-first rhythm: write *one* test expressing one small behavior (`test_slugify_lowercases`), run it, see it fail for the right reason, write the minimal code to pass, repeat. Resist implementing ahead of the tests.
- `slugify` is fiddly at the edges — that's why it's here. Tests to think about: all-punctuation input, internal runs of junk (`"a---b"`), leading/trailing junk, already-clean input, empty string.
- To count how many times `retry` called your function, use a closure or a mutable list in the test: a tiny "counting function" that fails N times then succeeds. This is the standard hand-rolled spy pattern.
- `workdays_between`: `date.weekday()` gives 0–6 (Mon–Sun). A generator expression over a date range with `timedelta` keeps it clean. Decide and test whether the range includes its endpoints — *the deciding is the design work*.
- Watch for tests that mirror the implementation ("assert it does what the code does"). Good tests state outcomes you could write *before* seeing the code.
- When a test is hard to write, that's the design smell talking: the function probably does too much or has vague failure behavior.

## Stretch Goals

- Measure coverage with `pytest-cov` (`pip install pytest-cov; pytest --cov=handytools`) and get above 90% — then find one *meaningful* missing test the number pointed you to
- Add type hints throughout and check them with `mypy`
- Property-style tests: for `clamp`, generate 100 random (value, low, high) triples in a loop and assert the invariant `low <= result <= high` always holds
- Package it properly with `pyproject.toml` so `pip install -e .` works in the venv, and import it from a *different* folder to prove it
- Write a `CHANGELOG.md` and cut a `v0.1.0` — practice speaking the release rituals of real libraries
