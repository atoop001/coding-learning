# Chapter 9: Error Handling (Result, `?`, panic vs recoverable)

## Overview

Rust has no exceptions. No `try`/`catch`, no `throw`, no invisible control flow that skips half your function because something five calls down raised. Instead, fallible functions *return* their failures as values — `Result<T, E>`, an enum like `Option` but carrying *why* it failed — and the `?` operator makes propagating them nearly as terse as exceptions, while keeping every failure path visible in the source. Rust splits errors into two families and forces you to pick: **recoverable** (`Result` — file not found, bad input) and **unrecoverable bugs** (`panic!` — index out of bounds, violated invariant). Getting this distinction right is most of error-handling wisdom.

## Definitions & Explanations

### `Result<T, E>`

```rust
enum Result<T, E> {
    Ok(T),    // success, carrying the value
    Err(E),   // failure, carrying the error
}
```

Any function that can fail returns `Result`. The signature *is* the documentation:

```rust
use std::fs;
use std::num::ParseIntError;

fn read_port(path: &str) -> Result<u16, String> {
    // fs::read_to_string returns Result<String, std::io::Error>
    let content = match fs::read_to_string(path) {
        Ok(text) => text,
        Err(e) => return Err(format!("cannot read {path}: {e}")),
    };
    match content.trim().parse::<u16>() {
        Ok(port) => Ok(port),
        Err(e) => Err(format!("bad port in {path}: {e}")),
    }
}
```

> **Contrast with what you know:** In Python/JS, `parseInt`/`int()`/`JSON.parse` *throw*, and nothing in a function's signature tells you it can. Callers forget try/except and ship crashes. In Rust, `fn parse(&str) -> Result<u16, _>` cannot be mistaken for infallible — you *cannot get the `u16` out* without addressing the `Err` case. It's Go's `if err != nil` discipline, but checked by the compiler and with far better ergonomics (see `?` below). TS people: `Result<T, E>` is the `Either` type you may have met in fp-ts — built into the language and used by the entire ecosystem.

Rust *warns* if you ignore a Result entirely:

```text
warning: unused `Result` that must be used
  = note: this `Result` may be an `Err` variant, which should be handled
```

### The `?` operator: propagation without ceremony

The match-and-return-Err pattern above is so common it has an operator. `expr?` means: *if Ok, unwrap the value and continue; if Err, convert the error (via `From`) and return it from the current function.*

```rust
fn read_port(path: &str) -> Result<u16, Box<dyn std::error::Error>> {
    let content = std::fs::read_to_string(path)?;   // io::Error? -> early return
    let port = content.trim().parse::<u16>()?;      // ParseIntError? -> early return
    Ok(port)
}
```

Three lines replace fourteen, and every `?` marks a spot where the function can exit early — visible, unlike exceptions. Notes:

- `?` works only inside functions returning `Result` (or `Option` — it propagates `None` the same way). Using it in a plain `fn main() -> ()` gives: `error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option``.
- `main` itself may return `Result`: `fn main() -> Result<(), Box<dyn std::error::Error>>` — then `?` works at top level and an Err prints and sets a nonzero exit code.
- The error conversion: `?` calls `From::from` on the error, which is why one function can bubble different error types into one umbrella type.

### Error types: from quick-and-dirty to production

1. **`String`** — fine for exercises: `Err(format!("bad thing: {e}"))`.
2. **`Box<dyn std::error::Error>`** — "any error at all", great for application `main`s and prototypes; `?` converts everything into it automatically. (Box/dyn mechanics in Chapters 11 and 14 — for now it's "a heap-allocated any-error".)
3. **A custom enum** — the library-grade answer; each failure mode a variant:

```rust
#[derive(Debug)]
enum ConfigError {
    Io(std::io::Error),
    BadPort(std::num::ParseIntError),
    Missing(String),
}

