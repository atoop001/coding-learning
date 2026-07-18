# Chapter 7: Structs & Methods

## Overview

Structs are how you build your own types in Rust: named bundles of data, with behavior attached via `impl` blocks. If you're coming from TypeScript classes or Python dataclasses, structs will feel familiar — minus inheritance (Rust has none; it uses traits instead, Chapter 11) and plus ownership: every field is *owned* by the struct, and methods must declare whether they read (`&self`), mutate (`&mut self`), or consume (`self`) the instance. That three-way choice is new, and it's the heart of this chapter.

## Definitions & Explanations

### Defining and instantiating

```rust
struct User {
    username: String,   // the struct OWNS this String
    email: String,
    sign_in_count: u64,
    active: bool,
}

let user = User {
    email: String::from("ada@example.com"),
    username: String::from("ada"),
    sign_in_count: 1,
    active: true,
};
println!("{}", user.email);   // field access with dot, like everywhere else
```

- Every field must be initialized — there are no "optional fields" or `undefined`. If a field can be absent, its type says so: `Option<String>` (Chapter 8).
- Mutability is all-or-nothing per binding: `let mut user` makes *all* fields assignable; there's no per-field `readonly` like TS. (Encapsulation comes from module privacy, Chapter 15, not per-field mutability.)
- Fields are private outside the module by default; `pub` exposes them (Chapter 15).

Why `String` fields and not `&str`? Because the struct should *own* its data — a struct holding borrowed data needs lifetime annotations (Chapter 12) and can't outlive what it borrows. Rule of thumb while learning: **structs own their fields.**

### Field init shorthand and struct update syntax

```rust
fn new_user(username: String, email: String) -> User {
    User {
        username,          // shorthand: same as username: username (like JS!)
        email,
        sign_in_count: 0,
        active: true,
    }
}

let user2 = User {
    email: String::from("ada@newjob.com"),
    ..user               // take remaining fields FROM user — like JS spread, BUT:
};
// `user.username` was a String -> it MOVED into user2.
// `user` as a whole is no longer usable (partially moved).
// println!("{}", user.username);  // error[E0382]: borrow of moved value
println!("{}", user.sign_in_count); // Copy fields are still fine, though!
```

The `..spread` analogy from JS holds *syntactically*, but ownership applies: non-`Copy` fields move.

### Tuple structs and unit structs

```rust
struct Point(f64, f64);          // tuple struct: fields by position (.0, .1)
struct Meters(f64);              // "newtype" pattern: a distinct type wrapping one value
struct Celsius(f64);

let p = Point(3.0, 4.0);
let altitude = Meters(120.5);
// let t: Celsius = altitude;    // ERROR — Meters is not Celsius, even though
                                 // both wrap f64. This is the point: the type
                                 // system now prevents unit-mixup bugs.
```

The newtype pattern is beloved in Rust: wrap a primitive to give it meaning, and mixups become compile errors (the Mars Climate Orbiter bug, made impossible).

### Methods: `impl` blocks and the three selfs

```rust
impl User {
    // Associated function (no self) — Rust's "static method" / constructor idiom.
    // Called as User::new(...). `Self` = the type being implemented.
    fn new(username: String, email: String) -> Self {
        Self { username, email, sign_in_count: 0, active: true }
    }

    // &self — read-only borrow of the instance. The overwhelmingly common case.
    fn display_name(&self) -> &str {
        &self.username
    }

    // &mut self — mutable borrow: this method changes the instance.
    fn record_sign_in(&mut self) {
        self.sign_in_count += 1;
    }

    // self — takes OWNERSHIP: the instance is consumed by the call.
    // Used for conversions/finalizers: "turn this into something else."
    fn into_email(self) -> String {
        self.email        // moved out; the rest of the User is dropped
    }
}

fn main() {
    let mut u = User::new("ada".into(), "ada@example.com".into());
    u.record_sign_in();                    // needs `mut u` — the signature says so
    println!("{}", u.display_name());
    let email = u.into_email();            // u is GONE after this line
    // u.record_sign_in();                 // error[E0382]: use of moved value
    println!("{email}");
}
```

