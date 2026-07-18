# Chapter 10: Collections (Vec, HashMap, iteration)

## Overview

Now that you can own, borrow, and handle errors, you're ready for the workhorse data structures: `Vec<T>` (Python's list / JS array) and `HashMap<K, V>` (dict / Map / object-as-map). Their APIs will feel familiar — but ownership changes the texture: collections *own* their elements, indexing can panic while `.get()` returns `Option`, and iterating comes in three ownership flavors (`iter`, `iter_mut`, `into_iter`). This chapter also touches `HashSet`, `VecDeque`, and `BTreeMap` so you know what exists.

## Definitions & Explanations

### `Vec<T>`: the growable array

```rust
let mut v: Vec<i32> = Vec::new();     // empty; type annotation or later usage needed
let mut v = vec![1, 2, 3];            // vec! macro: literal syntax
v.push(4);                            // append (needs mut)
let last = v.pop();                   // Option<i32> — None if empty
let n = v.len();                      // usize
let empty = v.is_empty();
```

A Vec is heap-allocated, contiguous, and homogeneous: **every element the same type** (this is where Python's `[1, "two", 3.0]` habit dies — if you need mixed, that's an enum: `Vec<Value>` where `Value` is an enum of your cases).

**Two ways to access by index:**

```rust
let third = v[2];        // PANICS if out of bounds — use when the index is proven valid
let third = v.get(2);    // Option<&i32>              — use when it might not be
```

This mirrors Chapter 9's philosophy exactly: `[]` asserts, `.get()` asks. Note `v[2]` on a `Vec<String>` also runs into ownership — you can't move an element out by indexing (`error[E0507]: cannot move out of index`), only borrow it (`&v[2]`) or clone it. Elements are owned by the Vec; that's the deal.

**Ownership with push:** `v.push(s)` *moves* `s` into the vector (Chapter 4). And the borrow rules from Chapter 5 apply across the whole Vec: holding `&v[0]` while calling `v.push(...)` is E0502, because push may reallocate and move every element — the borrow checker is protecting you from a genuine dangling pointer that C++ programmers debug for days.

### The three iteration flavors

This trio is *the* pattern to internalize — it's Chapter 4 and 5 projected onto loops:

```rust
let mut v = vec![String::from("a"), String::from("b")];

for s in &v { }        // 1. iter(): borrows — s is &String. Vec fully usable after.
for s in &mut v {      // 2. iter_mut(): mutable borrows — s is &mut String
    s.push('!');       //    mutate in place
}
for s in v { }         // 3. into_iter(): MOVES — s is String, v is CONSUMED.
// v.len();            //    error[E0382]: borrow of moved value: `v`
```

Python has one `for x in xs` because the GC makes everything a shared reference. Rust asks: are you *reading*, *editing*, or *taking*? Say which. Rule of thumb: write `&v` unless you specifically need the others.

### `HashMap<K, V>`

```rust
use std::collections::HashMap;        // not in the prelude — must import

let mut scores: HashMap<String, u32> = HashMap::new();
scores.insert(String::from("ada"), 95);    // key and value MOVED in
scores.insert(String::from("bob"), 82);

let ada = scores.get("ada");               // Option<&u32> — no KeyError, no undefined
let ada = scores.get("ada").copied().unwrap_or(0);  // common: default if absent

for (name, score) in &scores {             // iteration order is RANDOM (by design!)
    println!("{name}: {score}");
}

scores.remove("bob");                      // Option<u32> — the removed value
scores.contains_key("ada");                // bool
```

Key differences from dict/Map:

- **`get` returns `Option<&V>`** — absent keys are a typed possibility, not an exception (Python) or silent `undefined` (JS).
- **Iteration order is arbitrary** and varies run to run (the hasher is randomly seeded, which also defends against HashDoS). Need sorted iteration? `BTreeMap` keeps keys ordered. Need insertion order (like JS Map / modern dict)? The `indexmap` crate.
- **Keys must implement `Hash + Eq`** — true for ints, `String`, `&str`, tuples of them; not for `f64` (NaN breaks equality — the compiler will stop you, which is a bug class Python allows).

### The entry API: upsert done right

The counting idiom — Python's `d[k] = d.get(k, 0) + 1` or `defaultdict` — in Rust avoids the double lookup:

```rust
let text = "the quick the lazy the";
let mut counts: HashMap<&str, u32> = HashMap::new();
for word in text.split_whitespace() {
    *counts.entry(word).or_insert(0) += 1;
    // entry(word): a handle to the slot, existing or vacant
    // or_insert(0): insert 0 if vacant; returns &mut u32 either way
    // *...+= 1: mutate through the reference
}
// {"the": 3, "quick": 1, "lazy": 1}
```

Also: `or_insert_with(Vec::new)` for expensive defaults, `and_modify(|v| ...).or_insert(...)` for full upsert control.

### The rest of the family (know they exist)

| Type | Use when | Analogy |
|---|---|---|
| `HashSet<T>` | membership, dedup | `set` |
| `BTreeMap<K,V>` / `BTreeSet<T>` | need sorted keys / range queries | `sorted dict` (none in Py!) |
| `VecDeque<T>` | queue / push-pop both ends | `collections.deque` |
| `String` | it really is `Vec<u8>`+UTF-8 (Ch 6) | — |

Default choice: `Vec` for sequences, `HashMap` for lookups. Don't reach further until profiling or semantics demand it.

## Code Examples

### Realistic: grouping and reporting

