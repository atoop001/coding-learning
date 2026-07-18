# Chapter 2: Variables & Types

## Overview

In C#, every variable has a **type** that is fixed at compile time. This chapter covers the built-in types, how to declare and convert between them, the crucial distinction between **value types** and **reference types**, and how C# handles "no value" with `null`. Understanding value vs reference semantics early will save you from an entire family of confusing bugs later — it affects how assignment, comparison, and method calls behave.

## Definitions & Explanations

### Declaring variables

```csharp
int score = 42;          // explicit type
var total = 3.14;        // 'var' = compiler INFERS the type (double here)
```

`var` is *not* like JavaScript's `var` or Python's dynamic variables. The type is still fixed — the compiler just figures it out from the right-hand side. `var total = 3.14;` makes `total` a `double` forever; assigning a string to it later is a compile error.

### The core built-in types

| C# type | What it holds | Example | Python/JS analogue |
|---|---|---|---|
| `int` | 32-bit whole number (±2.1 billion) | `int n = 7;` | `int` / `number` |
| `long` | 64-bit whole number | `long big = 9_000_000_000;` | `int` |
| `double` | 64-bit floating point (default for decimals) | `double pi = 3.14;` | `float` / `number` |
| `decimal` | 128-bit precise decimal — use for money | `decimal price = 9.99m;` | `Decimal` |
| `bool` | `true` / `false` (lowercase!) | `bool ok = true;` | `True`/`true` |
| `char` | a single character, single quotes | `char c = 'A';` | 1-char string |
| `string` | text, double quotes | `string s = "hi";` | `str` / `string` |
| `object` | can hold anything (avoid overusing) | `object o = 5;` | any value |

Note the literal suffixes: `9.99m` (decimal), `3.5f` (float), `10L` (long). Underscores in number literals (`1_000_000`) are just for readability.

### Value types vs reference types

This is the most important concept in this chapter.

- **Value types** (`int`, `double`, `bool`, `char`, `decimal`, `struct`s, enums): the variable *contains the data itself*. Assignment **copies** the value. Two copies are independent.
- **Reference types** (`string`, arrays, `List<T>`, all classes): the variable contains a *reference* (think: an arrow pointing at an object elsewhere in memory). Assignment copies the **arrow**, not the object — two variables can point at the *same* object.

Python folks: this is like the mutable-object aliasing you already know (`list_b = list_a` aliases), except C# makes the rule explicit per type. JS folks: primitives vs objects works almost identically.

```csharp
// Value type: copies are independent
int a = 5;
int b = a;      // b gets a COPY of 5
b = 99;
Console.WriteLine(a);   // 5 — unchanged

// Reference type: both variables point at the same object
int[] xs = { 1, 2, 3 };
int[] ys = xs;          // ys points at the SAME array
ys[0] = 99;
Console.WriteLine(xs[0]); // 99 — changed through the other name!
```

(`string` is a reference type but *immutable*, so it behaves like a value in practice — more in Chapter 6.)

### null and nullable types

`null` means "no object" — like Python's `None` or JS's `null`.

- Reference types can be `null`.
- Value types normally **cannot** — an `int` always has a number. To allow "no value", use a nullable type: `int?`.

```csharp
int? maybeAge = null;        // ? makes a value type nullable
string? name = null;         // ? on reference types = "null is expected here"

if (maybeAge.HasValue)
    Console.WriteLine(maybeAge.Value);

int age = maybeAge ?? 0;     // ?? = null-coalescing: use 0 if null (like JS ??)
```

Modern C# projects enable **nullable reference types**: the compiler warns when you might dereference a possible `null`. Treat those warnings as errors — they prevent the #1 runtime crash (`NullReferenceException`).

### Constants

```csharp
const double TaxRate = 0.07;   // must be known at compile time; can never change
```

### Conversions

