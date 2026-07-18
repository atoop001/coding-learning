# Chapter 17: Unit Testing with xUnit

## Overview

Professional C# code ships with tests, and interviewers ask about them. A **unit test** is a small program that calls one piece of your code with known inputs and asserts the output is what you expect — automatically, repeatably, in milliseconds. **xUnit** is the de facto standard test framework in the .NET world (it's what ASP.NET Core itself uses). If you've seen `pytest` or `jest`, the concepts map directly: test discovery by attribute instead of by name, `Assert.Equal` instead of `assert`/`expect`.

## Definitions & Explanations

### Vocabulary

- **Test project** — a separate project that references your real code and contains only tests.
- **`[Fact]`** — an attribute marking a method as a test with no parameters ("this is always true").
- **`[Theory]` + `[InlineData]`** — one parameterized test run with many input rows (pytest's `@parametrize`).
- **Assertion** — the pass/fail check: `Assert.Equal(expected, actual)` and friends.
- **Arrange / Act / Assert (AAA)** — the universal three-step test structure: set up, do the thing, check the result.
- **Test runner** — `dotnet test` builds everything, finds all attributed methods, runs them, and reports.

### Setting up (the standard two-project layout)

```powershell
mkdir MathKit; cd MathKit
dotnet new sln -n MathKit
dotnet new classlib -o src/MathKit
dotnet new xunit    -o tests/MathKit.Tests
dotnet sln add src/MathKit tests/MathKit.Tests
dotnet add tests/MathKit.Tests reference src/MathKit

dotnet test      # runs the sample test that the template includes
```

### Anatomy of a test class

Code under test (in `src/MathKit`):

```csharp
namespace MathKit;

public class Calculator
{
    public int Add(int a, int b) => a + b;

    public double Divide(double a, double b) =>
        b == 0 ? throw new DivideByZeroException() : a / b;

    public bool IsEven(int n) => n % 2 == 0;
}
```

Tests (in `tests/MathKit.Tests`):

```csharp
namespace MathKit.Tests;

using Xunit;
using MathKit;

public class CalculatorTests
{
    // Naming convention: Method_Scenario_ExpectedResult
    [Fact]
    public void Add_TwoPositives_ReturnsSum()
    {
        // Arrange
        var calc = new Calculator();

        // Act
        int result = calc.Add(2, 3);

        // Assert
        Assert.Equal(5, result);           // (expected, actual) — in that order!
    }

    [Fact]
    public void Divide_ByZero_Throws()
    {
        var calc = new Calculator();

        // Asserting an exception: pass the action as a lambda
        Assert.Throws<DivideByZeroException>(() => calc.Divide(10, 0));
    }

    // One test, five rows — each row is a separately reported test case
    [Theory]
    [InlineData(2, true)]
    [InlineData(3, false)]
    [InlineData(0, true)]
    [InlineData(-4, true)]
    [InlineData(-7, false)]
    public void IsEven_VariousInputs_Classifies(int input, bool expected)
    {
        var calc = new Calculator();
        Assert.Equal(expected, calc.IsEven(input));
    }
}
```

### The assertion toolbox

```csharp
Assert.Equal(expected, actual);          // values (works for collections too)
Assert.Equal(3.14, actual, precision: 2);// doubles: compare with tolerance!
Assert.NotEqual(a, b);
Assert.True(condition);  Assert.False(condition);
Assert.Null(obj);        Assert.NotNull(obj);
Assert.Contains("needle", haystackString);
Assert.Contains(item, collection);
Assert.Empty(collection); Assert.Single(collection);
Assert.Throws<ArgumentException>(() => Thing());
Assert.IsType<Dog>(animal);
Assert.Same(a, b);                       // same object reference (rare)
```

### Shared setup — the constructor

xUnit creates a **fresh instance of the test class for every test**, so setup goes in the constructor (no `[SetUp]` attribute needed) and tests can't contaminate each other:

```csharp
public class ScoreBoardTests
{
    private readonly ScoreBoard _board;

    public ScoreBoardTests()             // runs before EVERY test
    {
        _board = new ScoreBoard();
        _board.Add("Ada", 100);
    }

    [Fact]
    public void Add_NewScore_IncreasesCount() { /* uses _board */ }
}
```

### What makes a good unit test

- **Fast** (no disk, network, or `Console.ReadLine` — this is why Chapter 10's interface trick matters: substitute fakes for slow dependencies).
- **Independent** — passes alone and in any order.
- **One behavior per test** — a failing name should tell you what broke without reading code.
- **Tests behavior, not implementation** — assert on outputs/state, not "method X called method Y".

## Code Examples

### Testing a class with state and edge cases

```csharp
// src: the class under test (from Chapter 12, slightly trimmed)
public class BoundedStack<T>
{
    private readonly List<T> _items = new();
    public int Capacity { get; }
    public BoundedStack(int capacity) =>
        Capacity = capacity > 0 ? capacity
                 : throw new ArgumentOutOfRangeException(nameof(capacity));
    public int Count => _items.Count;
    public bool TryPush(T item)
    {
        if (Count == Capacity) return false;
        _items.Add(item); return true;
    }
    public bool TryPop(out T? item)
    {
        if (Count == 0) { item = default; return false; }
        item = _items[^1]; _items.RemoveAt(Count - 1); return true;
    }
}
```

```csharp
// tests: notice edge cases get their own tests
public class BoundedStackTests
{
    [Fact]
    public void Constructor_NonPositiveCapacity_Throws() =>
        Assert.Throws<ArgumentOutOfRangeException>(() => new BoundedStack<int>(0));

    [Fact]
    public void TryPush_WhenFull_ReturnsFalseAndKeepsCount()
    {
        var stack = new BoundedStack<string>(1);
        stack.TryPush("a");

        bool result = stack.TryPush("b");

        Assert.False(result);
        Assert.Equal(1, stack.Count);          // state unchanged — important assert!
    }

    [Fact]
    public void TryPop_IsLifo()
    {
        var stack = new BoundedStack<int>(5);
        stack.TryPush(1);
        stack.TryPush(2);

        stack.TryPop(out int? top);

        Assert.Equal(2, top);                  // last in, first out
    }

    [Fact]
    public void TryPop_Empty_ReturnsFalse()
    {
        var stack = new BoundedStack<int>(5);
        Assert.False(stack.TryPop(out _));     // _ discards the out value
    }
}
```

### Running and filtering

```powershell
dotnet test                                   # everything
dotnet test --filter "FullyQualifiedName~BoundedStack"   # one class
dotnet test --logger "console;verbosity=detailed"        # see each test name
```

A failing test prints exactly what differed:

```
Assert.Equal() Failure
Expected: 5
Actual:   4
   at CalculatorTests.Add_TwoPositives_ReturnsSum() ...
```

## Common Pitfalls

**1. Swapped expected/actual.** `Assert.Equal(actual, expected)` still fails when it should — but the failure *message* lies to you, sending you debugging in the wrong direction. Expected comes first.

**2. Comparing doubles exactly.**

```csharp
Assert.Equal(0.3, 0.1 + 0.2);                 // ❌ fails! (floating point)
Assert.Equal(0.3, 0.1 + 0.2, precision: 10);  // ✅ tolerance
```

**3. Tests that depend on each other or on order.** Each test builds its own world (xUnit's per-test instances help). If test B only passes after test A, both are broken.

**4. Testing private methods.** If you feel the need, the private logic probably wants to be a public method on a new small class. Test through the public surface.

**5. Forgetting the project reference.** "Type or namespace not found" in the test project usually means the `dotnet add reference` step was skipped — or the class under test is `internal` (Chapter 11!) instead of `public`.

**6. One giant test asserting everything.** When it fails, which of the 12 asserts broke, and what still works? Split by behavior; use `[Theory]` for input variations rather than copy-paste.

## Practice Exercises

1. Set up the two-project solution from this chapter. Write a `StringKit` class with `Reverse`, `IsPalindrome`, and `CountVowels`, then a test class with at least one `[Fact]` per method and one `[Theory]` covering five palindrome cases (include punctuation and empty string — decide and document the empty-string policy in the test name).
2. Write tests FIRST (red), then implement: a `TemperatureConverter` with `CToF` and `FToC`. Include a round-trip theory (`CToF(FToC(x)) ≈ x`) using precision-based equality.
3. Add xUnit tests for your Chapter 13 `Fraction` class: normal construction, the zero-denominator throw, `Add` correctness, and the `TryCreate` false path. Aim for every branch of your code to be exercised by some test.
4. Take the `OrderService`/`INotifier` design from Chapter 10 and test that `PlaceOrder` sends exactly one notification containing the item name — using the in-memory fake notifier, no console output. This is the fakes-via-interfaces pattern in real use.
5. Break things on purpose: introduce three different bugs into your `StringKit` (off-by-one, wrong comparison, inverted condition) one at a time, and confirm each is caught by at least one failing test. If any bug survives, add the missing test.
