# Chapter 13: Exceptions & Error Handling

## Overview

Things go wrong: files are missing, input is garbage, networks drop. C# signals runtime failures with **exceptions** — objects that abort normal flow and travel up the call stack until something **catches** them (or the program crashes). The mechanics mirror Python's `try/except/finally` and JS's `try/catch` closely; the C#-specific parts are typed catch clauses, exception filters, `finally`/`using` for cleanup, and a culture of *specific* exception types. Just as important is judgment: when to throw, when to catch, and when to avoid exceptions entirely with `TryParse`-style methods.

## Definitions & Explanations

### The core mechanics

```csharp
try
{
    int n = int.Parse("not a number");    // throws FormatException
    Console.WriteLine("never reached");
}
catch (FormatException ex)                 // catch by TYPE (like `except ValueError as ex`)
{
    Console.WriteLine($"Bad input: {ex.Message}");
}
finally
{
    Console.WriteLine("Always runs — success, failure, or early return.");
}
```

- Execution jumps from the throwing line straight to the first matching `catch`.
- An uncaught exception unwinds the entire stack and terminates the program with a stack trace.
- `ex.Message` (human text), `ex.StackTrace` (where), `ex.InnerException` (wrapped cause) are the members you'll read most.

### Multiple catch clauses — most specific first

```csharp
try
{
    string text = File.ReadAllText(path);
    int value = int.Parse(text);
}
catch (FileNotFoundException)             // specific
{
    Console.WriteLine("File missing.");
}
catch (FormatException)                   // specific
{
    Console.WriteLine("File didn't contain a number.");
}
catch (Exception ex)                      // general fallback — LAST
{
    Console.WriteLine($"Unexpected: {ex.Message}");
    throw;                                // rethrow when you can't actually handle it
}
```

Order matters: clauses are tested top-down, and a base type (`Exception`) before a derived one makes the derived clause unreachable (compile error CS0160, helpfully).

**Exception filters** add a condition without catching-and-rethrowing:

```csharp
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
{
    // handle only 404s; other HttpRequestExceptions fly past
}
```

### Throwing

```csharp
public void SetAge(int age)
{
    if (age is < 0 or > 130)
        throw new ArgumentOutOfRangeException(nameof(age), age, "Age must be 0-130.");
    _age = age;
}
```

Common types to throw (and to recognize):

| Exception | Meaning |
|---|---|
| `ArgumentException` / `ArgumentNullException` / `ArgumentOutOfRangeException` | caller passed a bad value |
| `InvalidOperationException` | object is in the wrong state for this call |
| `FormatException` | text couldn't be parsed |
| `KeyNotFoundException`, `IndexOutOfRangeException` | lookup misses |
| `NullReferenceException` | you used a null — a **bug to fix**, never to catch |
| `IOException`, `FileNotFoundException` | file system trouble |

Rethrowing: use bare `throw;` (preserves the original stack trace), **not** `throw ex;` (resets it — a classic interview question).

### Custom exceptions

```csharp
public class InsufficientFundsException : Exception
{
    public decimal Shortfall { get; }

    public InsufficientFundsException(decimal shortfall)
        : base($"Insufficient funds: short by {shortfall:C}")
        => Shortfall = shortfall;
}

// throw new InsufficientFundsException(25.50m);
```

Name ends in `Exception`, inherit from `Exception`, add fields callers can act on.

### Exceptions vs Try-pattern — choosing