```csharp
int i = 100;
double d = i;            // implicit: no data loss possible, allowed silently
int j = (int)3.99;       // explicit CAST required — truncates to 3 (no rounding!)

string s = "123";
int parsed = int.Parse(s);            // throws if s isn't a number
bool ok = int.TryParse(s, out int n); // safe version: returns false instead of throwing

string text = 42.ToString();          // number -> string
```

## Code Examples

### A small, complete program using several types

```csharp
// TypeTour — demonstrates declarations, inference, conversion, nullability
const decimal PricePerItem = 4.25m;    // money => decimal, never double

Console.Write("How many items? ");
string? input = Console.ReadLine();

// TryParse: convert safely without crashing on bad input
if (int.TryParse(input, out int quantity) && quantity > 0)
{
    decimal subtotal = quantity * PricePerItem;
    double approxWeightKg = quantity * 0.35;   // doubles fine for measurements
    bool bulkOrder = quantity >= 10;

    Console.WriteLine($"Subtotal: {subtotal:C}");        // :C = currency format
    Console.WriteLine($"Weight:   {approxWeightKg:F2} kg"); // :F2 = 2 decimals
    Console.WriteLine($"Bulk?     {bulkOrder}");
}
else
{
    Console.WriteLine("That wasn't a positive whole number.");
}
```

### Demonstrating value vs reference semantics

```csharp
// A tiny class (reference type) — details in Chapter 8
class Counter
{
    public int Value;
}

class Program
{
    static void Main()
    {
        // struct-like behavior with plain int (value type)
        int x = 1;
        Bump(x);                       // x is COPIED into the method
        Console.WriteLine(x);          // still 1

        // reference behavior with a class instance
        Counter c = new Counter();
        c.Value = 1;
        Bump(c);                       // the REFERENCE is copied — same object
        Console.WriteLine(c.Value);    // 2 — the method mutated the shared object
    }

    static void Bump(int n) => n++;            // affects only the local copy
    static void Bump(Counter c) => c.Value++;  // affects the shared object
}
```

## Common Pitfalls

**1. Integer division silently truncates.**

```csharp
double half = 1 / 2;          // ❌ 0.0 — int / int is int division first
double half2 = 1.0 / 2;       // ✅ 0.5 — make one operand a double
double half3 = (double)1 / 2; // ✅ 0.5
```

**2. Using `double` for money.** Binary floating point can't represent 0.1 exactly.

```csharp
double bad = 0.1 + 0.2;        // 0.30000000000000004
decimal good = 0.1m + 0.2m;    // 0.3 exactly — always use decimal for currency
```

**3. Casting doesn't round — it truncates.**

```csharp
int n = (int)9.99;             // 9, not 10
int r = (int)Math.Round(9.99); // ✅ 10
```

**4. Assuming `var` means dynamic typing.**

```csharp
var x = 5;
x = "hello";   // ❌ CS0029 — x is an int forever
```

**5. Forgetting `True`/`False` are lowercase in C#.** Coming from Python: it's `true` and `false`.

**6. Comparing with `=` instead of `==`.** `if (x = 5)` is a compile error in C# (thankfully), but train the habit: `=` assigns, `==` compares.

## Practice Exercises

1. Declare one variable of each type in the core table above with a sensible example value, then print each with `Console.WriteLine`. Try changing one to the wrong kind of value and read the compiler error carefully.
2. Write a program that asks for a temperature in Fahrenheit and prints it in Celsius with exactly one decimal place (`:F1`). Make sure `70` converts to `21.1`, not `21`.
3. Predict, then verify: what do these print? `Console.WriteLine(7 / 2);` `Console.WriteLine(7 % 2);` `Console.WriteLine(7.0 / 2);` `Console.WriteLine((int)(7.0 / 2));`
4. Write a program that reads a string with `Console.ReadLine()` and uses `int.TryParse` to report either "you typed the number N" or "not a number" — without ever crashing.
5. Create two `int` variables and two arrays, assign one to the other in each pair, mutate the second of each pair, and print both. Explain in a comment why the results differ.
