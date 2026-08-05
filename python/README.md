# Python Learning Track — Back-End Web Development Path

A self-paced track for someone with some coding basics who wants structure, depth, and employable skills — ending at back-end web development with Flask. Chapters are primary study material (read + type out every example); projects are guided specs with **no solution code** — building them is where the learning sticks.

- **Chapters:** `learning-docs/` — 18 numbered chapters, designed to be read in order
- **Projects:** `projects/` — 8 numbered specs, easiest → hardest, each bundling several chapters

## Chapters

| # | File | Topic |
|---|------|-------|
| 01 | `learning-docs/01-getting-started.md` | Installing Python on Windows, REPL, scripts, VS Code |
| 02 | `learning-docs/02-variables-and-data-types.md` | Variables, int/float/str/bool/None, conversion |
| 03 | `learning-docs/03-strings-and-f-strings.md` | String methods, slicing, f-strings & formatting |
| 04 | `learning-docs/04-operators-and-conditionals.md` | Operators, truthiness, if/elif/else |
| 05 | `learning-docs/05-loops-and-iteration.md` | for/while, range, break/continue, enumerate/zip |
| 06 | `learning-docs/06-functions.md` | def, return, defaults, *args/**kwargs, scope |
| 07 | `learning-docs/07-lists-and-tuples.md` | Lists, mutation, sorting, copying, tuples, unpacking |
| 08 | `learning-docs/08-dictionaries-and-sets.md` | Dicts, counting/grouping patterns, sets |
| 09 | `learning-docs/09-comprehensions.md` | List/dict/set comprehensions, generator expressions |
| 10 | `learning-docs/10-modules-packages-pip-venv.md` | import, stdlib, packages, pip, virtual environments |
| 11 | `learning-docs/11-file-io-and-paths.md` | Reading/writing files, `with`, pathlib, CSV |
| 12 | `learning-docs/12-error-handling-and-exceptions.md` | try/except/finally, raise, custom exceptions |
| 13 | `learning-docs/13-object-oriented-programming.md` | Classes, `__init__`, inheritance, dunder methods |
| 14 | `learning-docs/14-decorators-and-closures.md` | First-class functions, closures, the decorator recipe |
| 15 | `learning-docs/15-generators-and-iterators.md` | Iterator protocol, yield, laziness, pipelines |
| 16 | `learning-docs/16-json-http-and-apis.md` | JSON, the `requests` library, consuming web APIs |
| 17 | `learning-docs/17-testing-with-pytest.md` | pytest, asserts, parametrize, fixtures, tmp_path |
| 18 | `learning-docs/18-intro-to-flask.md` | Flask routes, templates, forms, JSON endpoints |

## Projects

| # | File | Project | After chapters |
|---|------|---------|----------------|
| 1 | `projects/01-number-guessing-game.md` | Number Guessing Game | 1–5 |
| 2 | `projects/02-mad-libs-text-tools.md` | Mad Libs & Text Toolbox | 1–8 |
| 3 | `projects/03-contact-book.md` | Contact Book (persistent) | 1–12 |
| 4 | `projects/04-expense-tracker.md` | Expense Tracker (CSV/JSON) | 1–12 (+16's JSON sections) |
| 5 | `projects/05-library-system-oop.md` | Library Management System (OOP) | 1–13 |
| 6 | `projects/06-cli-weather-app.md` | CLI Weather App (live API) | 1–16 |
| 7 | `projects/07-tested-utility-package.md` | `handytools` — tested utility package | 1–17 |
| 8 | `projects/08-flask-capstone.md` | **Capstone:** TrackIt Flask web app | all 18 |

## Suggested Cadence

A comfortable pace is 2–3 chapters per week with every practice exercise attempted, pausing at each project milestone until it fully meets its checklist:

1. **Weeks 1–2:** Chapters 1–5 → **Project 1** (guessing game)
2. **Weeks 3–4:** Chapters 6–9 → **Project 2** (mad libs / text tools)
3. **Weeks 5–6:** Chapters 10–12 → **Projects 3 and 4** (contact book, then expense tracker — 4 is a bigger sibling of 3)
4. **Week 7:** Chapter 13 → **Project 5** (library system)
5. **Week 8:** Chapters 14–16 → **Project 6** (weather app)
6. **Week 9:** Chapter 17 → **Project 7** (tested package)
7. **Weeks 10–12:** Chapter 18 → **Project 8** (Flask capstone)

## How to Work

- **Type, don't paste.** Every chapter example should pass through your fingers into a running script.
- **One venv per project** from Project 6 onward (and ideally from Project 3) — see Chapter 10.
- **Meet the checklist, then stop.** Stretch goals are optional; finishing beats gold-plating.
- **Break things on purpose.** Half of debugging skill is having seen the error before.
- Put your projects under version control with git from Project 3 onward — an employer-visible GitHub of Projects 5–8 *is* your junior portfolio.

## Where This Track Ends and Others Begin

This track deliberately stops at JSON-file persistence — even the capstone's SQLite stretch goal (Project 8) is optional, so the `sqlite3` standard-library module and real relational modeling are only lightly touched here. Before attempting that swap, or once you want to back a Python app with a proper database, work through the sibling `sql/` track — it covers schema design, joins, and SQL itself in depth and pairs naturally with what you've built here.
