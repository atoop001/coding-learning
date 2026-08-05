# Project 8: Capstone — "TaskForge" Console Task Manager (Maven, Tested, Shippable)

## Description

The graduation project: a complete, polished console task-management application built **as a real Maven project**, with proper packages, a third-party dependency, JSON persistence, a full JUnit suite run by `mvn test`, and a runnable JAR a stranger could download and use. Features: tasks with priorities, due dates, tags, and statuses; filtering, searching, sorting, and statistics; data that survives restarts. Everything from the track appears here — but the *real* deliverable is the engineering around the code: layout, build, tests, README. This is the repo you put on GitHub and link in job applications.

## Difficulty

**Advanced / Capstone** — estimated effort: 15–25 hours over 1–2 weeks.

## Chapters used

All of them. Centrally: 18 (Maven, JARs), 17 (JUnit), 16 (persistence, `java.time` due dates), 15 (streams for queries), 14 (error handling), 10–13 (design: interfaces, `enum` for `Priority`, collections, generics), 11 (packages).

## Requirements checklist

### Project & build
- [ ] Standard Maven layout (`src/main/java`, `src/test/java`, `pom.xml`) with real packages (`dev.<you>.taskforge.model`, `.store`, `.service`, `.cli`)
- [ ] `pom.xml` declares Java release, JUnit 5 (test scope), and Gson (or another JSON library)
- [ ] `mvn clean verify` passes from a fresh clone: compiles, runs all tests, builds the JAR
- [ ] Runnable JAR: `java -jar target/taskforge-*.jar` starts the app (jar-plugin main class; shade plugin since Gson must ride along)
- [ ] A `README.md`: what it is, how to build, how to run, feature list, one screenshot-style sample session — written for a stranger
- [ ] `.gitignore` excluding `target/`; project committed to git with meaningful commit messages as you go (not one giant commit at the end)

### Domain & features
- [ ] `Task`: id, title, optional description, `Priority` enum (LOW/MEDIUM/HIGH), optional `LocalDate` due date, `Set<String>` tags, status (`OPEN`/`DONE`), created timestamp
- [ ] Commands (menu or command-line-style, your choice): add, list, show, edit, complete, delete, search, stats, quit
- [ ] List views via streams: filter by status / priority / tag / overdue; sort by due date, priority, or creation; combinable (e.g., "open HIGH tasks by due date")
- [ ] Search: case-insensitive substring across title and description
- [ ] Stats: counts by status and priority, overdue count, completion rate, most-used tags (top 3)
- [ ] Input never crashes the app: bad dates, unknown ids, malformed commands all produce friendly messages (specific exceptions caught at the CLI boundary only)

### Persistence
- [ ] Tasks auto-save to a JSON file on every mutation (or on exit — document the choice and its trade-off) and load on startup
- [ ] First run with no data file works; a corrupted data file produces a clear error and a safe fallback (e.g., backup the bad file, start fresh) rather than a stack trace
- [ ] Storage is behind an interface (e.g., `TaskStore` with `load()`/`save(List<Task>)`) so tests can swap in an in-memory fake

### Tests (the capstone's centerpiece — aim for 25+ tests)
- [ ] Service-layer tests for every command's logic (add assigns unique ids; complete is idempotent or errors — define it; delete of unknown id throws)
- [ ] Stream-query tests against a hand-built fixture (filters, sorts, and combinations verified)
- [ ] Parsing/formatting tests (date input, command parsing) including `assertThrows` cases
- [ ] Persistence round-trip test using a temp file (`@TempDir` — look it up), plus the corrupted-file behavior test
- [ ] All tests pass via `mvn test` on the command line, not just in IntelliJ

## Hints

- **Plan a milestone ladder and keep the app runnable at every rung**: (1) Maven skeleton + walking-skeleton CLI that echoes commands; (2) Task model + add/list in memory; (3) tests for what exists so far; (4) complete/edit/delete; (5) queries & stats; (6) persistence; (7) JAR + README polish. Resist building sideways.
- The architecture that makes this easy: `cli` (reads lines, prints — *zero* logic) → `service` (`TaskService`: all rules, fully tested) → `store` (persistence behind the interface) → `model` (records/enums). If a method both computes and prints, it's in the wrong layer.
- Ids: a simple `int` counter held by the service, persisted with the data (or derive `max(id)+1` on load).
- Gson + `LocalDate` needs a type adapter — a known speed bump; searching "gson localdate adapter" is authorized and is itself practice for real-world Java.
- Write the in-memory `TaskStore` fake *first*; develop the whole service against it; add the JSON store near the end. If the interface is right, the swap is one line in `Main`.
- When a bug appears, write the failing test *before* the fix — leave both as evidence of professionalism.

## Stretch goals

- Recurring tasks (e.g., weekly) that respawn on completion.
- Undo for the last destructive command (keep a snapshot or an inverse-operation stack — your generic `Stack<E>` from Chapter 13 finally pays off).
- Colored terminal output via ANSI escape codes (overdue in red) with a `--no-color` flag.
- Import/export CSV in addition to JSON, reusing Project 7 skills.
- GitHub Actions CI: a workflow file running `mvn verify` on every push — a green badge on the README.
- The true next step: rebuild the storage layer against SQLite (JDBC), or expose the service as a Spring Boot REST API — either one converts this capstone into your first portfolio *backend*.
