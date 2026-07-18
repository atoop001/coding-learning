# Chapter 14: Smart Pointers & Interior Mutability (Box, Rc, RefCell)

## Overview

The ownership rules — one owner, readers XOR one writer — cover the overwhelming majority of programs. But some shapes genuinely don't fit: recursive types, graph-like data where several places share one value, caches that mutate behind a shared reference. Rust's answer is not to abandon the rules but to package *controlled relaxations* of them into library types: `Box<T>` (heap allocation, still one owner), `Rc<T>` (multiple owners via reference counting), and `RefCell<T>` (the borrow rules enforced at *runtime* instead of compile time). Each buys a capability at an explicit, visible cost. Amusingly, this chapter is where Rust briefly becomes *more* like Python — `Rc` is Python's reference counting, made opt-in and explicit.

## Definitions & Explanations

### `Box<T>`: plain heap allocation

```rust
let b: Box<i32> = Box::new(5);   // 5 lives on the HEAP; b (a pointer) on the stack
println!("{}", *b);              // deref to reach it (often automatic)
```                              // b dropped -> heap freed. Normal ownership.

`Box` is the simplest smart pointer: single owner, no sharing, no runtime cost beyond the allocation itself. When do you need it?

1. **Recursive types.** A type containing itself has infinite size — unless the self-reference is a pointer:

```rust
enum List {
    Node(i32, List),   // error[E0072]: recursive type `List` has infinite size
    Empty,             //  = help: insert some indirection (e.g., a `Box`, ...)
}

enum List {
    Node(i32, Box<List>),   // fixed: Box is pointer-sized, size now known
    Empty,
}
```

The compiler literally suggests the fix. Same story for trees: `children: Vec<TreeNode>` is fine (Vec is already indirection), but `parent: TreeNode` would need a pointer type.

2. **Trait objects.** `Box<dyn Trait>` from Chapters 9 and 11 — `dyn Trait` has no compile-time size, so it must live behind a pointer.
3. **Moving big data cheaply.** A `Box<[f64; 1_000_000]>` moves by copying 8 bytes, not 8 MB.

> **Mental model:** In Python/JS, *everything* is effectively boxed — every object is a heap allocation behind a pointer, always. Rust puts values inline (stack, or inline in their container) by default and makes heap indirection a visible, deliberate choice. That inversion is a big part of why Rust is fast.

### `Rc<T>`: shared ownership (reference counting)

Sometimes "exactly one owner" is genuinely wrong: two lists sharing a tail; multiple UI nodes referencing one style object; a graph. `Rc<T>` (Reference Counted) allows *multiple owners* — the value is freed when the **last** owner drops:

```rust
use std::rc::Rc;

let a = Rc::new(String::from("shared data"));
println!("count = {}", Rc::strong_count(&a));   // 1

let b = Rc::clone(&a);   // NOT a deep copy — just increments the count (cheap)
let c = Rc::clone(&a);
println!("count = {}", Rc::strong_count(&a));   // 3

drop(b);
println!("count = {}", Rc::strong_count(&a));   // 2
// When a and c both drop -> count 0 -> String freed. Deterministic, still no GC.
```

Essentials:

- `Rc::clone(&a)` bumps a counter; the data is never copied. (Writing `a.clone()` works too, but `Rc::clone` signals "cheap count-bump, not deep copy" to readers.)
- **`Rc<T>` gives shared *read-only* access.** You cannot get `&mut` through an Rc (there are other owners — readers XOR writer still holds!). Mutation requires pairing with `RefCell`, below.
- **Single-threaded only.** The count isn't atomic. The thread-safe twin is `Arc<T>` (Chapter 16); the compiler will refuse to let `Rc` cross threads — a taste of `Send`/`Sync`.
- **Cycles leak.** Two Rc's owning each other never hit count 0. This is exactly why Python's ref-counting needs its backup cycle collector — Rust has none, so cyclic designs use `Weak<T>` (a non-owning downgrade: `Rc::downgrade`) for back-edges like `parent` pointers.

