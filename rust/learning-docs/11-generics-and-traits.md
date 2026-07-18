# Chapter 11: Generics & Traits

## Overview

You've been *using* generics (`Vec<T>`, `Option<T>`, `Result<T, E>`) and traits (`Copy`, `Debug`, `Clone`) for chapters. Now you learn to define your own. Generics are Rust's parametric polymorphism — TypeScript's `<T>` will feel familiar. Traits are Rust's answer to interfaces *and* to inheritance *and* to Python's duck typing — but checked at compile time and, crucially, implementable for types you didn't write. Together they are how Rust code gets reused, and they compile to code as fast as hand-written specializations (monomorphization — zero cost).

## Definitions & Explanations

### Generic functions and structs

```rust
// A function generic over T. The <T: PartialOrd> is a BOUND (see below):
// "any T, as long as it can be compared."
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

// A generic struct, and methods on it:
struct Pair<T> {
    first: T,
    second: T,
}

impl<T: PartialOrd> Pair<T> {                // impl<T> declares the parameter
    fn larger(&self) -> &T {
        if self.first > self.second { &self.first } else { &self.second }
    }
}

fn main() {
    println!("{}", largest(&[3, 7, 2]));          // T = i32
    println!("{}", largest(&["a", "zz", "m"]));   // T = &str — same function
    let p = Pair { first: 1.5, second: 9.9 };
    println!("{}", p.larger());
}
```

Unlike TypeScript — where generics are erased and anything goes at runtime — Rust **monomorphizes**: the compiler stamps out a specialized copy of `largest` for each concrete `T` used. Result: generic code runs exactly as fast as if you'd written `largest_i32` and `largest_str` by hand.

The other big difference from TS/Python: **you can't do anything with a bare `T`.** In TS, you can call any method and hope; in Python, duck typing shrugs. In Rust, `T` starts with *no* capabilities:

```rust
fn largest<T>(list: &[T]) -> &T { ... if item > largest ... }
// error[E0369]: binary operation `>` cannot be applied to type `&T`
// help: consider restricting type parameter `T`: `T: std::cmp::PartialOrd`
```

Capabilities come from **bounds** — and the compiler tells you which one you need.

### Traits: shared behavior as a contract

A trait declares methods a type promises to provide:

```rust
trait Summary {
    fn summarize(&self) -> String;                 // required method

    fn preview(&self) -> String {                  // DEFAULT method — free for
        format!("{}...", &self.summarize()[..10.min(self.summarize().len())])
    }                                              // implementors, overridable
}

struct Article { title: String, body: String }
struct Tweet { user: String, text: String }

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}: {}", self.title, self.body)
    }
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("@{}: {}", self.user, self.text)
    }
    // preview() inherited from the default
}
```

Compared to what you know:

- **vs TS interfaces:** same idea, but implementation is *explicit* (`impl Summary for Tweet`), not structural. A type doesn't accidentally satisfy a trait by having the right method names.
- **vs Python duck typing:** the contract is checked at compile time; "quacks like a duck" becomes "certified as a duck, or it doesn't compile."
- **vs inheritance:** traits carry behavior (default methods) but no state, no hierarchy. This is composition-first design enforced by the language.
- **The killer feature:** you can implement *your* trait for *someone else's* type (`impl Summary for String`), or someone else's trait for your type (`impl Display for Article`). The only rule (*orphan rule*): at least one of the trait or the type must be local to your crate — this keeps the ecosystem coherent.

### Trait bounds: generics + traits

```rust
// Three equivalent spellings, terse -> explicit:
fn notify(item: &impl Summary) { ... }                       // impl Trait sugar
fn notify<T: Summary>(item: &T) { ... }                      // classic bound
fn notify<T>(item: &T) where T: Summary { ... }              // where clause (scales best)

// Multiple bounds with +:
fn log_and_compare<T: Summary + PartialOrd>(a: &T, b: &T) { ... }

// Returning "some type that implements a trait" without naming it:
fn make_summarizable() -> impl Summary {
    Tweet { user: "rustlang".into(), text: "1.0 forever".into() }
}
```

### Static vs dynamic dispatch: `T: Trait` vs `dyn Trait`

Generics give **static dispatch**: each concrete type gets its own compiled copy; calls are direct (and inlinable). But sometimes you want *one collection of differently-typed things*:

