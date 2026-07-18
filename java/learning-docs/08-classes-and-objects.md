# Chapter 8: Classes and Objects

## Overview

This is the chapter where Java stops being "a stricter Python" and becomes itself. Java is an **object-oriented** language to its core: programs are structured as classes that bundle *data* (fields) with *behavior* (methods), and running programs are populations of *objects* — instances of those classes — collaborating.

You've used objects all along (`String`, `Scanner`, arrays). Now you'll build your own: fields, constructors, `this`, encapsulation with getters/setters, `toString`, and the difference between instance and static members finally lands.

## Definitions & Explanations

### Class vs object

- A **class** is a blueprint: "every BankAccount has an owner and a balance, and can deposit and withdraw."
- An **object** (instance) is one concrete thing built from the blueprint: *Alice's* account with *$250* in it.

```java
BankAccount acct = new BankAccount("Alice", 250.0);
//    │        │         └─ constructor call: builds and initializes the object
//    │        └─ variable holding a REFERENCE to the new object
//    └─ the type is the class name
```

Python: like `class` / `acct = BankAccount(...)` without the `new`. JS: like `class` with `new`, same keyword even.

### Fields (instance variables)

Fields are variables declared *in the class, outside any method*. Each object gets its own copy:

```java
public class BankAccount {
    private String owner;      // every account has its own owner...
    private double balance;    // ...and its own balance
}
```

Unlike locals, fields get default values (`0`, `false`, `null`) — though good constructors set them explicitly.

### Constructors

A constructor is a special method that runs at `new` time to initialize the object. Same name as the class, **no return type**:

```java
public BankAccount(String owner, double openingBalance) {
    this.owner = owner;
    this.balance = openingBalance;
}
```

- If you write *no* constructor, Java supplies an invisible no-argument default one.
- The moment you write any constructor, the default disappears.
- Constructors can be overloaded, and one can call another with `this(...)` (must be the first statement).

There's no `__init__`/`self` — the constructor is named after the class, and `this` plays the role of `self`, but it's implicit: inside instance methods you can write `balance` instead of `this.balance` unless a name conflict forces the prefix.

### `this`

`this` refers to "the current object." Its two main uses:

1. Disambiguating field from parameter: `this.owner = owner;`
2. Passing the current object along: `registry.add(this);`

### Encapsulation: private fields, public methods

**Encapsulation** = hiding an object's data behind a controlled interface. Fields are declared `private` (only this class can touch them); the outside world goes through public methods. Why it matters:

- The class can *enforce rules* (a balance that can't go negative) in one place.
- Internals can change later without breaking callers.
- It's the norm in every professional Java codebase — interviewers expect it.

The conventional accessors are **getters** and **setters**:

```java
public double getBalance() { return balance; }        // getter: read access
public void setOwner(String owner) {                  // setter: controlled write
    if (owner == null || owner.isBlank()) {
        throw new IllegalArgumentException("owner required");
    }
    this.owner = owner;
}
```

Don't reflexively generate a setter for every field — only expose what should be changeable. A `balance` should change via `deposit`/`withdraw` (which validate), not `setBalance`.

### `toString()`

Every class inherits a useless default `toString()` (`BankAccount@1b6d3586`). Override it to control how your object prints:

```java
@Override
public String toString() {
    return "BankAccount[owner=" + owner + ", balance=" + balance + "]";
}
```

`System.out.println(acct)` calls it automatically — like Python's `__str__`.

### static vs instance, revisited

- **Instance** members: one per object (`balance`).
- **Static** members: one per class, shared (`private static int totalAccounts;` — a counter every constructor bumps). Access as `BankAccount.getTotalAccounts()`.

Static methods can't read instance fields (whose object's?); instance methods *can* read statics.

## Code Examples

### A complete, well-encapsulated class

```java
public class BankAccount {

    // --- static: shared across ALL accounts ---
    private static int totalAccounts = 0;

    // --- instance fields: one set PER account ---
    private final String owner;      // final: set once in constructor, never changes
    private double balance;

    // Constructor: establish a valid object or refuse to build one
    public BankAccount(String owner, double openingBalance) {
        if (owner == null || owner.isBlank()) {
            throw new IllegalArgumentException("Owner name required");
        }
        if (openingBalance < 0) {
            throw new IllegalArgumentException("Opening balance cannot be negative");
        }
        this.owner = owner;
        this.balance = openingBalance;
        totalAccounts++;             // static counter: shared
    }

    // Overloaded constructor delegating to the main one
    public BankAccount(String owner) {
        this(owner, 0.0);            // must be first statement
    }

    // --- behavior: the ONLY ways to change the balance ---
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        balance += amount;
    }

    public boolean withdraw(double amount) {
        if (amount <= 0 || amount > balance) {
            return false;            // signal failure instead of corrupting state
        }
        balance -= amount;
        return true;
    }

    // --- accessors ---
    public String getOwner() { return owner; }
    public double getBalance() { return balance; }
    public static int getTotalAccounts() { return totalAccounts; }

    @Override
    public String toString() {
        return String.format("BankAccount[owner=%s, balance=%.2f]", owner, balance);
    }
}
```

