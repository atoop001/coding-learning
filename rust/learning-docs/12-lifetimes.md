# Chapter 12: Lifetimes

## Overview

Lifetimes are the part of Rust with the scariest reputation and the smallest actual surface. Here's the demystification up front: **lifetimes add no new rules.** Every rule was already in Chapter 5 — references must never outlive what they point to. Lifetime *annotations* (`'a`) are merely the notation for describing reference relationships to the compiler when it can't infer them — almost always at function boundaries and in structs that hold references. You are not telling the compiler how long things live (you can't change that); you are *describing* how the lifetimes of inputs and outputs relate so it can check callers.

If you've been following the track, you've already survived the hard part (the borrow checker). This chapter names what you've been doing.

## Definitions & Explanations

### The problem, concretely

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() { x } else { y }
}
```

```text
error[E0106]: missing lifetime specifier
 --> src\main.rs:1:33
  |
1 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the
    signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
1 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
```

Read the help text carefully — it states the entire problem: the function returns a borrow, and the *caller's* borrow checker needs to know which input it borrows from, to know how long the result may be used. The compiler can't inspect the body at every call site (signatures are contracts, Chapter 3), so the signature must say. The fix is literally in the error message:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

### What `'a` means (and doesn't)

`<'a>` declares a lifetime parameter — a generic, like `<T>`, but ranging over *regions of code where a borrow is valid* instead of types. The signature `fn longest<'a>(x: &'a str, y: &'a str) -> &'a str` says:

> "For some lifetime 'a: both inputs live at least as long as 'a, and the returned reference is valid for at most 'a."

In practice: **the result is only usable while *both* inputs are still alive** (the compiler picks `'a` = the shorter of the two). What annotations do NOT do: extend any lifetime, allocate anything, change runtime behavior in any way. They're compile-time documentation, checked for consistency. There is no runtime trace of them at all.

Watch it enforce:

```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }                          // string2 dropped here
    println!("{result}");      // but result MIGHT borrow from string2!
}
```

```text
error[E0597]: `string2` does not live long enough
  |
  |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^^^^^^^^^^^ borrowed value
  |     }                                        does not live long enough
  |     - `string2` dropped here while still borrowed
  |     println!("{result}");
  |               -------- borrow later used here
```

At runtime, `result` would point at `string1` (it's longer) and this would "work" — but the compiler judges by the contract, which says the result *may* borrow from either. This is the C use-after-free, caught before it can exist. Python survives the equivalent only because the GC keeps `string2` alive; Rust has no GC, so the proof happens at compile time instead.

### Elision: why you haven't written `'a` for eleven chapters

Most signatures don't need annotations because three **elision rules** fill them in:

1. Each input reference gets its own lifetime parameter.
2. If there is exactly **one input lifetime**, the output gets it: `fn first_word(s: &str) -> &str` desugars to `fn first_word<'a>(s: &'a str) -> &'a str`. (This is why Chapter 6 compiled!)
3. In methods, if `&self` is among the inputs, the output gets *self's* lifetime: `fn name(&self) -> &str` — borrowed from self. (Why Chapter 7's getters compiled.)

You only annotate when the rules can't decide — the classic case being *two* reference inputs and a reference output (`longest`). That's genuinely most of it: in day-to-day Rust you write explicit lifetimes rarely, and mostly in the two situations this chapter covers.

### Structs holding references

A struct field can be a reference — but then the struct must declare that it can't outlive the borrowed data:

```rust
// A zero-copy parser view into a log line someone else owns:
struct LogEntry<'a> {
    level: &'a str,
    message: &'a str,
}

fn parse_entry<'a>(line: &'a str) -> Option<LogEntry<'a>> {
    let (level, message) = line.split_once(": ")?;
    Some(LogEntry { level, message })
}

fn main() {
    let line = String::from("ERROR: disk full");
    let entry = parse_entry(&line).unwrap();
    println!("[{}] {}", entry.level, entry.message);
    // drop(line);                  // uncomment -> E0505: cannot move out of
    // println!("{}", entry.level); //   `line` because it is borrowed
}
```

`LogEntry<'a>` reads as: "a LogEntry borrowing data that lives for 'a." The compiler then guarantees no LogEntry outlives its line. This is *the* pattern behind Rust's famously fast zero-copy parsers (serde can deserialize JSON into structs borrowing the input buffer). The cost: a struct with lifetimes infects everything that stores it. Hence the Chapter 7 advice, still valid: **default to owned fields (`String`); use `&'a str` fields only for short-lived, performance-sensitive views.**

### `'static`

`'static` means "lives for the entire program." Two encounters:

- String literals: `let s: &'static str = "hello";` — baked into the binary, always valid (Chapters 3 and 6, explained at last).
- Bounds like `T: 'static` (you'll meet them in Chapter 16 with threads): "T contains no non-static borrows" — i.e., T owns its data. It does *not* mean "T lives forever"; a `String` satisfies `T: 'static`.

Do not "fix" lifetime errors by sprinkling `'static` — it's almost always a wrong promise, and the compiler will reject it somewhere else.

## Code Examples

### Two inputs, output tied to only one

```rust
// The annotation can express MORE than "all the same": here the result
// borrows only from `haystack`; `needle` can die immediately after the call.
fn find_line<'a>(haystack: &'a str, needle: &str) -> Option<&'a str> {
    haystack.lines().find(|line| line.contains(needle))
}

fn main() {
    let text = String::from("alpha\nbeta gamma\ndelta");
    let found;
    {
        let query = String::from("gamma");     // short-lived
        found = find_line(&text, &query);
    }                                          // query dropped — no problem:
    println!("{found:?}");                     // result borrows from text only.
}
```

Had we written the lazy `fn find_line<'a>(haystack: &'a str, needle: &'a str)`, this `main` would *fail to compile* — the contract would over-claim. Precise lifetimes make APIs more usable; this is the first place lifetime annotation becomes a design skill rather than appeasement.

### Methods on lifetime-carrying structs

```rust
struct Parser<'a> {
    input: &'a str,
    pos: usize,
}

impl<'a> Parser<'a> {
    fn new(input: &'a str) -> Self {
        Parser { input, pos: 0 }
    }

    // Elision rule 3 would tie the output to &mut self — but the token should
    // borrow from the INPUT (alive for 'a), not from the parser. Annotate:
    fn next_token(&mut self) -> Option<&'a str> {
        let rest = self.input[self.pos..].trim_start();
        if rest.is_empty() { return None; }
        let start = self.input.len() - rest.len();
        let end = rest.find(char::is_whitespace)
                      .map(|i| start + i)
                      .unwrap_or(self.input.len());
        self.pos = end;
        Some(&self.input[start..end])
    }
}

fn main() {
    let text = String::from("let x = 5");
    let mut tokens = Vec::new();
    {
        let mut p = Parser::new(&text);
        while let Some(tok) = p.next_token() {
            tokens.push(tok);       // tokens outlive the PARSER — legal, because
        }                           // they borrow from `text`, not from `p`
    }                               // p dropped; tokens still fine
    println!("{tokens:?}");         // ["let", "x", "=", "5"]
}
```

The subtle line is the return type `Option<&'a str>` instead of the elided `Option<&str>` (which would mean "borrowed from self"). With the elided version, `tokens.push(tok)` would fail — each token would lock the parser until dropped. This distinction ("borrowed from self vs borrowed from what self borrows") is the most advanced idea in the chapter, and the payoff of everything since Chapter 4.

## Common Pitfalls

- **Reading `'a` as an instruction rather than a description.** You cannot make anything live longer with annotations. If the E0597 "does not live long enough" is *true* — the data really does die too early — the fix is ownership: return `String` instead of `&str`, store owned data, restructure. Annotations only help when the compiler's assumption is *less precise than the truth* (like `find_line`).
- **The over-constrained signature.** Giving every input the same `'a` when the output borrows from only one. Symptoms: callers hit E0597 in code that's obviously fine. Fix: separate lifetimes, tie the output to the right one.
- **Structs with references as a default.** `struct Config<'a> { name: &'a str }` then trying to build it from a locally-read file → E0597, the config can't leave the function that read the text. Own the data (`String`) unless you *specifically* want a short-lived view.
- **Returning references to locals (E0106/E0515).** Still the #1 lifetime error: `fn make() -> &str { let s = format!(...); &s }` → `cannot return reference to local variable`. Nothing to annotate — return the `String`.
- **Panic-sprinkling `'static`.** `fn f() -> &'static str` compiles only for literals/leaked memory. If you wrote it to silence E0106, the real answer was `String`.
- **Fear.** Post-2018 ("non-lexical lifetimes") Rust accepts far more correct programs than the horror stories suggest. If you're writing application code and drowning in `'a`, it usually signals over-borrowed design — clone a little, own a little more, and the annotations evaporate. Idiomatic applications contain surprisingly few explicit lifetimes.

## Practice Exercises

1. Write `fn longer_of<'a>(a: &'a str, b: &'a str) -> &'a str` from memory. Then construct a `main` that compiles, and a second `main` (in comments) that triggers E0597, with the error pasted and explained in your own words.
2. Write `fn first_sentence(text: &str) -> &str` (up to and including the first `.`, or the whole text). No explicit lifetimes allowed — state in a comment which elision rule makes that legal.
3. Build a zero-copy `struct Csvrow<'a> { fields: Vec<&'a str> }` with `fn parse(line: &str) -> CsvRow` (annotate as needed). In `main`, parse a row, drop the source `String` inside a smaller scope, and document the compile error you get when using the row afterward. Then write the owning alternative (`Vec<String>`) and note the trade in a comment.
4. Take `fn find_line` from this chapter and deliberately change it to the over-constrained single-lifetime version. Confirm the chapter's `main` stops compiling; paste the error; restore the precise version. One sentence: why did the caller break when the *body* never changed?
5. Extend the `Parser` example with `fn remaining(&self) -> &'a str` returning the unparsed tail. Prove with a `main` that the tail stays usable after the parser is dropped. Then change the return type to the elided `&str` and document what breaks and why (which elision rule kicked in, and what it tied the borrow to).