```rust
use std::collections::HashMap;

#[derive(Debug)]
struct Employee {
    name: String,
    department: String,
    salary: u32,
}

fn main() {
    let staff = vec![
        Employee { name: "Ada".into(),  department: "Eng".into(),   salary: 130_000 },
        Employee { name: "Bob".into(),  department: "Sales".into(), salary: 90_000 },
        Employee { name: "Cleo".into(), department: "Eng".into(),   salary: 120_000 },
    ];

    // Group names by department. Values: Vec<&str> — borrows into `staff`,
    // fine because the map lives no longer than staff does.
    let mut by_dept: HashMap<&str, Vec<&str>> = HashMap::new();
    for e in &staff {
        by_dept.entry(e.department.as_str())
               .or_insert_with(Vec::new)
               .push(e.name.as_str());
    }

    // Sum salaries per department.
    let mut totals: HashMap<&str, u64> = HashMap::new();
    for e in &staff {
        *totals.entry(e.department.as_str()).or_insert(0) += e.salary as u64;
    }

    // Report in sorted order: collect keys, sort, iterate.
    let mut depts: Vec<&&str> = by_dept.keys().collect();
    depts.sort();
    for dept in depts {
        println!("{dept}: {:?} — total ${}", by_dept[*dept], totals[*dept]);
    }
}
```

### Sorting a Vec (several ways you'll actually use)

```rust
fn main() {
    let mut nums = vec![5, 1, 4, 2, 3];
    nums.sort();                              // in place, ascending
    nums.sort_unstable();                     // faster, order of equals unspecified

    let mut words = vec!["banana", "Apple", "cherry"];
    words.sort_by_key(|w| w.to_lowercase());  // case-insensitive (closure: Ch 13)
    words.sort_by(|a, b| b.len().cmp(&a.len())); // custom comparator: by length, desc

    let mut floats = vec![2.5, 1.0, 3.3];
    // floats.sort();  // ERROR: f64 isn't Ord (NaN!). The idiom:
    floats.sort_by(|a, b| a.partial_cmp(b).unwrap());
    println!("{nums:?} {words:?} {floats:?}");

    nums.dedup();          // remove CONSECUTIVE duplicates (sort first for full dedup)
    nums.retain(|&n| n % 2 == 1);   // keep only matching — filter in place, no borrow fight
    println!("{nums:?}");
}
```

### The remove-while-iterating problem, solved right

```rust
fn main() {
    let mut scores = vec![55, 90, 42, 78, 30];

    // WRONG (Chapter 5's E0502):
    // for (i, s) in scores.iter().enumerate() {
    //     if *s < 50 { scores.remove(i); }   // cannot borrow as mutable...
    // }

    // RIGHT: retain — designed for exactly this.
    scores.retain(|&s| s >= 50);
    println!("{scores:?}");   // [55, 90, 78]

    // Alternative when you need the removed ones too:
    let mut all = vec![55, 90, 42, 78, 30];
    let (kept, dropped): (Vec<i32>, Vec<i32>) = all.drain(..).partition(|&s| s >= 50);
    println!("kept {kept:?}, dropped {dropped:?}");
}
```

## Common Pitfalls

- **Holding a borrow across a push (E0502).** `let first = &v[0]; v.push(9); println!("{first}");` — reallocation would dangle the reference. Copy the value out first (`let first = v[0];` for Copy types) or finish reads before writes.
- **Trying to move out of a Vec by indexing (E0507).** `let s: String = v[0];` doesn't compile. Options: borrow `&v[0]`, clone `v[0].clone()`, take ownership of one element `v.remove(0)` / `v.swap_remove(0)`, or consume the Vec `for s in v`.
- **Mutating a map while iterating it.** Same E0502 family as Vec. Collect the keys to change first, then mutate; or use `retain`.
- **Assuming HashMap iteration order.** Tests that snapshot map iteration output are flaky *by design*. Sort keys before asserting, or use `BTreeMap`.
- **`&str` vs `String` keys.** `HashMap<&str, _>` borrows the keys — the map can't outlive the text they point into (E0597 when you try). Owned `String` keys are the default for maps that outlive their input; `get` accepts `&str` either way.
- **Indexing a map with `map[key]`.** Works only for keys present (panics otherwise) and only for Copy-able access patterns; `get` is the everyday accessor. (Also, unlike Python's `defaultdict`, `map[k]` never inserts.)
- **`with_capacity` ignorance.** Pushing a million elements one by one triggers repeated reallocation. If you know the size, `Vec::with_capacity(n)` / `HashMap::with_capacity(n)`. Rarely *matters*, but it's the systems-programming reflex this track is building.

## Practice Exercises

1. Read numbers from a hardcoded `&str` (one per line, some lines invalid), collect the valid ones into a `Vec<i64>`, and print count, min, max, mean, and median (sort for the median). Handle the empty-input case without panicking.
2. Build a word-frequency counter over a paragraph of text: lowercase everything, strip punctuation (`trim_matches(|c: char| !c.is_alphanumeric())`), count with the entry API, then print the top 5 by count (collect into a Vec of pairs and sort). This is a dry run for Project 2.
3. Write `fn dedup_preserving_order(items: Vec<String>) -> Vec<String>` that removes duplicates while keeping first-seen order, using a `HashSet<String>` alongside the output Vec. Mind the ownership: decide (and justify in a comment) whether the set stores clones or whether you restructure to avoid them.
4. Model a sparse grid with `HashMap<(i32, i32), char>`: place a few characters at coordinates, write `fn neighbors(map: &HashMap<(i32,i32),char>, at: (i32,i32)) -> Vec<char>` returning the up-to-8 occupied neighbors, and print a 5x5 window of the grid using `get` + `unwrap_or`.
5. Take a `Vec<Employee>` (from this chapter's example) and produce, in one pass each: (a) a `HashMap<String, u32>` of department → headcount; (b) the highest-paid employee per department (`HashMap<&str, &Employee>` — watch the borrows); (c) a Vec of names sorted by salary descending. No clones of `Employee` allowed.
