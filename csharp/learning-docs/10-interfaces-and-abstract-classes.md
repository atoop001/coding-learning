# Chapter 10: Interfaces & Abstract Classes

## Overview

Sometimes you want to define *what* a type must be able to do without saying *how*. C# gives you two tools: **interfaces** (a pure contract: "anything implementing me can do X") and **abstract classes** (a partial implementation that can't be instantiated and forces subclasses to fill in the gaps). These two are the backbone of professional C# design — dependency injection, testing, and the entire .NET library lean on interfaces heavily. Python has duck typing and ABCs; TypeScript has `interface`; C#'s versions are enforced at compile time.

## Definitions & Explanations

### Interfaces — pure contracts

```csharp
interface IShape                    // convention: interface names start with I
{
    double Area();                  // no body — just the signature
    double Perimeter();
    string Name { get; }            // properties can be required too
}
```

A class **implements** an interface by listing it and providing *every* member:

```csharp
class Circle : IShape
{
    public double Radius { get; }
    public Circle(double r) => Radius = r;

    public string Name => "Circle";
    public double Area() => Math.PI * Radius * Radius;
    public double Perimeter() => 2 * Math.PI * Radius;
}
```

Miss one member and the code won't compile — the contract is enforced, unlike Python duck typing where you find out at runtime.

Key powers:
- A class implements **any number** of interfaces (this replaces multiple inheritance).
- An interface variable can hold any implementer: `IShape s = new Circle(2);`
- Interfaces can inherit from other interfaces.

### Abstract classes — incomplete blueprints

```csharp
abstract class Shape                          // cannot do: new Shape()
{
    public string Name { get; }
    protected Shape(string name) => Name = name;

    public abstract double Area();            // abstract: NO body, MUST override

    // Concrete shared code — this is what interfaces (classically) can't give you
    public void PrintReport() =>
        Console.WriteLine($"{Name}: {Area():F2}");
}

class Square : Shape
{
    public double Side { get; }
    public Square(double side) : base("Square") => Side = side;
    public override double Area() => Side * Side;   // forced by 'abstract'
}
```

- `abstract` methods are like `virtual` with no body — deriving classes **must** `override` them.
- Abstract classes can also contain normal fields, constructors, and fully implemented methods.
- Python analogue: `abc.ABC` with `@abstractmethod` — same idea, compile-time enforced.

### Choosing between them

| Question | Interface | Abstract class |
|---|---|---|
| Models... | a capability ("can do") | an identity ("is a") |
| Multiple per class? | Yes, many | No, only one base |
| Can hold state (fields)? | No | Yes |
| Can provide shared code? | Mostly no (see note) | Yes — methods, constructors |
| Typical name | `IComparable`, `IDisposable` | `Stream`, `ControllerBase` |

