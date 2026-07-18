# Chapter 14: Delegates, Events & Lambdas

## Overview

In Python and JavaScript you pass functions around casually — as arguments, callbacks, event handlers. C# does all of this too, but with types: a **delegate** is a type that describes a method signature, a **lambda** is a compact inline function, and an **event** is a controlled broadcast mechanism built on delegates. This trio powers GUI programming, game engines, and — crucially for the next chapter — LINQ, whose every method takes a function as an argument.

## Definitions & Explanations

### Delegates — typed references to methods

```csharp
// Declare a delegate TYPE: "anything matching this signature"
delegate int MathOp(int a, int b);

static int Add(int a, int b) => a + b;
static int Mul(int a, int b) => a * b;

MathOp op = Add;                 // a variable holding a METHOD
Console.WriteLine(op(3, 4));     // 7 — call through the variable
op = Mul;
Console.WriteLine(op(3, 4));     // 12
```

JS analogue: `let op = add;` — except the C# variable's *type* guarantees the signature.

### Func, Action, Predicate — the built-in delegates you'll actually use

In practice you rarely declare delegate types; .NET ships generic ones:

| Type | Shape | Example |
|---|---|---|
| `Func<int, int, int>` | takes (int, int), returns int | `Func<int,int,int> add` |
| `Func<string, bool>` | takes string, returns bool | a test/filter |
| `Action<string>` | takes string, returns **nothing** | a side-effecting callback |
| `Action` | no params, no return | `() => Console.WriteLine("hi")` |
| `Predicate<T>` | takes T, returns bool | used by `List.Find` etc. |

The **last** type parameter of `Func<>` is the return type; everything before is parameters.

### Lambdas — inline anonymous functions

```csharp
Func<int, int> square = x => x * x;                 // like JS: x => x * x
Func<int, int, int> add = (a, b) => a + b;          // Python: lambda a, b: a + b
Action<string> shout = s => Console.WriteLine(s.ToUpper());

Func<int, int> weird = x =>                          // multi-statement body
{
    int y = x * 2;
    return y + 1;
};
```

Lambdas **capture** variables from their surrounding scope (closures), exactly like Python/JS:

```csharp
int factor = 10;
Func<int, int> scale = x => x * factor;   // captures 'factor'
factor = 100;
Console.WriteLine(scale(5));              // 500 — sees the CURRENT value
```

### Higher-order methods you've been near all along

```csharp
var nums = new List<int> { 5, 3, 8, 1, 9, 2 };

List<int> big = nums.FindAll(n => n > 4);          // filter: {5, 8, 9}
int firstEven = nums.Find(n => n % 2 == 0);        // 8
bool anyHuge = nums.Exists(n => n > 100);          // false
nums.RemoveAll(n => n < 3);                        // removes 1 and 2
nums.Sort((a, b) => b.CompareTo(a));               // custom order: descending
nums.ForEach(n => Console.WriteLine(n));
```

This is the exact idiom LINQ generalizes in Chapter 15.

### Multicast: one delegate, many methods

Delegates combine with `+=` — invoking the delegate calls *all* attached methods in order:

```csharp
Action notify = () => Console.WriteLine("first");
notify += () => Console.WriteLine("second");
notify();          // prints both
notify -= /* a stored reference */ null!;   // -= detaches (must be the same instance)
```

This is the machinery under events.

### Events — publish/subscribe done safely

An **event** is a multicast delegate that outsiders can only subscribe to (`+=`/`-=`) — they cannot invoke it or wipe other subscribers. The owning class alone can raise it:

```csharp
class Downloader
{
    // Standard shape: EventHandler<TPayload>
    public event EventHandler<int>? ProgressChanged;

    public void Download()
    {
        for (int pct = 0; pct <= 100; pct += 25)
        {
            // Raise the event; ?.Invoke handles "nobody is listening"
            ProgressChanged?.Invoke(this, pct);
        }
    }
}

var d = new Downloader();
d.ProgressChanged += (sender, pct) => Console.WriteLine($"Progress: {pct}%");
d.ProgressChanged += (sender, pct) => { if (pct == 100) Console.WriteLine("Done!"); };
d.Download();
```

Conventions: signature `(object? sender, TArgs args)`; the event is `null` when no one subscribed — hence `?.Invoke`; richer payloads use a class deriving from `EventArgs`.

## Code Examples

### An event-driven temperature monitor

