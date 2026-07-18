# Chapter 8: Classes & Objects

## Overview

Classes are blueprints; objects are the things built from them. C# is a deeply object-oriented language — essentially all code lives in classes — so this chapter is the gateway to everything that follows. You'll learn fields, **properties** (C#'s signature feature that Python/JS folks often love), constructors, methods on objects, `static` vs instance members, and encapsulation: the discipline of hiding internal state behind a controlled surface.

## Definitions & Explanations

### Class, object, instantiation

```csharp
class Player                       // the blueprint
{
    public string Name = "";       // fields: per-object data
    public int Health = 100;
}

Player p1 = new Player();          // instantiation — 'new' is required (like JS, unlike Python)
p1.Name = "Ada";
p1.Health -= 30;

var p2 = new Player();             // an independent object
Console.WriteLine(p2.Health);      // 100 — p2 unaffected by p1
```

Python comparison: no `self` parameter — inside methods, the current object is `this` (usually implicit). No `__init__` — constructors bear the class name.

### Access modifiers & encapsulation

- `public` — visible to everyone.
- `private` — visible only inside this class (**default** for members).
- `protected` — this class and subclasses (Chapter 9).

**Encapsulation** means: keep data `private`, expose behavior through `public` methods/properties, so the object can protect its own invariants (e.g., health never goes below zero). Python's `_underscore` is a polite request; C#'s `private` is enforced by the compiler.

### Properties — controlled access that looks like a field

```csharp
class Player
{
    // Auto-property: compiler generates a hidden backing field
    public string Name { get; set; }

    // Read-only from outside; only this class can set it
    public int Level { get; private set; } = 1;

    // Full property with logic — the classic encapsulation tool
    private int _health = 100;               // convention: _camelCase for private fields
    public int Health
    {
        get => _health;
        set => _health = Math.Clamp(value, 0, 100);   // 'value' = what was assigned
    }

    // Computed (get-only) property — like Python's @property
    public bool IsAlive => _health > 0;
}

var p = new Player { Name = "Ada" };   // object initializer syntax
p.Health = -50;                        // setter clamps it
Console.WriteLine(p.Health);           // 0
Console.WriteLine(p.IsAlive);          // False
```

JS comparison: like `get`/`set` accessors, but idiomatic and everywhere. Callers write `p.Health = 5` — no `p.setHealth(5)` ceremony.

### Constructors

```csharp
class BankAccount
{
    public string Owner { get; }               // get-only: set ONLY in constructor
    public decimal Balance { get; private set; }

    // Constructor: same name as class, no return type
    public BankAccount(string owner, decimal openingBalance = 0)
    {
        if (string.IsNullOrWhiteSpace(owner))
            throw new ArgumentException("Owner required", nameof(owner));
        Owner = owner;
        Balance = openingBalance;
    }

    public void Deposit(decimal amount)
    {
        if (amount <= 0) throw new ArgumentException("Must be positive");
        Balance += amount;
    }

    public bool TryWithdraw(decimal amount)
    {
        if (amount <= 0 || amount > Balance) return false;
        Balance -= amount;
        return true;
    }
}

var acct = new BankAccount("Ada", 100m);
acct.Deposit(50m);
// acct.Balance = 1_000_000m;   // ❌ compile error — encapsulation working as intended
```

If you write *no* constructor, you get a free parameterless one. Once you write any constructor, the free one disappears. Constructors can be overloaded like methods.

### static — belongs to the class, not to objects

```csharp
class Counter
{
    public static int TotalCreated { get; private set; }   // shared by ALL instances
    public int Id { get; }

    public Counter()
    {
        TotalCreated++;              // static state
        Id = TotalCreated;           // instance state
    }
}

new Counter(); new Counter();
Console.WriteLine(Counter.TotalCreated);   // 2 — accessed via the CLASS name
```

`Console.WriteLine` and `Math.Max` are static methods — that's why you never `new Console()`.

### ToString and records (a peek)

Every object inherits `ToString()`; override it for readable output:

