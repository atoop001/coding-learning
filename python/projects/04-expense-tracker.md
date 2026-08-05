# Project 4: Expense Tracker

## Description

Build a terminal expense tracker: record expenses (date, category, description, amount), then slice and summarize them — totals by category, by month, biggest purchases, and a simple budget warning. Data persists as **JSON** (primary store) with a **CSV export** for spreadsheet users.

Using it should feel like a personal finance sidekick: entering an expense takes seconds (today's date is the default), and the reports genuinely tell you something — "you spent 40% of July on food" — with clean, aligned, currency-formatted output.

## Difficulty & Effort

**Difficulty:** Intermediate
**Estimated effort:** 5–8 hours

## Chapters Used

- `06-functions.md` — decomposition into small functions
- `07-lists-and-tuples.md` / `08-dictionaries-and-sets.md` — the data model
- `09-comprehensions.md` — filtering and aggregating
- `10-modules-packages-pip-venv.md` — `datetime`, splitting code into modules
- `11-file-io-and-paths.md` — CSV export, `pathlib`
- `12-error-handling-and-exceptions.md` — robust parsing and loading
- `16-json-http-and-apis.md` — the JSON persistence sections (no HTTP needed)

## Requirements Checklist

### Data & persistence

- [ ] Each expense is a dict: `date` (ISO string `YYYY-MM-DD`), `category`, `description`, `amount` (float)
- [ ] All expenses persist in `expenses.json` (pretty-printed) located next to the script via `pathlib`
- [ ] Loading handles: missing file (start empty), corrupt JSON (warn, don't crash), and validates that amounts are numbers — bad records are reported and skipped, not silently dropped
- [ ] Code is split into at least two modules: e.g. `storage.py` (load/save) and `tracker.py` (menu + reports), with a main guard

### Entry

- [ ] Adding an expense prompts for each field; pressing Enter on the date uses today (`datetime.date.today()`)
- [ ] Dates typed by the user are validated as real dates (hint: `datetime.strptime` raises on garbage) and stored in ISO format
- [ ] Amounts are validated as positive numbers via try/except — `"12.5o"` re-asks, `"−3"` re-asks
- [ ] Categories are normalized (stripped, lowercased) so `Food` and `food ` are one category

### Reports

- [ ] List expenses, newest first, in aligned columns with amounts formatted like `$1,234.56`
- [ ] Optional filters when listing: by category and/or by month (`2026-07`)
- [ ] Summary report: total spent, count, average expense, single largest expense
- [ ] Per-category breakdown: each category's total, and its percentage of overall spending
- [ ] Monthly report: totals per `YYYY-MM`, in chronological order
- [ ] A monthly budget amount is stored in the JSON too; the monthly report flags months that exceeded it

### Export

- [ ] A menu option writes `expenses_export.csv` via `csv.DictWriter` with a header row, and confirms the path written

## Hints

- Deleting/editing entries is *not* required (but see stretch goals) — the focus here is aggregation and persistence done well.
- Grouping is the heart of this project: the `setdefault(key, []).append(...)` (or `dict.get`) grouping pattern from Chapter 8 powers the category, month, *and* budget reports. Write one generic helper `group_totals(expenses, key_function)` and reuse it — Chapter 6's function-as-value idea makes this elegant.
- The month of an ISO date is just `date_string[:7]` — sometimes string slicing beats date libraries.
- Keep *calculation* functions pure (take a list, return a dict/number) and *printing* functions separate. You'll thank yourself in Project 7 when you learn testing.
- For percentages, guard against dividing by a zero total (empty tracker).
- `sorted(expenses, key=lambda e: e["date"], reverse=True)` — ISO dates sort correctly as plain strings; that's *why* the ISO format is required.
- Round money at display time, not in storage.

## Stretch Goals

- Edit and delete entries (pick from a numbered recent-expenses list)
- Recurring expenses: a `recurring.json` of monthly items, applied automatically the first time the app runs in a new month
- A crude bar chart in the terminal: each category's row shows `#` characters proportional to spending
- Import back *from* CSV (round-trip), merging without duplicating identical rows
- Multi-currency amounts with a fixed conversion table, reported in a single base currency
