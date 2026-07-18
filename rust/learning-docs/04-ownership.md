# Chapter 4: Ownership — The Core Concept

## Overview

This is the most important chapter in the track. Ownership is Rust's answer to the question every language must answer: *when is memory freed, and who decides?* Python and JavaScript answer it with a garbage collector at runtime. C answers it with "you, manually, good luck." Rust answers it with a set of compile-time rules — and those rules are what make Rust memory-safe *without* a garbage collector, and what make Rust feel utterly different from every language you know.

Give this chapter double the time you'd give any other. Everything after it — borrowing, lifetimes, why strings are weird, why the compiler yells about moves — builds on these rules.

## Definitions & Explanations

### The problem being solved

Every value in a running program occupies memory. That memory must be reclaimed when the value is no longer needed — exactly once, and never while the value is still in use.

- **Python/JS approach (garbage collection):** the runtime tracks references to every heap object; when nothing references an object anymore, it's freed *eventually*. Cost: runtime overhead, unpredictable pauses, memory held longer than needed. Benefit: you never think about it.
- **C approach (manual):** you call `free()` yourself. Cost: use-after-free, double-free, leaks — the majority of serious security vulnerabilities in existence.
- **Rust approach (ownership):** the *compiler* tracks, at compile time, exactly one "owner" for every value, and inserts the free at the precise point the owner goes away. Zero runtime cost, zero GC, and the bug classes above are compile errors.

### Stack vs heap (quick, necessary detour)

- The **stack** holds fixed-size, short-lived data: integers, floats, bools, fixed arrays, and the "handle" part of bigger structures. Allocation is trivially fast.
- The **heap** holds data whose size can grow or isn't known at compile time: the character data of a `String`, the elements of a `Vec`. Allocating costs more and *someone must free it*.

An `i32` lives entirely on the stack. A `String` is a small stack part (pointer, length, capacity) pointing at heap memory holding the actual text. Ownership rules exist to manage that heap part.

### The three rules of ownership

1. **Every value in Rust has exactly one owner** (a variable).
2. **There can only be one owner at a time.**
3. **When the owner goes out of scope, the value is dropped** (its memory freed, its cleanup run).

```rust
{
    let s = String::from("hello"); // s owns the String (heap allocation happens)
    // ... use s ...
}   // scope ends -> s is dropped -> heap memory freed. Automatically. At compile
    // time, the compiler inserted the cleanup call right here.
```