### `RefCell<T>`: the borrow checker at runtime

`RefCell` holds a value and enforces the *same* rules as Chapter 5 — any number of readers XOR one writer — but at **runtime**, panicking on violation instead of failing compilation:

```rust
use std::cell::RefCell;

let cell = RefCell::new(5);

{
    let r1 = cell.borrow();     // like &T   -> Ref<i32>
    let r2 = cell.borrow();     // two readers: fine
    println!("{} {}", *r1, *r2);
}                               // readers dropped

{
    let mut w = cell.borrow_mut();   // like &mut T -> RefMut<i32>
    *w += 1;
}

// The crime, moved to runtime:
let _r = cell.borrow();
let _w = cell.borrow_mut();     // PANIC: already borrowed: BorrowMutError
```

Why would you ever trade a compile error for a panic? Because some correct programs can't be *proven* correct by the static checker — most famously, **mutating state behind a shared reference** ("interior mutability"): a cache inside a logger shared across your app, a visit-counter inside a graph node owned by many parents. `RefCell` says: "I take responsibility; check me at runtime." The cost: a flag check per borrow, and bugs become panics instead of compile errors — so it's a scalpel, not a default.

(Sibling: `Cell<T>` for `Copy` types — get/set whole values, no borrows, no panics. Cheaper when it fits.)

### The workhorse combination: `Rc<RefCell<T>>`

Shared ownership *plus* mutation — the closest Rust gets to "a normal Python object":

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Player { name: String, score: u32 }

fn main() {
    // One Player, owned by two lists (teams):
    let ada = Rc::new(RefCell::new(Player { name: "Ada".into(), score: 0 }));

    let team_red: Vec<Rc<RefCell<Player>>> = vec![Rc::clone(&ada)];
    let team_all: Vec<Rc<RefCell<Player>>> = vec![Rc::clone(&ada)];

    // Mutate through EITHER handle; both see it (it's the same Player):
    team_red[0].borrow_mut().score += 10;
    ada.borrow_mut().score += 5;

    println!("{:?}", team_all[0].borrow());   // Player { name: "Ada", score: 15 }
}
```

Read the type out loud — it's self-documenting: "shared ownership (`Rc`) of runtime-borrow-checked (`RefCell`) mutable state." When you see `Rc<RefCell<T>>` in code, you know exactly which guarantees were traded and which retained (memory safety: fully retained; static borrow checking: traded for runtime checks).

## Code Examples

### A tree with shared, mutable nodes and parent pointers

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,        // Weak: non-owning -> no ref cycle
    children: RefCell<Vec<Rc<Node>>>,   // owning edges point DOWN the tree
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    // Wire the back-edge: leaf.parent = weak ref to branch
    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    // Weak must be UPGRADED to use (returns Option — parent may be gone):
    if let Some(p) = leaf.parent.borrow().upgrade() {
        println!("leaf's parent value: {}", p.value);      // 5
    }

    println!("branch strong = {}, weak = {}",
        Rc::strong_count(&branch), Rc::weak_count(&branch)); // 1, 1
    // No cycle: when branch drops, its strong count hits 0 and it's freed,
    // even though leaf still holds a Weak to it. upgrade() then returns None.
}
```

This is the canonical "parent pointers without leaks" pattern. In Python you'd just write `child.parent = parent` and let the cycle collector clean up; Rust makes the ownership direction explicit: strong down, weak up.

### RefCell for a cache behind &self