The receiver forms map exactly onto Chapter 4/5 concepts:

| Receiver | Meaning | Python/TS equivalent |
|---|---|---|
| `&self` | "I read the instance" | any normal method |
| `&mut self` | "I mutate the instance" | any normal method (undeclared!) |
| `self` | "I consume the instance" | no equivalent — closest is "don't use after calling" comments |

In Python, *every* method can mutate `self` and nothing in the signature warns you. In Rust, `record_sign_in(&mut self)` is a contract: callers need a mutable instance, and readers of the call site know mutation may occur. This is the borrow checker paying rent in API design.

Borrowing rules apply to method calls too: you can't call a `&mut self` method while any borrow of the instance is live — same E0502 as Chapter 5, same reasoning.

### Deriving common behavior

Printing a struct with `{}` fails — Rust doesn't assume a display format. For debugging, *derive* one:

```rust
#[derive(Debug, Clone, PartialEq)]   // attribute: auto-generate these trait impls
struct Point { x: f64, y: f64 }

let p = Point { x: 1.0, y: 2.0 };
println!("{p:?}");     // Point { x: 1.0, y: 2.0 }   (Debug)
println!("{p:#?}");    // pretty multi-line version
let q = p.clone();     // Clone: explicit deep copy (Ch 4)
assert!(p == q);       // PartialEq: field-by-field equality
```

`#[derive(...)]` is a macro generating code at compile time. `Debug` is Python's `__repr__`, `PartialEq` is `__eq__`, `Clone` is `__deepcopy__` — except you opt in per type, and get them for free. Full trait story in Chapter 11; for now, cargo-cult `#[derive(Debug)]` onto every struct you write — you'll want it.

## Code Examples

### A worked type: Rectangle

```rust
#[derive(Debug)]
struct Rectangle {
    width: f64,
    height: f64,
}

impl Rectangle {
    fn new(width: f64, height: f64) -> Self {
        Self { width, height }
    }

    fn square(side: f64) -> Self {        // multiple constructors? just more
        Self::new(side, side)             // associated functions. No overloading
    }                                     // in Rust — distinct names instead.

    fn area(&self) -> f64 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }

    fn scale(&mut self, factor: f64) {
        self.width *= factor;
        self.height *= factor;
    }
}

fn main() {
    let mut r = Rectangle::new(3.0, 4.0);
    let s = Rectangle::square(2.0);

    println!("r = {r:?}, area {}", r.area());
    println!("r can hold s? {}", r.can_hold(&s));  // note: &s — can_hold borrows

    r.scale(2.0);
    println!("after scale: {r:?}");
}
```

### Method chaining with `self` (builder pattern preview)

```rust
#[derive(Debug, Default)]           // Default: gives a zero-ish ::default()
struct RequestBuilder {
    url: String,
    method: String,
    retries: u32,
}

impl RequestBuilder {
    fn url(mut self, url: &str) -> Self {     // takes self, returns Self:
        self.url = url.to_string();           // each call consumes and returns,
        self                                  // enabling chains
    }
    fn method(mut self, m: &str) -> Self {
        self.method = m.to_string();
        self
    }
    fn retries(mut self, n: u32) -> Self {
        self.retries = n;
        self
    }
}

fn main() {
    let req = RequestBuilder::default()
        .url("https://example.com")
        .method("GET")
        .retries(3);
    println!("{req:?}");
}
```

Note `mut self` (owned and locally mutable) — the value moves through the chain, each step modifying and passing it on. This is the fluent-API pattern from JS (`fetch().then()`-style builders), expressed through ownership.

### The struct-borrow interaction you will hit

