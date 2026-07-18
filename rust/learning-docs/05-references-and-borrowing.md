# Chapter 5: References & Borrowing

## Overview

Chapter 4 left us with a problem: if passing a value to a function *moves* it, how do we let a function merely *look at* our data? The answer is **references** — and the compile-time rules governing them, called **borrowing**, are enforced by the famous **borrow checker**. This chapter is where you meet the borrow checker properly. It will reject code that would run fine in Python; understanding *why* is the point. The rules boil down to one sentence: **any number of readers, or exactly one writer — never both.**

## Definitions & Explanations

### References: using without owning

A reference (`&T`) lets you refer to a value without taking ownership. Creating one is called *borrowing* — the language of lending is apt: the owner lends the value out and must get it back intact.

```rust
fn len_of(s: &String) -> usize {   // borrows a String; does NOT own it
    s.len()
}                                   // s (the reference) goes away; the String does NOT get dropped

fn main() {
    let msg = String::from("hello");
    let n = len_of(&msg);           // lend msg with &
    println!("{msg} has {n} bytes"); // msg still valid — we never gave it away
}
```

Compare with Chapter 4's `takes_ownership`: the only changes are `&String` in the signature and `&msg` at the call site, and now the caller keeps ownership.

> **Python/JS contrast:** In Python, *everything* is effectively a reference and any code holding one can mutate the object ("spooky action at a distance": you pass a list to a function and it mutates it without warning). Rust references are explicit at both ends (`&` in the signature *and* the call), and whether mutation is allowed is part of the type.

### Shared vs mutable references

- `&T` — a **shared** (immutable) reference. You may read through it. Any number may exist at once.
- `&mut T` — a **mutable** (exclusive) reference. You may read and write through it. If one exists, *nothing else* — no other `&mut`, no `&`, not even the owner — may access the value until it's gone.

```rust
fn add_exclamation(s: &mut String) {
    s.push('!');            // mutate through the reference
}

fn main() {
    let mut msg = String::from("hello"); // owner must be mut to lend mutably
    add_exclamation(&mut msg);           // explicit &mut at the call site too
    println!("{msg}");                   // hello!
}
```

Note the ceremony: `mut` on the binding, `&mut` in the signature, `&mut` at the call. Rust makes mutation loud. In JS, `addExclamation(msg)` gives no clue whether `msg` changes; in Rust, `&mut` at the call site tells every reader "this call may mutate msg."

### The borrowing rules

At any given time, for any given value:

1. You may have **either** any number of shared references (`&T`) **or** exactly one mutable reference (`&mut T`) — not both.
2. References must never outlive the value they point to (no dangling references).

Rule 1 is "readers XOR one writer." Why? Because a writer changing data while readers look at it is exactly what causes iterator invalidation, torn reads, and (across threads) data races. Rust bans the pattern *in the type system*, single-threaded or not. Rule 2 kills the C dangling-pointer bug at compile time.

Crucial refinement (this makes the rules livable): the compiler tracks where each reference is *last used*, not just its scope. A borrow ends at its last use — so this compiles:

```rust
let mut s = String::from("hi");
let r1 = &s;
let r2 = &s;
println!("{r1} {r2}");   // last use of r1, r2 — the shared borrows end HERE
let w = &mut s;          // fine: no live shared borrows anymore
w.push('!');
```

### Dangling references: impossible by construction

```rust
fn dangle() -> &String {
    let s = String::from("hello");
    &s                              // returning a reference to a local...
}                                   // ...that is dropped right here
```

```text
error[E0106]: missing lifetime specifier
 --> src\main.rs:1:16
  |
1 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  = help: this function's return type contains a borrowed value, but there
    is no value for it to be borrowed from
```

The compiler is saying: "you promise to return a borrow, but of *what*? Everything you could borrow dies when the function ends." Fix: return the owned `String` (move it out). Lifetimes get full treatment in Chapter 12; for now, know that this entire bug class cannot compile.

## Code Examples

### The classic borrow-checker fight: reading while writing

```rust
fn main() {
    let mut s = String::from("hello");
    let r = &s;              // shared borrow starts
    s.push_str(" world");    // requires &mut s — CONFLICT
    println!("{r}");         // shared borrow still live (used here)
}
```

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src\main.rs:4:5
  |
3 |     let r = &s;
  |             -- immutable borrow occurs here
4 |     s.push_str(" world");
  |     ^^^^^^^^^^^^^^^^^^^^ mutable borrow occurs here
5 |     println!("{r}");
  |               --- immutable borrow later used here
