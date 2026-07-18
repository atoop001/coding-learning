# Chapter 15: LINQ

## Overview

LINQ (**L**anguage **IN**tegrated **Q**uery) is C#'s built-in toolkit for querying and transforming data — filtering, mapping, sorting, grouping, aggregating — over any collection. If you like Python list comprehensions or JavaScript's `map`/`filter`/`reduce`, LINQ is that idea grown into a complete, composable system. It is arguably the single most-used C# feature in professional code: almost every method that touches a list of anything runs through LINQ.

Everything in this chapter builds on Chapter 14: LINQ methods take lambdas.

## Definitions & Explanations

### The shape of LINQ

LINQ methods are **extension methods** on `IEnumerable<T>` — they appear on every array, `List<T>`, `Dictionary`, string, etc. (via `using System.Linq;`, included in implicit usings). They chain:

```csharp
var result = people
    .Where(p => p.Age >= 18)          // filter   (Python: if clause / JS: filter)
    .OrderBy(p => p.LastName)         // sort
    .Select(p => p.FirstName)         // transform (Python: expression / JS: map)
    .ToList();                        // materialize into a List<string>
```

### The essential operators

```csharp
int[] nums = { 5, 3, 8, 1, 9, 2, 8 };

// Filtering & transforming
nums.Where(n => n > 4)               // 5, 8, 9, 8
nums.Select(n => n * n)              // 25, 9, 64, 1, 81, 4, 64
nums.Distinct()                      // 5, 3, 8, 1, 9, 2

// Ordering
nums.OrderBy(n => n)                 // ascending
nums.OrderByDescending(n => n)
people.OrderBy(p => p.Last).ThenBy(p => p.First)   // tie-breaker

// Slicing
nums.Take(3)                         // first 3
nums.Skip(3)                         // everything after the first 3
nums.TakeWhile(n => n > 2)           // 5, 3 (stops at first failure)

// Single-element retrieval
nums.First()                         // 5 — throws if empty
nums.First(n => n > 6)               // 8 — first match, throws if none
nums.FirstOrDefault(n => n > 100)    // 0 — default instead of throwing
nums.Single(n => n == 9)             // exactly one match or throw
nums.Last(); nums.ElementAt(2);

// Aggregation (returns a value, ends the chain)
nums.Count()                         // 7
nums.Count(n => n % 2 == 0)          // 2
nums.Sum(); nums.Min(); nums.Max();
nums.Average()                       // double
nums.Any(n => n > 8)                 // true — "does at least one..."
nums.All(n => n > 0)                 // true — "do all..."
nums.Contains(3)                     // true

// Materialization
.ToList()  .ToArray()  .ToDictionary(x => x.Key, x => x.Value)  .ToHashSet()
```

### Deferred execution — the big conceptual point

Most LINQ operators don't *do* anything when called — they build a query that runs only when enumerated (by `foreach`, `ToList`, `Count`, ...). Like Python generators:

```csharp
var query = nums.Where(n =>
{
    Console.Write($"[test {n}] ");
    return n > 4;
});
// Nothing printed yet!

var list = query.ToList();   // NOW the tests run
var list2 = query.ToList();  // ...and they run AGAIN (re-enumeration!)
```

Consequences: (1) queries see the *current* data each time they're enumerated; (2) enumerating twice does the work twice — call `.ToList()` once if you'll reuse results.

### GroupBy — the power tool

```csharp
var byLength = words.GroupBy(w => w.Length);
// a sequence of groups; each group has .Key and is itself enumerable

foreach (var group in byLength)
    Console.WriteLine($"{group.Key} letters: {string.Join(", ", group)}");
```

This replaces the manual `Dictionary<K, List<V>>` pattern from Chapter 12 in one call.

### Query syntax (the SQL-ish alternative)

```csharp
var adults = from p in people
             where p.Age >= 18
             orderby p.LastName
             select p.FirstName;
```

Identical meaning to method syntax. Most codebases prefer method syntax; recognize both.

## Code Examples

### A realistic data-crunching session

