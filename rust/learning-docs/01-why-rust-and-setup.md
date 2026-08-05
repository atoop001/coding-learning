# Chapter 1: Why Rust, and Getting Set Up

## Overview

Rust is a systems programming language that gives you the low-level control of C or C++ — direct memory management, no garbage collector, predictable performance — while guaranteeing memory safety at compile time. That combination did not exist before Rust, and it is why Rust has been voted the "most loved/admired language" on the Stack Overflow survey for years running, why it is now allowed in the Linux kernel and Windows internals, and why it is a genuine career differentiator.

You already know JavaScript, TypeScript, and Python. Those are *managed* languages: a runtime (V8, the Python interpreter) owns your memory and cleans it up for you with a garbage collector. Rust has no such runtime. Instead, the **compiler** proves at build time that your program manages memory correctly. This is the single biggest mental shift you will make in this track, and it is worth making: understanding it will make you a better programmer in *every* language.

This chapter gets Rust installed on Windows, builds your first program, and explains the vocabulary ("systems language", "memory safety", "zero-cost abstractions") that the rest of the track relies on.

## Definitions & Explanations

### What is a "systems language"?

A systems programming language is one suitable for building the software that other software runs on: operating systems, browsers, databases, game engines, embedded firmware, network services with strict latency budgets. The defining traits:

- **No mandatory runtime or garbage collector.** The program you compile is the program that runs. Nothing pauses your code to sweep memory.
- **Precise control over memory layout.** You decide whether data lives on the stack or the heap, and how structs are laid out.
- **Compiles to native machine code.** No interpreter, no JIT warm-up.

Python and JavaScript are *applications* languages — superb for productivity, but the runtime makes decisions for you and costs you predictability and speed.

### What is "memory safety"?

A memory-safe program can never:

- read or write memory it doesn't own (buffer overflows, out-of-bounds access),
- use memory after it has been freed (use-after-free),
- free the same memory twice (double free),
- have two threads write the same data at the same time unsynchronized (data races).

C and C++ are memory-*unsafe*: these bugs compile fine and blow up at runtime — Microsoft and Google both report that ~70% of their serious security vulnerabilities are memory-safety bugs. Python and JS are memory-safe *because* of the garbage collector. Rust is memory-safe **without** a garbage collector: the compiler's *borrow checker* (Chapters 4–5) statically proves your code can't commit these crimes. If it can't prove it, your code doesn't compile.

> **Mental model shift for Python/JS developers:** In Python, "who cleans up this object?" is not your problem — the GC finds unreachable objects eventually. In Rust, every value has exactly one *owner*, and the compiler knows precisely when that owner goes away, so it inserts the cleanup code at compile time. Same safety, zero runtime cost — but *you* must structure code so the compiler can prove it.

There is an escape hatch: the `unsafe` keyword lets you opt out of some of these compile-time guarantees for the rare cases (raw pointers, FFI, certain low-level optimizations) where the compiler can't prove safety but you can. It is deliberately **out of scope for this track** — everything here is safe Rust, which is also almost everything you'll write day to day.

### Zero-cost abstractions

Rust's iterators, generics, and closures compile down to the same machine code you'd write by hand in C. "You don't pay for what you don't use, and what you do use, you couldn't hand-code any faster." Contrast: a Python list comprehension is elegant but runs through the interpreter; a Rust iterator chain compiles to a tight loop.

### The toolchain vocabulary

| Tool | What it is | Closest analogy |
|---|---|---|
| `rustup` | Toolchain installer/updater | `nvm` / `pyenv` |
| `rustc` | The compiler itself | `tsc` (but produces native binaries) |
| `cargo` | Build tool + package manager + test runner | `npm`/`pnpm` + `vite` + `jest` in one |
| crates.io | The package registry | npm registry / PyPI |
| a *crate* | A compilation unit / package | an npm package |
| `Cargo.toml` | Project manifest | `package.json` |
| `Cargo.lock` | Locked dependency versions | `package-lock.json` |

## Setup on Windows

1. **Install the Visual Studio C++ Build Tools.** Rust on Windows links with the MSVC linker. Download "Build Tools for Visual Studio" from Microsoft, and in the installer select **"Desktop development with C++"**. (rustup will prompt you and can launch this installer for you.)
2. **Install rustup.** Download `rustup-init.exe` from <https://rustup.rs> and run it. Accept the defaults (stable toolchain, MSVC target).
3. **Verify** in a new PowerShell window:

```text
> rustc --version
rustc 1.85.0 (or newer)
> cargo --version
cargo 1.85.0 (or newer)
```

(1.85 is the version that introduced the 2024 edition this track uses; any fresh
rustup install will be far newer than that.)

