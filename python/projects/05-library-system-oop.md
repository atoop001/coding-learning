# Project 5: Library Management System (OOP)

## Description

Build a library system as a proper object-oriented design: `Book`s and `Member`s managed by a `Library`, with borrowing, returning, due dates, and late detection — all driven from a terminal menu, with the whole library state persisted to JSON.

Using it should feel like software with *rules*: a member can't borrow a book that's checked out, can't exceed their borrowing limit, and returning late is noticed and reported. The interesting work is in the classes enforcing those rules — the menu is just a thin skin over them.

## Difficulty & Effort

**Difficulty:** Intermediate
**Estimated effort:** 6–9 hours

## Chapters Used

- `13-object-oriented-programming.md` — the core: classes, properties, dunders, inheritance
- `12-error-handling-and-exceptions.md` — custom exception classes for rule violations
- `08-dictionaries-and-sets.md` — indexes by id
- `10-modules-packages-pip-venv.md` — multi-module layout, `datetime`
- `11-file-io-and-paths.md` / `16-json-http-and-apis.md` — JSON save/load of object state
- `09-comprehensions.md` — searches and reports

## Requirements Checklist

### Classes & rules

- [ ] `Book`: id, title, author, year; tracks whether it's currently borrowed and by whom; has a useful `__repr__` and `__eq__` (by id)
- [ ] `Member`: id, name; holds its set/list of currently borrowed book ids; a regular member may borrow at most 3 books at once
- [ ] `PremiumMember(Member)`: inherits, raises the limit to 10, and overrides whatever method enforces the limit using `super()` where sensible
- [ ] `Library`: owns the collections of books and members; all state changes go through its methods — outside code never pokes `_underscore` internals
- [ ] `Library.borrow(member_id, book_id)` enforces: book exists, member exists, book not already borrowed, member under their limit — each violation raising a *distinct custom exception* (all subclassing one `LibraryError`)
- [ ] Borrowing records a due date 14 days from today (`datetime.date`); `Library.return_book(...)` reports whether it was returned late and by how many days
- [ ] `Library` supports `len(library)` (number of books) and `book_id in library` via dunder methods
- [ ] A read-only `@property` on `Library` reports how many books are currently available

### Behavior & interface

- [ ] Menu: add book, add member (regular/premium), borrow, return, search books by title/author (partial, case-insensitive), list overdue loans, quit
- [ ] The menu layer catches `LibraryError` subclasses and prints their messages — rule enforcement lives in the classes, *presentation* of failures lives in the menu; the menu contains no rule logic of its own
- [ ] Overdue report lists member name, book title, due date, and days overdue, sorted most-overdue first

### Persistence

- [ ] The entire library state (books, members, active loans with due dates) saves to and loads from JSON
- [ ] Since JSON can't hold objects or dates directly, each class provides `to_dict()` and a classmethod-style constructor from a dict (a plain function is acceptable), and dates round-trip through ISO strings
- [ ] Loading a missing or corrupt file starts an empty library with a warning

## Hints

- Start with the classes and *no menu at all*: script a few borrow/return scenarios at the bottom of the module under a main guard, printing outcomes. When the rules all work, the menu is easy. (This ordering — model first, interface last — is how experienced developers actually build.)
- Store books and members in dicts keyed by id inside `Library` — O(1) lookup and no duplicate-id worries.
- The "distinct exceptions" requirement pays off in the menu: one `except LibraryError as e: print(e)` handles everything, yet tests (Project 7 foreshadowing) can assert precise failures.
- For due dates: `datetime.date.today() + datetime.timedelta(days=14)`; lateness is just date subtraction, which yields a `timedelta` with a `.days`.
- `to_dict` on `Library` composes the `to_dict`s of everything it owns — persistence becomes three layered calls.
- Resist the urge to have `Book` know about `Library`. Dependencies should point one way: Library → Book/Member.

## Stretch Goals

- Reservations: members can queue for a checked-out book; returning it notifies (prints) the next member in line
- Fines: $0.50/day late, tracked per member, with a "pay fine" menu option and a block on borrowing while owing more than $5
- A `Librarian(Member)` role that can remove books — but only ones not currently borrowed
- Search returning results ranked: exact title matches first, then startswith, then substring
- Inventory statistics report using comprehensions: most-borrowed author, average loan duration from a history log of completed loans