```csharp
record Order(int Id, string Customer, string Product, decimal Price, int Quantity);

var orders = new List<Order>
{
    new(1, "Ada",   "Keyboard", 79.99m, 1),
    new(2, "Grace", "Monitor",  249.00m, 2),
    new(3, "Ada",   "Mouse",    25.50m, 3),
    new(4, "Linus", "Keyboard", 79.99m, 1),
    new(5, "Grace", "Cable",    9.99m, 5),
    new(6, "Ada",   "Monitor",  249.00m, 1),
};

// Total revenue
decimal revenue = orders.Sum(o => o.Price * o.Quantity);
Console.WriteLine($"Revenue: {revenue:C}");

// Orders over $100, most expensive first
var bigOrders = orders
    .Where(o => o.Price * o.Quantity > 100)
    .OrderByDescending(o => o.Price * o.Quantity)
    .ToList();

// Spend per customer — GroupBy + projection into an anonymous type
var perCustomer = orders
    .GroupBy(o => o.Customer)
    .Select(g => new
    {
        Customer = g.Key,
        Orders   = g.Count(),
        Total    = g.Sum(o => o.Price * o.Quantity),
    })
    .OrderByDescending(x => x.Total);

foreach (var c in perCustomer)
    Console.WriteLine($"{c.Customer,-6} {c.Orders} orders  {c.Total,10:C}");

// Which products did Ada buy?
var adaProducts = orders
    .Where(o => o.Customer == "Ada")
    .Select(o => o.Product)
    .Distinct()
    .ToList();
Console.WriteLine("Ada bought: " + string.Join(", ", adaProducts));

// Quick lookups
bool anyCables  = orders.Any(o => o.Product == "Cable");
var priciest    = orders.MaxBy(o => o.Price);          // the whole Order object
var byId        = orders.ToDictionary(o => o.Id);      // Dictionary<int, Order>
```

### Text processing with LINQ

```csharp
string text = "the quick brown fox jumps over the lazy dog the end";

var topWords = text
    .Split(' ', StringSplitOptions.RemoveEmptyEntries)
    .GroupBy(w => w)
    .Select(g => (Word: g.Key, Count: g.Count()))
    .OrderByDescending(t => t.Count)
    .ThenBy(t => t.Word)
    .Take(3);

foreach (var (word, count) in topWords)
    Console.WriteLine($"{word}: {count}");
// the: 3, then alphabetical singles
```

### Generating sequences

```csharp
var squares = Enumerable.Range(1, 10).Select(n => n * n);       // 1,4,9,...100
var grid = Enumerable.Range(0, 3)
    .SelectMany(r => Enumerable.Range(0, 3).Select(c => (r, c)));  // all 9 cells
```

## Common Pitfalls

**1. Re-enumerating expensive queries.**

```csharp
var q = hugeList.Where(Slow);
if (q.Count() > 0)                    // ❌ enumerates once...
    Console.WriteLine(q.First());     // ...and again

var results = hugeList.Where(Slow).ToList();   // ✅ run once, reuse
if (results.Count > 0) Console.WriteLine(results[0]);
```

(Also: prefer `.Any()` over `.Count() > 0` — `Any` stops at the first hit.)

**2. `First()` on an empty sequence throws.**

```csharp
var match = people.First(p => p.Age > 200);            // ❌ InvalidOperationException
var match = people.FirstOrDefault(p => p.Age > 200);   // ✅ null if none — then check
```

**3. Expecting LINQ to mutate the source.** LINQ never modifies the input; it returns new sequences. `list.OrderBy(x => x)` leaves `list` unsorted — assign the result, or use `list.Sort()` for in-place.

**4. Modifying the source while a deferred query is alive.** The query sees the data *at enumeration time*, and mutating the collection mid-`foreach` still throws. Materialize with `ToList()` before mutating.

**5. Select vs Where confusion.** `Where` picks *which* elements survive (same type out); `Select` transforms *each* element (possibly new type). Filtering with `Select(x => x > 4)` gives you a sequence of booleans — a classic head-scratcher.

**6. Overlong chains nobody can read.** LINQ rewards taste. If a chain exceeds ~5 operators, name intermediate variables — the compiler doesn't care, reviewers do.

## Practice Exercises

1. From `Enumerable.Range(1, 100)`, produce with single LINQ chains: (a) all multiples of 3 or 5 and their sum, (b) the squares of even numbers, largest first, top five only, (c) whether any number's square exceeds 9000.
2. Using the `Order` record from this chapter (copy the sample data), find: the average order value, the product generating the most total revenue, and every customer who bought more than one distinct product. GroupBy required.
3. Take a paragraph of text and produce a case-insensitive word-frequency table sorted by count descending then alphabetically, excluding words shorter than 3 letters. Output aligned columns.
4. Demonstrate deferred execution: build a `Where` query over a `List<int>`, enumerate it, then `Add` a matching element and enumerate again, showing the new element appears. Then show that a `.ToList()` snapshot doesn't change. Explain in a comment.
5. Rewrite your Chapter 12 gradebook queries in LINQ: each student's average via one `Select`, the top student via `MaxBy`, and a grouping of students into grade bands (A/B/C) via `GroupBy` over a switch expression.