```rust
use std::cell::RefCell;
use std::collections::HashMap;

struct Fibber {
    cache: RefCell<HashMap<u64, u64>>,   // interior mutability
}

impl Fibber {
    fn new() -> Self { Fibber { cache: RefCell::new(HashMap::new()) } }

    // NOTE: &self — callers see a READ operation, and can share the Fibber
    // freely. The mutation (memoization) is an internal detail.
    fn fib(&self, n: u64) -> u64 {
        if let Some(&hit) = self.cache.borrow().get(&n) {
            return hit;
        }   // <- the borrow() ends HERE, before the recursive calls below.
        let result = if n < 2 { n } else { self.fib(n - 1) + self.fib(n - 2) };
        self.cache.borrow_mut().insert(n, result);
        result
    }
}

fn main() {
    let f = Fibber::new();
    println!("{}", f.fib(50));   // instant, thanks to the cache
}
```

Subtle and important: if the `if let Some` line held its `borrow()` across the recursive `self.fib(...)` calls, the recursion's `borrow_mut()` would panic (`already borrowed`). The early `return` + statement-end drop makes the scopes disjoint. This is the RefCell discipline: **keep borrows short**, exactly as with references.

## Common Pitfalls

- **Reaching for `Rc<RefCell<T>>` to silence the borrow checker.** The classic beginner escape hatch: any E0502 → wrap it in RefCell. You've traded compile errors for runtime panics and made the code harder to read. First exhaust the honest options: restructure, split structs, pass `&mut` down, return values up. `Rc<RefCell>` is for *genuine* shared mutable state, which is rarer than it feels.
- **`already borrowed: BorrowMutError` panics.** Almost always a `borrow()` still alive (often hidden in a temporary or held across a call that also borrows) when `borrow_mut()` runs. Fix: shrink borrow scopes — bind, copy out, drop early (`drop(guard)` works). The Fibber example shows the pattern.
- **Expecting `Rc` to allow mutation.** `rc_value.field = x` → `error[E0594]: cannot assign... behind a reference`. Rc = shared = read-only. Mutation needs the RefCell layer (or `Rc::get_mut`, which succeeds only when the count is 1).
- **Rc cycles leaking silently.** Parent and child both holding `Rc` of each other never free. No error, no panic — just memory that never returns, the one leak Rust's rules don't catch (leaks are safe, just wasteful). Back-edges are `Weak`; audit any bidirectional Rc design.
- **Using `Rc`/`RefCell` in threaded code.** `Rc<T>` across threads → `error[E0277]: Rc<...> cannot be sent between threads safely`. The compiler names the fix implicitly: `Arc` (and `Mutex` instead of RefCell) — Chapter 16.
- **Boxing reflexively for "performance" or out of GC-language habit.** Newcomers box things to "put them on the heap like objects." Rust values are fine where they are; Box has three jobs (recursion, trait objects, huge moves) and otherwise just adds an allocation and a hop.

## Practice Exercises

1. Build the classic cons list: `enum List { Cons(i32, Box<List>), Nil }` with methods `len(&self) -> usize` and `sum(&self) -> i32` (recursive matches). Start WITHOUT the Box, paste the E0072 error and its help text in a comment, then fix it.
2. Create two shopping-cart lists that share a common `Rc<Vec<String>>` of default items plus their own extras. Print `strong_count` at three points; predict each count in a comment *before* running.
3. Write a `Logger { messages: RefCell<Vec<String>> }` with `fn log(&self, msg: &str)` (note: `&self`) and `fn dump(&self) -> Vec<String>`. Share one logger between two functions that each log. Then deliberately provoke a `BorrowMutError` by holding a `borrow()` of messages while calling `log`, capture the panic message in a comment, and fix it.
4. Extend the tree example: add `fn add_child(parent: &Rc<Node>, value: i32) -> Rc<Node>` that creates a node, wires both directions (children strong, parent weak), and returns it. Then demonstrate `upgrade()` returning `None` after the parent is dropped.
5. Design decision drill — for each scenario, pick plain ownership, `&`/`&mut`, `Box`, `Rc`, `RefCell`, or a combination, with one-line justifications (no code needed): (a) a config struct read by many modules, never mutated after startup; (b) an AST for a calculator; (c) a memoization cache inside a struct with only `&self` methods; (d) a doubly-linked structure; (e) an 8 MB buffer returned from a function.
