# Chapter 10: Modules, Packages, pip & Virtual Environments

## Overview

Until now, each program fit in one file. Real projects don't: code is split across files (**modules**) and folders (**packages**), reuses Python's huge **standard library**, and pulls in third-party libraries with **pip** — isolated per-project inside **virtual environments**. This chapter is the bridge from "writing scripts" to "working like a developer," and it's non-negotiable for employability: every Python job expects you to know `import`, `pip`, `venv`, and `requirements.txt`.

## Definitions & Explanations

**Module** — any `.py` file. If you have `helpers.py`, then `import helpers` in another file (same folder) gives you access to its functions as `helpers.some_function()`.

**Import forms:**

```python
import math                      # use as math.sqrt(...)
import math as m                 # alias: m.sqrt(...)
from math import sqrt, pi        # bring specific names in directly: sqrt(...)
from math import *               # bring in EVERYTHING — avoid; pollutes your namespace
```

Prefer the first or third forms. Imports go at the **top of the file** by convention.

**Standard library** — hundreds of modules that ship with Python, no installation needed. Essential ones to know exist:

- `math` — sqrt, floor, ceil, pi
- `random` — random numbers, choices, shuffling
- `datetime` — dates, times, arithmetic on them
- `pathlib` / `os` — files & folders (Chapter 11)
- `json`, `csv` — data formats (Chapters 11 & 16)
- `collections` — Counter, defaultdict
- `sys` — command-line args (`sys.argv`), exit codes

**`__name__` and the main guard** — every module has a `__name__` variable. When a file is *run directly*, `__name__` is `"__main__"`; when it's *imported*, `__name__` is the module's filename. Hence the idiom:

```python
if __name__ == "__main__":
    main()
```

meaning: "run `main()` only when this file is executed as a script, not when another file imports it." This is what lets a file be both a usable library and a runnable program — and it's why importing a module *executes* its top-level code (put logic in functions!).

**Package** — a folder of modules. A folder `mytools/` containing `text.py` and `numbers.py` can be imported as `from mytools import text`. (An `__init__.py` file inside marks it explicitly as a package and runs on import; often empty.)

**pip** — the installer for third-party packages from PyPI (the Python Package Index, ~500k packages):

```powershell
pip install requests          # install
pip install requests==2.32.0  # a specific version
pip list                      # what's installed here
pip uninstall requests
```

**Virtual environment (venv)** — a private, per-project copy of Python's package space. Without one, every project shares one global pile of packages, and version conflicts eventually wreck something. With one, each project's dependencies are isolated and reproducible. **Professional rule: one project = one venv. Never `pip install` into your global Python.**

Windows workflow:

```powershell
cd D:\path\to\project
python -m venv .venv               # create it (a .venv folder appears)
.venv\Scripts\Activate.ps1         # activate — prompt now shows (.venv)
pip install requests               # installs INTO the venv only
deactivate                         # leave the venv
```

If PowerShell blocks activation ("running scripts is disabled"), run once:
`Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`

**requirements.txt** — the conventional file listing a project's dependencies so anyone (including future-you) can recreate the environment:

```powershell
pip freeze > requirements.txt      # record current versions
pip install -r requirements.txt    # recreate elsewhere
```

Commit `requirements.txt` to version control; never commit the `.venv` folder itself.

## Code Examples

### Using the standard library

```python
import math
import random
from datetime import date, timedelta

print(math.sqrt(144))                      # 12.0
print(math.ceil(3.1), math.floor(3.9))     # 4 3

print(random.randint(1, 6))                # a die roll, 1..6 inclusive
print(random.choice(["red", "green", "blue"]))
deck = list(range(1, 11))
random.shuffle(deck)                       # shuffles in place
print(deck)

today = date.today()
print(today)                               # 2026-07-17
print(today + timedelta(days=30))          # 30 days from now
print(today.strftime("%B %d, %Y"))         # July 17, 2026
```

### Splitting a program across modules

`text_utils.py`:

```python
"""Small text helpers — my first module."""

def shout(text):
    """Uppercase with an exclamation mark."""
    return text.upper() + "!"

def initials(full_name):
    """Return the initials of a full name."""
    return "".join(word[0].upper() for word in full_name.split())

# Top-level test code, guarded so imports stay silent:
if __name__ == "__main__":
    print(shout("this runs only when executed directly"))
    print(initials("grace brewster hopper"))
```

