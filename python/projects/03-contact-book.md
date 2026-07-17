# Project 3: Contact Book

## Description

Build a terminal contact manager: add, view, search, edit, and delete contacts, with everything saved to disk so contacts survive restarts. Each contact has a name, phone, email, and optional notes.

Using it should feel like a tiny, trustworthy database: the menu is always clear about what it can do, searches are forgiving (partial, case-insensitive matches), destructive actions ask for confirmation, and nothing is ever lost between sessions. This is the first project where your program has *state that matters* — treat the data with respect.

## Difficulty & Effort

**Difficulty:** Beginner–Intermediate
**Estimated effort:** 4–6 hours

## Chapters Used

- `05-loops-and-iteration.md` — the menu loop
- `06-functions.md` — one function per operation
- `07-lists-and-tuples.md` — the list of contacts
- `08-dictionaries-and-sets.md` — each contact as a dict
- `09-comprehensions.md` — filtering search results
- `11-file-io-and-paths.md` — persistence, `pathlib`
- `12-error-handling-and-exceptions.md` — safe loading, input validation

## Requirements Checklist

- [ ] Contacts are stored in memory as a list of dicts with keys `name`, `phone`, `email`, `notes`
- [ ] A menu loop offers: add, list all, search, edit, delete, quit — and survives any invalid menu input
- [ ] **Add** validates: name is required and non-empty; phone contains only digits, spaces, `+`, and `-`; email contains `@` with text on both sides; notes may be empty
- [ ] Adding a contact whose name already exists (case-insensitive) asks whether to update the existing one instead of silently duplicating
- [ ] **List all** prints contacts sorted by name in aligned columns using f-string formatting, plus a total count
- [ ] **Search** finds partial, case-insensitive matches against name *and* email, and displays all hits
- [ ] **Edit** lets the user pick a found contact and change any single field, keeping the current value when they just press Enter
- [ ] **Delete** shows the contact and requires a typed `yes` to proceed
- [ ] Contacts persist to a file (your choice of format — a structured text format you design, or CSV via the `csv` module) in the same folder as the script, using `pathlib` to build the path
- [ ] On startup, a missing data file means "first run" and starts empty — no crash, no scary message
- [ ] A corrupt/unreadable data file is handled with a clear warning rather than a traceback
- [ ] Saving happens either after every change or on quit — but a hard kill (closing the window) after "add" then "quit" must never lose the add if you chose save-on-quit... so choose wisely and defend the choice in a comment

## Hints

- Design the data flow before coding: `load_contacts() -> list`, then every operation takes and/or mutates that list, then `save_contacts(list)`. Draw it as a comment at the top.
- CSV via `csv.DictWriter`/`DictReader` (Chapter 11) fits this data perfectly and hands you the dict shape for free — remember `newline=""` and that everything read back is a string.
- "Keep current value on empty Enter" is one line: `new = input(...) or current` — think about why that works (truthiness).
- Search with a comprehension: build the list of matches once, then reuse it for display, edit, and delete — a numbered pick-list of matches is friendlier than re-asking for exact names.
- Sorting dicts by a key: `sorted(contacts, key=...)` with a small lambda or a named function; don't forget case-insensitivity.
- Test the persistence seams hardest: quit and restart after each kind of change; delete the data file; put garbage in it.

## Stretch Goals

- Export to a differently formatted "pretty" text file (aligned table) alongside the machine-readable data file
- Contact groups/tags with a filter-by-tag menu option (a set per contact serializes nicely as a delimited string)
- An undo for the most recent delete (keep the last deleted contact in memory until quit)
- Duplicate detection on phone number as well as name
- Command-line fast path: `python contacts.py search ada` performs the search and exits without entering the menu (Chapter 10's `sys.argv`)