```rust
// A Vec of "anything Summary" — types differ, so sizes differ, so we need
// indirection: Box (heap pointer, Ch 14) + dyn (dynamic dispatch via vtable).
let feed: Vec<Box<dyn Summary>> = vec![
    Box::new(Article { title: "Rust 1.0".into(), body: "...".into() }),
    Box::new(Tweet { user: "ada".into(), text: "hi".into() }),
];
for item in &feed {
    println!("{}", item.summarize());   // method resolved at RUNTIME via vtable
}
```

This is the closest Rust gets to classic OO polymorphism. Trade-offs: `dyn` costs a pointer hop per call and disallows some traits (those with generic methods); generics cost compile time and binary size. Default to generics; reach for `dyn` when you need heterogeneous collections or to avoid generic infection through APIs. (You already met `Box<dyn Error>` in Chapter 9 — now you know what it means.)

### The traits you'll implement or derive constantly

| Trait | Gives you | Usually |
|---|---|---|
| `Debug` | `{:?}` printing | derive |
| `Clone` | `.clone()` | derive |
| `Copy` | implicit copy semantics (Ch 4) | derive (small types only) |
| `PartialEq`/`Eq` | `==` | derive |
| `PartialOrd`/`Ord` | `<`, `sort()` | derive |
| `Hash` | usable as HashMap key | derive |
| `Default` | `T::default()` | derive |
| `Display` | `{}` printing | **manual** impl |
| `From`/`Into` | conversions, `?` error conversion (Ch 9) | manual |
| `Iterator` | `for` loops, adapters (Ch 13) | manual |

`impl Display` is the canonical first manual trait impl:

```rust
use std::fmt;

impl fmt::Display for Tweet {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "@{}: {}", self.user, self.text)
    }
}
// Now println!("{tweet}") works — and so does .to_string(), for free
// (blanket impl: ToString is auto-implemented for everything with Display).
```

That last line demonstrates a **blanket implementation** — `impl<T: Display> ToString for T` in std. Traits implemented *for all types satisfying a bound* are how the std library multiplies your one impl into many features.

## Code Examples

### A realistic trait: pluggable output formats

```rust
use std::fmt;

#[derive(Debug)]
struct Report {
    title: String,
    rows: Vec<(String, f64)>,
}

trait Render {
    fn render(&self, report: &Report) -> String;
}

struct PlainText;
struct Csv;

impl Render for PlainText {
    fn render(&self, r: &Report) -> String {
        let mut out = format!("== {} ==\n", r.title);
        for (label, value) in &r.rows {
            out.push_str(&format!("{label:<12} {value:>8.2}\n"));
        }
        out
    }
}

impl Render for Csv {
    fn render(&self, r: &Report) -> String {
        let mut out = String::from("label,value\n");
        for (label, value) in &r.rows {
            out.push_str(&format!("{label},{value}\n"));
        }
        out
    }
}

// Generic over ANY renderer — static dispatch, zero overhead:
fn print_report<R: Render>(renderer: &R, report: &Report) {
    println!("{}", renderer.render(report));
}

fn main() {
    let report = Report {
        title: "Q3 Sales".into(),
        rows: vec![("North".into(), 1250.5), ("South".into(), 980.0)],
    };
    print_report(&PlainText, &report);
    print_report(&Csv, &report);

    // Or choose at runtime — dynamic dispatch:
    let choice = "csv";
    let renderer: Box<dyn Render> = match choice {
        "csv" => Box::new(Csv),
        _ => Box::new(PlainText),
    };
    println!("{}", renderer.render(&report));
}
```

This pattern — a trait for the *strategy*, structs for each implementation — replaces both the class hierarchies you'd write in Python and the function-props you'd pass in JS, with compile-time checking either way.

### From/Into in practice

```rust
struct Celsius(f64);
struct Fahrenheit(f64);

impl From<Celsius> for Fahrenheit {
    fn from(c: Celsius) -> Self {
        Fahrenheit(c.0 * 9.0 / 5.0 + 32.0)
    }
}

fn main() {
    let c = Celsius(100.0);
    let f: Fahrenheit = c.into();      // Into<T> is FREE once From is written
    println!("{}", f.0);                // 212
    let f2 = Fahrenheit::from(Celsius(0.0));
    println!("{}", f2.0);               // 32
}
```

