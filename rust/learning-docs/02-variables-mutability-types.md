# Chapter 2: Variables, Mutability & Basic Types

## Overview

This chapter covers how Rust binds names to values, why everything is *immutable by default*, and the primitive type system. If TypeScript taught you that types catch bugs, Rust takes the idea much further: types are checked, inferred, and there is no `any`, no `null`, and no implicit conversion — not even from `i32` to `i64`. The habits you build here (asking "what type is this, exactly, and can it change?") are the foundation for ownership in Chapter 4.

## Definitions & Explanations

### `let` bindings and immutability by default

```rust
let x = 5;
```

This looks like JavaScript, but the defaults are flipped. In JS, `let` means mutable and `const` means immutable *binding* (the object behind it can still mutate!). In Rust:

- `let x = 5;` — **immutable**. You cannot reassign `x`, ever.
- `let mut x = 5;` — mutable. You may reassign.
- `const MAX_POINTS: u32 = 100_000;` — a true compile-time constant: type annotation required, value must be computable at compile time, name conventionally SCREAMING_SNAKE_CASE.

Why immutable by default? Because most values never need to change, and when the compiler *knows* a value can't change, whole categories of bugs (and, later, data races between threads) become impossible. In Rust you must *ask* for mutability, which documents intent: `mut` in a signature is a signal to every reader.

> **Python/JS contrast:** In Python, everything is mutable unless the type happens to be immutable (tuples, strings). In Rust, mutability is a property of the *binding*, declared up front, and enforced.

### Type inference and annotations

Rust is statically typed but you rarely write types locally — the compiler infers them:

```rust
let count = 42;          // inferred i32
let price = 9.99;        // inferred f64
let active = true;       // bool
let letter = 'A';        // char (note: single quotes, 4-byte Unicode scalar)
let annotated: u64 = 42; // explicit annotation when you need a specific type
```

Unlike TypeScript, inference in Rust is *bidirectional* — later usage can determine an earlier type. And unlike TS, there is no escape hatch: no `any`, no `as unknown as`. If the compiler can't determine a type, it asks you to annotate (common with `.parse()`, see below).

### The integer types

| Length | Signed | Unsigned |
|---|---|---|
| 8-bit | `i8` | `u8` |
| 16-bit | `i16` | `u16` |
| 32-bit | `i32` | `u32` |
| 64-bit | `i64` | `u64` |
| 128-bit | `i128` | `u128` |
| pointer-size | `isize` | `usize` |

