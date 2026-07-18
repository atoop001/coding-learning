# Project 4: Mini-Grep — File Search Tool

## Description

Build a grep-like tool: `minigrep <pattern> <path> [flags]` searches for a pattern in a file — or recursively through a directory tree — and prints matching lines with filename and line number, like the real ripgrep you've been using. The centerpiece of this project is **error handling done properly**: a custom error enum, `?` propagation through clean layers, precise exit codes, and a `main` that is nothing but a boundary. This is also the project where you structure a program as library + binary for the first time.

## Difficulty

**Intermediate.** Estimated effort: **6–10 hours.**

## Chapters Used

- Chapter 6 (string searching and slices — matching without allocating)
- Chapter 9 (the whole chapter: Result, `?`, custom error enum with `Display` and `From`, panic vs recoverable)
- Chapter 10 (collections for gathering results)
- Chapter 11 (first taste: implement `Display` for your error type; maybe a `Matcher` trait for the stretch)
- Chapter 12 (lifetimes in practice: search functions returning `&str` slices borrowed from the file contents)
- Chapter 15 patterns previewed (lib.rs + main.rs split, unit tests — read the first half of Chapter 15 if you haven't reached it)

## Requirements Checklist

- [ ] Project structured as `src/lib.rs` (all logic) + `src/main.rs` (arg parsing, error printing, exit codes only)
- [ ] `minigrep <pattern> <path>` searches a single file and prints `path:line_number:line content` for each match
- [ ] If `<path>` is a directory, recurses into it, searching every file (skip files that aren't valid UTF-8 — report them to stderr as skipped, don't die)
- [ ] Flags: `-i` (case-insensitive), `-n` off switch is not needed (numbers always on), `-c` (print only a per-file match count), `-v` (invert: print non-matching lines)
- [ ] A custom `enum GrepError` with at least variants for: file read failure (wrapping `std::io::Error` and the path), invalid arguments (with a usage message), and pattern-empty; implements `Display` and the `From` conversions your `?` operators need
- [ ] The core search is a pure function `fn search<'a>(pattern: &str, contents: &'a str) -> Vec<Match<'a>>` where `Match<'a>` is a struct holding the line number and the matched line as a `&'a str` — zero copies of file content (this is the lifetime requirement)
- [ ] `main` matches on the result: user-facing message to stderr, exit code 0 (matches found), 1 (no matches — like real grep), 2 (error)
- [ ] At least 6 unit tests in lib.rs covering: basic match, no match, case-insensitive, invert, multiple matches on one line's file, empty contents
- [ ] Case-insensitive search does not allocate a lowercased copy of the whole file per line pair more than once per line (think before you `to_lowercase()` in a loop condition)
- [ ] No `unwrap()`/`expect()` outside tests; `cargo clippy -- -D warnings` passes

## Hints

- Start from the Book's classic minigrep (The Rust Programming Language, ch. 12) if stuck on structure — but this spec goes further; don't just transcribe it.
- Recursion over directories: `std::fs::read_dir(path)?` yields entries; `entry.path().is_dir()` decides recurse vs search. A `fn walk(path: &Path, out: &mut Vec<PathBuf>) -> Result<(), GrepError>` collecting files first keeps search logic separate from traversal.
- Non-UTF-8 files: `fs::read_to_string` fails with an `io::Error` of kind `InvalidData` — match on `e.kind()` to distinguish "skip quietly-ish" from "real error".
- The lifetime in `search` is Chapter 12's `find_line` example scaled up: results borrow from `contents`, not from the pattern. Write the signature first, let the compiler hold you to it.
- `line.to_lowercase().contains(&pattern_lower)`: fine. Computing `pattern_lower` *inside* the per-line loop: not fine — hoist it. (This is the allocation requirement.)
- Exit codes in Rust: `std::process::exit(code)` — but it skips destructors, so call it only from `main` after everything is dropped, or return the code from a `run()` and exit last.
- Windows note: test with both `\` and `/` separators in arguments; `PathBuf` handles both, your code shouldn't string-mangle paths.

## Stretch Goals

- `--ext txt,rs` filter to search only certain extensions.
- Context flags `-A 2` / `-B 2` (lines after/before each match) — trickier than it looks; think in terms of a window over `lines().enumerate()`.
- Highlight the matched substring in red ANSI color (find the byte range with `line.find(...)`, print in three pieces).
- A `Matcher` trait (`fn is_match(&self, line: &str) -> bool`) with `PlainMatcher` and `CaseInsensitiveMatcher` implementations chosen at startup — your first real Chapter 11 design; add `RegexMatcher` with the `regex` crate if you want a dependency.
- Parallel directory search with one thread per file batch (return after Chapter 16 and retrofit — instructive diff).
