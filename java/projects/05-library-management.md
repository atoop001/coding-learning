# Project 5: Library Management System

## Description

A console system for a small library: catalog books, register members, check books out and in, track due state, and search the catalog. Data lives in the right **collections** (`List`, `Set`, `Map`) chosen deliberately, keys behave correctly because `equals`/`hashCode` are handled properly, and one small **generic** utility of your own ties in Chapter 13. This is the project where programs start feeling like *systems* — multiple classes with distinct responsibilities collaborating.

## Difficulty

**Intermediate** — estimated effort: 6–10 hours.

## Chapters used

- 08 (Classes & Objects), 09 (Inheritance & Polymorphism), 10 (Interfaces) — the domain model
- 11 (Packages) — organized package layout
- 12 (Collections Framework) — List/Set/Map selection and iteration
- 13 (Generics) — a generic utility class or method of your own

## Requirements checklist

- [ ] Proper package structure (e.g., `dev.<you>.library.model`, `.store`, `.app`) with deliberate access modifiers
- [ ] `Book` with ISBN, title, author, and copy count — implemented as a `record` or a class with correct `equals`/`hashCode` on ISBN
- [ ] `Member` with a unique member ID and name
- [ ] Catalog stored as `Map<String, Book>` keyed by ISBN — O(1) lookup, no duplicate ISBNs possible
- [ ] Each member's borrowed books tracked as `Map<String, List<String>>` (memberId → list of ISBNs) or a design you can justify in a comment
- [ ] Genre tags per book held in a `Set<String>` — duplicates impossible by construction
- [ ] Checkout: fails cleanly when the book is unknown, out of copies, or the member already holds 3 books (the limit)
- [ ] Return: fails cleanly when the member doesn't hold that book; otherwise restores the copy count
- [ ] Search by title substring (case-insensitive) and filter by genre — both return new lists, never expose internal collections directly
- [ ] "Library report": total titles, total copies, checked-out count, and the most-borrowed book (maintain a borrow counter map)
- [ ] Iteration rules respected: no structural modification during for-each (use `removeIf`/iterators where needed)
- [ ] A generic class or method you wrote yourself, used somewhere real — e.g., `Pair<A,B>` for search results with relevance, or `static <T> List<T> filter(List<T>, Predicate<T>)`-style helper (pre-streams, hand-rolled)
- [ ] Menu-driven `main` exercising everything; invalid input never crashes the app

## Hints

- Choose each collection by asking the interview question out loud: *Do I need ordering? Duplicates? Lookup by key?* Write the one-line justification as a comment above each field — this habit is résumé-grade.
- Do the model first (Book, Member) with tiny hardcoded tests in `main`, then the storage layer (a `Library` class wrapping the maps — nothing outside it touches a collection), then the menu UI last. UI-first is the classic mistake.
- `computeIfAbsent(memberId, k -> new ArrayList<>())` makes "get or create the borrow list" one line.
- For most-borrowed: `Map<String, Integer>` + `merge(isbn, 1, Integer::sum)` at checkout time beats recounting.
- Returning `new ArrayList<>(internalList)` (a defensive copy) from getters keeps outsiders from mutating your state — encapsulation applies to collections too.
- Seed the library with 8–10 books in code so every manual test doesn't start from zero.

## Stretch goals

- Due dates with `java.time.LocalDate` (`.plusDays(14)`), an overdue report, and late fees.
- Reservation queue per book: research `Queue<E>`/`ArrayDeque` — first member in line gets the next returned copy.
- Sort catalog views three ways (title, author, availability) — with `Comparator`s if you've peeked at Chapter 15, or `Comparable` plus manual comparators if not.
- A `Storable` interface (`String toCsvLine()`) on Book and Member — groundwork you'll cash in during Project 7 when persistence arrives.
- Multi-copy modeling done properly: a `Copy` class with its own ID, so *specific physical copies* are tracked rather than a count.
