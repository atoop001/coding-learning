# Project 2: Word-Count CLI Tool (`rwc`)

## Description

Build a command-line tool in the spirit of Unix `wc`: given one or more file paths as arguments, print the number of lines, words, characters, and bytes for each file, plus a totals row when multiple files are given. Then go one better than `wc`: add a `--top N` flag that prints the N most frequent words in the input.

This project moves you from interactive toys to a real tool: arguments instead of menus, files instead of stdin, stderr vs stdout discipline, and exit codes. You'll feel `String` vs `&str` (Chapter 6) in your hands constantly.

## Difficulty

**Beginner-plus.** Estimated effort: **4–6 hours.**

## Chapters Used

- Chapters 2–3 (types, functions, loops)
- Chapter 4 (ownership — you'll hit your first real moves passing Strings around)
- Chapter 5 (borrowing — functions should take `&str`, not `String`)
- Chapter 6 (slices & strings — the heart of this project: `lines()`, `split_whitespace()`, `chars()`, byte vs char counts)
- Chapter 10 (HashMap + entry API for the `--top` feature)
- Light use of Chapter 9 patterns (`std::fs::read_to_string` returns a Result — handle it per-file without crashing)

## Requirements Checklist

- [ ] Reads file paths from `std::env::args()`; running with no paths prints usage to **stderr** and exits with code 2
- [ ] For each file, prints one aligned row: lines, words, chars, bytes, filename
- [ ] Character count and byte count are computed separately (test with a file containing `é` or emoji — they must differ)
- [ ] A file that can't be read prints an error to stderr naming the file and the OS reason, then processing **continues** with the remaining files
- [ ] Exit code is 0 if all files succeeded, 1 if any failed
- [ ] With 2+ files, a final `total` row sums all columns
- [ ] Counting logic lives in a function `fn count(text: &str) -> Counts` where `Counts` is a struct you define — no counting in `main`
- [ ] `--top N` flag: prints the N most frequent words (case-insensitive, punctuation trimmed) with their counts, most frequent first; ties broken alphabetically
- [ ] No `unwrap()`/`expect()` on any I/O or parse in the final version
- [ ] `cargo fmt` and `cargo clippy` clean

## Hints

- Deriving the argument list: `let args: Vec<String> = std::env::args().skip(1).collect();` — then separate flags from paths yourself (a simple loop is fine; clap comes in Project 7).
- `text.lines()`, `text.split_whitespace()`, `text.chars().count()`, `text.len()` are the four counters. Three of them are one-liners; notice which is O(n).
- Your `Counts` struct wants a method like `fn add(&mut self, other: &Counts)` for the totals row — that's Chapter 7 receiver practice.
- For alignment, `{:>8}` right-pads numbers in `println!`.
- Word normalization for `--top`: `word.to_lowercase()` then `trim_matches(|c: char| !c.is_alphanumeric())` — and skip the word if it's empty after trimming.
- "Top N" from a HashMap: collect the pairs into a `Vec`, `sort_by` on `(count desc, word asc)` — sorting a Vec of tuples with a custom comparator is the exercise.
- To test stderr separation on PowerShell: `cargo run -- .\a.txt 2>$null` should show only the clean table.

## Stretch Goals

- Read from stdin when no paths are given (like real `wc`), so `Get-Content big.txt | rwc` works.
- Add `--json` output for the counts (hand-rolled formatting — serde arrives later).
- Add a `--min-len N` filter to `--top` to skip short words like "the".
- Benchmark against a large file (concatenate a book from Project Gutenberg several times) and try `read_to_string` vs `BufReader` line streaming; note the memory difference.
