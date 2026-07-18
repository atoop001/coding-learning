# Chapter 15: Modules, Crates & the Cargo Ecosystem

## Overview

Time to zoom out from language features to *programs*: how Rust code is organized into modules and files, how crates depend on each other, how tests live alongside code, and the cargo ecosystem (crates.io, workspaces, publishing, and the handful of third-party crates everyone uses). If npm/PyPI shaped your instincts, cargo will feel like the best version of that world: one blessed tool, lockfiles from day one, docs auto-generated for every published crate, and semver actually taken seriously.

## Definitions & Explanations

### Crates, packages, modules

- A **crate** is the unit of compilation: either a **binary crate** (has `main`, builds an .exe) or a **library crate** (a `lib.rs`, builds a library others link).
- A **package** is what `cargo new` makes: a `Cargo.toml` plus one or more crates — at most one library (`src/lib.rs`) and any number of binaries (`src/main.rs`, plus extras in `src/bin/`).
- A **module** (`mod`) is a *namespace within* a crate — the unit of privacy and organization.

The classic CLI-app layout uses both: a thin `main.rs` (argument handling, calls into the library) and a `lib.rs` holding the real logic — because library code is testable and reusable. Project 7 will demand this.

### Modules and privacy

```rust
// src/lib.rs — the crate ROOT of the library
pub mod inventory;         // "there is a module `inventory`; find it in
                           //  src/inventory.rs (or src/inventory/mod.rs)"

// src/inventory.rs
pub struct Item {                   // pub: visible to crate users
    pub name: String,
    pub price_cents: u64,
    internal_id: u64,               // NOT pub: private to this module
}

pub fn total(items: &[Item]) -> u64 {
    items.iter().map(|i| i.price_cents).sum()
}

mod pricing {                       // private child module (no pub)
    pub(crate) fn tax(cents: u64) -> u64 { cents / 10 }
    // pub(crate): visible anywhere in THIS crate, but not to external users
}

pub fn total_with_tax(items: &[Item]) -> u64 {
    let t = total(items);
    t + pricing::tax(t)
}
```

Rules worth internalizing:

