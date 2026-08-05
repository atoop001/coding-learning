# Project 8 (Capstone): "TrackIt" — a Flask Web App

## Description

The capstone: a small but complete web application. **TrackIt** is a personal task/habit tracker you use from the browser: create tasks with categories and due dates, mark them done, filter and search them, and view a stats dashboard — plus a JSON API exposing the same data for programs. Data persists to disk through a storage module developed with tests.

Using it should feel like a real (if minimalist) web product: pages share a common layout and navigation, forms validate input and report problems politely, actions redirect properly so refresh never double-submits, and the numbers on the dashboard are actually true. This project deliberately pulls together *every* chapter of the track.

If tasks bore you, reskin the same requirements as a reading log, workout tracker, recipe box, or job-application tracker — the architecture is identical; the requirements below use task vocabulary.

## Difficulty & Effort

**Difficulty:** Advanced (for this track)
**Estimated effort:** 10–16 hours, best split over multiple sessions

## Chapters Used

All of them — most directly:

- `18-intro-to-flask.md` — routes, templates, forms, PRG, JSON endpoints
- `16-json-http-and-apis.md` — persistence format + being an API server
- `17-testing-with-pytest.md` — tests for the storage layer
- `13-object-oriented-programming.md` — the `Task` model class
- `11-file-io-and-paths.md`, `12-error-handling-and-exceptions.md` — safe storage
- `10-modules-packages-pip-venv.md` — project layout, venv, requirements
- `09-comprehensions.md`, `08-dictionaries-and-sets.md`, `14-decorators-and-closures.md` — throughout

## Requirements Checklist

### Project structure

- [ ] A venv and `requirements.txt` (`flask`, `pytest`)
- [ ] Separation of concerns across modules: `app.py` (routes only), `models.py` (a `Task` class with `to_dict`/`from_dict`), `storage.py` (load/save, no Flask imports), `templates/` with a `base.html` all pages extend
- [ ] `storage.py` has a pytest suite using `tmp_path` covering: round-trip, missing file, corrupt file — written *before or alongside* the storage code, and the suite passes

### Task model

- [ ] A task has: id, title, category, optional due date (ISO string or None), done flag, created timestamp
- [ ] Invalid construction (empty title, malformed date) raises a clear exception — enforced in the model, not just the form

### Web pages

- [ ] **Home / task list**: all tasks in a table — title, category, due date, status — with visual distinction for done tasks and an "overdue" marker computed against today
- [ ] Filtering via query parameters: by category, by status (open/done), and a case-insensitive text search — combinable, with the active filters reflected in the page
- [ ] **Add task** form (GET shows, POST creates): title required, category from a select of existing categories plus free text, optional due date; invalid submissions re-render the form with a specific error message and the user's input preserved
- [ ] Mark-done and delete actions as POSTs (not GETs) that redirect back — the PRG pattern throughout; refresh after any action never repeats it
- [ ] **Stats dashboard**: total tasks, open vs. done counts, completion percentage, per-category counts, and number overdue — every figure computed from the real data
- [ ] A custom 404 page using the shared layout

### JSON API

- [ ] `GET /api/tasks` — all tasks as JSON, honoring the same filters via query parameters
- [ ] `GET /api/tasks/<id>` — one task or a JSON 404
- [ ] `POST /api/tasks` — create from a JSON body, validating and returning 400 with an error message on bad input, 201 with the created task on success
- [ ] `POST /api/tasks/<id>/done` — mark done; JSON response
- [ ] A separate script `api_demo.py` using `requests` that exercises every endpoint (including failure cases) and prints status codes — your own client for your own server

### Robustness

- [ ] The app starts with no data file (first run) and survives a corrupt one with a warning
- [ ] Every state change is saved immediately; killing and restarting the server never loses acknowledged changes
- [ ] No route handler crashes on any input you can produce from a browser or from `requests`

## Hints

- Build in vertical slices, each one fully working before the next: (1) storage + tests, (2) hardcoded task list page, (3) real data on the page, (4) add form, (5) done/delete, (6) filters, (7) stats, (8) API. A working thin app at every stage beats a broken ambitious one.
- Give `storage.py` a `path` parameter (defaulting to the real file) — that single design choice is what makes it testable with `tmp_path`.
- Task ids: `max(existing ids, default=0) + 1` is fine here. Note in a comment why this would break with concurrent users — interviewers love that awareness.
- Overdue = has a due date AND not done AND due date < today. ISO strings compare correctly, but convert to `date` objects for clarity. Compute it in Python and pass a ready-made flag to the template — keep Jinja logic shallow.
- Preserving form input on error: pass the submitted values back into `render_template` and set the fields' `value=` attributes from them.
- The filter logic will tempt you into route-handler spaghetti. Extract `filter_tasks(tasks, category=None, status=None, query=None)` into a plain module — comprehensions shine, and it's unit-testable *and* reusable by both the HTML route and the API route. One function, two front doors.
- If HTML/CSS styling threatens to eat your time, timebox it: one tiny CSS file in `static/`, or a classless CSS framework via a single `<link>`. The Python is the point.

## Stretch Goals

- Edit-task page (pre-filled form, same validation path as add — refactor toward sharing it)
- Sort controls on the task list (by due date, by created, by title) via query parameters
- Flash messages ("Task added") using Flask's `flash`/`get_flashed_messages` — research required
- Tests for the Flask routes themselves using Flask's built-in `test_client()` — research required, very employable
- Swap JSON storage for SQLite via the standard-library `sqlite3` module, keeping `storage.py`'s function signatures identical — witness the payoff of layering: nothing else changes (see the `sql/` track before attempting this — schema design and SQL itself aren't covered here)
- Deploy it: run behind a production server (`waitress` on Windows) and access it from your phone on the same network
