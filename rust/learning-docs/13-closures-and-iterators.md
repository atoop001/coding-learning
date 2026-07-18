# Chapter 13: Closures & Iterators

## Overview

This chapter is where Rust starts feeling *expressive* — the functional style you love from JS array methods (`map`/`filter`/`reduce`) and Python comprehensions, but compiled to code as fast as a hand-written loop (often faster, thanks to eliminated bounds checks). The new Rust twist: closures interact with ownership (they *capture* their environment by reference, mutable reference, or move — and the compiler classifies each closure accordingly), and iterators are **lazy** — nothing happens until a consumer runs.

## Definitions & Explanations

### Closure syntax

```rust
let add = |a: i32, b: i32| a + b;      // params in pipes; body an expression
let square = |x| x * x;                 // types inferred from first use
let announce = |name: &str| {           // multi-statement body: braces
    let msg = format!("hello {name}");
    println!("{msg}");
};
println!("{}", add(2, 3));
```

Closures are JS arrow functions / Python lambdas (but multi-statement, unlike lambda). Type inference works — but is locked in at first use: call `square(2)` then `square(2.5)` and the second call is E0308; a closure gets *one* concrete type.

### Capturing: the ownership story

A closure can reference variables from its enclosing scope. Rust infers the *least demanding* capture that works, in this order:

```rust
let name = String::from("Ada");

// 1. By shared reference (&) — closure only reads:
let greet = || println!("hi {name}");
greet();
println!("{name}");             // fine — closure only borrowed it

// 2. By mutable reference (&mut) — closure mutates:
let mut count = 0;
let mut bump = || count += 1;   // note: the closure binding itself needs mut
bump();
bump();
println!("{count}");            // 2 — but note: while `bump` is alive, `count`
                                // is mutably borrowed (Chapter 5 rules apply!)

// 3. By move — the `move` keyword forces taking ownership:
let owned = String::from("mine now");
let take = move || println!("{owned}");
take();
// println!("{owned}");         // error[E0382]: borrow of moved value: `owned`
```

The three capture modes correspond to three **traits** the compiler assigns automatically:

| Trait | The closure... | Callable |
|---|---|---|
| `Fn` | reads captures (or captures nothing) | many times |
| `FnMut` | mutates captures | many times (needs mut) |
| `FnOnce` | consumes captures (moves something out) | **once** |

Every closure implements `FnOnce`; most also `FnMut`; read-only ones also `Fn`. You'll meet these as bounds in APIs: `fn for_each<F: FnMut(&T)>(...)`, `thread::spawn<F: FnOnce() + Send + 'static>` (Chapter 16 — that `move ||` habit starts there).

> **JS/Python contrast:** JS closures capture *variables* by reference, always, forever — leading to the classic loop-variable bug and accidental memory retention. Python's late-binding closures have the same trap. Rust closures state their relationship with the environment in the type system: the compiler tells you if a closure holds a borrow too long (E0502/E0505 — same rules as Chapter 5, no new magic), and `move` makes ownership transfer explicit.

### Iterators: lazy pipelines

An iterator is any type implementing the `Iterator` trait — one required method:

```rust
trait Iterator {
    type Item;                              // associated type: what it yields
    fn next(&mut self) -> Option<Self::Item>;   // Some(item) until None
    // ...plus ~75 default methods (map, filter, ...) built on next()
}
```

`for x in xs` is sugar over calling `next()` until `None` — and Chapter 10's three flavors are three iterator constructors: `.iter()` → `&T`, `.iter_mut()` → `&mut T`, `.into_iter()` → `T`.

**Adapters** transform iterators into other iterators — lazily. **Consumers** actually drive the loop:

```rust
let v = vec![1, 2, 3, 4, 5, 6];

let evens_doubled: Vec<i32> = v.iter()
    .filter(|&&x| x % 2 == 0)     // adapter: keep evens (nothing runs yet!)
    .map(|&x| x * 2)              // adapter: double them (still nothing!)
    .collect();                   // CONSUMER: now the whole pipeline runs once
