# Chapter 16: Concurrency & Threads (plus a look at async)

## Overview

Here is the payoff chapter — where every ownership rule you've fought for fifteen chapters reveals what it was *really* protecting you from. Concurrent programming's defining bug, the **data race** (two threads touching the same data, at least one writing, unsynchronized), is undefined behavior in C++, a runtime roulette in Python (partly masked by the GIL), and simply **a compile error in Rust**. "Fearless concurrency" is the honest slogan: the same `Send`/`Sync` machinery that let the compiler reject your single-threaded aliasing bugs makes threading errors unrepresentable. This chapter covers real OS threads, sharing state safely (`Arc`, `Mutex`), channels, the easy path (`rayon`) — and a working introduction to `async`/`await`, which you should recognize but not lead with.

## Definitions & Explanations

### Spawning threads

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {                 // a REAL OS thread
        for i in 1..=3 {
            println!("worker: {i}");
            thread::sleep(Duration::from_millis(10));
        }
        42                                          // threads can return values
    });

    println!("main keeps going...");
    let result = handle.join().unwrap();            // block until done; get the value
    println!("worker returned {result}");
}
```

Unlike Python (GIL: threads never run Python bytecode truly in parallel) and Node (one JS thread + event loop), Rust threads are genuine parallel OS threads using every core, with no runtime referee. Which is exactly why the compiler must be strict about what crosses the boundary.

### `move` and the `'static` bound: why closures must own their data

```rust
let data = vec![1, 2, 3];
let handle = thread::spawn(|| {
    println!("{data:?}");           // borrowing main's data from another thread...
});
```

```text
error[E0373]: closure may outlive the current function, but it borrows `data`,
              which is owned by the current function
help: to force the closure to take ownership of `data`, use the `move` keyword
```

The compiler's reasoning is airtight: the spawned thread might run *after* `main`'s stack frame (and `data`) is gone — a use-after-free across threads. Hence `thread::spawn` requires `F: FnOnce() + Send + 'static` (Chapters 12–13 vocabulary: the closure must own its captures and be safe to send). The fix is the suggested `move`:

```rust
let data = vec![1, 2, 3];
let handle = thread::spawn(move || println!("{data:?}"));  // data MOVED to the thread
handle.join().unwrap();
```

### `Send` and `Sync`: thread safety as traits

- **`Send`**: the type can be *moved* to another thread. (Almost everything; `Rc` is not — its counter isn't atomic.)
- **`Sync`**: the type can be *shared* (`&T`) between threads. (`RefCell` is not — its borrow flag isn't atomic.)

These are auto-derived structurally and checked at compile time. Try to send an `Rc` across threads and you get `error[E0277]: Rc<...> cannot be sent between threads safely` — the compiler enumerating exactly which field breaks the rule. This is the mechanism that makes the guarantees *compositional*: any type you build from thread-safe parts is thread-safe, provably, with no annotations. Nothing like this exists in Python, JS, C++, or Go.

### Sharing state: `Arc<Mutex<T>>`

Chapter 14's `Rc<RefCell<T>>` has an exact thread-safe twin: `Arc` (Atomically Reference Counted) for shared ownership, `Mutex` for exclusive access:

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));          // shared, protected state
    let mut handles = vec![];

    for _ in 0..8 {
        let counter = Arc::clone(&counter);         // each thread gets a handle
        handles.push(thread::spawn(move || {
            for _ in 0..1000 {
                let mut n = counter.lock().unwrap();   // acquire -> MutexGuard
                *n += 1;
            }                                          // guard drops -> UNLOCKS (RAII!)
        }));
    }

    for h in handles { h.join().unwrap(); }
    println!("total: {}", *counter.lock().unwrap());   // exactly 8000, every run
}
```

Two things Python/JS/C++ people should savor:

1. **You cannot forget the mutex.** The data lives *inside* the `Mutex<T>` — the only way to reach it is `lock()`. Compare C++/Python, where the mutex and data are only associated by convention and comments.
2. **You cannot forget to unlock.** Unlocking is the guard's `Drop` (Chapter 4's determinism, again). Early returns, `?`, panics — the lock releases on every path.

(`lock().unwrap()`: lock only errors if another thread *panicked* while holding it — "poisoning". Unwrapping is the standard idiom.)

For plain counters/flags, skip the mutex: `std::sync::atomic::AtomicUsize` with `fetch_add(1, Ordering::Relaxed)` is cheaper.

### Channels: share by communicating

Often better than shared state: move data between threads through a channel (Go's philosophy; Rust's ownership makes it airtight — a sent value is *moved*, so the sender provably can't touch it again):

```rust
use std::sync::mpsc;      // multi-producer, single-consumer
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    for id in 0..3 {
        let tx = tx.clone();                        // multi-producer: clone senders
        thread::spawn(move || {
            tx.send(format!("result from worker {id}")).unwrap();
        });                                          // tx dropped when thread ends
    }
    drop(tx);   // drop the original — rx ends when ALL senders are gone

    for msg in rx {                                  // rx is an iterator! blocks until
        println!("{msg}");                           // a message or all senders dropped
    }
}
```

### The easy button: rayon

For data parallelism (parallelize a loop), don't manage threads at all. The `rayon` crate (cargo add rayon) turns any iterator chain parallel with a one-word change:

```rust
use rayon::prelude::*;

