# Project 1: Console Unit Converter

## Description

Build an interactive console program that converts between units: temperatures (Celsius / Fahrenheit / Kelvin), distances (kilometers / miles), and weights (kilograms / pounds). The program runs in a loop: it shows a menu, reads the user's choice and value from stdin, prints the converted result nicely formatted, and repeats until the user quits.

This is your "hello, real program" — small enough to finish in an evening or two, but it forces you through the full edit–compile–fight-the-compiler–run cycle with real user input, which is where Rust actually gets learned.

## Difficulty

**Beginner (in Rust).** Estimated effort: **3–5 hours.**

## Chapters Used

- Chapter 1 (setup, cargo, println!)
- Chapter 2 (numeric types, parsing strings to numbers, format specifiers)
- Chapter 3 (functions, loops, `loop`/`break`, match-on-input basics)
- Light preview of Chapters 8–9 (you will meet `Option`/`Result` through `.parse()` — handle them with `match` as shown in Chapter 3's guessing-game example)

## Requirements Checklist

- [ ] `cargo new unit-converter` project that builds with no warnings
- [ ] A menu printed on start listing at least 6 conversions (C→F, F→C, C→K, km→mi, mi→km, kg→lb) plus a quit option
- [ ] Reads the user's menu choice from stdin; invalid choices re-prompt instead of crashing
- [ ] Reads the value to convert from stdin; non-numeric input re-prompts instead of crashing (no `unwrap()` on the parse in the final version)
- [ ] Each conversion implemented as its own function with signature `fn xxx(value: f64) -> f64`
- [ ] Results printed with exactly 2 decimal places and both units labeled (e.g. `100.00 °C = 212.00 °F`)
- [ ] Temperature conversions reject values below absolute zero with a friendly message
- [ ] The program loops until the user picks quit; quitting prints a goodbye message
- [ ] All numeric work uses `f64`; no `as` casts needed anywhere
- [ ] `cargo fmt` and `cargo clippy` run clean

## Hints

- Chapter 3's guessing-game example contains the exact stdin-reading pattern you need (`std::io::stdin().read_line(&mut input)` plus `trim()` plus `parse()`); wrap it in a helper function like `fn read_f64(prompt: &str) -> f64` that loops until valid.
- `read_line` *appends* to the String and includes the newline — that's why `trim()` matters before parsing.
- A `loop { match choice.as_str() { "1" => ..., "q" => break, _ => ... } }` shape keeps `main` readable.
- Format to 2 decimals with `{:.2}` inside `println!`.
- Absolute zero: -273.15 °C, -459.67 °F, 0 K. Put the check in one helper, not three places.
- If the borrow checker complains about reusing your input String across loop iterations, `input.clear()` before each `read_line` — or just declare the String inside the loop.

## Stretch Goals

- Accept a one-shot mode: `cargo run -- c2f 100` prints the answer and exits (peek at `std::env::args()`).
- Add a "history" feature: remember the last 5 conversions in a `Vec<String>` and add a menu option to print them (a preview of Chapter 10).
- Support compound input like `100C` or `62.5kg` — parse the unit suffix yourself and pick the sensible conversion.
- Let the user chain conversions (C→F→K) and show all steps.
