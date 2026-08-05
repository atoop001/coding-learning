# Project 7: Modular Expense Tracker (Classes + Modules + localStorage)

## Description

Build a personal expense tracker with real software architecture. Users add expenses (description, amount, category, date), see them in a filterable/sortable list, get summary statistics (total, per-category breakdown with percentage bars, this-month total), and everything **persists across page reloads** via `localStorage`.

What makes this project different from the to-do app is the *engineering*: the code is split into ES modules with clear responsibilities, the core domain logic lives in classes (`Expense`, `ExpenseTracker`), and storage is isolated behind its own module so the tracker class never touches `localStorage` directly. You're practicing the shape of professional codebases: layers that can be developed, tested, and replaced independently.

Because you're using modules, run the project through a local server (`npx serve`, Live Server, etc.) — revisit Chapter 17 if imports mysteriously fail.

## Difficulty & Effort

- **Difficulty:** Advanced−
- **Estimated effort:** 9–14 hours

## Chapters Used

- `14-classes-and-prototypes.md` — `Expense`, `ExpenseTracker` classes, getters, static methods, private fields
- `17-modules.md` — the file structure IS the exercise
- `12-error-handling.md` — validation in the class layer, safe JSON parsing
- `07-arrays-and-array-methods.md` — filtering, sorting, `reduce` for summaries
- `08-objects.md` — immutability discipline, serialization shapes
- `10-dom-manipulation.md` + `11-events.md` — the UI layer
- `18-modern-js-and-tooling.md` (only the destructuring/spread/`??` sections — previewed already in Chapters 8 and 13; the full chapter comes later, before Project 8)

## Requirements Checklist

**Architecture**
- [ ] At least four modules: `expense.js` (the `Expense` class), `tracker.js` (the `ExpenseTracker` class), `storage.js` (load/save), `ui.js` and/or `main.js` (DOM + wiring); HTML loads only `main.js` with `type="module"`
- [ ] `expense.js` and `tracker.js` contain **zero** references to `document` or `localStorage` — verifiable with a text search
- [ ] `storage.js` exposes `save(expenses)` / `load()` and is the only module touching `localStorage`; `load()` survives corrupted JSON (try `try/catch` and manually mangling the stored value in DevTools)

**Domain classes**
- [ ] `Expense` has `id`, `description`, `amount`, `category`, `date`; its constructor **throws** descriptive errors for invalid data (empty description, non-positive/non-numeric amount, unknown category)
- [ ] Valid categories are defined once as an exported constant (array), used by both validation and the UI's `<select>`
- [ ] `Expense` has at least one getter (e.g., `formattedAmount` or `monthKey`) and a static `fromJSON(obj)` for rebuilding instances after `JSON.parse` (parsed objects lose their class — this is the key serialization lesson!)
- [ ] `ExpenseTracker` holds the collection in a **private field** (`#expenses`), exposing `add(expense)`, `remove(id)`, `getAll()` (returning a copy, not the internal array), `totalFor(category)`, `get total`, and `summaryByCategory()` (built with `reduce`)

**UI & behavior**
- [ ] Add-expense form with validation errors surfaced inline (catch the class's thrown errors and display their messages — the class is the single source of validation truth)
- [ ] Expense list with delete buttons (delegated listener), category filter, and at least two sort options (date, amount)
- [ ] Summary panel: overall total, per-category totals with percentage bars (a styled `div` width works fine), and current-month total
- [ ] Every mutation persists immediately; a full page reload restores the exact state, with working class instances (getters functional — proof `fromJSON` is wired correctly)
- [ ] A "clear all data" button with a `confirm()` guard

## Hints

- Build order that works: `Expense` class alone (test in console via a temporary import or Node) → `ExpenseTracker` with hardcoded data → `storage.js` round-trip → UI last. Each layer should be demonstrably working before the next.
- The serialization trap in one line: `JSON.parse(JSON.stringify(expense))` gives a plain object — `instanceof Expense` is `false` and getters are gone. `storage.load()` should therefore return raw objects, and *someone* (tracker constructor or `main.js`) must map them through `Expense.fromJSON`. Decide where and comment why.
- Persist-on-mutation without littering `save()` calls everywhere: let `main.js` wrap mutations in a single `commit()` helper (`tracker.add(...); commit();`), or have tracker accept an `onChange` callback (Chapter 13!) that `main.js` sets to "save + render."
- `summaryByCategory()` is a `reduce` into an object keyed by category — Chapter 7's vote-tally example is the template.
- Percentages: guard against dividing by a zero total when the tracker is empty.
- For month math, a `monthKey` getter like `"2026-07"` (from the date string) makes "this month" a simple string comparison.

## Stretch Goals

- **Income + expenses:** signed amounts or a type field, with a net balance display.
- **Budgets:** a per-category monthly budget with an over-budget warning state in the summary bars.
- **CSV export:** generate a CSV string from the data and trigger a download (`Blob` + a temporary `<a download>`).
- **Charts:** hand-rolled bar chart of monthly spending using sized divs — no libraries needed.
- **Multiple profiles:** a profile switcher, with `storage.js` keying saved data by profile name (its isolation should make this an easy change — that's the architecture paying off).