`drop` is deterministic — it happens at the closing brace, not "eventually when the GC runs." (This is the same idea as C++ RAII, or Python's `with` block, but applied to *every* value automatically.)

### Moves: assignment transfers ownership

Here is the rule that shocks everyone coming from Python/JS:

```rust
let s1 = String::from("hello");
let s2 = s1;                // ownership MOVES from s1 to s2
println!("{s1}");           // ERROR: s1 is no longer valid!
```

In Python, `s2 = s1` makes both names point to the same object — fine, because the GC handles cleanup. Rust can't allow two owners: when both went out of scope, the memory would be freed twice. So assignment of a heap-owning value **moves** it: `s2` is now the owner, and `s1` is dead. The compiler enforces this:

```text
error[E0382]: borrow of moved value: `s1`
 --> src\main.rs:3:15
  |
1 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not
  |            implement the `Copy` trait
2 |     let s2 = s1;
  |              -- value moved here
3 |     println!("{s1}");
  |               ^^^^ value borrowed here after move
  |
help: consider cloning the value if the performance cost is acceptable
  |
2 |     let s2 = s1.clone();
  |                ++++++++
```

Memorize the shape of E0382 — it will be your companion for weeks. Note the mechanics: only the small stack part (pointer/len/capacity) is copied; the heap data doesn't move. What changes is *who is responsible for freeing it*.

### `Copy` types: the exception

Simple stack-only types — all integers, floats, `bool`, `char`, and tuples of them — implement the `Copy` trait. For them, assignment copies the bits and both variables stay valid:

```rust
let x = 5;
let y = x;          // x is COPIED, not moved
println!("{x} {y}"); // fine — copying an i32 is trivially cheap and safe
```

Rule of thumb: if a type owns heap data (`String`, `Vec`, most structs), it moves. If it's a plain fixed-size value, it copies. The compiler error above even tells you which case you're in ("does not implement the `Copy` trait").

### `clone`: explicit deep copy

When you genuinely want two independent copies of heap data:

```rust
let s1 = String::from("hello");
let s2 = s1.clone();      // heap data duplicated; two owners of two values
println!("{s1} {s2}");    // both valid
```

`clone()` is Rust making costs visible: in Python, you never know when a "copy" is a reference or real copy (`copy.deepcopy`, anyone?). In Rust, deep copies only happen where you can see the word `clone`.

### Functions move too

Passing a value to a function moves it, exactly like assignment. Returning moves it back out:

```rust
fn takes_ownership(s: String) {   // s takes ownership of the argument
    println!("{s}");
}                                  // s dropped here; memory freed

fn gives_ownership() -> String {
    String::from("yours now")      // moved out to the caller
}

fn main() {
    let s = String::from("hello");
    takes_ownership(s);
    // println!("{s}");            // ERROR E0382: s was moved into the function

    let t = gives_ownership();     // t owns the returned String
    println!("{t}");
}
```

This seems brutal — does every function eat its arguments? Passing ownership in and threading it back out (`fn process(s: String) -> String`) works but is clumsy. The real solution is *borrowing* (Chapter 5): letting functions temporarily use a value without taking ownership. This chapter's job is to make you feel the problem borrowing solves.

## Code Examples

### Watching drops happen

```rust
struct Noisy(&'static str); // a tiny struct wrapping a name (Chapter 7 covers structs)

impl Drop for Noisy {
    fn drop(&mut self) {
        // This runs at the exact moment the value is dropped.
        println!("dropping {}", self.0);
    }
}

fn main() {
    let _a = Noisy("a");
    {
        let _b = Noisy("b");
        println!("inner scope ends next");
    }                           // <- "dropping b" prints here, deterministically
    println!("main ends next");
}                               // <- "dropping a" prints here
```

Output:

```text
inner scope ends next
dropping b
main ends next
dropping a
```

No GC decided this. The compiler placed the cleanup calls at the closing braces. (Drop order within a scope is reverse declaration order.)

### Move in a loop — a classic first fight

```rust
fn shout(s: String) {
    println!("{}!", s.to_uppercase());
}

fn main() {
    let msg = String::from("hello");
    for _ in 0..3 {
        shout(msg);   // ERROR on the 2nd iteration conceptually —
    }                 // and the compiler catches it before you even run:
}
```

```text
error[E0382]: use of moved value: `msg`
  |
  |         shout(msg);
  |               ^^^ value moved here, in previous iteration of loop
```

The first iteration moves `msg` into `shout`; the second iteration has nothing to move. Fixes, from worst to best: clone each time (`shout(msg.clone())` — works, wasteful), restructure, or — the real answer — make `shout` *borrow* instead of take (`fn shout(s: &str)`, next chapter).

### Ownership through data structures

```rust
fn main() {
    let name = String::from("Ada");
    let names = vec![name];          // `name` moved INTO the vector
    // println!("{name}");           // ERROR: the Vec owns it now

    let first = names.into_iter().next().unwrap(); // move it back out
    println!("{first}");
    // `names` was consumed by into_iter — also gone now
}
```

Collections *own* their contents. Putting a value in a `Vec` moves it in; the `Vec` dropping drops every element. One owner at every level, all the way down — this composability is why the rules scale.

## Common Pitfalls

- **Treating `=` like Python's name binding.** In Python, assignment never invalidates anything. In Rust, assignment of a non-`Copy` value is a *transfer*. When you see E0382, your first question should be "where did it move, and did I actually need two owners?"
- **Cloning everything to make errors go away.** It compiles, and while learning it's an acceptable crutch — but each `clone` is a real allocation. Before reaching for it, ask: could this function *borrow* instead (Chapter 5)? Experienced Rust has surprisingly few clones.
- **Thinking moves copy the heap data.** They don't — a move is byte-for-byte copy of a small handle, essentially free. `clone` copies heap data. Don't avoid moves for "performance"; they *are* the fast path.
- **Confusion about why `i32` behaves differently from `String`.** It's the `Copy` trait. When an example works with numbers but breaks with strings, this is why — reread the E0382 note: "which does not implement the `Copy` trait".
- **Returning references to local variables (previewing Chapter 5/12).** "The function made it, the function's scope ends, so it's dropped — I can't hand out a pointer to it." You return the *owned value* instead, moving it out. Rust makes the C dangling-pointer bug a compile error.

## Practice Exercises

1. Without running it, annotate each line of the following with OK/ERROR and why; then verify with `cargo check`:
   ```rust
   let a = String::from("x");
   let b = a;
   let c = b.clone();
   println!("{b}");
   println!("{a}");
   let n = 5;
   let m = n;
   println!("{n} {m}");
   ```
2. Write `fn longest_word_owned(sentence: String) -> String` that returns the longest whitespace-separated word as a new `String`. In `main`, demonstrate that the original sentence is unusable after the call, with the compiler error pasted in a comment.
3. Implement the `Noisy`/`Drop` example, then predict (in comments, before running) the exact drop-message order for: two values in `main`, one value inside an inner block between them, and one value moved into a function that is called between the blocks. Verify.
4. Write a program where a `String` is moved into a `Vec<String>`, then moved back out (any method), then moved into a function that consumes it. Prove at each stage (with commented-out lines that fail to compile) that exactly one owner exists.
5. Take this deliberately clone-heavy program and reduce it to at most one `clone` while keeping it compiling, using only what you know so far (moves, returns, restructuring): a `main` that builds a `String` greeting, "logs" it with a function, appends to it, and prints it at the end. (You may change function signatures to take and return ownership.)
