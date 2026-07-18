# Chapter 5: Methods

## Overview

Methods are C#'s functions. Coming from Python (`def`) or JavaScript (`function`), the ideas transfer directly — the differences are that every method declares its **return type** and the **type of every parameter**, methods live inside classes, and C# supports **overloading** (several methods with the same name but different parameter lists). This chapter covers defining and calling methods, parameters (including optional, named, `out`, and `ref`), return values, and expression-bodied shorthand.

## Definitions & Explanations

### Anatomy of a method

```csharp
//  access   static  return-type  Name        (parameter list)
    static  int         Add        (int a, int b)
    {
        return a + b;      // must return an int on every path
    }
```

- **Return type** — what the method gives back. `void` means "returns nothing" (like a Python function with no `return`).
- **`static`** — belongs to the class itself rather than an instance. Until we cover objects (Chapter 8), all our methods are `static`.
- **Parameters** are typed. Calling `Add("a", "b")` is a compile error, not a runtime surprise.

```csharp
int result = Add(3, 4);       // calling — same as Python/JS
Console.WriteLine(result);    // 7
```

### Expression-bodied methods

One-liners can use `=>` (like a concise arrow function):

```csharp
static int Add(int a, int b) => a + b;
static void Greet(string name) => Console.WriteLine($"Hi, {name}!");
```

### Optional parameters and named arguments

```csharp
static void Log(string message, string level = "INFO", bool timestamp = true)
{
    string prefix = timestamp ? $"[{DateTime.Now:HH:mm:ss}] " : "";
    Console.WriteLine($"{prefix}{level}: {message}");
}

Log("Server started");                        // uses both defaults
Log("Disk low", "WARN");                      // positional
Log("Crashed", level: "ERROR");               // named argument
Log("Quiet note", timestamp: false);          // skip the middle parameter by name
```

Just like Python's default/keyword arguments. Defaults must be compile-time constants and must come after required parameters.

### Method overloading — new for Python/JS folks

Multiple methods may share a name if their **parameter lists differ** (count or types). The compiler picks the right one from the arguments:

```csharp
static double Area(double radius) => Math.PI * radius * radius;   // circle
static double Area(double w, double h) => w * h;                  // rectangle
static int    Area(int side) => side * side;                      // square

Area(2.0);      // circle overload
Area(3.0, 4.0); // rectangle overload
Area(5);        // square overload
```

You cannot overload on return type alone. In Python you'd simulate this with `*args` or type checks; C# resolves it at compile time.

### out and ref parameters

Normally arguments are passed **by value** (the method gets a copy — for reference types, a copy of the reference; see Chapter 2). Two keywords change this:

- **`out`** — the method *must* assign the parameter; used to return extra values. You've already met it: `int.TryParse(text, out int n)`.
- **`ref`** — the method receives the caller's variable itself and may read and modify it.

```csharp
static bool TryDivide(int a, int b, out int quotient)
{
    if (b == 0) { quotient = 0; return false; }
    quotient = a / b;
    return true;
}

if (TryDivide(10, 3, out int q))
    Console.WriteLine(q);      // 3
```

Modern C# often prefers returning a **tuple** for multiple results:

```csharp
static (int Min, int Max) MinMax(int[] xs)
{
    int min = xs[0], max = xs[0];
    foreach (int x in xs)
    {
        if (x < min) min = x;
        if (x > max) max = x;
    }
    return (min, max);
}

var (lo, hi) = MinMax(new[] { 4, 1, 9 });   // destructuring, like Python
Console.WriteLine($"{lo}..{hi}");           // 1..9
```

### params — variable-length arguments

```csharp
static int Sum(params int[] numbers)     // like Python's *args
{
    int total = 0;
    foreach (int n in numbers) total += n;
    return total;
}

Sum();            // 0
Sum(1, 2, 3);     // 6 — pass as many as you like
```

## Code Examples

### A small program organized into methods

```csharp
// TemperatureReport — shows decomposition into focused methods
class Program
{
    static void Main()
    {
        double[] readingsC = { 21.5, 23.0, 19.8, 25.2, 22.1 };

        PrintHeader("Weekly Temperatures");

        foreach (double c in readingsC)
            Console.WriteLine($"{c,6:F1} °C = {ToFahrenheit(c),6:F1} °F");

        var (min, max) = MinMax(readingsC);
        Console.WriteLine($"Range: {min:F1}–{max:F1} °C");
        Console.WriteLine($"Average: {Average(readingsC):F1} °C");
    }

    // Each method does ONE thing and is independently testable
    static double ToFahrenheit(double celsius) => celsius * 9 / 5 + 32;

    static double Average(double[] values)
    {
        double sum = 0;
        foreach (double v in values) sum += v;
        return values.Length == 0 ? 0 : sum / values.Length;
    }

    static (double Min, double Max) MinMax(double[] values)
    {
        double min = values[0], max = values[0];
        foreach (double v in values)
        {
            if (v < min) min = v;
            if (v > max) max = v;
        }
        return (min, max);
    }

    static void PrintHeader(string title)
    {
        Console.WriteLine(title);
        Console.WriteLine(new string('=', title.Length));   // repeat '=' chars
    }
}
```

### Recursion works as expected

```csharp
static long Fibonacci(int n) =>
    n <= 1 ? n : Fibonacci(n - 1) + Fibonacci(n - 2);
```

## Common Pitfalls

**1. Missing return on some path.** The compiler checks *every* branch.

```csharp
static string Sign(int n)
{
    if (n > 0) return "positive";
    if (n < 0) return "negative";
}                                   // ❌ CS0161: not all code paths return a value

static string Sign(int n)
{
    if (n > 0) return "positive";
    if (n < 0) return "negative";
    return "zero";                  // ✅
}
```

**2. Ignoring the return value.** `text.ToUpper()` returns a *new* string; it doesn't modify anything in place (Chapter 6). Same trap as Python's `sorted(xs)` vs `xs.sort()`.

**3. Calling a non-static method from `Main` without an object.** Until Chapter 8, mark your helper methods `static`, or you'll hit "CS0120: An object reference is required".

**4. Expecting mutation of value-type arguments to stick.**

```csharp
static void Double(int n) { n *= 2; }     // modifies only the local copy
int x = 5;
Double(x);
Console.WriteLine(x);     // still 5 — return the new value instead:
static int Doubled(int n) => n * 2;       // ✅ x = Doubled(x);
```

**5. Overload ambiguity with optional parameters.** If `Print(string s)` and `Print(string s, int n = 0)` both exist, `Print("hi")` is ambiguous-ish design even where the compiler picks one. Keep overload sets simple.

## Practice Exercises

1. Write `static bool IsPrime(int n)` and use it in `Main` to print all primes below 100. Then add an overload `IsPrime(long n)`.
2. Write `static string Repeat(string text, int times, string separator = ", ")` that returns the text repeated with separators (`Repeat("ha", 3)` → `"ha, ha, ha"`). Call it with positional, default, and named arguments.
3. Write `static (double Avg, int Count) Stats(params double[] values)` and call it with 0, 1, and 5 arguments. Handle the empty case without crashing.
4. Write `static bool TryGetInitials(string fullName, out string initials)` returning false for empty/whitespace input, and initials like `"A.L."` for `"Ada Lovelace"` otherwise.
5. Refactor your Chapter 4 guessing game so `Main` is at most 10 lines, with all logic moved into well-named methods. Note which methods you found easiest to extract.
