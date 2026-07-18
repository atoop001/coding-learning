# Project 5: Inventory Management System (Traits & Generics)

## Description

Build a warehouse inventory system as a *library* with a small demo binary: products of different kinds (physical goods with weight, digital goods with download size, perishables with expiry), a generic storage layer, pluggable pricing rules, and multiple report formats. Unlike Projects 1–4, the deliverable here is a well-designed **API**: this project is a workout for traits, generics, trait objects, and the standard traits (`Display`, `From`, `PartialOrd`, `Default`) — the chapter-11-and-beyond muscles that separate "can write Rust" from "designs in Rust."

## Difficulty

**Intermediate-advanced.** Estimated effort: **8–12 hours.**

## Chapters Used

- Chapter 7 (structs, builders)
- Chapter 8 (enums for domain modeling)
- Chapter 9 (error enum for domain rules)
- Chapter 10 (HashMap-backed storage)
- Chapter 11 (the core: traits, generics, bounds, trait objects, standard traits)
- Chapter 13 (iterator pipelines for the reports)
- Chapter 14 (one deliberate use of interior mutability — see requirements)

## Requirements Checklist

- [ ] Library crate (`src/lib.rs` + modules) with a demo `src/main.rs` exercising everything
- [ ] A `Product` **trait** with: `fn sku(&self) -> &str`, `fn name(&self) -> &str`, `fn base_price_cents(&self) -> u64`, and a default method `fn display_line(&self) -> String` built from the others
- [ ] At least three implementing types with genuinely different data: `PhysicalItem` (weight, shelf location), `DigitalItem` (file size, license kind as an enum), `PerishableItem` (expiry date string, storage temp)
- [ ] A generic `Inventory<T: Product>` storing items in a `HashMap<String, T>` keyed by SKU, with `add`, `get(&self, sku: &str) -> Option<&T>`, `remove -> Option<T>`, and `iter()` — plus a **separate** heterogeneous `MixedInventory` storing `Vec<Box<dyn Product>>`; the demo must use both and a comment must state when each is the right tool
- [ ] Adding a duplicate SKU returns `Err(InventoryError::DuplicateSku(..))` — your error enum with `Display`; at least 3 variants used in practice
- [ ] A `PricingRule` trait (`fn price_cents(&self, product: &dyn Product) -> u64`) with at least three implementations: flat markup, percentage margin, and a clearance rule that discounts perishables (downcasting is NOT required — design the trait so it isn't needed, e.g. via a `fn category(&self) -> Category` method on `Product`)
- [ ] A report function generic over a `Render`-style trait (Chapter 11's example evolved): plain text and CSV renderers minimum; adding a renderer must require zero changes to existing code
- [ ] Stock levels tracked with interior mutability: a `StockCounter` using `Cell<u32>` or `RefCell` so that `reserve(&self, n)` works through a shared reference — with a comment justifying why interior mutability is warranted here (or arguing it isn't and showing the `&mut` design — a reasoned "no" with working code is a full pass)
- [ ] Standard traits implemented meaningfully: `Display` for at least one product type, `From<(&str, &str, u64)>` for one, `Default` for `Inventory<T>`, `PartialOrd`/`Ord` (or `sort_by_key`) used to produce a price-sorted report
- [ ] Iterator-only report: total inventory value, most expensive item, and count per category computed with iterator chains — no `for` loops in report code
- [ ] At least 8 unit tests, including one that verifies trait-object dispatch (a `Vec<Box<dyn Product>>` containing all three types produces the right display lines)
- [ ] `cargo clippy -- -D warnings` and `cargo fmt` clean; every `pub` item has a `///` doc comment

## Hints

- Write the traits *first*, on paper, before any impl. The requirement that clearance pricing must not downcast forces the key design question: what must `Product` expose? A `Category` enum method is one answer; a `fn expiry(&self) -> Option<&str>` is another. Choose and defend in a comment.
- `Inventory<T>` vs `MixedInventory` is the static-vs-dynamic dispatch lesson made concrete: a warehouse of one known type vs a catalog page of anything. Your demo should make the difference feel obvious.
- The orphan rule will bite if you try `impl Display for Box<dyn Product>` — remember the newtype escape hatch from Chapter 11's pitfalls.
- For `From<(&str, &str, u64)>`: tuples of borrowed strings mean your `from` allocates owned Strings — that's correct and normal.
- Generic method on the storage: `fn total_value<P: PricingRule>(&self, rule: &P) -> u64` — or take `&dyn PricingRule`; try both signatures and note in a comment which callers prefer.
- If `RefCell` panics in a test (`already borrowed`), you've found the Chapter 14 lesson in the wild: shrink your borrow scopes.
- Doc comments with runnable examples (```` ``` ````) will be tested by `cargo test` — write at least two.

## Stretch Goals

- A query DSL via closures: `inventory.find(|p| p.base_price_cents() > 500)` returning an iterator (return `impl Iterator` — harder than it looks with borrows; a `Vec` is an acceptable fallback and the difference is worth a comment).
- Implement `IntoIterator` for `&Inventory<T>` so `for item in &inv` works.
- A `Discountable` marker trait + blanket impl experiment: `impl<T: Product + Discountable> ...` — feel how bounds compose.
- Persist and reload the mixed inventory using a hand-rolled tagged format (`physical|SKU1|...`) — trait objects can't be naively serialized; the tag-and-match pattern you'll invent is exactly what serde's enum tagging automates.
- Property-style testing with the `proptest` crate on the pricing rules (never negative, clearance ≤ base).
