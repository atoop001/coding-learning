# Chapter 3: Operators & Control Flow

## Overview

Programs make decisions. This chapter covers C#'s operators (arithmetic, comparison, logical) and its branching constructs: `if`/`else`, the `switch` statement, the modern `switch` expression, and the ternary operator. If you know Python or JavaScript, most of this will feel familiar — the differences are braces instead of indentation, mandatory parentheses around conditions, and a much stricter attitude about what counts as "true".

## Definitions & Explanations

### Arithmetic operators

```csharp
int sum = 7 + 3;    // 10
int diff = 7 - 3;   // 4
int prod = 7 * 3;   // 21
int quot = 7 / 3;   // 2   (integer division — truncates!)
int rem = 7 % 3;    // 1   (remainder / modulo)
```

C# has no `**` power operator (use `Math.Pow(2, 10)`), and no `//` floor division — `int / int` already truncates.

Increment/decrement (familiar from JS, absent in Python):

```csharp
int i = 5;
i++;        // i is now 6
i--;        // back to 5
i += 10;    // compound assignment: also -=, *=, /=, %=
```

### Comparison operators

`==`, `!=`, `<`, `>`, `<=`, `>=` — all return `bool`.

Key difference from JavaScript: **there is no `===`**. C# `==` never does type coercion, because the compiler already guarantees both sides are compatible types. `5 == "5"` is a *compile error*, not `true` or `false`.

Key difference from Python: no chained comparisons. `0 < x < 10` does not work; write `x > 0 && x < 10`.

### Logical operators

| C# | Python | Meaning |
|---|---|---|
| `&&` | `and` | both must be true (short-circuits) |
| `\|\|` | `or` | at least one true (short-circuits) |
| `!` | `not` | negation |

**Truthiness does not exist in C#.** A condition must be an actual `bool`. `if (myList)` or `if (count)` are compile errors — write `if (myList.Count > 0)` or `if (count != 0)`. This removes a whole category of subtle bugs, at the cost of a few extra characters.

### if / else if / else

```csharp
if (score >= 90)          // parentheses REQUIRED around the condition
{
    grade = "A";          // braces define the block — indentation is cosmetic
}
else if (score >= 80)
{
    grade = "B";
}
else
{
    grade = "C or below";
}
```

Braces are optional for single statements but strongly recommended — always use them.

### The ternary (conditional) operator

```csharp
string label = age >= 18 ? "adult" : "minor";
// Python equivalent: label = "adult" if age >= 18 else "minor"
```

### switch statement and switch expression

The classic statement (like JS `switch`, but *no fall-through* between non-empty cases):

```csharp
switch (dayNumber)
{
    case 6:
    case 7:                          // stacking empty cases is allowed
        Console.WriteLine("Weekend");
        break;                       // break is REQUIRED
    case 1:
        Console.WriteLine("Monday, ugh");
        break;
    default:
        Console.WriteLine("Midweek");
        break;
}
```

The modern **switch expression** (C# 8+) is often cleaner — it *returns a value*, like Python 3.10's `match` but as an expression:

```csharp
string mood = dayNumber switch
{
    6 or 7 => "Weekend!",
    1      => "Monday, ugh",
    _      => "Midweek",       // _ is the default (discard) pattern
};
```

Switch expressions also support **pattern matching** on ranges and types:

```csharp
string category = score switch
{
    >= 90           => "A",
    >= 80           => "B",
    >= 70           => "C",
    < 0             => "invalid",
    _               => "F",
};
```

## Code Examples

### A complete decision-making program

```csharp
// TicketPricer — realistic branching with input validation
Console.Write("Enter your age: ");

if (!int.TryParse(Console.ReadLine(), out int age) || age < 0 || age > 130)
{
    Console.WriteLine("Please enter a valid age.");
    return;                                  // exit early — a common clean pattern
}

Console.Write("Is it a weekday matinee? (y/n): ");
bool matinee = Console.ReadLine()?.Trim().ToLower() == "y";

decimal basePrice = age switch
{
    < 5   => 0m,           // under 5: free
    < 13  => 6.50m,        // child
    < 65  => 11.00m,       // adult
    _     => 8.00m,        // senior
};

// Matinee discount applies only to paying customers
decimal finalPrice = matinee && basePrice > 0
    ? basePrice * 0.8m
    : basePrice;

string note = finalPrice == 0 ? " (free!)" : matinee ? " (matinee -20%)" : "";
Console.WriteLine($"Ticket price: {finalPrice:C}{note}");
```

### Short-circuit evaluation in practice

```csharp
string? input = Console.ReadLine();

// Safe: if input is null, the right side never runs (short-circuit),
// so there's no NullReferenceException.
if (input != null && input.Length > 3)
{
    Console.WriteLine("Long enough.");
}

// The null-conditional operator ?. is an even shorter guard:
if (input?.Length > 3) { /* same effect; null?.Length is null, null > 3 is false */ }
```

## Common Pitfalls

**1. Missing parentheses around conditions.** Python habits will bite you.

```csharp
if x > 5 { }        // ❌ syntax error
if (x > 5) { }      // ✅
```

**2. Using `and`/`or`/`not`.** Those are Python. C# wants `&&`, `||`, `!`.

```csharp
if (a > 0 and b > 0) { }    // ❌
if (a > 0 && b > 0) { }     // ✅
```

**3. Relying on truthiness.**

```csharp
if (name) { }               // ❌ cannot convert string to bool
if (!string.IsNullOrEmpty(name)) { }   // ✅ say what you mean
```

**4. Forgetting `break` in a switch *statement*.** Non-empty cases must end with `break` (or `return`/`throw`). The compiler enforces this — unlike JS, silent fall-through can't happen, but the error message ("Control cannot fall out of switch") confuses newcomers. Add the `break`.

**5. Comparing strings with wrong case expectations.**

```csharp
if (answer == "Yes") { }                     // fails for "yes", "YES"
if (answer.Equals("yes", StringComparison.OrdinalIgnoreCase)) { }  // ✅
```

**6. Chained comparisons.**

```csharp
if (0 <= x <= 100) { }        // ❌ compile error
if (x >= 0 && x <= 100) { }   // ✅
```

## Practice Exercises

1. Write a program that reads an integer and prints whether it is negative, zero, or positive — and additionally whether it's even or odd (use `%`).
2. Build a simple BMI calculator: read weight (kg) and height (m), compute `weight / (height * height)`, and print the WHO category (underweight < 18.5, normal < 25, overweight < 30, otherwise obese) using a **switch expression** with relational patterns.
3. Write a "rock, paper, scissors" round: read two players' choices as strings and print who wins. Handle invalid input and ignore letter case.
4. Create a leap-year checker: a year is a leap year if divisible by 4, except centuries, unless divisible by 400. Express it as a single boolean expression assigned to a variable, then print it. Test with 1900 (false), 2000 (true), 2024 (true).
5. Rewrite this chain as a switch expression: an HTTP status code maps to "OK" (200), "Redirect" (301 or 302), "Client error" (400–499), "Server error" (500–599), "Unknown" otherwise.