fn main() {
    let inputs: Vec<u64> = (1..=1_000_000).collect();
    let total: u64 = inputs
        .par_iter()                       // was: iter(). That's the entire diff.
        .map(|&n| expensive_work(n))
        .sum();                           // work-stealing pool, all cores, no races
    println!("{total}");
}

fn expensive_work(n: u64) -> u64 { (1..=n % 1000).sum() }
```

This is safe *because* of everything in this track: rayon's API bounds (`Send`/`Sync`) let the compiler verify your closure can't race. In C++ this one-liner is a minefield; in Python it's a `multiprocessing` pickle-fest. Project 6 uses this.

### A look at async/await

Threads suit CPU-bound and modestly-concurrent work. For *massive I/O concurrency* (10k network connections), Rust has `async`/`await` — syntactically your home turf from JS/Python:

```rust
// cargo add tokio --features full
use std::time::Duration;

async fn fetch_pretend(name: &str, ms: u64) -> String {
    tokio::time::sleep(Duration::from_millis(ms)).await;   // yields; doesn't block
    format!("{name} done after {ms}ms")
}

#[tokio::main]                            // sets up the async runtime around main
async fn main() {
    // Sequential: ~300ms
    let a = fetch_pretend("a", 100).await;

    // Concurrent on one thread: ~150ms total — like Promise.all / asyncio.gather
    let (b, c) = tokio::join!(
        fetch_pretend("b", 150),
        fetch_pretend("c", 120),
    );
    println!("{a}\n{b}\n{c}");
}
```

The crucial differences from JS to keep straight:

- **Futures are lazy.** Calling `fetch_pretend(...)` does *nothing* until `.await`ed (a JS Promise starts immediately). Forgetting `.await` earns a compiler warning: `unused implementer of Future that must be used`.
- **No built-in runtime.** JS ships an event loop; Rust's std defines only the `Future` trait, and you pick a runtime crate — in practice **tokio**. Hence `#[tokio::main]`.
- Async is *cooperative* concurrency on few threads (great for I/O-bound), threads are *preemptive* parallelism (great for CPU-bound); tokio actually layers both.
- Honest note: async Rust is genuinely harder than sync Rust (lifetimes across `.await` points, `Pin`, `Send` bounds on futures). The guidance for this track: **be able to read it, write servers with it later, and default to threads/rayon for everything CPU-shaped.** When you go there, tokio's tutorial is the canonical next step.

## Code Examples

### Worked pattern: parallel word count with channels (Project 6 preview)

```rust
use std::sync::mpsc;
use std::thread;

fn count_words(text: &str) -> usize {
    text.split_whitespace().count()
}

fn main() {
    // Pretend these are files read from disk:
    let documents = vec![
        String::from("the quick brown fox"),
        String::from("jumped over the lazy dog and kept running"),
        String::from("rust makes concurrency boring in the best way"),
    ];

    let (tx, rx) = mpsc::channel();
    let n_docs = documents.len();

    for (id, doc) in documents.into_iter().enumerate() {   // MOVE each doc out
        let tx = tx.clone();
        thread::spawn(move || {                            // doc owned by its thread
            let count = count_words(&doc);
            tx.send((id, count)).unwrap();                 // ownership of result moves back
        });
    }
    drop(tx);

    let mut total = 0;
    for (id, count) in rx {
        println!("doc {id}: {count} words");
        total += count;
    }
    assert_eq!(n_docs, 3);
    println!("total: {total}");
}
```