```csharp
public override string ToString() => $"{Owner}: {Balance:C}";
```

For pure-data classes, C# offers `record` — value-style equality and a nice `ToString` for free:

```csharp
public record Point(double X, double Y);      // done. Immutable, comparable, printable.
var a = new Point(1, 2);
Console.WriteLine(a);                          // Point { X = 1, Y = 2 }
```

## Code Examples

### A complete, realistic class in a small program

```csharp
// Library book tracker — encapsulation with real invariants
class Book
{
    public string Title { get; }
    public string Author { get; }
    public bool IsCheckedOut { get; private set; }
    private readonly List<string> _history = new();   // readonly: field can't be reassigned

    public Book(string title, string author)
    {
        Title = title;
        Author = author;
    }

    public bool CheckOut(string borrower)
    {
        if (IsCheckedOut) return false;               // protect the invariant
        IsCheckedOut = true;
        _history.Add($"{borrower} @ {DateTime.Now:d}");
        return true;
    }

    public void Return() => IsCheckedOut = false;

    public int TimesBorrowed => _history.Count;       // expose a summary, not the list

    public override string ToString() =>
        $"\"{Title}\" by {Author}" + (IsCheckedOut ? " [OUT]" : "");
}

class Program
{
    static void Main()
    {
        var books = new List<Book>
        {
            new Book("Dune", "Frank Herbert"),
            new Book("The Pragmatic Programmer", "Hunt & Thomas"),
        };

        books[0].CheckOut("Ada");
        bool ok = books[0].CheckOut("Grace");     // false — already out

        foreach (var b in books)
            Console.WriteLine($"{b}  (borrowed {b.TimesBorrowed}x)");
        Console.WriteLine($"Second checkout succeeded? {ok}");
    }
}
```

## Common Pitfalls

**1. Forgetting `new`.**

```csharp
Player p = Player();          // ❌ Python habit
Player p = new Player();      // ✅  (or: Player p = new();)
```

**2. NullReferenceException from an unassigned reference.**

```csharp
Player p = null!;
p.Name = "Ada";               // ❌ NullReferenceException at runtime
```

The #1 C# runtime error. Keep nullable warnings on and heed them.

**3. Public fields instead of properties.** `public int Health;` works but gives up all control and breaks conventions/tooling. Use `public int Health { get; set; }` at minimum — it costs nothing and lets you add validation later without changing callers.

**4. Doing real work in a field initializer that needs constructor parameters.** Field initializers run before the constructor body; they can't see its parameters. Put dependent setup in the constructor.

**5. Confusing class-level and instance-level.**

```csharp
Counter.Id            // ❌ Id is per-instance
myCounter.TotalCreated // ❌ (warning/error) — statics via the class name
Counter.TotalCreated  // ✅
```

**6. Comparing objects with `==` expecting content equality.** For classes, `==` compares *references* by default (same object?). Two different `Book` objects with identical titles are not `==`. Use records for data-shaped types, or override `Equals`.

## Practice Exercises

1. Create a `Rectangle` class with `Width`/`Height` properties that reject non-positive values (throw `ArgumentException`), plus computed `Area` and `Perimeter` properties and a sensible `ToString`. Exercise all of it in `Main`.
2. Build a `Temperature` class that stores Celsius internally but exposes both `Celsius` and `Fahrenheit` properties — setting either updates the single internal value correctly.
3. Write a `Stopwatch`-like class `TaskTimer` with `Start()`, `Stop()`, and a get-only `Elapsed` (use `DateTime.Now`). Prevent nonsense like stopping before starting (decide: ignore, or throw?).
4. Extend `BankAccount` above with a transaction history (private list, exposed as a formatted report string), an overloaded constructor, and a static property tracking the bank's total balance across all accounts.
5. Model a `Deck` of cards: a `Card` record (Rank, Suit), a `Deck` class that fills all 52 in its constructor, `Shuffle()` (swap random pairs), and `Deal()` returning and removing the top card — throwing when empty. Deal a 5-card hand and print it.
