# Project 7: Capstone — A Polished, Publishable CLI Crate

## Description

The graduation project: design and build a complete, polished command-line application, packaged as a proper crate the way real Rust tools are — library + binary split, clap-derived interface with subcommands and good `--help`, serde-based persistence, a real error strategy, unit + integration + doc tests, CI-grade lint cleanliness, a README, and a dry-run-verified `cargo publish` setup (actually publishing to crates.io is optional and should be done only if you genuinely want to maintain it).

**Choose your own tool.** Requirements below are domain-agnostic. Good candidates:

- `kanban` — a terminal kanban/task board (evolve Project 3 into a real tool)
- `snip` — a code-snippet manager with tags and clipboard integration
- `logview` — Project 6's engine grown into a queryable log explorer
- `budget` — a plain-text accounting tool over CSV transactions
- `flashcards` — a spaced-repetition drill tool
- or your own — anything with state, subcommands, and files. Rule of thumb: you should be *wanting* to use it yourself, on your machine, weekly.

## Difficulty

**Advanced — the hardest project in the track.** Estimated effort: **15–25 hours** over 2–3 weeks. Scope discipline is part of the assignment: cut features, not quality.

## Chapters Used

All sixteen. Explicitly exercised: 9 (error architecture), 11 (traits in your design), 13 (iterators throughout), 15 (the whole chapter: modules, tests, docs, workspace/publishing, clap/serde/anyhow/thiserror), plus 16 if your tool has any parallel or async surface (optional).

## Requirements Checklist

### Architecture
- [ ] Library (`src/lib.rs` + at least 3 modules) contains ALL logic; `src/main.rs` is under ~50 lines: parse args, call `run`, map errors to stderr + exit codes
- [ ] Public API of the library is deliberate: minimal `pub`, re-exports at the root (`pub use`), every public item doc-commented
- [ ] At least one trait of your own design with 2+ implementations doing real work (output formats, storage backends, strategies — not ceremony)
- [ ] Error strategy: library errors are a `thiserror`-derived enum (or hand-written equivalent with `Display` + `From` — your choice, justified in the README); the binary may use `anyhow`. No `unwrap()`/`expect()` outside tests, ever, with `expect` allowed only with an invariant-stating message

### Interface
- [ ] clap derive API with at least 3 subcommands, each with its own args/flags; `--help` output reads like a tool you'd trust
- [ ] Human output to stdout, diagnostics to stderr; a `--json` (or similar machine-readable) mode on at least one subcommand
- [ ] Exit codes documented in the README and correct (0 success, distinct nonzero for user error vs internal error)
- [ ] Data persisted with serde (JSON or TOML) in a sensible location (`directories` crate or a `--data-dir` flag); corrupt data files produce a *helpful* error naming the file, not a panic or a serde stack trace

### Quality
- [ ] ≥ 15 unit tests across the library modules, covering error paths, not just happy paths
- [ ] ≥ 3 integration tests in `tests/` driving the compiled binary end-to-end (`assert_cmd` + `predicates` crates, or hand-rolled `std::process::Command`), using a temp directory so tests never touch real user data (`tempfile` crate)
- [ ] ≥ 3 doc examples that run under `cargo test` (the Doc-tests section must be non-empty)
- [ ] `cargo clippy --all-targets -- -D warnings` and `cargo fmt --check` both pass
- [ ] `cargo doc --open` produces docs you'd be willing to show someone

### Packaging
- [ ] `Cargo.toml` fully filled: description, license (`MIT OR Apache-2.0` is the ecosystem norm), repository, keywords, categories, `rust-version`
- [ ] A README.md with: what it is, install (`cargo install --path .`), 3+ usage examples with real output, exit codes, data-file location and format
- [ ] `cargo publish --dry-run` succeeds (a `.crate` file is produced and verified); actually publishing is optional
- [ ] `cargo install --path .` works and the tool runs from anywhere in a fresh terminal (this is your "ship it" moment — the tool is now on your PATH like ripgrep is)
- [ ] Version `0.1.0` tagged in git with a short CHANGELOG.md

## Hints

- Spend the first session writing (a) the README *first* — usage examples before code force interface clarity ("README-driven development"), and (b) the module map. Both will drift; starting without them drifts worse.
- Steal the shape of tools you admire: `cargo`'s own `verb --flags` grammar, ripgrep's stderr discipline, `git`'s exit codes. Run them and study their `--help`.
- The trait requirement is where designs go baroque. Two honest implementations beat four speculative ones — a `Storage` trait with `JsonFile` and `InMemory` (the test double!) is the classic that also makes your integration tests fast.
- Integration testing pattern: `Command::cargo_bin("yourtool")?.args(["add", "x"]).env("YOURTOOL_DATA_DIR", tmp.path())...` — design the data-dir override *early* precisely so tests can exist.
- serde tip: version your data file (`{"version": 1, ...}`) from day one; future-you will send thanks.
- When a design decision feels hard (owned vs borrowed in an API, enum vs trait, `Result` granularity), write the two candidate signatures in a scratch file and imagine the *caller's* code for each. The nicer call site wins. This is the Rust design instinct the whole track has been building.
- Scope check at the halfway point: if the checklist above isn't ~60% green, cut a subcommand. A small finished tool teaches (and demos in interviews) infinitely better than a large abandoned one.

## Stretch Goals

- Shell completions generated via `clap_complete` (PowerShell included) and a `completions` subcommand.
- A GitHub repository with CI: a workflow running fmt-check, clippy, and tests on push (the `actions-rs`-style YAML is ~20 lines; ask the rustup/cargo docs, not your memory).
- Cross-compile a Linux binary from Windows (`cargo build --target x86_64-unknown-linux-gnu` via `cross` or WSL) and note what it took.
- Benchmarks for your hottest function with `criterion`, plus one optimization proven by numbers.
- If your tool wants it: a long-running mode using Chapter 16 (a watch mode with a channel-fed worker, or a small tokio-based fetch) — only if the tool genuinely benefits.
- Actually publish to crates.io, announce nowhere, and enjoy the quiet professional satisfaction of `cargo install yourtool` working on any machine.
