# Chapter 7: Arrays & Lists

## Overview

Arrays and `List<T>` are C#'s two everyday sequence types. An **array** is fixed-size and fast; a **`List<T>`** grows and shrinks like a Python list or JS array. Both hold elements of a *single declared type* — a `List<int>` can only contain ints, which is the biggest mental shift from Python/JS. In practice you'll reach for `List<T>` most of the time and arrays when the size is known and fixed.

## Definitions & Explanations

### Arrays — fixed size, one type

```csharp
int[] scores = new int[5];              // five ints, all initialized to 0
int[] primes = { 2, 3, 5, 7, 11 };      // initializer syntax
string[] names = new string[] { "Ada", "Grace" };

primes[0]          // 2 — zero-indexed
primes[^1]         // 11 — last element (index from end)
primes.Length      // 5
primes[1] = 30;    // elements are mutable...
primes[5] = 13;    // ❌ runtime IndexOutOfRangeException — size is FIXED
```

Key facts:
- The size is set at creation and can never change.
- Unset elements get a default: `0` for numbers, `false` for bool, `null` for reference types.
- Arrays are **reference types** — assigning copies the reference (Chapter 2).

### List<T> — the resizable workhorse

`List<T>` lives in `System.Collections.Generic`. The `<T>` is a **generic type parameter** — you fill in the element type (fully explained in Chapter 12; for now read `List<int>` as "list of ints").

```csharp
using System.Collections.Generic;   // usually implicit in modern templates

List<string> tasks = new List<string>();          // empty
var nums = new List<int> { 3, 1, 4 };             // collection initializer
List<int> nums2 = [3, 1, 4];                      // collection expression (C# 12)

tasks.Add("write chapter");        // append (Python append / JS push)
tasks.Insert(0, "wake up");        // insert at index
tasks.Remove("wake up");           // remove first matching value (true/false)
tasks.RemoveAt(0);                 // remove by index
tasks.Count                        // number of items (NOT Length!)
tasks.Contains("x")                // membership test (Python's `in`)
tasks.IndexOf("x")                 // index or -1
tasks.Clear();                     // empty it
tasks.Sort();                      // in-place sort
tasks.Reverse();                   // in-place reverse
tasks.ToArray();                   // snapshot as an array
```

### Comparison table

| Feature | Array | `List<T>` | Python list | JS array |
|---|---|---|---|---|
| Size | Fixed | Dynamic | Dynamic | Dynamic |
| Element types | One type | One type | Anything | Anything |
| Size property | `.Length` | `.Count` | `len()` | `.length` |
| Add | — | `.Add(x)` | `.append(x)` | `.push(x)` |

### 2D data

```csharp
int[,] grid = new int[3, 4];            // rectangular 3x4 grid
grid[2, 3] = 9;
int rows = grid.GetLength(0);           // 3

int[][] jagged = new int[3][];          // array of arrays (rows can differ in length)
jagged[0] = new[] { 1, 2, 3 };
```

## Code Examples

### Grade book with a List

```csharp
// GradeBook — collect scores, then report
var scores = new List<double>();

while (true)
{
    Console.Write($"Score #{scores.Count + 1} (blank to finish): ");
    string? line = Console.ReadLine();

    if (string.IsNullOrWhiteSpace(line)) break;

    if (double.TryParse(line, out double s) && s is >= 0 and <= 100)
        scores.Add(s);
    else
        Console.WriteLine("  Enter a number 0-100.");
}

if (scores.Count == 0)
{
    Console.WriteLine("No scores entered.");
    return;
}

double sum = 0;
double min = scores[0], max = scores[0];
foreach (double s in scores)
{
    sum += s;
    if (s < min) min = s;
    if (s > max) max = s;
}

Console.WriteLine($"Count: {scores.Count}  Avg: {sum / scores.Count:F1}  Min: {min}  Max: {max}");

scores.Sort();
scores.Reverse();
Console.WriteLine("Sorted high→low: " + string.Join(", ", scores));
```

### A tiny to-do list (List CRUD in a menu loop)

```csharp
var todos = new List<string>();

while (true)
{
    Console.Write("\n[a]dd  [r]emove  [l]ist  [q]uit > ");
    switch (Console.ReadLine()?.Trim().ToLower())
    {
        case "a":
            Console.Write("Task: ");
            string? task = Console.ReadLine();
            if (!string.IsNullOrWhiteSpace(task)) todos.Add(task.Trim());
            break;

        case "r":
            Console.Write("Number to remove: ");
            if (int.TryParse(Console.ReadLine(), out int i)
                && i >= 1 && i <= todos.Count)
                todos.RemoveAt(i - 1);          // user sees 1-based numbers
            else
                Console.WriteLine("No such task.");
            break;

        case "l":
            if (todos.Count == 0) Console.WriteLine("(nothing to do!)");
            for (int t = 0; t < todos.Count; t++)
                Console.WriteLine($"{t + 1}. {todos[t]}");
            break;

        case "q":
            return;
    }
}
```

### Copying vs aliasing (recap in collection form)

```csharp
var a = new List<int> { 1, 2, 3 };
var alias = a;                    // SAME list, two names
var copy = new List<int>(a);      // NEW list with copied elements

alias.Add(4);
Console.WriteLine(a.Count);       // 4 — alias changed "both"
Console.WriteLine(copy.Count);    // 3 — copy is independent
```

## Common Pitfalls

**1. `Length` vs `Count` confusion.** Arrays and strings have `.Length`; `List<T>` has `.Count`. Muscle memory comes fast; until then, IntelliSense.

**2. Index out of range.**

```csharp
var xs = new List<int> { 10, 20, 30 };
Console.WriteLine(xs[3]);               // ❌ ArgumentOutOfRangeException
Console.WriteLine(xs[xs.Count - 1]);    // ✅ 30
Console.WriteLine(xs[^1]);              // ✅ 30 — nicer
```

**3. Removing items inside `foreach`.** (Chapter 4's rule, repeated because everyone hits it.)

```csharp
foreach (var t in todos)
    if (t.StartsWith("old")) todos.Remove(t);     // ❌ InvalidOperationException

todos.RemoveAll(t => t.StartsWith("old"));        // ✅
```

**4. `Remove` vs `RemoveAt`.** `Remove(2)` on a `List<int>` removes *the value 2*, not index 2. Use `RemoveAt(2)` for the index.

**5. Expecting mixed types.**

```csharp
var stuff = new List<int> { 1, "two" };   // ❌ CS0029 — one type per list
```

If you genuinely need mixed data, that's usually a sign you want a class (Chapter 8) with named fields instead.

**6. Uninitialized array of reference types is full of nulls.**

```csharp
string[] names = new string[3];
Console.WriteLine(names[0].Length);   // ❌ NullReferenceException — it's null, not ""
```

## Practice Exercises

1. Create an array of the 12 month names. Ask the user for a number 1–12 and print the corresponding month, handling bad input. Then print all months that contain the letter "r".
2. Read integers into a `List<int>` until the user types "done". Print the list, then print it with duplicates removed (build a second list, adding only unseen values).
3. Write `static List<int> EvensOnly(List<int> input)` that returns a *new* list of just the even numbers, leaving the input untouched. Prove the input is unchanged.
4. Simulate 1,000 rolls of two dice using `Random` (`new Random().Next(1, 7)`), tallying totals 2–12 in an array of counters. Print a text histogram (one `#` per 10 rolls).
5. Extend the to-do example with a "move up" command that swaps a task with the one above it, and a "clear completed" that removes all tasks starting with `[x]`. Which List methods did you need?