```csharp
// Sensors raise events; displays/alarms subscribe. Publisher knows NOTHING
// about its subscribers — the decoupling is the whole point.
class TemperatureSensor
{
    private double _current;
    public event EventHandler<double>? ReadingTaken;
    public event EventHandler<double>? ThresholdExceeded;

    public double Threshold { get; set; } = 30.0;

    public void RecordReading(double celsius)
    {
        _current = celsius;
        ReadingTaken?.Invoke(this, celsius);
        if (celsius > Threshold)
            ThresholdExceeded?.Invoke(this, celsius);
    }
}

class Program
{
    static void Main()
    {
        var sensor = new TemperatureSensor { Threshold = 28 };

        // Subscriber 1: logger (lambda)
        sensor.ReadingTaken += (s, t) => Console.WriteLine($"  log: {t:F1}°C");

        // Subscriber 2: alarm (method group — a named method as handler)
        sensor.ThresholdExceeded += SoundAlarm;

        foreach (double t in new[] { 24.1, 27.9, 31.2, 26.0, 33.8 })
            sensor.RecordReading(t);

        // Unsubscribe works only with the same reference — easy with methods:
        sensor.ThresholdExceeded -= SoundAlarm;
        sensor.RecordReading(40);          // no alarm this time
    }

    static void SoundAlarm(object? sender, double temp) =>
        Console.WriteLine($"  ALARM! {temp:F1}°C exceeds threshold!");
}
```

### Strategy via Func — swapping behavior without subclassing

```csharp
// One method, pluggable policies — compare to writing three subclasses
static decimal ApplyDiscount(decimal price, Func<decimal, decimal> policy) =>
    policy(price);

Func<decimal, decimal> none = p => p;
Func<decimal, decimal> tenPercent = p => p * 0.9m;
Func<decimal, decimal> capAtFifty = p => Math.Min(p, 50m);

Console.WriteLine(ApplyDiscount(80m, none));        // 80.0
Console.WriteLine(ApplyDiscount(80m, tenPercent));  // 72.0
Console.WriteLine(ApplyDiscount(80m, capAtFifty));  // 50.0
```

## Common Pitfalls

**1. Invoking a null event.**

```csharp
ProgressChanged.Invoke(this, 50);     // ❌ NullReferenceException if no subscribers
ProgressChanged?.Invoke(this, 50);    // ✅ the standard idiom
```

**2. Unsubscribing a lambda that was never stored.**

```csharp
sensor.ReadingTaken += (s, t) => Log(t);
sensor.ReadingTaken -= (s, t) => Log(t);   // ❌ different lambda instance — no-op!

EventHandler<double> handler = (s, t) => Log(t);
sensor.ReadingTaken += handler;
sensor.ReadingTaken -= handler;            // ✅ same reference
```

**3. Calling instead of referencing.**

```csharp
Func<int,int,int> op = Add(1, 2);   // ❌ that's the RESULT (an int) — type error
Func<int,int,int> op = Add;         // ✅ the method itself
```

**4. Forgotten subscriptions leaking memory.** A long-lived publisher holds references to every subscriber; short-lived objects that subscribe and never unsubscribe can't be garbage-collected. In long-running apps, pair every `+=` with a `-=` in cleanup.

**5. Captured loop variable surprises.**

```csharp
var actions = new List<Action>();
for (int i = 0; i < 3; i++)
    actions.Add(() => Console.Write(i + " "));  // captures the VARIABLE i
foreach (var a in actions) a();                 // "3 3 3" in for-loops!

for (int i = 0; i < 3; i++)
{
    int copy = i;                               // ✅ capture a per-iteration copy
    actions.Add(() => Console.Write(copy + " "));
}                                               // (foreach loops don't have this trap)
```

## Practice Exercises

1. Write a `Calculate(double a, double b, Func<double,double,double> op)` method and drive a calculator menu (+, −, ×, ÷) where each menu choice selects a lambda. No `if`/`switch` inside `Calculate` itself.
2. Using only `List<T>` methods with lambdas (`FindAll`, `RemoveAll`, `Sort`, `Exists`), process a list of product names: keep those under 10 characters, drop any containing a digit, sort by length, and report whether any start with "z".
3. Build a `CountdownTimer` class with a `Tick` event (raised each simulated second with seconds remaining) and a `Finished` event. Attach two different subscribers to `Tick` and demonstrate unsubscribing one midway.
4. Create a tiny publish/subscribe stock ticker: a `Stock` class with a `Price` property whose setter raises `PriceChanged` only when the price actually changes, carrying old and new price in a custom `EventArgs` subclass. Subscribe a display and a "drop over 5%" alert.
5. Reproduce the captured-loop-variable bug from the pitfalls, verify the "3 3 3" behavior, fix it, and then write a `MakeCounter()` method that returns a `Func<int>` which increments and returns a private captured counter — proving closures maintain state between calls.