impl From<std::io::Error> for ConfigError {          // enables `?` conversion
    fn from(e: std::io::Error) -> Self { ConfigError::Io(e) }
}
impl From<std::num::ParseIntError> for ConfigError {
    fn from(e: std::num::ParseIntError) -> Self { ConfigError::BadPort(e) }
}

fn read_port(path: &str) -> Result<u16, ConfigError> {
    let content = std::fs::read_to_string(path)?;    // io::Error -> ConfigError::Io
    Ok(content.trim().parse::<u16>()?)               // ParseIntError -> BadPort
}
```

Callers can now `match` on *which* failure occurred — exhaustively. (In the real ecosystem: the `thiserror` crate generates this boilerplate for libraries, and `anyhow` provides a supercharged Box-dyn-style for applications. Chapter 15.)

### `panic!`: for bugs, not for errors

```rust
panic!("row {row} out of bounds for grid of {n} rows");
```

A panic unwinds the stack (running Drop cleanups) and kills the thread/program with a backtrace (`$env:RUST_BACKTRACE=1` in PowerShell to see it). Panics are for **violated assumptions — bugs**: an index you *proved* in range isn't; an invariant broken. They are not for expected failures like missing files or bad user input — those are `Result`. Std follows this rule: `v[i]` panics (you claimed i was valid), `v.get(i)` returns Option (you're asking).

Decision table:

| Situation | Use |
|---|---|
| File missing, network down, invalid user input | `Result` |
| Programmer error / "impossible" state reached | `panic!` / `unreachable!()` |
| Prototype code, tests | `unwrap()`/`expect()` (formalized panics) |
| Invariant documented and checked | `assert!`, `debug_assert!` |

`unwrap()` and `expect("...")` on Result work exactly as on Option: give me Ok or panic. In tests and examples they're fine; in library code they should be rare and each `expect` message should state *why it can't fail* ("regex is statically valid").

## Code Examples

### The full pattern, end to end

```rust
use std::fs;
use std::path::Path;

#[derive(Debug)]
enum StatsError {
    Io(std::io::Error),
    Empty,
}

impl std::fmt::Display for StatsError {              // human-readable form
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            StatsError::Io(e) => write!(f, "I/O error: {e}"),
            StatsError::Empty => write!(f, "file has no numbers"),
        }
    }
}

impl From<std::io::Error> for StatsError {
    fn from(e: std::io::Error) -> Self { StatsError::Io(e) }
}

/// Mean of all parseable numbers in the file, one per line.
/// Unparseable lines are SKIPPED (a policy decision — visible in the code).
fn mean_of_file(path: &Path) -> Result<f64, StatsError> {
    let text = fs::read_to_string(path)?;            // Io error propagates via ?
    let mut sum = 0.0;
    let mut count = 0u32;
    for line in text.lines() {
        if let Ok(n) = line.trim().parse::<f64>() {  // per-line failure: handled locally
            sum += n;
            count += 1;
        }
    }
    if count == 0 {
        return Err(StatsError::Empty);
    }
    Ok(sum / count as f64)
}

fn main() {
    // The application boundary: HERE errors become user messages + exit codes.
    match mean_of_file(Path::new("numbers.txt")) {
        Ok(mean) => println!("mean: {mean:.2}"),
        Err(e) => {
            eprintln!("error: {e}");                 // errors go to stderr
            std::process::exit(1);
        }
    }
}
```

Note the layering — this is the idiomatic architecture: inner functions *return* Results and never print or exit; only `main` (the boundary) decides how failures are shown. Compare with Python codebases where `sys.exit()` and `print` calls hide at every depth.

### `?` in the wrong place (learn the error)

```rust
fn main() {
    let text = std::fs::read_to_string("data.txt")?;   // main returns ()
    println!("{text}");
}
```

```text
error[E0277]: the `?` operator can only be used in a function that returns
              `Result` or `Option` (or another type that implements `FromResidual`)