```rust
#[derive(Debug)]
struct Inventory { items: Vec<String>, log: Vec<String> }

impl Inventory {
    fn add(&mut self, item: &str) {
        self.items.push(item.to_string());
        self.log.push(format!("added {item}"));   // two fields, one &mut self: fine
    }
}

fn main() {
    let mut inv = Inventory { items: vec![], log: vec![] };
    inv.add("hammer");

    let first = &inv.items[0];   // borrow of one FIELD
    inv.log.push("x".into());    // mutating a DIFFERENT field: OK! The compiler
                                 // tracks disjoint field borrows on locals...
    println!("{first}");

    let first = &inv.items[0];
    inv.add("saw");              // ...but a METHOD call borrows ALL of self:
    // println!("{first}");      // error[E0502]: cannot borrow `inv` as mutable
                                 // because `inv.items` is also borrowed
}
```

Field-level borrows are tracked when you access fields directly, but a method taking `&mut self` borrows the *whole* struct — the compiler can't see inside the method's signature to know which fields it touches. This asymmetry surprises everyone once. Workarounds: do field accesses directly at the conflict site, or finish with the borrow before calling the method.

## Common Pitfalls

- **`&str` fields without understanding lifetimes.** `struct User { name: &str }` → `error[E0106]: missing lifetime specifier`. Until Chapter 12, the answer is: make it `String` and own the data.
- **Forgetting `#[derive(Debug)]`,** then being confused that `println!("{u}")` and even `println!("{u:?}")` fail. `{}` needs `Display` (manual impl, Chapter 11); `{:?}` needs `Debug` (derive it).
- **Calling a `&mut self` method on an immutable binding.** `error[E0596]: cannot borrow `u` as mutable`. The fix is at the *binding*: `let mut u`.
- **Using a value after a `self`-consuming method.** E0382 again. Check the receiver: if a method takes `self` (builder steps, `into_*` conversions), the old binding is dead. If that's not what you wanted, the method probably should have been `&self`/`&mut self`.
- **Expecting inheritance.** There is no `struct Admin extends User`. Composition (`Admin { user: User, .. }`) and traits (Chapter 11) cover the use cases. Don't fight this; Rust code genuinely doesn't miss it.
- **Getters returning owned clones by reflex.** `fn name(&self) -> String { self.name.clone() }` allocates per call. Idiomatic: `fn name(&self) -> &str { &self.name }` — hand out a borrow; the caller clones only if needed.

## Practice Exercises

1. Define a `Temperature` struct storing degrees Celsius (`f64`) with: constructor `new`, `to_fahrenheit(&self) -> f64`, `to_kelvin(&self) -> f64`, and `warm_by(&mut self, delta: f64)`. Derive `Debug` and demonstrate all four in `main`.
2. Build a `BankAccount` struct (owner `String`, balance `i64` in cents) with `deposit(&mut self, cents: u64)`, `withdraw(&mut self, cents: u64) -> bool` (false if insufficient — you'll upgrade this to `Result` after Chapter 9), and `balance(&self) -> i64`. Why cents and not `f64`? Answer in a comment.
3. Create newtypes `Miles(f64)` and `Kilometers(f64)` with a method `to_km(&self) -> Kilometers` on `Miles`. Show that passing `Miles` where `Kilometers` is expected fails to compile (comment + error code).
4. Implement a `TodoItem` (title, done, priority `u8`) and a `TodoList { items: Vec<TodoItem> }` with methods: `add(&mut self, title: &str, priority: u8)`, `complete(&mut self, index: usize)`, `count_open(&self) -> usize`, and `into_titles(self) -> Vec<String>` (consuming). Exercise every receiver kind in `main`, and demonstrate the "method borrows all of self" conflict from this chapter, then resolve it.
5. Recreate the builder pattern for a `Pizza` (size, Vec of toppings, extra_cheese bool) with chained methods ending in a plain struct. Then try to reuse the builder after the chain and explain the E0382 you get.