Rule of thumb: **default to interfaces**; use an abstract class when several related classes genuinely share code and state. (Since C# 8 interfaces *can* carry default method bodies, but treat that as an advanced tool.)

### Interfaces you'll meet constantly

- `IEnumerable<T>` — "can be foreach-ed". Arrays, Lists, LINQ results all implement it.
- `IComparable<T>` — "can be ordered" — enables `list.Sort()`.
- `IDisposable` — "has cleanup to do" — enables `using` blocks (Chapter 16).

```csharp
class Player : IComparable<Player>
{
    public string Name { get; set; } = "";
    public int Score { get; set; }

    public int CompareTo(Player? other) =>
        other is null ? 1 : other.Score.CompareTo(Score);  // descending by score
}

var players = new List<Player> { /* ... */ };
players.Sort();          // works because Player is IComparable<Player>
```

## Code Examples

### A plugin-style design with interfaces

```csharp
// Notification system — swap implementations freely; THE core pattern
// behind testable, professional C# (and dependency injection)
interface INotifier
{
    void Send(string recipient, string message);
}

class EmailNotifier : INotifier
{
    public void Send(string recipient, string message) =>
        Console.WriteLine($"[EMAIL to {recipient}] {message}");
}

class SmsNotifier : INotifier
{
    public void Send(string recipient, string message) =>
        Console.WriteLine($"[SMS to {recipient}] {message}");
}

class ConsoleNotifier : INotifier          // great fake for testing!
{
    public List<string> Sent { get; } = new();
    public void Send(string recipient, string message) =>
        Sent.Add($"{recipient}: {message}");
}

class OrderService
{
    private readonly INotifier _notifier;                 // depends on the CONTRACT,
    public OrderService(INotifier notifier) =>            // not any concrete class
        _notifier = notifier;

    public void PlaceOrder(string customer, string item)
    {
        // ... imagine real order logic here ...
        _notifier.Send(customer, $"Your {item} is on the way!");
    }
}

class Program
{
    static void Main()
    {
        // Same service, three behaviors — chosen at one single point:
        var service = new OrderService(new EmailNotifier());
        service.PlaceOrder("ada@example.com", "keyboard");

        var testDouble = new ConsoleNotifier();
        new OrderService(testDouble).PlaceOrder("test@example.com", "mouse");
        Console.WriteLine($"Captured {testDouble.Sent.Count} notification(s).");
    }
}
```

### Combining an abstract base with interfaces

```csharp
interface IFlyer  { void Fly(); }
interface ISwimmer { void Swim(); }

abstract class Animal
{
    public string Name { get; }
    protected Animal(string name) => Name = name;
    public abstract string Speak();
}

class Duck : Animal, IFlyer, ISwimmer       // one base class + many interfaces
{
    public Duck(string name) : base(name) { }
    public override string Speak() => "Quack";
    public void Fly() => Console.WriteLine($"{Name} flies south.");
    public void Swim() => Console.WriteLine($"{Name} paddles around.");
}

class Penguin : Animal, ISwimmer            // swims, does NOT fly — no dilemma
{
    public Penguin(string name) : base(name) { }
    public override string Speak() => "Noot";
    public void Swim() => Console.WriteLine($"{Name} torpedoes through water.");
}

// Polymorphism over a capability:
static void SwimTeamPractice(List<ISwimmer> team)
{
    foreach (var s in team) s.Swim();
}
```

## Common Pitfalls

**1. Trying to instantiate an interface or abstract class.**

```csharp
IShape s = new IShape();       // ❌ CS0144
Shape sh = new Shape("x");     // ❌ CS0144 (abstract)
IShape s = new Circle(2);      // ✅ instantiate a concrete implementer
```

**2. Forgetting a member of the contract.** Error CS0535 ("does not implement interface member...") — read it; it names exactly what's missing. In VS Code, the lightbulb quick-fix "Implement interface" stubs everything.

**3. Interface members are implicitly public — don't add modifiers in the interface, and don't make the implementation private.**

```csharp
class Circle : IShape
{
    double Area() => ...;            // ❌ private — doesn't satisfy the contract
    public double Area() => ...;     // ✅
}
```

**4. Abstract method with a body / virtual method without one.**

```csharp
abstract class A
{
    public abstract void Go() { }    // ❌ abstract = no body
    public virtual void Stop();      // ❌ virtual = needs a body
}
```

**5. Making an interface for everything.** One-class-one-interface pairs everywhere is ceremony without benefit. Introduce an interface when there are (or will plausibly be) *multiple* implementations, or when you need to substitute a fake in tests.

## Practice Exercises

1. Define `IPlayable` with `Play()` and `Stop()`, implemented by `Song`, `Podcast`, and `AudioBook` (each printing something distinct). Write a `Playlist` class holding `List<IPlayable>` with `PlayAll()`.
2. Create an abstract `PaymentMethod` class (abstract `Pay(decimal amount)`, concrete shared `Receipt(decimal)` method) with `CreditCard`, `PayPal`, and `GiftCard` subclasses — the gift card must refuse payments above its balance. Drive them all through a `List<PaymentMethod>`.
3. Make your Chapter 8 `Book` class implement `IComparable<Book>` (order by title) and verify `Sort()` works on a list of books. Then change the ordering to author-then-title.
4. Revisit exercise 5 from Chapter 9 (Bird/Penguin/Plane) and solve it properly with an `IFlyer` interface plus an appropriate base class. Write a method that takes `List<IFlyer>` and launches everything.
5. Design a two-implementation storage contract: `ISaver` with `Save(string data)` — one `ConsoleSaver` and one `MemorySaver` (accumulates into a public list). Write a `ReportGenerator` that takes an `ISaver` in its constructor. In comments: why does this design make `ReportGenerator` testable without touching a real disk?