println!("{evens_doubled:?}");    // [4, 8, 12]
```

Laziness matters: `v.iter().map(|x| expensive(x))` does *zero work* until consumed — the compiler even warns: `unused Map that must be used: iterators are lazy and do nothing unless consumed`. Contrast JS, where each `.map().filter()` allocates an intermediate array and runs eagerly; a Rust chain compiles into a single fused loop with no intermediates. Python's generators are lazy too — Rust iterators are generators with static types and zero overhead.

### The adapter/consumer vocabulary

Adapters (lazy): `map`, `filter`, `filter_map` (map + drop Nones), `take(n)`, `skip(n)`, `enumerate` (→ `(i, item)`), `zip`, `chain`, `rev`, `flat_map`, `take_while`, `skip_while`, `peekable`, `inspect` (debug peeks).

Consumers (run the pipeline): `collect()`, `sum()`, `product()`, `count()`, `min()`/`max()`/`min_by_key`, `find` (→ Option, short-circuits), `any`/`all` (short-circuit), `position`, `fold` (general reduce), `for_each`, `last`, `nth`.

`collect()` deserves special note — it's generic over the *target*, chosen by type annotation:

```rust
let words = ["apple", "banana", "cherry"];
let v: Vec<String> = words.iter().map(|w| w.to_uppercase()).collect();
let joined: String = words.concat();                       // Strings concatenate
let lengths: std::collections::HashMap<&str, usize> =
    words.iter().map(|&w| (w, w.len())).collect();         // pairs -> map!
let nums: Result<Vec<i32>, _> = ["1","2","3"].iter().map(|s| s.parse()).collect();
// Result<Vec>, not Vec<Result> — Chapter 9's all-or-nothing trick, explained at last
```

## Code Examples

### From loop-code to iterator-code

```rust
#[derive(Debug)]
struct Sale { region: String, amount: f64 }

fn main() {
    let sales = vec![
        Sale { region: "North".into(), amount: 120.0 },
        Sale { region: "South".into(), amount: 87.5 },
        Sale { region: "North".into(), amount: 43.0 },
        Sale { region: "West".into(),  amount: 250.0 },
    ];

    // Total of North sales — imperative:
    let mut total = 0.0;
    for s in &sales {
        if s.region == "North" { total += s.amount; }
    }

    // The same, iterator style — no mut, no intermediate state to get wrong:
    let total2: f64 = sales.iter()
        .filter(|s| s.region == "North")
        .map(|s| s.amount)
        .sum();
    assert_eq!(total, total2);

    // Largest sale (Option — empty input handled by the types):
    let biggest = sales.iter()
        .max_by(|a, b| a.amount.partial_cmp(&b.amount).unwrap());
    println!("biggest: {biggest:?}");

    // Index of first sale over 100:
    let idx = sales.iter().position(|s| s.amount > 100.0);
    println!("first big sale at {idx:?}");

    // Names of regions, deduped, sorted:
    let mut regions: Vec<&str> = sales.iter().map(|s| s.region.as_str()).collect();
    regions.sort();
    regions.dedup();
    println!("{regions:?}");
}
```

### fold: when no ready-made consumer fits

```rust
fn main() {
    let text = "never gonna give you up";

    // (word_count, char_count) in one pass — fold carries an accumulator:
    let (words, chars) = text.split_whitespace()
        .fold((0, 0), |(w, c), word| (w + 1, c + word.len()));
    println!("{words} words, {chars} chars");

    // Python: functools.reduce. JS: Array.reduce. Same shape:
    // .fold(initial, |accumulator, item| new_accumulator)
}
```

### Implementing Iterator yourself

```rust
struct Fibonacci { a: u64, b: u64 }

impl Iterator for Fibonacci {
    type Item = u64;
    fn next(&mut self) -> Option<u64> {
        let out = self.a;
        (self.a, self.b) = (self.b, self.a.checked_add(self.b)?); // stop on overflow
        Some(out)
    }
}

