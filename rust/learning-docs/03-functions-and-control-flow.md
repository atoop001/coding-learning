# Chapter 3: Functions & Control Flow

## Overview

Rust's functions and control flow look familiar coming from TypeScript — until you notice that almost everything is an *expression* that produces a value. `if` is an expression. Blocks are expressions. `loop` can return a value. Understanding the expression-oriented style is what makes Rust code read cleanly, and it explains oddities like "why did removing a semicolon fix my code?"

## Definitions & Explanations

### Function syntax

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

- Parameter types are **required** — no inference in signatures. This is deliberate: signatures are contracts, and keeping them explicit means the compiler's inference never leaks across function boundaries (a big reason Rust errors stay local and comprehensible, unlike some TS inference chains).
- Return type after `->`. Omit it and the function returns `()` (unit), Rust's `void`.
- Naming convention: `snake_case` for functions and variables (like Python, unlike JS).

### Statements vs expressions — the key idea

- A **statement** performs an action and produces no value: `let x = 5;`
- An **expression** evaluates to a value: `5`, `a + b`, `add(1, 2)`, `if cond { 1 } else { 2 }`, and even a block `{ ... }`.

The last expression in a block — *without a trailing semicolon* — is the block's value:

```rust
let y = {
    let x = 3;
    x + 1        // no semicolon: this is the block's value
};               // y == 4
```

Adding a semicolon turns an expression into a statement that evaluates to `()`. This is why Rust functions usually end with a bare expression instead of `return`:

```rust
fn double(x: i32) -> i32 {
    x * 2      // idiomatic
    // `return x * 2;` also works, but is used mainly for EARLY returns
}
```

> **Python/JS contrast:** In Python, `if` is a statement; you need the ternary `a if cond else b` for expression form. In Rust there is no ternary because `if` already *is* one. JS arrow functions have "implicit return" for one-liners; Rust has it for every block.

### `if` / `else if` / `else`

```rust
let n = 6;
if n % 4 == 0 {
    println!("divisible by 4");
} else if n % 2 == 0 {
    println!("even");
} else {
    println!("odd");
}
```

- No parentheses around the condition; braces are **mandatory** (no brace-less single statements — a whole class of C bugs eliminated).
- The condition must be a `bool`. `if n { }` → `error[E0308]: mismatched types — expected bool, found integer`.
- As an expression, both arms must produce the **same type**:

```rust
let label = if n % 2 == 0 { "even" } else { "odd" };
```

```rust
// This does NOT compile:
let x = if cond { 5 } else { "five" };
// error[E0308]: `if` and `else` have incompatible types
```

### The three loops

```rust
// 1. `loop` — infinite until `break`. break can carry a value!
let mut attempts = 0;
let result = loop {
    attempts += 1;
    if attempts == 3 {
        break attempts * 10;   // `loop` is an expression; this is its value
    }
};

// 2. `while` — condition-driven
let mut countdown = 3;
while countdown > 0 {
    println!("{countdown}!");
    countdown -= 1;            // note: no ++ or -- in Rust
}

// 3. `for` — iterate over anything iterable (the workhorse; ~95% of loops)
for i in 0..5 {                // range 0,1,2,3,4  (end-exclusive, like Python range)
    println!("i = {i}");
}
for i in (1..=3).rev() {       // 3,2,1 — inclusive range, reversed
    println!("{i}");
}
```

`for` in Rust is Python's `for`, not C's: it iterates a collection or range. There is no C-style `for (i = 0; i < n; i++)` — use ranges or iterators. Also note: Rust has no `++`/`--` operators; write `x += 1`.

### Loop labels

Nested loops can break/continue an *outer* loop by label — cleaner than Python's flag-variable workaround:

```rust
'outer: for x in 0..5 {
    for y in 0..5 {
        if x * y > 6 {
            break 'outer;   // exits BOTH loops
        }
    }
}
```

## Code Examples

### FizzBuzz, expression style

```rust
fn fizzbuzz(n: u32) -> String {
    // The whole function body is one `if` expression.
    // `format!` builds a String (like a template literal / f-string).
    if n % 15 == 0 {
        String::from("FizzBuzz")
    } else if n % 3 == 0 {
        String::from("Fizz")
    } else if n % 5 == 0 {
        String::from("Buzz")
    } else {
        format!("{n}")
    }
}

fn main() {
    for n in 1..=20 {
        println!("{}", fizzbuzz(n));
    }
}
```

