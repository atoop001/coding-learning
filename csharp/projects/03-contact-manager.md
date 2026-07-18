# Project 3: Contact Manager

## Description

Build a menu-driven console application for managing contacts: add, list, search, edit, and delete entries, each with a name, phone, email, and category. This is the classic CRUD (Create, Read, Update, Delete) pattern that underlies most business software — here powered by a `List<T>` of objects, with your first real class doing encapsulation work.

## Difficulty

**Beginner–Intermediate** — estimated effort: 4–6 hours.

## Chapters Used

- 05 Methods
- 06 Strings
- 07 Arrays & Lists
- 08 Classes & Objects
- 03/04 Control flow & loops (menu)

## Requirements Checklist

- [ ] A `Contact` class with properties: `Name`, `Phone`, `Email`, `Category` (e.g., "family", "work", "other")
- [ ] `Contact` validates its own data: setting an empty name throws or is rejected; email must contain "@" (keep the rule simple, but enforce it *inside the class*, not in the menu code)
- [ ] `Contact` overrides `ToString()` for one-line display
- [ ] A menu loop offering: [1] add, [2] list all, [3] search, [4] edit, [5] delete, [6] stats, [0] quit
- [ ] Contacts are stored in a `List<Contact>`
- [ ] List view shows numbered contacts in aligned columns
- [ ] Search is case-insensitive and matches partial names ("ad" finds "Ada Lovelace")
- [ ] Edit lets the user pick a contact by number, shows current values, and keeps any field the user leaves blank
- [ ] Delete asks for confirmation before removing
- [ ] Stats shows: total contacts and a per-category count
- [ ] Duplicate names are rejected on add (case-insensitive), with a clear message
- [ ] All user input is validated — no crash on any input you can think to type
- [ ] Program logic is split into methods; `Main` is under ~20 lines

## Hints

- Give `Contact` a constructor that takes all fields and does the validation once; the "edit" flow can construct a new candidate or use property setters that validate.
- A separate `ContactBook` class holding the private `List<Contact>` with methods like `Add`, `Find`, `RemoveAt` is a big step up in design — the menu then never touches the list directly. Highly recommended.
- For "keep field if blank" during edit: read the input, and use `string.IsNullOrWhiteSpace(input) ? oldValue : input`.
- For per-category stats without dictionaries: get the distinct categories first (loop + a `List<string>` of seen values), then count matches per category.
- Number the contacts from 1 in the UI but remember the list is 0-indexed — a classic off-by-one source. Convert in exactly one place.
- Test the nasty paths deliberately: edit with an empty list, delete index 0, delete index Count, search with no matches.

## Stretch Goals

- Sort options in the list view: by name or by category (use `List.Sort` with a lambda comparison).
- Add a `DateTime CreatedAt` property, set in the constructor, displayed in list view.
- Support multiple phone numbers per contact (a `List<string>` inside `Contact` — a collection inside an object).
- Add an "export" option printing all contacts in CSV format to the console (name-with-comma handling: wrap in quotes).
- When you reach Chapter 16, add JSON save/load so contacts persist between runs — if you built `ContactBook`, only that class changes.
