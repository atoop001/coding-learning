# Chapter 8: Enums & Pattern Matching (Option, match, exhaustiveness)

## Overview

Rust enums are not the anemic enums of TypeScript or C. They are full *algebraic data types*: each variant can carry its own data, and the `match` expression forces you to handle every variant. Together they eliminate two entire bug families you've lived with for years: "forgot a case" and — via `Option<T>` — the billion-dollar mistake, `null`/`undefined`/`None` errors. Rust has **no null**. This chapter shows what replaces it and why the replacement is strictly better.

## Definitions & Explanations

### Enums with data

```rust
enum Shape {
    Circle { radius: f64 },              // struct-like variant
    Rectangle { width: f64, height: f64 },
    Triangle(f64, f64, f64),             // tuple-like variant
    Point,                               // unit variant (no data)
}
```

A `Shape` value is *exactly one* of these variants at a time, tagged so the program always knows which. TS folks: this is a **discriminated union** (`type Shape = {kind:"circle", radius:number} | ...`) with the discriminant handled for you. Python folks: it's what you fake with class hierarchies or `Union[...]` + isinstance chains — but closed, checked, and impossible to leave half-handled.

Enums can have methods too — `impl Shape { fn area(&self) -> f64 { ... } }` works exactly like structs.

### `match`: exhaustive pattern matching

```rust
fn area(shape: &Shape) -> f64 {
    match shape {
        Shape::Circle { radius } => std::f64::consts::PI * radius * radius,
        Shape::Rectangle { width, height } => width * height,
        Shape::Triangle(a, b, c) => {           // Heron's formula; blocks are fine
            let s = (a + b + c) / 2.0;
            (s * (s - a) * (s - b) * (s - c)).sqrt()
        }
        Shape::Point => 0.0,
    }
}
```

Key properties:

- **Destructuring**: patterns pull the variant's data into variables (`radius`, `width`...).
- **Exhaustiveness**: delete the `Shape::Point` arm and you get:

```text
error[E0004]: non-exhaustive patterns: `&Shape::Point` not covered
  = note: the matched value is of type `&Shape`
help: ensure that all possible cases are being handled
```

This is the superpower. Add a variant next month, and the compiler lists *every match in the codebase* that must now handle it. In TS you approximate this with `never`-checking tricks; in Python you hope your tests catch it. In Rust it's the default.

- `match` is an expression (all arms must have the same type) — like everything else in Rust.
- `_` is the wildcard ("anything else"): useful, but every `_` arm trades away exhaustiveness checking. Use deliberately.
- Patterns go further: literals (`1 => ...`), ranges (`1..=5 => ...`), or-patterns (`'a' | 'e' | 'i' => ...`), guards (`n if n < 0 => ...`), bindings on tuples/structs, nesting.

### `Option<T>`: the end of null

Rust's standard library defines:

```rust
enum Option<T> {     // T is a generic parameter (Chapter 11) — works for any type
    Some(T),
    None,
}
```

There is no null in Rust. A `String` is *always* a valid string. When a value might be absent, the type says so: `Option<String>`. And because `Option<T>` is a *different type* from `T`, you cannot forget to check:

```rust
let maybe_name: Option<String> = Some(String::from("Ada"));
// let len = maybe_name.len();   // ERROR: Option<String> has no method `len` —
                                  // the compiler forces you to handle None first
let len = match &maybe_name {
    Some(name) => name.len(),
    None => 0,
};
```

> **The contrast that matters:** In JS, `user.name.length` throws `TypeError: Cannot read properties of undefined` *at runtime, in production, on Saturday*. In Python it's `AttributeError: 'NoneType'`. TypeScript's `string | null` with strict null checks is the same idea as `Option` — but TS has `!` to bypass it and `any` to smuggle nulls. Rust has no bypass: absence is in the type, and handling it is compile-enforced.

The everyday `Option` toolkit (learn these — they replace 90% of matches):

```rust
let x: Option<i32> = Some(5);
x.is_some(); x.is_none();          // checks
x.unwrap();                        // the value, or PANIC if None (crash). Prototyping only.
x.expect("config missing");        // unwrap with a message — always prefer over unwrap
x.unwrap_or(0);                    // the value, or a default
x.unwrap_or_else(|| compute());    // default computed lazily (closure, Ch 13)
x.map(|n| n * 2);                  // transform the inside if Some -> Option<i32>
x.and_then(|n| checked_op(n));     // chain operations that themselves return Option
```

Std returns `Option` everywhere: `vec.first()`, `vec.get(i)`, `map.get(&k)`, `iterator.next()`, `s.find('x')`, `s.split_once('=')`...

### `if let` / `let else` / `while let`: ergonomic single-pattern matching

A full `match` to handle one interesting case is noisy. Sugar:

```rust
// if let: "if it matches this pattern, bind and run"
if let Some(name) = &maybe_name {
    println!("hello {name}");
} else {
    println!("hello anonymous");
}

// let-else: "bind it or bail" — great for guard clauses
fn process(input: Option<&str>) {
    let Some(text) = input else {
        println!("nothing to process");
        return;                      // else block MUST diverge (return/panic/continue)
    };
    println!("processing {text}");   // text is a plain &str from here on
}

// while let: loop while a pattern keeps matching
let mut stack = vec![1, 2, 3];
while let Some(top) = stack.pop() {   // pop returns Option<i32>
    println!("{top}");
}
```

## Code Examples

### Modeling a domain: no invalid states