- **Everything is private by default** — the inverse of Python (where `_underscore` is a plea, not a law) and of JS's everything-exported habits. `pub` is a considered API decision, checked by the compiler: private items are *uncallable* from outside, so refactoring them can never break users.
- Visibility gradations: `pub` (world), `pub(crate)` (this crate), `pub(super)` (parent module), private (this module + children).
- A `pub` struct does **not** expose its fields — each field needs its own `pub`. Structs with private fields can only be built via constructors you provide (this is how you enforce invariants — recall Chapter 7's `BankAccount`).
- `use` brings paths into scope: `use crate::inventory::Item;` (absolute, from crate root), `use super::x` (relative to parent), `use std::collections::HashMap;` (std). Idiom: `use` the *parent* for functions (`use crate::inventory; inventory::total(...)`) but the *type itself* for structs/enums (`use crate::inventory::Item`).
- Re-exports shape your public API: `pub use inventory::Item;` in lib.rs lets users write `mycrate::Item` regardless of your internal file layout.

### File layout, mapped

```text
my-cli/
├── Cargo.toml
├── src/
│   ├── main.rs          # binary crate root:  `use my_cli::run;`
│   ├── lib.rs           # library crate root: `pub mod parser; pub mod output;`
│   ├── parser.rs        # mod parser
│   └── output/
│       ├── mod.rs       # mod output  (or a sibling `output.rs` — both styles legal)
│       └── json.rs      # mod output::json  (declared in mod.rs: `pub mod json;`)
├── tests/               # INTEGRATION tests: each file its own crate, tests the
│   └── cli_test.rs      #   public API only — like an external user
└── target/              # build output (never commit)
```

Unlike Python/Node, files do not automatically become modules — the `mod` declaration in the parent is what creates the module; the file is just where its body lives. Forgetting the `mod parser;` line is the #1 "why can't it see my file" confusion.

### Testing (built in, no framework to choose)

```rust
// Unit tests live IN the same file as the code, in a cfg-gated module:
pub fn add(a: i32, b: i32) -> i32 { a + b }

#[cfg(test)]                       // compiled only for `cargo test`
mod tests {
    use super::*;                  // pull in the parent module's items

    #[test]
    fn adds_positive_numbers() {
        assert_eq!(add(2, 3), 5);
    }

    #[test]
    fn adds_negatives() {
        assert!(add(-2, -3) < 0);
        assert_ne!(add(-2, 2), 1);
    }

    #[test]
    #[should_panic(expected = "index out of bounds")]
    fn panics_on_bad_index() {
        let v: Vec<i32> = vec![];
        let _ = v[0];
    }

    #[test]
    fn results_work_too() -> Result<(), String> {
        if add(1, 1) == 2 { Ok(()) } else { Err("math broke".into()) }
    }
}
```

`cargo test` runs unit tests, integration tests (`tests/`), *and* code examples in your doc comments. Yes — doc examples are compiled and executed:

```rust
/// Adds two numbers.
///
/// # Examples
/// ```
/// assert_eq!(my_cli::add(2, 2), 4);
/// ```
pub fn add(a: i32, b: i32) -> i32 { a + b }
```

`cargo doc --open` renders `///` comments into the same HTML docs you see on docs.rs. Documentation that can't go stale because it's tested — internalize this; it's a genuine Rust superpower and the reason ecosystem docs are so uniformly good.

### Dependencies and the ecosystem

```toml
[dependencies]
serde = { version = "1", features = ["derive"] }   # opt-in feature flags
clap = { version = "4", features = ["derive"] }
anyhow = "1"

[dev-dependencies]        # only for tests/benches
assert_cmd = "2"
```

Add with `cargo add serde --features derive` (like `npm install`). Versions are semver ranges (`"1"` = `>=1.0, <2.0`); `Cargo.lock` pins exact versions (commit it for binaries). Where to find things: **crates.io** (registry), **docs.rs** (auto-built docs for every crate ever published), **lib.rs** (curated browsing).

The short list of crates the whole ecosystem standardizes on — learn these names:

| Crate | Role | You'd know it as |
|---|---|---|
| `serde` + `serde_json` | (de)serialization via derive | `JSON.parse` / pydantic |
| `clap` | CLI argument parsing via derive | argparse / commander |
| `anyhow` | app-level error handling (Ch 9's Box-dyn, perfected) | — |
| `thiserror` | derive for library error enums | — |
| `regex` | regular expressions | `re` |
| `rand` | random numbers | `random` |
| `tokio` | async runtime (Ch 16) | Node's event loop |
| `reqwest` | HTTP client | fetch / requests |
| `rayon` | data parallelism (Ch 16) | multiprocessing, but easy |
| `criterion` | benchmarking | — |

### Workspaces and publishing

**Workspace** = several packages sharing one `Cargo.lock` and `target/` (a monorepo, like pnpm workspaces):

```toml
# ./Cargo.toml (workspace root — no [package] of its own)
[workspace]
members = ["core", "cli", "server"]
resolver = "2"
```

Each member is a normal package; path dependencies wire them: `core = { path = "../core" }`. `cargo build -p cli` targets one member; bare `cargo test` runs everything.

**Publishing** to crates.io: `cargo login` (token from crates.io), fill the manifest metadata (`description`, `license = "MIT OR Apache-2.0"`, `repository`, `keywords`), then `cargo publish --dry-run` and `cargo publish`. Names are first-come global, and **publishes are permanent** — you can `cargo yank` a version (stops new use) but never delete. Semver is a promise the resolver relies on: breaking change → major bump.

### The daily toolbelt

```text
cargo fmt                 # format (rustfmt) — non-negotiable, zero-config
cargo clippy              # lints with fixes; treat warnings as homework
cargo clippy -- -D warnings   # CI mode: warnings are errors
cargo doc --open          # your crate's docs, rendered
cargo tree                # dependency tree (audit what you're pulling in)
cargo test some_name      # run tests matching a name
cargo run --bin other     # choose among multiple binaries
cargo install ripgrep     # install a Rust CLI globally (how users get YOUR tool)
```

## Code Examples

### A testable library + thin binary (the shape of Project 7)

```rust
// src/lib.rs
pub mod stats;

// src/stats.rs
/// Mean of a slice. Returns `None` for empty input.
///
/// ```
/// assert_eq!(my_cli::stats::mean(&[1.0, 3.0]), Some(2.0));
/// ```
pub fn mean(data: &[f64]) -> Option<f64> {
    if data.is_empty() {
        return None;
    }
    Some(data.iter().sum::<f64>() / data.len() as f64)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn empty_is_none() { assert_eq!(mean(&[]), None); }

    #[test]
    fn simple_mean() { assert_eq!(mean(&[2.0, 4.0]), Some(3.0)); }
}
```

```rust
// src/main.rs — knows about I/O and exit codes; the library knows math.
use my_cli::stats::mean;

fn main() {
    let nums: Vec<f64> = std::env::args()
        .skip(1)
        .filter_map(|a| a.parse().ok())
        .collect();
    match mean(&nums) {
        Some(m) => println!("mean: {m}"),
        None => {
            eprintln!("usage: my_cli <numbers...>");
            std::process::exit(2);
        }
    }
}
```

### serde + clap in twenty lines (why derive macros sell Rust)

```rust
// cargo add serde --features derive; cargo add serde_json clap --features clap/derive
use clap::Parser;
use serde::{Deserialize, Serialize};

#[derive(Parser)]                      // clap: --file <FILE> [--pretty]
struct Args {
    #[arg(long)]
    file: std::path::PathBuf,
    #[arg(long, default_value_t = false)]
    pretty: bool,
}

#[derive(Serialize, Deserialize, Debug)]
struct Task { title: String, done: bool, tags: Vec<String> }

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let args = Args::parse();                       // parsing, validation, --help: free
    let text = std::fs::read_to_string(&args.file)?;
    let tasks: Vec<Task> = serde_json::from_str(&text)?;   // typed JSON — no `any`
    let open: Vec<&Task> = tasks.iter().filter(|t| !t.done).collect();
    let out = if args.pretty {
        serde_json::to_string_pretty(&open)?
    } else {
        serde_json::to_string(&open)?
    };
    println!("{out}");
    Ok(())
}
```

Compare with the TS equivalent: no `zod` schema duplicating the type, no `JSON.parse` returning `any`, no argv library choices — derive macros generate the parsing/validation/serialization from the type definitions themselves.

## Common Pitfalls

- **Forgetting `mod` declarations.** Creating `src/parser.rs` and getting `error[E0432]: unresolved import crate::parser`. Files don't self-register: add `pub mod parser;` to `lib.rs`/`main.rs`.
- **`private` errors that are actually design feedback.** `error[E0603]: module `pricing` is private` / `field `internal_id` of struct `Item` is private`. Before adding `pub`, ask if the *caller* is reaching somewhere it shouldn't — often the fix is a new public method, not a hole in the wall.
- **Binary + library confusion.** In `main.rs`, importing library items via `crate::` fails — they are *different crates*. Import by package name: `use my_cli::stats::mean;`.
- **Test module missing `use super::*;`** — then nothing from the file is in scope and every test line errors. It's the first line of the `mod tests` idiom for a reason.
- **Version-pinning paranoia or promiscuity.** `"=1.2.3"` everywhere fights the resolver; `"*"` is a supply-chain shrug. The idiom: major-version ranges (`"1"`) + committed `Cargo.lock`.
- **npm-instinct dependency sprawl.** Cargo makes adding deps easy, and each dep adds compile time and audit surface. Run `cargo tree` before adding; std + a handful of blessed crates covers most CLI work. (There is no `is-odd` culture here; don't import one.)
- **Publishing prematurely.** Names are global forever. Practice with `cargo publish --dry-run` and keep hobby experiments local (path deps or git deps work fine without publishing).

## Practice Exercises

1. Take any earlier chapter's exercise code and restructure it as lib + bin: logic in `src/lib.rs` (+ one module file), thin `main.rs` calling it by package name. Everything must still run via `cargo run`, and `cargo test` must find at least three unit tests you add.
2. Build a `shapes` module with a private helper, a `pub(crate)` function, and a fully `pub` API — then write a `tests/integration.rs` that proves (via compile errors, kept in comments) that only the `pub` layer is reachable from outside.
3. Add doc comments with runnable examples to three public functions; make `cargo test` execute them (watch the "Doc-tests" section of the output) and `cargo doc --open` render them. Break one doc example deliberately and observe the test failure.
4. Create a workspace with members `converter-core` (library: the Chapter 7 temperature logic) and `converter-cli` (binary using clap that depends on core by path). `cargo run -p converter-cli -- --celsius 100` should print the conversions.
5. Recreate the serde+clap example but for a CSV-ish format of your own design (parse manually — no csv crate), with `--filter <tag>` and `--json` output flags. Then run `cargo clippy -- -D warnings` and `cargo fmt` until both are clean; fix what clippy flags rather than allowing it.