One `From` impl buys you: `Fahrenheit::from(c)`, `c.into()`, and `?`-conversion if these were error types. This is why Chapter 9's custom errors implemented `From`.

### Reading a bound error (you'll see this weekly)

```rust
#[derive(Debug)]
struct Point { x: i32, y: i32 }

fn main() {
    let points = vec![Point { x: 1, y: 2 }, Point { x: 0, y: 0 }];
    let mut sorted = points;
    sorted.sort();
}
```

```text
error[E0277]: the trait bound `Point: Ord` is not satisfied
   --> src\main.rs:7:12
    |
7   |     sorted.sort();
    |            ^^^^ the trait `Ord` is not implemented for `Point`
    = help: the following other types implement trait `Ord`: ...
help: consider annotating `Point` with `#[derive(PartialEq, Eq, PartialOrd, Ord)]`
```

The method exists only when the bound holds (`impl<T: Ord> Vec<T> { fn sort... }`). The fix is in the help text: derive the ordering traits (fields compare lexicographically, in declaration order), or use `sort_by_key(|p| (p.x, p.y))` for explicit control.

## Common Pitfalls

- **E0277 "trait bound not satisfied."** The defining error of this chapter. Read the *help* section — it nearly always names the derive or bound to add. If it names a trait you've never heard of, that trait's doc page is your next stop.
- **Forgetting bounds on generic code.** Writing `fn f<T>(x: T)` then using `x == y`, `x.clone()`, or `println!("{x:?}")` — each use needs its bound (`PartialEq`, `Clone`, `Debug`). Let the compiler errors accrete your `where` clause; that's the normal workflow.
- **Deriving `Copy` on heap-owning types.** `#[derive(Copy)]` on a struct containing `String` → `error[E0204]: the trait `Copy` cannot be implemented`: Copy means bitwise duplication is safe, which it isn't for owned heap data. Copy is for small plain-data types only.
- **Fighting the orphan rule (E0117).** `impl Display for Vec<String>` fails — both trait and type are foreign. Fix: the *newtype pattern* — `struct Names(Vec<String>); impl Display for Names { ... }`. This comes up constantly; remember the newtype answer.
- **`Box<dyn Trait>` reflexively, everywhere.** Coming from OO, everything becomes a boxed interface. Idiomatic Rust prefers generics (static dispatch) until heterogeneity forces `dyn`. If your function takes `&Box<dyn Trait>`, it should almost certainly take `&impl Trait` or `&dyn Trait`.
- **Expecting structural typing.** A type with a `summarize()` method does *not* satisfy `Summary` — only `impl Summary for T` does. Method-name coincidence means nothing (a feature: no accidental contracts).

## Practice Exercises

1. Write `fn smallest<T: PartialOrd>(items: &[T]) -> Option<&T>` (None for empty input). Test with integers, floats, and string slices. Then remove the bound and paste the compiler error in a comment.
2. Define a `Shape` **trait** with `area(&self) -> f64` and a default method `describe(&self) -> String` that uses `area`. Implement it for `Circle`, `Rect`, and `Triangle` structs. Print descriptions via (a) a generic function and (b) a `Vec<Box<dyn Shape>>`. Comment on when you'd choose each.
3. Implement `Display` and `From<(f64, f64)>` for a `Point` struct, so that `let p: Point = (3.0, 4.0).into();` and `println!("{p}")` both work. Then add `PartialEq` by hand (not derived) that treats points equal within 1e-9 tolerance and explain in a comment why deriving would be wrong here.
4. Build the `Render` example from this chapter, then add a third renderer `Markdown` (table output) *without touching any existing code* — note in a comment which OO principle this demonstrates. Then add a new method `file_extension(&self) -> &'static str` to the trait and follow the compile errors to every implementor.
5. Write a generic `struct Stack<T> { items: Vec<T> }` with `push`, `pop -> Option<T>`, `peek -> Option<&T>`, and `len`. Add `impl<T: Ord> Stack<T> { fn max(&self) -> Option<&T> }` — a method that exists *only* for orderable contents. Demonstrate that `max` compiles for `Stack<i32>` but not for a `Stack<P>` of a struct without `Ord` (error in comment).
