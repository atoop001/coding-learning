# Project 3: Todo List Manager

## Description

Build an interactive todo-list application: add tasks, list them, mark them done, set priorities, filter and sort, and persist everything to a file between runs (a simple text format you design and parse yourself — no serde yet). This is the project where structs, enums, `Option`, and pattern matching stop being chapter examples and become your data model — and where ownership of data moving between your collection, your functions, and your file format has to actually work.

## Difficulty

**Intermediate.** Estimated effort: **6–9 hours.**

## Chapters Used

- Chapter 4–5 (ownership & borrowing — the `TodoList` owns its items; methods borrow)
- Chapter 6 (parsing your save-file format with string slices)
- Chapter 7 (structs & methods — `Task`, `TodoList`, receiver discipline)
- Chapter 8 (enums & matching — `Priority`, `Status`, command parsing, `Option` everywhere)
- Chapter 9 (Result — file I/O and parse errors with real error messages)
- Chapter 10 (Vec operations: retain, sort_by_key, iteration flavors)

## Requirements Checklist

- [ ] A `Task` struct with at least: id (`u32`), title (`String`), priority (enum `Low`/`Medium`/`High`), status (enum `Open`/`Done`), and an optional due date stored as `Option<String>` (plain `YYYY-MM-DD` text is fine)
- [ ] A `Priority` enum deriving `Debug`, `Clone`, `Copy`, `PartialEq`, `PartialOrd`, `Ord`, `Eq` — so tasks can be sorted by it directly
- [ ] A `TodoList` struct owning `Vec<Task>`, with ALL mutation going through methods (`add`, `complete`, `remove`, `set_priority`) — `main` never touches the Vec directly
- [ ] Interactive commands (parse a line like `add Buy milk !high @2026-08-01`): `add`, `list`, `done <id>`, `rm <id>`, `pri <id> <level>`, `quit` — unknown commands print help, never crash
- [ ] Command parsing returns an `enum Command`, produced by a function `fn parse_command(line: &str) -> Result<Command, String>` — an exhaustive `match` then dispatches
- [ ] `list` shows open tasks sorted by priority (High first) then id; done tasks shown separately or via `list all`
- [ ] Completing or removing a nonexistent id reports a friendly error (this is an `Option` → message path, not a panic)
- [ ] On quit (and on start), the list is saved to / loaded from `todo.txt` in a line-based format you design; a missing file on first run is NOT an error
- [ ] A malformed line in the save file reports the line number and content, skips the line, and keeps loading
- [ ] Ids are stable across runs (persist the next-id counter or derive it from the max loaded id)
- [ ] No `unwrap()`/`expect()` outside of tests; `cargo clippy` clean

## Hints

- Design the save format before coding it, e.g. `1|open|high|2026-08-01|Buy milk` — then `split('|')` with a fixed field count parses it. `splitn(5, '|')` protects titles containing `|`... or forbid the character on input. Your call; document it in a comment.
- `parse_command`: `line.split_whitespace()` for the command word, but careful — the title of `add` is the *rest of the line*, so `split_once(' ')` first is cleaner.
- The `!high` / `@date` markers: iterate the words, peel off ones starting with `!` or `@`, and join the rest as the title. `strip_prefix` returns an `Option<&str>` — perfect `if let` practice.
- Mapping strings to `Priority` and back is two small functions (or `FromStr`/`Display` impls if you want to reach ahead to Chapter 11 — both are legitimate).
- For `complete(&mut self, id: u32) -> Result<(), String>`: `self.items.iter_mut().find(|t| t.id == id)` gives `Option<&mut Task>` — a one-line borrow-checker lesson.
- Sorting: `self.items.sort_by_key(|t| (std::cmp::Reverse(t.priority), t.id))` — `Reverse` is why deriving `Ord` on the enum pays off. Variant order in the enum declaration determines `Ord` order — declare `Low` first.
- Loading: build the whole `Vec<Task>` first, then construct the TodoList — don't mutate a half-loaded list.

## Stretch Goals

- `edit <id> <new title>` and `due <id> <date>` commands.
- A `stats` command: counts by priority and status, oldest open task.
- Search: `find <substring>` (case-insensitive) across titles.
- Undo for the last destructive operation (keep the removed/modified `Task` — ownership makes this design interesting: who holds the undo state?).
- Colored output on Windows Terminal using ANSI escape codes (`\x1b[31m` etc.) with a `--no-color` fallback.