Trace the ownership: each `String` moves into exactly one thread; results move back through the channel; at no point can two threads see one document. The design that ownership *forces* is the design a concurrency expert would have chosen anyway — that's the deep lesson of this chapter.

### The race that cannot compile

```rust
use std::thread;

fn main() {
    let mut total = 0;
    let h1 = thread::spawn(move || { total += 1; });  // moves total (a COPY of it!)
    let h2 = thread::spawn(move || { total += 1; });  // moves another copy
    h1.join().unwrap();
    h2.join().unwrap();
    println!("{total}");   // 0 — each thread incremented its own copy
}
```

This compiles (i32 is Copy) but the *sharing you intended never happened* — and the printed `0` shows it. Try to actually share (`&mut total` in both closures) and you get E0499/E0373: the compiler refuses two mutable captures. The only ways to express "two threads increment one counter" are the safe ones: `Arc<Mutex<_>>` or atomics. The race isn't caught at runtime — it is *unwritable*.

## Common Pitfalls

- **E0373 "closure may outlive the current function".** The chapter's signature error. Fix is `move` — and if you need the value back afterward, that's what return values, channels, or `Arc` are for. Don't fight to borrow across `spawn`; scoped threads (`thread::scope`) exist for the borrow case and are worth looking up.
- **Cloning the Arc wrong.** `let c = Arc::clone(&counter)` *before* each `move` closure. Moving the original into thread 1 leaves nothing for thread 2 (E0382). The `let counter = Arc::clone(&counter);` shadowing inside the loop is the idiom.
- **Holding a lock across slow work.** `let g = m.lock().unwrap(); expensive();` serializes all threads — correctness without concurrency. Lock late, copy out, drop early (explicit `drop(g)` or a `{ }` scope). Deadlocks (two locks, two orders) also remain *possible* — Rust prevents races, not all logic bugs.
- **Forgetting `drop(tx)`.** The `for msg in rx` loop never ends because the original sender still exists. If your program hangs at the receive loop, count your senders.
- **`Rc`/`RefCell` in threads.** E0277 `cannot be sent between threads safely`. Mechanical translation: `Rc→Arc`, `RefCell→Mutex` (or `RwLock` for many-readers). The compiler enforces the swap; Chapter 14 taught you the semantics already.
- **Reaching for async because JS habits say I/O = async.** For a CLI reading 20 files, sync + threads (or just sync) is simpler and equally fast. Async earns its complexity at *scale* (thousands of concurrent connections). Threads first; rayon for parallel compute; tokio when you build the server.
- **Forgetting `.await`** (the future does nothing, with a warning) — or conversely, `.await`ing sequentially when you meant `join!` (correct, but you just serialized your concurrency — the 300ms vs 150ms difference above).

## Practice Exercises

1. Spawn 4 threads that each compute the sum of a distinct quarter of `1..=10_000_000` and return it from the closure; join and combine. Verify against the closed-form total. Then redo it with one `par_iter().sum()` via rayon and compare wall-clock times with `std::time::Instant`.
2. Build the `Arc<Mutex<HashMap<String, u32>>>` version of the word counter: N threads each process one document and merge counts into the shared map. Then refactor to the channel design (threads send their own local `HashMap`, main merges). One comment: which contends less, and why?
3. Take the "race that cannot compile" example and (a) demonstrate the Copy-trap prints 0, (b) fix it with `Arc<Mutex<i32>>`, (c) fix it with `AtomicI32`. Confirm both fixes print 2 across many runs.
4. Create a pipeline with two channel stages: a producer thread sends numbers 1..=100; a squarer thread receives, squares, and forwards to a second channel; main receives and sums. Shut the whole pipeline down cleanly (no hangs) — document which `drop` makes each stage terminate.
5. (Async taster) With tokio: write `async fn simulated_download(name: &str, ms: u64) -> usize` (sleep, return `ms as usize`), then download five "files" first sequentially with `.await` in a loop, then concurrently with `tokio::join!` or `futures::future::join_all`. Print both elapsed times and explain the ratio in a comment.