### Early return for guard clauses

```rust
fn describe_grade(score: u32) -> &'static str {
    // (&'static str = a string literal; details in Chapters 6 and 12.)
    if score > 100 {
        return "invalid"; // early exit: `return` earns its keep here
    }
    if score >= 90 { "A" }
    else if score >= 80 { "B" }
    else if score >= 70 { "C" }
    else { "F" }
}

fn main() {
    println!("{}", describe_grade(85)); // B
    println!("{}", describe_grade(500)); // invalid
}
```

### The missing-semicolon error (learn to read this one now)

```rust
fn double(x: i32) -> i32 {
    x * 2;   // <- accidental semicolon
}
```

```text
error[E0308]: mismatched types
 --> src\main.rs:1:22
  |
1 | fn double(x: i32) -> i32 {
  |    ------            ^^^ expected `i32`, found `()`
  |
2 |     x * 2;
  |          - help: remove this semicolon to return this value
```

The semicolon turned the expression into a statement, so the function body evaluates to `()` — but the signature promises `i32`. The compiler tells you the exact fix. You *will* hit this error; now you know what it means.

### A realistic function: interactive guessing loop

```rust
use std::io::Write; // brings the flush() method into scope (Chapter 11 explains why)

fn read_number(prompt: &str) -> u32 {
    loop {
        print!("{prompt}");
        std::io::stdout().flush().unwrap(); // print! doesn't flush; println! does
        let mut input = String::new();
        std::io::stdin().read_line(&mut input).unwrap();
        // If parsing fails, loop again; if it succeeds, break WITH the value.
        match input.trim().parse() {
            Ok(n) => break n,          // match: full treatment in Chapter 8
            Err(_) => println!("Please enter a whole number."),
        }
    }
}

fn main() {
    let secret = 7;
    loop {
        let guess = read_number("Guess (1-10): ");
        if guess == secret {
            println!("Correct!");
            break;
        } else if guess < secret {
            println!("Too low.");
        } else {
            println!("Too high.");
        }
    }
}
```

Don't worry about `&mut`, `match`, `Ok`/`Err`, or `unwrap` yet — they're Chapters 5, 8, and 9. The takeaway here is the *shape*: `loop` + `break value` is Rust's idiom for "retry until valid input."

## Common Pitfalls

- **Trailing semicolon on the return expression.** The E0308 "expected i32, found `()`" error above. Remove the semicolon.
- **`if` without `else` used as an expression.** `let x = if cond { 5 };` fails — if `cond` is false there'd be no value. The compiler says: `` `if` may be missing an `else` clause``. Either add `else` or don't use it as an expression.
- **Mismatched arm types.** Both branches of an expression `if` (and later, all `match` arms) must be the same type. `{ 5 } else { "five" }` is E0308.
- **Trying to write C-style `for`.** `for i = 0; i < n; i += 1` is a syntax error. Use `for i in 0..n`.
- **Off-by-one with ranges.** `0..5` excludes 5; `0..=5` includes it. Same convention as Python's `range` (exclusive), plus an inclusive form Python lacks.
- **Using `while` with manual indices to walk a collection.** It compiles, but it's slower (bounds checks) and non-idiomatic. `for item in collection` — and later, iterators — is the Rust way.
- **Expecting `print!` to show immediately.** `print!` (no newline) is line-buffered; call `.flush()` or your prompt appears after the user types.

## Practice Exercises

1. Write `fn is_leap_year(year: u32) -> bool` using a single boolean expression (no `if`). Test it on 1900, 2000, 2024, and 2025 in `main`.
2. Write a function `celsius_to_fahrenheit(c: f64) -> f64` and use a `for` loop over an inclusive range to print a conversion table from -10°C to 40°C in steps of 5 (hint: `step_by`).
3. Rewrite this Python in idiomatic Rust, without a `mut` accumulator flag: *"loop numbers 1..100; find the first number divisible by both 7 and 9 and store it in a variable"*. Use `loop`/`break value` or a labeled loop.
4. Write `fn collatz_steps(mut n: u64) -> u32` that counts steps for n to reach 1 (n → n/2 if even, 3n+1 if odd). Use a `while` loop. Print steps for 6, 27, and 97.
5. Deliberately create each of these compile errors, read the message, then fix it: (a) trailing semicolon on a returned expression, (b) `if` expression with incompatible arm types, (c) non-bool condition. Keep the error text in a comment above each fix.
