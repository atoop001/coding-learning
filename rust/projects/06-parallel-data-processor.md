# Project 6: Multithreaded Data Processor

## Description

Build a log-file statistics engine that processes many files in parallel: point it at a directory of large text logs (you'll also write a small generator to create test data), and it produces aggregate statistics — line counts, per-level counts (ERROR/WARN/INFO), top error messages, busiest hour — using multiple threads, then prints a comparison of sequential vs parallel wall-clock time. You will build the same pipeline **three ways**: sequential, hand-rolled threads with channels, and rayon — and measure all three. This is Chapter 16 as a lived experience: `move` closures, `Arc`, channels, `Send` bounds, and the discovery of what does and doesn't get faster.

## Difficulty

**Advanced.** Estimated effort: **8–12 hours.**

## Chapters Used

- Chapter 9 (error handling for I/O at scale)
- Chapter 10 (HashMaps for aggregation, merging maps)
- Chapter 11 (a trait to abstract the three processing strategies)
- Chapter 13 (iterator pipelines inside each worker)
- Chapter 14 (Arc's sibling-relationship to Rc, made real)
- Chapter 16 (the whole chapter: threads, move, channels, Arc<Mutex>, rayon)

## Requirements Checklist

- [ ] A `generate` subcommand (`cargo run -- generate 8 200000`) that writes N synthetic log files of M lines each into `.\testdata\` — lines like `2026-07-18T14:32:07 ERROR db: connection refused`, with randomized-ish levels, hours, and a rotating set of ~20 messages (a simple deterministic pseudo-random scheme is fine; the `rand` crate is also fine)
- [ ] A `Stats` struct (total lines, per-level counts, `HashMap<String, u32>` of messages, per-hour counts `[u32; 24]`) with a `fn merge(&mut self, other: Stats)` — merging is the heart of every parallel design here
- [ ] A pure `fn process_file(path: &Path) -> Result<Stats, ProcessError>` used *unchanged* by all three strategies (this is the design constraint that keeps the comparison honest)
- [ ] Strategy 1 — sequential: iterate files, merge results
- [ ] Strategy 2 — manual threads: spawn one thread per file (or a bounded pool of K threads pulling paths from a shared work queue — bounded pool preferred), results returned via `mpsc` channel, merged in main; no `Mutex` needed in this design and a comment must explain why
- [ ] Strategy 3 — rayon: `par_iter()` over the paths with `map` + `reduce` (look up `reduce`'s identity argument) — the whole strategy should be under ~10 lines
- [ ] A `Strategy` trait or enum unifying the three behind `fn run(&self, paths: &[PathBuf]) -> Result<Stats, ProcessError>`, selected by CLI arg (`--mode seq|threads|rayon`)
- [ ] Timing with `std::time::Instant` printed for each mode; a `--race` flag runs all three and prints a comparison table, verifying all three produce **identical** Stats (implement `PartialEq` for Stats — determinism check)
- [ ] Unreadable files are reported to stderr and skipped in *all* strategies without aborting the run
- [ ] Report output: totals, level breakdown with percentages, top 5 messages, busiest hour — via iterator chains
- [ ] At least one deliberately-broken variant kept in a `#[cfg(never)]`-style comment block or doc comment: the version where you tried to share a `&mut Stats` across threads, with the actual compiler error text and two sentences on what it prevented
- [ ] No `unwrap()` outside tests/main-boundary; `cargo clippy -- -D warnings` clean

## Hints

- Write and perfect the sequential version FIRST. Parallelizing broken code parallelizes the brokenness.
- One-thread-per-file is easy but degrades with many files; the bounded pool teaches more: share the path list as `Arc<Mutex<Vec<PathBuf>>>` and have workers `pop()` until empty — or feed paths through a channel, since `Receiver` can be shared behind `Arc<Mutex<Receiver>>`. Either is accepted.
- The channel design's no-Mutex answer: each worker builds a *local* Stats and sends it once; only main merges. Contrast with the `Arc<Mutex<Stats>>` design (workers merge into shared state) — implement that one too if curious, and watch it be slower under contention.
- rayon's `reduce` wants `|| Stats::default()` as identity and a combining closure — this is `merge` again. If you derived `Default`, it's nearly free.
- Windows timing noise: run each mode a few times; the first run pays for file-cache warmup. For honest numbers, either warm the cache first or note it.
- If speedup disappoints: your workload may be I/O-bound (files cached → CPU-bound → good speedup; cold disk → threads wait on the same drive). Add a `--cpu-heavy` flag that, say, computes a checksum per line, and watch parallel efficiency change. Write down what you observe — it's the most valuable output of this project.
- Thread count: `std::thread::available_parallelism()` gives you a sensible K.

## Stretch Goals

- Streaming parse with `BufReader` instead of `read_to_string`, and compare peak memory (Task Manager is fine) on 4× bigger files.
- A live progress display: workers send `Progress` events over a second channel; main renders `files done: 7/32` in place with `\r`.
- Graceful Ctrl+C: the `ctrlc` crate + an `Arc<AtomicBool>` checked in worker loops; partial results still reported.
- Async epilogue: port Strategy 2 to tokio with `spawn_blocking` for the file reads, and write three sentences on whether async bought anything here (spoiler to verify: for local-disk CPU-bound batch work, it shouldn't — articulating *why* is the exercise).
- Scoped threads (`std::thread::scope`) variant that borrows the path slice instead of Arc-ing it — compare the ergonomics.
