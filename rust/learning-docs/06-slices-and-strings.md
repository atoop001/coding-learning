# Chapter 6: Slices & Strings (String vs &str)

## Overview

"Why are there two string types?" is the most-asked beginner Rust question. The answer falls straight out of ownership: `String` is an *owned, growable* string; `&str` is a *borrowed view* of string data living somewhere else. More generally, this chapter introduces **slices** — borrowed views into contiguous data — which apply to vectors and arrays too (`&[T]`). Along the way: why Rust strings are UTF-8, why you can't index them with `s[0]`, and which type to use where.

## Definitions & Explanations

### Slices: a borrowed view of a run of elements

A slice is a reference to a *contiguous portion* of a collection: a pointer plus a length. It doesn't own anything.

```rust
let v = vec![10, 20, 30, 40, 50];
let middle: &[i32] = &v[1..4];    // view of elements 20, 30, 40
println!("{middle:?}");            // [20, 30, 40]  ({:?} = debug print)
```

- Range syntax: `&v[1..4]` (end-exclusive), `&v[..2]`, `&v[3..]`, `&v[..]` (whole thing). Same conventions as Python slicing — but Python slicing *copies* the elements into a new list; a Rust slice copies **nothing**. It's a window onto the original data, which means it's a borrow, and all of Chapter 5's rules apply: while `middle` is live, `v` cannot be mutated.
- Slices are "fat pointers": pointer + length (16 bytes on your machine). Bounds are checked; `&v[1..99]` panics rather than reading out of bounds.

### `String` vs `&str`

| | `String` | `&str` ("string slice") |
|---|---|---|
| Owns its data? | Yes (heap) | No — borrows from somewhere |
| Growable/mutable? | Yes (`push_str`, etc.) | No |
| Analogy | a `Vec<u8>` of UTF-8, with string methods | a `&[u8]` of UTF-8, with string methods |
| Where you meet it | building/returning strings | literals, function parameters, views |

Key realizations:

- **String literals are `&str`.** `let s = "hello";` — the text is baked into your compiled binary, and `s` borrows it. Its full type is `&'static str`: a borrow that lives as long as the program (that's the `'static` you saw in Chapter 3).
- **`&String` coerces to `&str`.** Borrow a `String` and you can use it anywhere a `&str` is wanted. This is why the idiomatic parameter type is `&str`:

```rust
fn greet(name: &str) { println!("Hello, {name}!"); }

fn main() {
    let owned = String::from("Ada");
    greet(&owned);   // &String -> &str, automatic (deref coercion)
    greet("Bob");    // literal is already &str
    greet(&owned[..2]); // a slice of a String: "Ad"
}
```

  A `fn greet(name: &String)` would accept *only* `&String` — strictly less useful. **Rule: take `&str`, return/store `String`.**

- **Converting:** `&str → String`: `.to_string()` or `String::from(...)` (allocates + copies). `String → &str`: just borrow, `&s` or `s.as_str()` (free).

> **Python/JS contrast:** Python has one string type because strings are immutable heap objects managed by the GC — copies and views are the runtime's business. Rust surfaces the distinction because *you* control allocation: `String` means "I own and may grow this buffer"; `&str` means "I'm just reading someone's text." TS folks: think of `&str` as a read-only view type, and `String` as the owned builder — roughly `readonly` view vs owned array, but enforced.

### UTF-8, and why `s[0]` doesn't compile

Rust strings are guaranteed valid UTF-8 bytes. In UTF-8, characters occupy 1–4 bytes: `"é"` is 2 bytes, `"日"` is 3, `"🦀"` is 4. So "the first character" is not "the first byte", and `s[0]` would be an O(1) lie. Rust refuses:

```rust
let s = String::from("héllo");
let c = s[0];  // error[E0277]: the type `str` cannot be indexed by `{integer}`
```

Instead you choose your view explicitly:

```rust
let s = "héllo";
s.len();                 // 6 — BYTES, not characters!
s.chars().count();       // 5 — characters (Unicode scalar values)
s.chars().nth(1);        // Some('é') — O(n), and honest about it
s.bytes();               // iterator of u8
&s[0..2];                // byte-range slice: "hé"[0..2] is "h" + first byte of é? NO —
                         // slicing must land on char boundaries or it PANICS at runtime.
&s[..1];                 // "h" — fine, boundary-aligned
```

Python 3 hides similar costs (`s[0]` works but strings are internally re-encoded); JS `s[0]` can hand you half a surrogate pair. Rust makes the byte/char distinction explicit — annoying for a day, then you realize you've been shipping Unicode bugs in other languages for years.

### `&[T]` in function signatures

The same "borrow the view, not the container" logic applies to vectors:

```rust
fn sum(values: &[i32]) -> i32 {     // takes Vec, array, or slice of either
    let mut total = 0;
    for v in values { total += v; }
    total
}

fn main() {
    let v = vec![1, 2, 3];
    let a = [4, 5, 6];
    println!("{} {} {}", sum(&v), sum(&a), sum(&v[..2]));
}
```

**Rule: take `&[T]`, not `&Vec<T>`.** Same reasoning as `&str` vs `&String`.

## Code Examples

### First word — the canonical slice example

```rust
/// Returns the first whitespace-separated word as a slice of the input.
/// No allocation, no copying — just a view.
fn first_word(s: &str) -> &str {
    match s.split_whitespace().next() {
        Some(word) => word,
        None => "",
    }
}

fn main() {
    let sentence = String::from("hello brave new world");
    let word = first_word(&sentence);
    println!("first word: {word}");    // "hello"

    // The slice BORROWS from sentence, so this fails while `word` is live:
    // sentence.clear();
    // error[E0502]: cannot borrow `sentence` as mutable because it is
    //               also borrowed as immutable
    // The compiler just prevented a use-after-free: clear() would free the
    // buffer `word` points into.
}
```

This example is the payoff of Chapters 4–6 combined: a zero-copy function whose misuse is a compile error instead of a crash.

### Building strings

```rust
fn main() {
    let mut report = String::new();          // empty owned string
    report.push_str("Items: ");              // append &str
    report.push('3');                        // append single char
    let more = format!("{report} (updated)"); // format! allocates a NEW String
    println!("{more}");

    // Concatenation with + moves the left side (it's a method taking `self`):
    let a = String::from("Hello, ");
    let b = String::from("world");
    let c = a + &b;      // a is MOVED — unusable after. b still fine (only borrowed).
    // println!("{a}");  // error[E0382]: borrow of moved value: `a`
    println!("{c}");

    // For joining many pieces, prefer format! or join:
    let parts = ["2026", "07", "18"];
    let date = parts.join("-");
    println!("{date}");
}
```

### Realistic parsing with slices

```rust
/// Parses "key=value" into (key, value) slices — zero allocation.
fn parse_pair(line: &str) -> Option<(&str, &str)> {
    // split_once: splits at the FIRST '='; returns None if absent (Option: Ch 8)
    line.split_once('=')
        .map(|(k, v)| (k.trim(), v.trim()))
}

fn main() {
    let config = "  name = Ada Lovelace ";
    match parse_pair(config) {
        Some((k, v)) => println!("key={k:?} value={v:?}"),
        None => println!("not a key=value line"),
    }
}
```

Everything here is a view into `config` — a parser that allocates nothing. This pattern is why Rust parsers are fast, and why "who owns the text?" becomes second nature.

## Common Pitfalls

- **Writing `&String` / `&Vec<T>` parameters.** Compiles, works, but needlessly rejects literals, arrays, and sub-slices. Clippy will nag you: take `&str` / `&[T]`. (Owned `String`/`Vec` parameters are for when the function needs to *keep* the data.)
- **`len()` means bytes.** Truncating a string to "20 characters" with `&s[..20]` will eventually panic at 2 a.m. on a non-ASCII input: `byte index 20 is not a char boundary`. Use `s.chars().take(20)` or `char_indices` to find safe boundaries.
- **Expecting `+` to behave like Python.** `a + b` with two `String`s doesn't compile (`&` needed on the right), and it *moves* `a`. Prefer `format!` — clearer and no ownership surprises.
- **Holding a slice while mutating the source.** `let w = first_word(&s); s.clear(); println!("{w}");` → E0502. The slice borrows the buffer; mutation could invalidate it. Finish with the view first, or copy it out (`w.to_string()`) if you need it past the mutation.
- **"expected `String`, found `&str`" (E0308).** Struct fields and return values want owned data; you handed a borrow. Add `.to_string()`. The reverse error means add `&`. Knowing which direction converts freely (`String → &str`: free; `&str → String`: allocates) tells you which fix is cheap.
- **`chars()` gives Unicode scalars, not "user-perceived characters".** "é" can be *two* scalars (e + combining accent). For grapheme clusters you need the `unicode-segmentation` crate. Rarely matters for tooling; matters a lot for text editors.

## Practice Exercises

1. Write `fn last_word(s: &str) -> &str` returning the final whitespace-separated word (empty string if none). Demonstrate in `main` that mutating the source `String` after your last use of the result compiles, but mutating *before* it does not (keep the failing line in a comment with its error code).
2. Write `fn stats(values: &[f64]) -> (f64, f64, f64)` returning (min, max, mean). Call it with a `Vec`, an array, and a slice of the middle of the Vec.
3. Write `fn char_count(s: &str) -> usize` and `fn byte_count(s: &str) -> usize`, then print both for `"hello"`, `"héllo"`, and `"🦀🦀🦀"`. Add a comment explaining each result.
4. Write `fn initials(full_name: &str) -> String` turning `"ada mae lovelace"` into `"A.M.L."`. Handle extra internal whitespace. (Hint: `split_whitespace`, `chars().next()`, `to_uppercase()`.)
5. Write `fn parse_csv_line(line: &str) -> Vec<&str>` splitting on commas and trimming each field, allocating no new strings (the Vec of borrows is fine). Then explain in a comment: what would have to change if the function needed to *return the fields to a caller that then drops the line*? (You'll prove your answer formally in Chapter 12.)