```rust
#[derive(Debug)]
enum PaymentMethod {
    Cash,
    Card { number: String, expiry: String },
    Transfer { iban: String },
}

#[derive(Debug)]
struct Order {
    id: u32,
    payment: PaymentMethod,
    coupon: Option<String>,     // absence is explicit and typed
}

fn describe(order: &Order) -> String {
    let pay = match &order.payment {
        PaymentMethod::Cash => String::from("cash"),
        PaymentMethod::Card { number, .. } => {          // `..` ignores other fields
            // show last 4 digits only
            let last4 = &number[number.len().saturating_sub(4)..];
            format!("card ending {last4}")
        }
        PaymentMethod::Transfer { iban } => format!("transfer from {iban}"),
    };
    match &order.coupon {
        Some(code) => format!("order {} paid by {pay} with coupon {code}", order.id),
        None => format!("order {} paid by {pay}", order.id),
    }
}

fn main() {
    let order = Order {
        id: 7,
        payment: PaymentMethod::Card {
            number: String::from("4242424242424242"),
            expiry: String::from("12/27"),
        },
        coupon: None,
    };
    println!("{}", describe(&order));
}
```

Notice what's *impossible* here: a Card payment without a number, a Cash payment carrying an IBAN, a coupon field holding `undefined`-but-not-really. "Make invalid states unrepresentable" is a core Rust design mantra, and enums are how.

### Guards, ranges, and bindings

```rust
fn categorize(n: i32) -> &'static str {
    match n {
        0 => "zero",
        1..=9 => "single digit",
        n if n < 0 => "negative",           // guard: arbitrary condition
        10 | 100 | 1000 => "round number",  // or-pattern
        _ => "something else",
    }
}

fn main() {
    for n in [-5, 0, 7, 100, 42] {
        println!("{n}: {}", categorize(n));
    }
}
```

Arms are checked top-to-bottom; first match wins. Guards (`if` in an arm) opt that arm out of exhaustiveness counting, so a final catch-all is still required.

### Option in a realistic pipeline

```rust
struct Config {
    entries: Vec<(String, String)>,
}

impl Config {
    fn get(&self, key: &str) -> Option<&str> {
        self.entries
            .iter()
            .find(|(k, _)| k == key)       // find returns Option<&(String,String)>
            .map(|(_, v)| v.as_str())      // transform inside the Option
    }
}

fn main() {
    let cfg = Config {
        entries: vec![
            ("port".into(), "8080".into()),
            ("host".into(), "localhost".into()),
        ],
    };

    // Chain: get -> parse -> default. No null checks, no exceptions.
    let port: u16 = cfg
        .get("port")
        .and_then(|v| v.parse().ok())   // parse gives Result; .ok() -> Option (Ch 9)
        .unwrap_or(3000);

    println!("port = {port}");
    println!("timeout = {:?}", cfg.get("timeout")); // None — visibly, safely
}
```

## Common Pitfalls

- **`unwrap()` everywhere.** Each `unwrap` is a latent crash with a useless message. Ladder of virtue: `unwrap()` → `expect("why I believe this is Some")` → `unwrap_or`/`if let`/`match` → restructure so the Option never occurs. Move up it as you learn.
- **Matching on a reference and fighting moves.** `match some_string_option { Some(s) => ... }` *moves* the String out — then the Option is partially moved and unusable. Match on `&opt` (or call `.as_ref()`): patterns then bind references. If you see E0382 around a match, this is usually it.
- **Forgetting `match` arms must agree in type.** One arm returns `String`, another `&str` → E0308. Unify (usually `.to_string()` the borrowed one).
- **Over-using `_` catch-alls.** `_ => {}` today means no compiler help when you add a variant next month. Prefer listing variants; reserve `_` for genuinely open sets (like matching on integers).
- **Reinventing Option methods with verbose matches.** A `match` that maps `Some(x)` to `Some(f(x))` and `None` to `None` *is* `.map(f)`. When your match arms look mechanical, a method exists. Clippy will often tell you which.
- **TS instinct: "I'll just add `| null`".** In Rust you never widen a type to include absence — you wrap in `Option`. The difference: `Option<Option<T>>` is representable and meaningful ("key present but value empty" vs "key absent"), whereas `null | null` collapses. This precision becomes genuinely useful in real APIs.

## Practice Exercises

1. Define `enum Command { Go { x: i32, y: i32 }, Turn(f64), Stop, Report }` and `fn execute(cmd: &Command, position: &mut (i32, i32))` using an exhaustive match (`Report` prints the position; `Turn` may just print for now). Then add a new variant `Reverse` and follow the compiler errors to every place that must change.
2. Write `fn safe_divide(a: f64, b: f64) -> Option<f64>` (None for division by zero or non-finite results). Chain it: compute `a/b/c` for user-supplied values using `and_then`, printing a friendly message on None. No `unwrap` allowed.
3. Model a traffic light: `enum Light { Red, Yellow, Green }` with method `next(&self) -> Light` (match) and `duration_secs(&self) -> u32`. Simulate 10 transitions in a loop starting from Red.
4. Write `fn first_char_uppercased(s: &str) -> Option<char>` returning the first character uppercased, using only `Option` combinators (`chars().next()`, `map`) — no `match`, no `if`. Then rewrite it as an explicit `match` and compare readability in a comment.
5. Build `fn parse_command(input: &str) -> Option<Command>` (for the enum from exercise 1) parsing lines like `"go 3 4"`, `"turn 90"`, `"stop"`, `"report"`. Use `split_whitespace`, nested `match`/`if let`, and `?`-free Option handling. Bad input → None, never panic. Feed it a mix of valid and invalid lines from `main`.