Exceptions are for *exceptional* situations, not expected control flow (they're also slow when thrown). For "this might reasonably fail" cases, .NET offers Try-methods, and you should mimic that shape in your own APIs:

```csharp
// Expected-to-sometimes-fail: use the Try pattern
if (int.TryParse(input, out int n)) { /* use n */ }

// Programming errors and broken invariants: throw
if (capacity <= 0) throw new ArgumentOutOfRangeException(nameof(capacity));
```

Rules of thumb:
- Validating **user input**? Try-pattern (bad input is normal, not exceptional).
- A **caller violated your contract**? Throw `Argument...` immediately.
- Can't do anything useful about it here? **Don't catch it** — let it rise to a level that can (often just the top-level handler that logs and shows a friendly message).

## Code Examples

### A robust console workflow

```csharp
// SafeCalc — layered handling: validate cheaply, catch what's left
static void Main()
{
    while (true)
    {
        Console.Write("Divide: enter 'a b' (blank to quit): ");
        string? line = Console.ReadLine();
        if (string.IsNullOrWhiteSpace(line)) return;

        try
        {
            Console.WriteLine($"Result: {Divide(line)}");
        }
        catch (FormatException)
        {
            Console.WriteLine("  Please enter two numbers separated by a space.");
        }
        catch (DivideByZeroException)
        {
            Console.WriteLine("  Cannot divide by zero.");
        }
        // Anything else is a bug — let it crash loudly so we notice and fix it.
    }
}

static double Divide(string line)
{
    string[] parts = line.Split(' ', StringSplitOptions.RemoveEmptyEntries);
    if (parts.Length != 2)
        throw new FormatException("Expected exactly two numbers.");

    double a = double.Parse(parts[0]);   // FormatException on garbage — fine,
    double b = double.Parse(parts[1]);   // caller handles it

    if (b == 0)
        throw new DivideByZeroException();   // int division throws this itself;
                                             // double division would give Infinity,
    return a / b;                            // so we make the rule explicit
}
```

### finally and using for cleanup

```csharp
// Guaranteed cleanup even when things explode
StreamWriter? writer = null;
try
{
    writer = new StreamWriter("log.txt");
    writer.WriteLine("started");
    // ... work that might throw ...
}
finally
{
    writer?.Dispose();      // runs no matter what
}

// The idiomatic shorthand for exactly this pattern (details in Chapter 16):
using (var w = new StreamWriter("log.txt"))
{
    w.WriteLine("started");
}   // Dispose called automatically here, even on exceptions
```

### Custom exception in a domain model

```csharp
class BankAccount
{
    public decimal Balance { get; private set; }

    public void Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentOutOfRangeException(nameof(amount));
        if (amount > Balance)
            throw new InsufficientFundsException(amount - Balance);
        Balance -= amount;
    }
}

// Caller can react to the SPECIFIC failure with its extra data:
try { account.Withdraw(100m); }
catch (InsufficientFundsException ex)
{
    Console.WriteLine($"Denied — you need {ex.Shortfall:C} more.");
}
```

## Common Pitfalls

**1. Swallowing exceptions.** The cardinal sin:

```csharp
try { DoWork(); }
catch (Exception) { }        // ❌ failure vanishes; bugs become ghosts

catch (Exception ex)         // ✅ at minimum, make it visible
{
    Console.WriteLine($"ERROR: {ex}");
    throw;                   // and usually rethrow what you can't handle
}
```

**2. Catching `Exception` everywhere.** Catch the *specific* types you can genuinely handle; let the rest rise. Broad catches belong in exactly one place: the outermost layer (main loop / request handler) for logging.

**3. `throw ex;` instead of `throw;`.** Destroys the stack trace pointing at the real crash site. Bare `throw;` inside a catch preserves it.

**4. Using exceptions for expected flow.**

```csharp
try { n = int.Parse(input); }        // ❌ user typos are not exceptional
catch (FormatException) { n = 0; }

if (!int.TryParse(input, out n)) n = 0;   // ✅ cleaner AND faster
```

**5. Catching `NullReferenceException`.** It always means a bug upstream. Fix the null (nullable annotations, guards) — don't paper over it.

**6. Cleanup outside `finally`.** Code after the `try` block may never run if an exception escapes; only `finally` (or `using`) guarantees cleanup.

## Practice Exercises

1. Write a program that keeps asking for a number until parsing succeeds — first using try/catch on `int.Parse`, then rewritten with `TryParse`. In a comment, state which you'd ship and why.
2. Write `static int[] LoadScores(string path)` that reads a file of one-integer-per-line. Handle: file missing (return empty array with a warning), a non-numeric line (skip it, report the line number), any other exception (rethrow). Test all three paths.
3. Create a `Fraction` class whose constructor throws `ArgumentException` for a zero denominator, plus an `Add(Fraction)` method. Then add `static bool TryCreate(int num, int den, out Fraction? result)` — the non-throwing sibling. Demonstrate both.
4. Build a custom `InvalidMoveException` (with `FromSquare`/`ToSquare` properties) for a tiny board game: a 3x3 grid where a move to an occupied or out-of-range square throws. Catch it and print a friendly message using the properties.
5. Write a method chain `A() → B() → C()` where `C` throws. In `B`, catch, log, and rethrow with bare `throw;`. Run it once with `throw;` and once with `throw ex;`, compare the printed stack traces, and write one sentence about the difference.