- Default integer is `i32`. Default float is `f64`.
- `usize` is the type used for indexing and lengths (it's 64-bit on your machine). You will meet it constantly with collections.
- Underscores are legal separators: `1_000_000`.
- Literals can carry their type: `57u8`, `1_000i64`.

**Why so many?** This is systems programming: you control exactly how many bytes each number occupies. Python's `int` is arbitrary-precision (a heap object!); JS numbers are all `f64` doubles. Rust numbers are raw machine integers — fast, fixed-size, and they can **overflow**. In debug builds, overflow panics (crashes with a clear message); in release builds it wraps around silently. If you *want* wrapping or saturating math, say so explicitly: `x.wrapping_add(1)`, `x.saturating_sub(1)`, `x.checked_add(1)` (returns an `Option`, Chapter 8).

### No implicit conversions — ever

```rust
let a: i32 = 5;
let b: i64 = 10;
// let c = a + b;      // ERROR: mismatched types
let c = a as i64 + b;  // OK: explicit cast with `as`
```

Coming from JS, where `"5" + 3` is `"53"`, this strictness is a gift. `as` casts are explicit and can truncate (`300_i32 as u8` is `44`), so prefer `i64::from(a)` or `.try_into()` when you want checked conversions.

### Other primitives

- **`bool`** — `true` / `false`. Only `bool` works in conditions: `if 1 { }` is a compile error. No truthiness. No `if some_list:` idiom — you write `if !list.is_empty()`.
- **`char`** — a Unicode scalar value (4 bytes), written `'a'`, `'🎉'`. Not the same as a 1-character string.
- **Tuples** — fixed-size, mixed types: `let pair: (i32, f64) = (1, 2.5);` Access by `.0`, `.1`, or destructure: `let (x, y) = pair;`. The empty tuple `()` is called *unit* and is Rust's "no value" (what functions return when they return nothing — closer to `void` than to `None`).
- **Arrays** — fixed length, on the stack: `let a: [i32; 3] = [1, 2, 3];` Length is part of the type. Out-of-bounds indexing panics at runtime (safely — no buffer overflow) or errors at compile time if the index is a constant. Growable lists are `Vec<T>` (Chapter 10).

### Shadowing

Rust lets you re-declare a name with `let`, even changing its type:

```rust
let input = "42";            // &str
let input: i32 = input.trim().parse().expect("not a number"); // now i32
```

This is *not* mutation — it's a new binding that shadows the old one. It's idiomatic for pipelines like "raw string → parsed value" where the intermediate form should become inaccessible. Python allows rebinding too, but silently and without types; Rust's version is deliberate and type-checked.

## Code Examples

### Mutability enforced

```rust
fn main() {
    let x = 5;
    println!("x = {x}");   // {x} inline capture works in modern Rust
    x = 6;                 // <- compile error
}
```

```text
error[E0384]: cannot assign twice to immutable variable `x`
 --> src\main.rs:4:5
  |
2 |     let x = 5;
  |         - first assignment to `x`
  |         |
  |         help: consider making this binding mutable: `mut x`
3 |     println!("x = {x}");
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
```

The fix is exactly what the compiler says:

```rust
fn main() {
    let mut x = 5;
    println!("x = {x}");
    x = 6;
    println!("x = {x}");
}
```

### Types are strict, inference is smart

```rust
fn main() {
    // Inference flows backward: `parse` can produce many types,
    // so we must say which one we want — either on the binding...
    let port: u16 = "8080".parse().expect("invalid port");

    // ...or with the "turbofish" syntax on the call:
    let retries = "3".parse::<u32>().expect("invalid number");

    println!("port {port}, retries {retries}");
}
```

Remove the `: u16` and the turbofish and you get:

```text
error[E0284]: type annotations needed
  = note: cannot satisfy `<_ as FromStr>::Err == _`
help: consider giving `port` an explicit type
```

### A realistic example: unit-aware arithmetic

```rust
fn main() {
    const SECONDS_PER_MINUTE: u32 = 60;
    const MINUTES_PER_HOUR: u32 = 60;

    let total_seconds: u32 = 5_000;

    // Integer division and remainder, like Python's // and %
    let hours = total_seconds / (SECONDS_PER_MINUTE * MINUTES_PER_HOUR);
    let minutes = (total_seconds / SECONDS_PER_MINUTE) % MINUTES_PER_HOUR;
    let seconds = total_seconds % SECONDS_PER_MINUTE;

    println!("{total_seconds}s = {hours}h {minutes}m {seconds}s");

    // Floats and ints don't mix implicitly:
    let ratio = total_seconds as f64 / 3600.0;
    println!("= {ratio:.2} hours"); // format spec: 2 decimal places
}
```

### Tuples and destructuring

```rust
fn min_max(values: &[i32]) -> (i32, i32) {
    // (Don't worry about &[i32] yet — Chapter 6. It's "a view of some i32s".)
    let mut min = values[0];
    let mut max = values[0];
    for &v in values {
        if v < min { min = v; }
        if v > max { max = v; }
    }
    (min, max) // no `return` needed — last expression is the value (Chapter 3)
}

fn main() {
    let data = [3, 9, -2, 7];
    let (lo, hi) = min_max(&data); // destructure the tuple
    println!("min {lo}, max {hi}");
}
```

## Common Pitfalls

- **Reaching for `mut` everywhere.** Beginners sprinkle `mut` to silence errors. Resist: most `mut` can be removed by restructuring (shadowing, expressions, iterators). The compiler even warns you: `variable does not need to be mutable`.
- **Expecting truthiness.** `if count { ... }` doesn't compile. Write the comparison: `if count != 0`. This feels verbose for a week and then reads as clearer forever.
- **Integer division surprise.** `5 / 2` is `2` (both are integers). For `2.5`, make them floats: `5.0 / 2.0`, or cast: `5 as f64 / 2 as f64`.
- **`as` truncates silently.** `let small = 1000_i32 as u8;` compiles and yields `232`. For safety use `u8::try_from(1000)` which returns a `Result` you must handle (Chapter 9).
- **Confusing `char` and `&str`.** `'a'` is a char; `"a"` is a string. `"a" == 'a'` is a type error. Python has no char type, so this trips people.
- **Shadowing a `mut` when you meant to assign.** `let mut x = 1; let x = 2;` — the second line creates a *new immutable* `x`. If you then try `x = 3`, you'll get the E0384 error and wonder where your `mut` went.

## Practice Exercises

1. Write a program that declares an immutable binding, tries to reassign it, and then fix it two different ways: (a) with `mut`, (b) with shadowing. Add a comment stating when you'd prefer each.
2. Write a program that converts a temperature stored as `f64` Celsius into Fahrenheit and Kelvin, printing each with exactly 1 decimal place using format specifiers.
3. Predict — then verify — the output/behavior of: `255_u8 + 1` in a `cargo run` (debug) build. Then rewrite it with `wrapping_add` and `checked_add` and print the results.
4. Create a tuple `(name, age, height_m)` with types `(&str, u8, f64)`. Destructure it into three variables and print a sentence. Then try to compare `age` against `height_m` directly and read the compiler error; fix it with an explicit conversion.
5. Take the string `"  49  "` and, using shadowing for each step, trim it and parse it into a `u32`, then print its square. Do not introduce any new variable names.
