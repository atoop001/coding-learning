# Project 1: CLI Toolbox

## Description

Build a small collection of command-line utilities in Node — no server, no browser, just you, the terminal, and the filesystem. The finished toolbox contains three tools you run with `node`:

1. **File organizer** — points at a folder and sorts its files into subfolders by extension (`.jpg` files into `images/`, `.pdf` into `documents/`, and so on).
2. **Word counter** — reads a text file and reports its line, word, and character counts (like a friendly `wc`).
3. **JSON pretty-printer** — reads a `.json` file and prints it re-indented and readable, with a clear error if the file isn't valid JSON.

This project is your first real server-side JavaScript: the same language you know, but with file access, command-line arguments, exit codes, and no `document` in sight. When you're done you'll have a repo you can genuinely use on your own machine.

## Difficulty & Estimated Effort

**Beginner — 3–5 hours.**

## Chapters Used

- Chapter 1: Node.js & the Runtime (`process`, command-line args, running scripts)
- Chapter 2: npm & Modules (package.json, npm scripts, your own modules)
- Chapter 3: Async Node & the Event Loop (`fs/promises`, async/await with Node APIs)

## Requirements

Work top to bottom — the order is a sensible build order.

### Setup
- [ ] Create a `cli-toolbox` folder with a `package.json` (via `npm init`) that has `"type": "module"` so you can use `import`/`export`.
- [ ] Create one file per tool inside a `src/` folder (e.g., `src/organize.js`, `src/count.js`, `src/pretty.js`).
- [ ] Add an npm script for each tool (e.g., `npm run count -- .\notes.txt` runs the word counter). Note for PowerShell: the `--` before your arguments is required so npm passes them through.
- [ ] Initialize a git repository and commit after each tool works.

### Word counter (start here — it's the gentlest)
- [ ] Accepts exactly one argument: a path to a text file.
- [ ] Prints line count, word count, and character count, clearly labeled.
- [ ] Uses `fs/promises` (not the sync or callback APIs) with `async`/`await`.
- [ ] If the argument is missing, prints a usage message (e.g., `Usage: node src/count.js <file>`) and exits with a **non-zero** exit code.
- [ ] If the file doesn't exist or can't be read, prints a human-friendly error (not a raw stack trace) and exits non-zero.
- [ ] Works with both relative and absolute paths, including paths with spaces (test with quotes in PowerShell).

### JSON pretty-printer
- [ ] Accepts a path to a `.json` file and prints the parsed, re-stringified JSON with 2-space indentation.
- [ ] Invalid JSON produces a clear one-line error message and a non-zero exit code — the stack trace stays hidden.
- [ ] Accepts an optional second argument for indent width (defaults to 2) and validates that it's a number.
- [ ] Missing-file and missing-argument handling matches the word counter's behavior (be consistent across tools).

### File organizer (the main event)
- [ ] Accepts a path to a folder; refuses politely (non-zero exit) if the path doesn't exist or isn't a directory.
- [ ] Lists the files in that folder (top level only — no recursion) and determines each file's extension.
- [ ] Moves each file into a subfolder named for its category — decide your own mapping (at minimum: images, documents, and an `other/` catch-all) and document it in the README.
- [ ] Creates the category subfolders only when they're needed.
- [ ] Skips directories and skips its own category folders on re-runs (running it twice must be harmless).
- [ ] Supports a `--dry-run` flag that prints what *would* move where without moving anything.
- [ ] Prints a summary at the end: how many files moved, into which folders, how many skipped.

### Finishing
- [ ] Shared logic used by more than one tool (e.g., "exit with an error message" or argument checking) lives in a module of its own and is imported — no copy-paste between tools.
- [ ] A `README.md` documents each tool: what it does, its usage line, and one example invocation with real output.
- [ ] All three tools handle being run from a different working directory than the project root.

## Hints

- Log `process.argv` before anything else and study its shape — where does *your* first real argument actually live? Everything about argument handling follows from that.
- The `node:path` module (`path.join`, `path.extname`, `path.basename`) exists so you never glue paths together with string concatenation. Windows path separators are exactly why.
- Skim the `fs/promises` docs and shortlist the functions you'll need — reading a file, reading a directory, making a directory, moving/renaming a file, and checking what something is (`stat`). That's nearly the whole toolbox.
- For the organizer: what happens if a file with the same name already exists at the destination? Decide the behavior on purpose (skip? rename? overwrite?) instead of discovering it by accident.
- Test the organizer against a **disposable copy** of a folder, never your real Downloads. In PowerShell, `Copy-Item -Recurse` makes you a sandbox in one line. Build `--dry-run` *before* the code that actually moves files — it will save you.
- "Exit codes" are how scripts report success to the shell. Find out how a Node process sets one, and check your last exit code in PowerShell with `$LASTEXITCODE`.
- If your error handling is `try { everything } catch { console.log(err) }`, ask: what does the *user* of this tool need to know, and what should stay hidden?

## Stretch Goals

- Add a `--recursive` flag to the word counter that accepts a folder and reports totals per file plus a grand total.
- Make the organizer's category → extension mapping configurable via a JSON file in the project (dogfood your pretty-printer on it).
- Add a fourth tool of your own invention that solves a real annoyance on your machine (deduplicating filenames, batch-renaming, finding large files…).
- Give every tool a consistent `--help` flag, generated from one shared help-text module.
- Package the toolbox so it installs as real commands via `npm link` (research the `bin` field in package.json).