fn main() {
    let fib = Fibonacci { a: 0, b: 1 };
    // One `next()` impl and EVERY adapter/consumer works on your type:
    let firsts: Vec<u64> = fib.take(10).collect();
    println!("{firsts:?}");   // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

    let big_even_sum: u64 = Fibonacci { a: 0, b: 1 }
        .take_while(|&n| n < 1_000_000)
        .filter(|n| n % 2 == 0)
        .sum();
    println!("{big_even_sum}");
}
```

This is the trait system (Chapter 11) at full power: implement one method, inherit an entire vocabulary. It's also Python's iterator protocol (`__next__` + `StopIteration`) with `Option` instead of exceptions.

### The closure-borrow fight you will have

```rust
fn main() {
    let mut names = vec![String::from("ada"), String::from("bob")];

    let has_ada = names.iter().any(|n| n == "ada");   // borrow begins & ENDS here
    names.push(String::from("cleo"));                 // fine — any() consumed the iter

    // But store the closure/iterator, and the borrow lives on:
    let finder = || names.iter().find(|n| n.len() > 3);  // closure BORROWS names
    // names.push(String::from("dan"));
    // error[E0502]: cannot borrow `names` as mutable because it is also
    //               borrowed as immutable (the closure holds the borrow)
    println!("{:?}", finder());
    names.push(String::from("dan"));   // fine now — finder's last use has passed
}
```

Closures are borrows with a trigger. The E0502 is pure Chapter 5; the only novelty is that the borrower is a closure. Same fix vocabulary: reorder uses, shrink the closure's life, or `move` + clone what it needs.

## Common Pitfalls

- **Forgetting the consumer.** A pipeline ending in `.map(...)` does nothing (warning: "iterators are lazy"). End with `collect`/`sum`/`for_each`/a `for` loop.
- **Reference-level confusion in closures.** `.filter(|&&x| ...)` vs `|&x|` vs `|x| *x` — `iter()` yields `&T`, and `filter`'s closure receives `&Item` = `&&T`. Don't memorize; let E0308 tell you, and add/remove `&` in the pattern until it fits. It becomes muscle memory within a week.
- **`collect()` type mysteries (E0282 "type annotations needed").** `collect` can build many containers, so the target must be known: annotate the binding (`let v: Vec<_> = ...`) or turbofish (`.collect::<Vec<_>>()`).
- **Using `into_iter()` then wanting the collection back.** E0382. `into_iter` consumes. If the collection must survive, `iter()`. Symmetric trap: returning `&T` items from `iter()` when you need owned values → `.cloned()` / `.copied()` adapters exist for exactly this.
- **Index-loops out of habit.** `for i in 0..v.len() { v[i] }` compiles but is un-idiomatic, bounds-checked per access, and blind to the iterator vocabulary. `for x in &v`, `enumerate` when you need the index. Clippy flags this (`needless_range_loop`).
- **Over-chaining.** A 9-adapter pipeline nobody can read is worse than a clear `for` loop. Rust makes both zero-cost; choose by *readability*. Rule of thumb: if you're nesting `if let` inside `map`, break the chain or use a loop.
- **Mutating through `iter()` (E0594: cannot assign).** Reading flavor can't write. Reach for `iter_mut()` — `for x in v.iter_mut() { *x *= 2 }`.

## Practice Exercises

1. Rewrite Chapter 10's word-frequency counter as a single iterator pipeline feeding the entry API, then extract "top 5" with a sort. Compare line count with your loop version in a comment.
2. Using only iterator methods (no `for`, no `mut` accumulators): from `Vec<i32>` produce (a) sum of squares of odd numbers, (b) the string `"1,4,9,16"` of the first four squares (hint: `map` + `collect::<Vec<String>>` + `join`), (c) `true` iff all elements are positive, short-circuiting.
3. Write `fn running_total(values: &[f64]) -> Vec<f64>` using `scan` (look it up in the Iterator docs — reading iterator docs is the real exercise). `[1.0, 2.0, 3.0]` → `[1.0, 3.0, 6.0]`.
4. Implement `struct Countdown(u32)` as an Iterator yielding n, n-1, ..., 1. Then use it with three different adapters/consumers you haven't used yet in this chapter (candidates: `step_by`, `inspect`, `fold`, `zip`).
5. Create the closure-borrow conflict deliberately: a closure capturing a Vec by reference, a mutation attempt inside the borrow's span (E0502 in a comment), then fix it twice — (a) by reordering, (b) with `move` and a clone. One sentence on the cost difference between the fixes.