### Using it

```java
public class BankDemo {
    public static void main(String[] args) {
        BankAccount alice = new BankAccount("Alice", 250.0);
        BankAccount bob = new BankAccount("Bob");          // overloaded ctor: balance 0

        alice.deposit(100.0);
        boolean ok = alice.withdraw(500.0);                // false — insufficient funds
        System.out.println("Big withdrawal worked? " + ok);

        bob.deposit(40.0);

        System.out.println(alice);   // toString() called automatically
        System.out.println(bob);
        System.out.println("Accounts open: " + BankAccount.getTotalAccounts()); // 2

        // Each object has its OWN state:
        System.out.println(alice.getBalance());  // 350.0
        System.out.println(bob.getBalance());    // 40.0
    }
}
```

### Objects reference each other

```java
public class Author {
    private final String name;
    public Author(String name) { this.name = name; }
    public String getName() { return name; }
}

public class Book {
    private final String title;
    private final Author author;      // a field can hold another object

    public Book(String title, Author author) {
        this.title = title;
        this.author = author;
    }

    @Override
    public String toString() {
        return "\"" + title + "\" by " + author.getName();
    }

    public static void main(String[] args) {
        Author tolkien = new Author("J.R.R. Tolkien");
        Book b1 = new Book("The Hobbit", tolkien);
        Book b2 = new Book("The Silmarillion", tolkien);  // SAME author object, shared
        System.out.println(b1);
        System.out.println(b2);
    }
}
```

(One public class per file: `Author.java` and `Book.java` in practice — Chapter 11.)

## Common Pitfalls

### 1. Shadowing fields in the constructor

```java
public BankAccount(String owner) {
    owner = owner;          // ❌ assigns the parameter to itself; field stays null
    this.owner = owner;     // ✅ this. targets the field
}
```

IntelliJ warns about this; heed the squiggle.

### 2. Forgetting `new`

```java
BankAccount acct;
acct.deposit(50);           // ❌ compile error: variable might not be initialized
BankAccount acct = null;
acct.deposit(50);           // ❌ compiles, then NullPointerException at runtime
BankAccount acct = new BankAccount("Ada", 0);   // ✅ declaring ≠ creating
```

### 3. Accidentally declaring a return type on a constructor

```java
public void BankAccount(String owner) { ... }   // ❌ this is a METHOD named BankAccount
public BankAccount(String owner) { ... }        // ✅ no return type = constructor
```

The broken version compiles silently and your "constructor" never runs.

### 4. Public fields breaking invariants

```java
public double balance;              // ❌ anyone can write acct.balance = -999999;
private double balance;             // ✅ force changes through deposit/withdraw
```

### 5. Comparing objects with `==`

```java
new BankAccount("A", 1) == new BankAccount("A", 1)    // false: different objects
```

For value comparison, override `equals()` (and `hashCode()` — Chapter 12 explains why they travel together). Until then, compare meaningful fields explicitly.

### 6. Static method touching instance state

```java
public static double getBalance() { return balance; }  // ❌ non-static field referenced from static context
```

A static method has no `this` — which account's balance would it be? Remove `static`.

## Practice Exercises

1. **`Dog` class.** Fields: `name`, `breed`, `ageInYears` (all private). Constructor validating non-blank name and non-negative age. Methods: `bark()` (prints), `humanYears()` (returns age × 7), getters, and a `toString`. In `main`, create three dogs and exercise everything.
2. **`Rectangle` with invariants.** Private `width`/`height` (positive doubles enforced in the constructor). Methods `area()`, `perimeter()`, `isSquare()`, and `scaledBy(double factor)` that returns a **new** Rectangle rather than mutating (immutable style — note both fields can be `final`). Explain in a comment one advantage immutability gave you.
3. **`Counter` with statics.** Instance method `increment()` and getter for the per-object count; static field tracking total increments across *all* counters with a static getter. Create two counters, increment them differently, and print per-object and global totals. Predict the outputs before running.
4. **`Temperature` conversions.** Store degrees Celsius privately. Provide `getCelsius()`, `getFahrenheit()`, `getKelvin()` (computed, not stored), and a static *factory method* `Temperature.fromFahrenheit(double f)` that converts and returns a new object. Why is a factory method nicer than a second constructor here? (Hint: what would `new Temperature(double)` be ambiguous about?)
5. **`Playlist` of objects.** A `Song` class (title, artist, seconds) and a `Playlist` class holding a `Song[]` plus a count (fixed capacity, e.g., 10). Playlist methods: `add(Song)` (refuse when full, return boolean), `totalDuration()` formatted `mm:ss`, and `longestSong()`. No `ArrayList` yet — manage the array yourself; you'll appreciate Chapter 12 more.