`app.py` (same folder):

```python
"""Main program that USES the module."""
from text_utils import shout, initials

print(shout("modules work"))          # MODULES WORK!
print(initials("ada lovelace"))       # AL
```

Run `python app.py` — note that `text_utils`'s test prints do **not** appear, thanks to the main guard. Run `python text_utils.py` and they do.

### A tiny package

```
myproject/
├── app.py
└── helpers/
    ├── __init__.py        (can be empty)
    ├── validation.py
    └── formatting.py
```

`app.py`:

```python
from helpers.validation import is_valid_email
from helpers import formatting

if is_valid_email("user@example.com"):
    print(formatting.as_header("Welcome"))
```

### Full venv + pip session (terminal, not Python)

```powershell
# One-time project setup
mkdir weather-app
cd weather-app
python -m venv .venv
.venv\Scripts\Activate.ps1

# (.venv) now prefixes your prompt
pip install requests
pip freeze > requirements.txt
python -c "import requests; print(requests.__version__)"

# Later, on another machine / after cloning:
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

### Command-line arguments with `sys`

```python
# repeat.py — usage: python repeat.py hello 3
import sys

def main():
    if len(sys.argv) != 3:                     # argv[0] is the script name itself
        print("usage: python repeat.py WORD COUNT")
        sys.exit(1)                            # non-zero exit code = error
    word, count = sys.argv[1], int(sys.argv[2])
    for _ in range(count):
        print(word)

if __name__ == "__main__":
    main()
```

## Common Pitfalls

**1. Naming your file after a module you import**

`random.py`, `json.py`, `requests.py` in your folder will shadow the real module and produce errors like `AttributeError: module 'random' has no attribute 'randint'`. Also delete any stale `__pycache__` folder after renaming.

**2. Forgetting the main guard**

```python
# helpers.py  (no guard)
print("loading helpers...")     # this print fires every time ANYONE imports helpers
```

Top-level code runs on import. Keep modules quiet: put behavior in functions, execution under `if __name__ == "__main__":`.

**3. Installing packages without an active venv**

If `pip install` works but `import` fails, you likely installed into a different Python than the one running your script. Checklist: is `(.venv)` visible in your prompt? Does `pip --version` show a path inside your project's `.venv`? In VS Code, did you select the venv interpreter (Ctrl+Shift+P → "Python: Select Interpreter" → the `.venv` one)?

**4. `ModuleNotFoundError` for your own module**

Python searches the *script's folder* and installed packages — not arbitrary other folders. Keep related modules in the same directory (or a package inside it), and run Python from the project root.

**5. `from module import *`**

Dumps unknown names into your file, invites collisions, and defeats editors' ability to trace where things come from. Import explicitly.

**6. Committing the venv / no requirements.txt**

The `.venv` folder is machine-specific and huge — add it to `.gitignore`. The `requirements.txt` is what you share.

**7. Circular imports**

If `a.py` imports `b.py` and `b.py` imports `a.py`, you'll get partially-initialized-module errors. It's a design smell: move the shared code into a third module both can import.

## Practice Exercises

1. **Dice module.** Create `dice.py` with functions `roll()` (1–6), `roll_many(n)` (list of n rolls), and `roll_stats(rolls)` (returns min, max, and average as a tuple). Add a main guard that demo-rolls 10 dice. Then write `game.py` that imports it and rolls until it gets a 6, reporting how many tries it took.
2. **Venv from scratch.** Create a new folder, set up a venv, activate it, install `requests`, generate `requirements.txt`, deactivate, delete the `.venv` folder entirely, and rebuild it from `requirements.txt`. Confirm `import requests` works in the rebuilt environment.
3. **Date maths.** Using `datetime`, write a script that asks for a birthdate as `YYYY-MM-DD`, then prints the day of the week it fell on, the person's age in days, and the date of their 10,000th day alive.
4. **Package practice.** Build the `helpers/` package layout shown above with real implementations: `validation.is_valid_email` (contains `@` and a dot after it) and `formatting.as_header(text)` (text framed in `=` lines). Import and use both from `app.py` using two *different* import styles.
5. **Argument-driven tool.** Write `countdown.py` that takes one command-line argument (a number of seconds ≤ 10) and prints a countdown using a loop with `time.sleep(1)` from the `time` module. Print a usage message and exit non-zero when the argument is missing or invalid.
