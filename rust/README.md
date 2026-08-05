# Rust Learning Track

Deep-dive track into systems programming with Rust: memory management without a garbage collector, the ownership model, fearless concurrency, and the cargo ecosystem — culminating in a polished, publishable CLI tool.

**This is the most challenging track in this collection.** It assumes you are already comfortable in at least one other language (JavaScript/TypeScript and Python are the reference points used throughout the chapters). Do not take this as a first programming track: the chapters lean on your existing mental models — especially around garbage collection, dynamic typing, and async — and spend their effort on where Rust *differs*. The reward for the difficulty: a genuine understanding of how memory, ownership, and concurrency actually work, which will improve your code in every language, plus a meaningful career differentiator.

## Why Rust for a web developer?

It's not required — you can build a full web career on JS/TS alone. But Rust does show up on the edges of web development in ways worth knowing about: **Axum** and **Actix Web** are fast, increasingly popular backend frameworks (some teams pick Rust specifically for API services under heavy load); and **WebAssembly** via `wasm-bindgen`/`wasm-pack` lets you compile Rust to run *in the browser* alongside your JS, typically for performance-critical slices (image/audio processing, parsers, simulations) rather than whole apps. Treat this track as depth and curiosity, not a job requirement — nobody is going to reject your web dev application for not knowing Rust.

## How to use this track

- Chapters in `learning-docs\` are the primary study material — read them in order; each assumes only its predecessors. Type out and run every code example; deliberately trigger the compiler errors shown. In Rust, the error messages are course content.
- Projects in `projects\` are guided specs (no solutions). Start each one as soon as its listed chapters are done — the projects are where the knowledge sets.
- Expect to fight the borrow checker in Chapters 4–6. This is normal, universal, and temporary. Every "Common pitfalls" section documents the exact fights and their resolutions.
- Setup is Windows-specific where it matters (MSVC build tools, PowerShell commands) — see Chapter 1.

## Chapters (in order)

| # | File | Topic |
|---|------|-------|
| 1 | `learning-docs\01-why-rust-and-setup.md` | Why Rust, systems languages, memory safety, rustup/cargo on Windows, first program |
| 2 | `learning-docs\02-variables-mutability-types.md` | Bindings, immutability by default, integer/float/bool/char, tuples, shadowing |
| 3 | `learning-docs\03-functions-and-control-flow.md` | Functions, expressions vs statements, if/loop/while/for, loop labels |
| 4 | `learning-docs\04-ownership.md` | **The core concept**: moves, drops, Copy vs Clone, stack vs heap |
| 5 | `learning-docs\05-references-and-borrowing.md` | `&`/`&mut`, the borrow checker, readers-XOR-writer, dangling refs |
| 6 | `learning-docs\06-slices-and-strings.md` | Slices, `String` vs `&str`, UTF-8, zero-copy views |
| 7 | `learning-docs\07-structs-and-methods.md` | Structs, impl blocks, `&self`/`&mut self`/`self`, derives, builders |
| 8 | `learning-docs\08-enums-and-pattern-matching.md` | Data-carrying enums, `match` exhaustiveness, `Option`, if let |
| 9 | `learning-docs\09-error-handling.md` | `Result`, `?`, custom error types, panic vs recoverable |
| 10 | `learning-docs\10-collections.md` | `Vec`, `HashMap`, entry API, three iteration flavors |
| 11 | `learning-docs\11-generics-and-traits.md` | Generics, trait bounds, standard traits, static vs dynamic dispatch |
| 12 | `learning-docs\12-lifetimes.md` | Lifetime annotations demystified, elision, structs holding references |
| 13 | `learning-docs\13-closures-and-iterators.md` | Closures & capture modes, lazy iterator pipelines, implementing Iterator |
| 14 | `learning-docs\14-smart-pointers-interior-mutability.md` | `Box`, `Rc`/`Weak`, `RefCell`, `Rc<RefCell<T>>` |
| 15 | `learning-docs\15-modules-crates-cargo.md` | Modules & privacy, testing, docs, workspaces, publishing, key crates |
| 16 | `learning-docs\16-concurrency-and-async.md` | Threads, `Send`/`Sync`, `Arc<Mutex>`, channels, rayon, async/tokio intro |

## Projects (easiest → hardest)

| # | File | Project | Start after chapter |
|---|------|---------|---------------------|
| 1 | `projects\01-unit-converter.md` | Console unit converter | 3 |
| 2 | `projects\02-word-count-cli.md` | Word-count CLI tool (`wc`-like + top words) | 6 (uses 10 for `--top`; do the base first) |
| 3 | `projects\03-todo-list.md` | Persistent todo list manager | 10 |
| 4 | `projects\04-mini-grep.md` | Grep-like file search with real error handling | 12 |
| 5 | `projects\05-inventory-system.md` | Inventory system — traits & generics API design | 14 |
| 6 | `projects\06-parallel-data-processor.md` | Multithreaded log processor (seq vs threads vs rayon) | 16 |
| 7 | `projects\07-capstone-cli-crate.md` | **Capstone**: polished CLI app as a publishable crate with tests | 16 (all) |

## Suggested cadence

Self-paced at roughly **10–12 weeks** with 6–10 hours/week:

- **Weeks 1–2** — Chapters 1–3, then Project 1. Get the toolchain solid and the edit-check-run loop fast.
- **Weeks 3–4** — Chapters 4–6 (*go slow here — this is the heart of Rust; re-read Chapter 4 after Chapter 5*), then Project 2.
- **Weeks 5–6** — Chapters 7–10, then Project 3.
- **Week 7** — Chapters 11–12, then Project 4.
- **Week 8** — Chapters 13–14, then Project 5.
- **Week 9** — Chapters 15–16, then Project 6.
- **Weeks 10–12** — Capstone (Project 7). Scope small, finish completely.

If a week slips, let the *projects* absorb the slip, not the chapter order — but never skip a project entirely; they are where ownership becomes instinct.

## Short on time?

This track is optional and long (10–12 weeks). If life intervenes, Chapters 1–11 plus Projects 1–3 form a complete, worthwhile arc on their own: ownership, borrowing, structs, enums, error handling, collections, and generics/traits — the ideas that actually reshape how you think about code in any language. Stopping there, without ever touching lifetimes, smart pointers, or concurrency, is a legitimate and respectable place to end.

## Companion resources (optional but recommended)

- **The Rust Programming Language** ("the Book") — the official free book; this track parallels its arc and cites it where useful: <https://doc.rust-lang.org/book/>
- **Rustlings** — tiny compiler-error-driven exercises, superb alongside Chapters 4–6: <https://github.com/rust-lang/rustlings>
- **Rust Playground** for quick experiments: <https://play.rust-lang.org>
- `rustc --explain E0382` (etc.) whenever an error code appears — mini-tutorials built into the compiler.

## Ground rules

- Always `cargo fmt` and `cargo clippy` before considering anything finished — both are habits this track grades on from Project 1.
- No copying solutions. The projects contain hints, not answers, by design.
- When the borrow checker rejects your code, read the *full* error including the `help:` lines before changing anything. Half of this track's teaching is delivered through those messages.