```

Read the three annotations — the compiler shows the whole story: where the read borrow began, where the write happened, where the read was still needed. Why is this worth banning? `push_str` may reallocate the String's heap buffer, which would leave `r` pointing at freed memory. Python survives the equivalent because the GC keeps the old object alive; C crashes; Rust refuses to compile. Fix: finish reading before writing (move the `println!` above the `push_str`), or don't hold the borrow across the mutation.

### Two mutable references

```rust
let mut n = 5;
let a = &mut n;
let b = &mut n;    // ERROR E0499: cannot borrow `n` as mutable more than once at a time
*a += 1;
*b += 1;
```

Note `*a` — the **dereference operator** reaches through the reference to the value. (Method calls like `s.len()` auto-dereference for you, so `*` mostly appears when assigning through references to primitives.)

### Iterating while mutating — the fight everyone has

```rust
fn main() {
    let mut scores = vec![70, 85, 92];
    for s in &scores {              // shared borrow of the whole Vec for the loop
        if *s < 80 {
            scores.push(100);       // ERROR: needs &mut while & is live
        }
    }
}
```

```text
error[E0502]: cannot borrow `scores` as mutable because it is also borrowed as immutable
```

This is the same bug as modifying a list while iterating in Python — except Python lets it run and produces skipped elements or infinite loops, and Rust stops you at compile time. Fixes: collect what to add first, then push after the loop; or iterate over indices; or use iterator adapters like `retain` (Chapter 13).

### Borrowing in a realistic function

```rust
/// Counts words longer than `min_len`. Borrows; never takes ownership.
fn count_long_words(text: &String, min_len: usize) -> usize {
    let mut count = 0;
    for word in text.split_whitespace() {
        if word.len() > min_len {
            count += 1;
        }
    }
    count
}

fn main() {
    let essay = String::from("the borrow checker is your strict but fair friend");
    let long = count_long_words(&essay, 4);
    println!("{long} long words in: {essay}"); // essay fully usable
}
```

(Preview: `&String` works, but `&str` is the more flexible parameter type — Chapter 6 explains why and you'll switch to it.)

## Common Pitfalls

- **"cannot borrow as mutable because it is also borrowed as immutable" (E0502).** The #1 beginner error. Strategy: find the *last use* of the shared reference and ask whether the mutation can move after it, or the read can complete (often by copying a small value out: `let len = s.len();` ends the borrow immediately because `usize` is `Copy`).
- **"cannot borrow as mutable more than once" (E0499).** Usually a sign you're trying to hold two handles into one structure (e.g., two `&mut` elements of one Vec). Often solved by scoping one borrow to end before the next begins, or using methods like `split_at_mut` designed for this.
- **Forgetting `mut` on the owner.** `let s = String::new(); s.push('a');` → `error[E0596]: cannot borrow s as mutable, as it is not declared as mutable`. The fix is in the message: `let mut s`.
- **Adding `&` or `mut` randomly until it compiles.** A common flailing pattern. Instead, ask the two questions in order: *Who owns this value?* *Does this code need to read it or write it?* Then the right signature (`T` / `&T` / `&mut T`) falls out.
- **Trying to keep a reference to an element while growing the collection.** `let first = &v[0]; v.push(x);` → E0502. The push may reallocate and move every element. Copy the element out (if cheap) or restructure.
- **Thinking `&mut` is about the reference variable being reassignable.** `&mut T` means "can mutate the T behind it." Whether the reference *variable itself* can be re-pointed is a separate, ordinary `let mut r` question.

## Practice Exercises

1. Convert Chapter 4's exercise function `longest_word_owned(sentence: String) -> String` into `longest_word(sentence: &String) -> String`, and show in `main` that the sentence remains usable afterward.
2. Write `fn double_all(values: &mut Vec<i32>)` that doubles every element in place (a plain indexed `for` is fine). In `main`, print the Vec before and after. Then try calling it on a non-`mut` binding and record the error.
3. Predict which of these compile, then verify each in isolation:
   a) shared then shared, both printed after;
   b) shared, printed, then mutable, used;
   c) shared, then mutable, then the shared printed last;
   d) mutable, used, then shared, printed.
4. Write `fn swap_halves(s: &mut String)` that rearranges "hello world" into "world hello" (split on the space, rebuild). You'll likely hit E0502 on your first attempt — copy the pieces into owned `String`s to end the borrows, then mutate. Keep your first failing attempt in a comment.
5. Write a program with a `Vec<String>` of names, a function that *borrows* the Vec to find the longest name (returning its length, a `usize`), and a function that *mutably borrows* it to uppercase every name. Call them in an order that compiles, then reorder the calls to force E0502 and explain the error in a comment.