help: consider adding return type `Result<(), Box<dyn std::error::Error>>`
```

Apply the compiler's suggestion — and add `Ok(())` as the final expression:

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let text = std::fs::read_to_string("data.txt")?;
    println!("{text}");
    Ok(())
}
```

### Result combinators (the Option toolkit's sibling)

```rust
fn demo() {
    let r: Result<i32, String> = "42".parse::<i32>().map_err(|e| e.to_string());

    r.is_ok();                       // bool checks
    let _ = r.as_ref().map(|n| n * 2);        // transform the Ok side
    let _ = r.as_ref().map_err(|e| format!("!{e}")); // transform the Err side
    let _ = r.as_ref().ok();                  // Result -> Option (drop the error)
    let n = r.unwrap_or_default();            // 42, or 0 on Err
    println!("{n}");

    // Collecting many Results: all-or-nothing in one line —
    let nums: Result<Vec<i32>, _> = ["1", "2", "x"].iter().map(|s| s.parse()).collect();
    println!("{nums:?}");            // Err(ParseIntError) — first failure wins
}
```

## Common Pitfalls

- **Unwrap-driven development leaking into real code.** Prototyping with `unwrap()` is fine; shipping it means users see `thread 'main' panicked at 'called Result::unwrap() on an Err value'`. Before "done", grep for `unwrap`/`expect` and justify or replace each.
- **`?` type mismatches (E0277 "the trait `From<X>` is not implemented").** Your function returns `Result<_, MyError>` but `?` is propagating an `io::Error` and no `From<io::Error> for MyError` exists. Fix: write the From impl, or `.map_err(...)` at the call site, or widen to `Box<dyn Error>`.
- **Stringly-typed errors everywhere.** `Result<T, String>` is fine in chapter exercises, but callers can't match on it and context gets lost. Graduate to enums (or `anyhow`/`thiserror`) as programs grow.
- **Treating panics as catchable exceptions.** `std::panic::catch_unwind` exists but is for FFI/thread boundaries, not control flow. If you're "catching" panics to handle bad input, the input path should have been `Result` from the start.
- **Swallowing errors with `let _ = ...` or `.ok()`.** Silences the must-use warning *and* the failure. Occasionally correct (best-effort cleanup); usually a bug. At minimum log: `if let Err(e) = try_cleanup() { eprintln!("cleanup failed: {e}"); }`.
- **Printing errors to stdout.** Diagnostics belong on stderr (`eprintln!`), so users can pipe your real output. This matters the moment you write CLI tools (Projects 2 and 4).

## Practice Exercises

1. Write `fn parse_age(input: &str) -> Result<u8, String>` rejecting non-numbers, zero, and ages over 130 with distinct messages. Drive it from `main` with a list of test inputs, printing `Ok`/`Err` for each. No `unwrap`.
2. Convert Chapter 7's `BankAccount::withdraw` from `-> bool` to `-> Result<(), WithdrawError>` where `WithdrawError` is an enum with `InsufficientFunds { missing_cents: u64 }` and `ZeroAmount` variants. Implement `Display` for it. Show a `match` on the specific variant in `main`.
3. Write a program with `fn main() -> Result<(), Box<dyn std::error::Error>>` that reads a filename from `std::env::args()`, reads the file, and prints its line count — using `?` exactly three times and no `match`/`unwrap`. Trigger and observe both failure modes (missing arg → make that an error too; missing file).
4. Write `fn sum_file(path: &str) -> Result<i64, StatsError>` (enum: Io, Parse with the offending line number and text) that FAILS on the first bad line — contrast with this chapter's skip-bad-lines policy. Include the line number in the Display output. Which policy is right for a bank statement vs a log analyzer? Comment.
5. Take `["10", "20", "abc", "30"]` and produce BOTH: (a) `Result<Vec<i32>, _>` via `collect` (all-or-nothing), and (b) a `Vec<i32>` of just the successes plus a count of failures, using `filter_map`. Print both outcomes and note in a comment when each policy is appropriate.