4. **Editor:** VS Code + the **rust-analyzer** extension. rust-analyzer gives you inline type hints, which are enormously helpful while learning — it shows you the types the compiler infers.
5. **Keep updated** later with `rustup update`.

## Code Examples

### Your first program, the manual way

Create `hello.rs`:

```rust
// hello.rs
// `fn` declares a function. `main` is the entry point of every Rust binary,
// just like it isn't in Python (`if __name__ == "__main__"` is a convention;
// in Rust, main is required and enforced).
fn main() {
    // println! is a MACRO (note the `!`), not a function.
    // Macros generate code at compile time; println! type-checks its
    // format string against its arguments — a compile error, not a runtime
    // surprise, if they mismatch.
    println!("Hello, Rust!");
}
```

Compile and run:

```text
> rustc hello.rs
> .\hello.exe
Hello, Rust!
```

Note what happened: you got a standalone `hello.exe`. No interpreter, no `node_modules`, no venv. You could copy that .exe to any Windows machine and it runs.

**A note on macros:** you'll spend this whole track *using* macros — `println!`, `vec!`, `format!`, and others — recognizable by their trailing `!`. Writing your own with `macro_rules!` is a distinct, more advanced skill (metaprogramming: code that writes code) and is out of scope here. Using them well is all you need.

### The real way: cargo

Nobody invokes `rustc` by hand in practice. Use cargo:

```text
> cargo new hello_cargo
> cd hello_cargo
> cargo run
   Compiling hello_cargo v0.1.0
    Finished dev [unoptimized + debuginfo] target(s)
     Running `target\debug\hello_cargo.exe`
Hello, world!
```

`cargo new` generated:

```text
hello_cargo/
├── Cargo.toml      <- manifest (like package.json)
├── .gitignore      <- cargo even inits a git repo
└── src/
    └── main.rs     <- your code
```

`Cargo.toml`:

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

The commands you will use constantly:

```text
cargo run            # compile (if needed) and run
cargo build          # compile debug build to target/debug
cargo build --release  # optimized build to target/release (much faster code)
cargo check          # type-check WITHOUT producing a binary — fastest feedback
cargo test           # run tests
cargo fmt            # auto-format (like prettier, but universal & uncontroversial)
cargo clippy         # linter with genuinely good advice
```

Habit to build now: run `cargo check` constantly while writing code. It's much faster than a full build.

### The compiler as a teacher

Rust's compiler errors are the best in the industry — they are a core part of how you learn the language. Try this deliberately broken program:

```rust
fn main() {
    let name = "world";
    println!("Hello, {}!", nam); // typo: `nam`
}
```

```text
error[E0425]: cannot find value `nam` in this scope
 --> src\main.rs:3:28
  |
3 |     println!("Hello, {}!", nam);
  |                            ^^^ help: a local variable with a similar name exists: `name`
```

It found the typo *and suggested the fix*. Throughout this track we will intentionally write broken code to read the errors, because in Rust the error messages teach the rules. When an error has a code like `E0425`, you can run `rustc --explain E0425` for a mini-tutorial.

## Common Pitfalls

- **Skipping the C++ Build Tools install.** Symptom: `error: linker 'link.exe' not found`. Fix: install "Desktop development with C++" workload, then restart your terminal.
- **Expecting a REPL.** There is no first-class `python`-style REPL. Your feedback loop is `cargo check`/`cargo run`, which is fast. (The Rust Playground at <https://play.rust-lang.org> is great for quick experiments.)
- **Fighting the compiler emotionally.** Coming from dynamic languages, having the compiler reject working-looking code feels hostile. Reframe it now: every rejection is a runtime bug (in another language) caught before it existed. Rustaceans call the eventual click "if it compiles, it works" — an exaggeration, but a surprisingly mild one.
- **Antivirus slowing builds.** Windows Defender can slow compilation noticeably. Consider excluding your `target\` directories from real-time scanning if builds feel sluggish.
- **Editing without rust-analyzer.** You lose inferred-type hints, which are half your learning feedback. Install it before Chapter 2.

## Practice Exercises

1. Install the toolchain and confirm `rustc --version`, `cargo --version`, and `cargo clippy --version` all work. Create, build, and run a project with `cargo new`.
2. Modify the hello program to print three lines: your name, the language you're coming from, and one thing you want to build with Rust. Use one `println!` per line, then rewrite it as a single `println!` using `\n`.
3. Deliberately break your program three different ways (misspell a variable, remove a semicolon, remove the closing brace of `main`) and read each compiler error carefully. For each, write down (in a comment) what the error code was and what the compiler suggested.
4. Run `cargo build --release` and compare the file sizes and locations of the debug and release executables under `target\`. Then run `rustc --explain E0425` and skim the output.
5. In `Cargo.toml`, change the `name` field to something invalid (e.g., containing a space) and run `cargo check`. What does cargo tell you? Restore it afterward.
