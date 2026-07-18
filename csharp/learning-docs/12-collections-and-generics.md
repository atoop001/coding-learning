# Chapter 12: Collections & Generics

## Overview

You've used `List<T>` for a while; now we make sense of that `<T>` and meet the rest of the collection family: `Dictionary<TKey, TValue>` (Python dict / JS Map), `HashSet<T>` (Python set), `Queue<T>`, and `Stack<T>`. Then we flip to the other side and *write* generic classes and methods ourselves — the mechanism that lets one class work safely with any element type. Generics are everywhere in C#; after this chapter, signatures like `Dictionary<string, List<Order>>` will read naturally.

## Definitions & Explanations

### What generics are

A generic type has one or more **type parameters** — placeholders filled in at use site:

```csharp
List<int> a;                 // T = int      → a list that only holds ints
List<string> b;              // T = string
Dictionary<string, int> c;   // TKey = string, TValue = int
```

The compiler then enforces it: `a.Add("hi")` is a compile error. Contrast with Python/JS, where any list holds anything and mistakes surface at runtime. Type hints in Python (`list[int]`) are the same *idea*, but optional and unenforced; in C# they're the real thing.

### Dictionary<TKey, TValue>

Key → value lookup, like `dict` / `Map`:

```csharp
var ages = new Dictionary<string, int>
{
    ["Ada"] = 36,               // index-style initializer
    ["Grace"] = 85,
};

ages["Linus"] = 55;             // add or overwrite
int a = ages["Ada"];            // read — THROWS KeyNotFoundException if absent!

// The safe patterns:
if (ages.TryGetValue("Bob", out int bobAge))     // preferred: test + get in one call
    Console.WriteLine(bobAge);

if (ages.ContainsKey("Ada")) { /* ... */ }

ages.Remove("Grace");
Console.WriteLine(ages.Count);

foreach (var kvp in ages)                        // KeyValuePair<string,int>
    Console.WriteLine($"{kvp.Key} is {kvp.Value}");

foreach (var (name, age) in ages)                // or deconstruct directly
    Console.WriteLine($"{name}: {age}");
```

Notes: keys must be unique; lookup is O(1); Python's `d["missing"]` raises KeyError — C# throws `KeyNotFoundException`; `TryGetValue` is the equivalent of `d.get(k)`.

### HashSet<T>

Unordered collection of **unique** values with O(1) membership tests:

```csharp
var seen = new HashSet<string>();
seen.Add("apple");           // true — added
seen.Add("apple");           // false — already there (no exception, no duplicate)
bool has = seen.Contains("apple");

var evens = new HashSet<int> { 2, 4, 6 };
var small = new HashSet<int> { 1, 2, 3 };
evens.IntersectWith(small);  // evens is now { 2 } — also UnionWith, ExceptWith
```

Use it whenever you're tracking "have I seen this before?" — vastly faster than `List.Contains` for large data.

### Queue<T> and Stack<T>

```csharp
var line = new Queue<string>();     // FIFO — first in, first out
line.Enqueue("Ada");
line.Enqueue("Grace");
string next = line.Dequeue();       // "Ada"

var undo = new Stack<string>();     // LIFO — last in, first out
undo.Push("typed A");
undo.Push("typed B");
string last = undo.Pop();           // "typed B"
```

### Writing your own generic class

```csharp
// A type-safe box that works for ANY element type — written once
public class Pair<T>
{
    public T First { get; }
    public T Second { get; }

    public Pair(T first, T second) { First = first; Second = second; }

    public Pair<T> Swapped() => new Pair<T>(Second, First);
}

var p1 = new Pair<int>(1, 2);           // T = int
var p2 = new Pair<string>("a", "b");    // T = string
int x = p1.First;                       // fully typed — no casting anywhere
```

### Generic methods and constraints

```csharp
// A generic METHOD — T inferred from the argument
static T Middle<T>(List<T> items) => items[items.Count / 2];

int m = Middle(new List<int> { 1, 2, 3 });          // T inferred as int

// Constraints: 'where' limits what T may be, unlocking capabilities
static T Max<T>(T a, T b) where T : IComparable<T>  // T must be comparable
    => a.CompareTo(b) >= 0 ? a : b;

Max(3, 7);            // 7
Max("ant", "bee");    // "bee"
```

Common constraints: `where T : class`, `where T : new()` (has parameterless constructor), `where T : SomeBase`, `where T : ISomeInterface`.

## Code Examples

### Word frequency counter (Dictionary in anger)

```csharp
string text = "the cat and the dog and the bird";

var counts = new Dictionary<string, int>();
foreach (string word in text.Split(' ', StringSplitOptions.RemoveEmptyEntries))
{
    // Classic upsert pattern:
    if (counts.TryGetValue(word, out int current))
        counts[word] = current + 1;
    else
        counts[word] = 1;
}

foreach (var (word, count) in counts)
    Console.WriteLine($"{word,-6} {new string('#', count)}");
// the    ###
// cat    #
// and    ##
// dog    #
// bird   #
```

### A generic bounded stack with constraint-free design

```csharp
// BoundedStack<T> — our own collection type, built on List<T>
public class BoundedStack<T>
{
    private readonly List<T> _items = new();
    public int Capacity { get; }

    public BoundedStack(int capacity)
    {
        if (capacity <= 0) throw new ArgumentOutOfRangeException(nameof(capacity));
        Capacity = capacity;
    }

    public int Count => _items.Count;
    public bool IsFull => Count == Capacity;

    public bool TryPush(T item)
    {
        if (IsFull) return false;
        _items.Add(item);
        return true;
    }

    public bool TryPop(out T? item)
    {
        if (Count == 0) { item = default; return false; }   // default: null/0/false per T
        item = _items[^1];
        _items.RemoveAt(_items.Count - 1);
        return true;
    }
}

var history = new BoundedStack<string>(3);
history.TryPush("page1");
history.TryPush("page2");
if (history.TryPop(out string? page))
    Console.WriteLine($"Back to {page}");     // Back to page2
```

### Grouping with Dictionary<string, List<T>>

```csharp
var students = new[] { "Ada", "Alan", "Grace", "Guido", "Anders" };

var byInitial = new Dictionary<char, List<string>>();
foreach (var name in students)
{
    char key = name[0];
    if (!byInitial.TryGetValue(key, out var group))
    {
        group = new List<string>();
        byInitial[key] = group;
    }
    group.Add(name);
}

foreach (var (initial, group) in byInitial)
    Console.WriteLine($"{initial}: {string.Join(", ", group)}");
// A: Ada, Alan, Anders
// G: Grace, Guido
```

(LINQ's `GroupBy` will do this in one line — Chapter 15.)

## Common Pitfalls

**1. Indexing a missing dictionary key.**

```csharp
int n = counts["zzz"];                    // ❌ KeyNotFoundException
counts.TryGetValue("zzz", out int n2);    // ✅ n2 = 0, returns false
```

(Assigning `counts["zzz"] = 1` is always fine — write access creates the key.)

**2. Expecting HashSet or Dictionary to stay in insertion order.** Enumeration order is *unspecified* — mostly insertion-ish in practice, never guaranteed. Need order? Sort at output time, or use `List`.

**3. Using a mutable class as a dictionary key without care.** Keys are located by hash; if a key object's hash-relevant data changes after insertion, it gets "lost". Use immutable keys: strings, numbers, records.

**4. `default` surprises in generics.** `default(T)` is `null` for reference types but `0`/`false` for value types — hence the `out T?` in `TryPop`. Don't assume `default` means null.

**5. Modifying a dictionary while enumerating it.** Same rule as lists in `foreach`: runtime exception. Collect keys to change first, or build a new dictionary.

**6. Reaching for non-generic legacy types.** If you see `ArrayList` or `Hashtable` in old tutorials — those predate generics (2005!). Never use them; `List<T>` and `Dictionary<K,V>` replaced them entirely.

## Practice Exercises

1. Read a line of text and use a `Dictionary<char, int>` to count each letter (ignore case and non-letters). Print counts sorted alphabetically by letter.
2. Given two hardcoded `List<int>` with duplicates, use `HashSet<int>` operations to print: unique values in either, values in both, and values only in the first. Then explain (comment) the running time advantage over nested loops.
3. Simulate a print queue: a `Queue<string>` of jobs, a loop that processes one per "tick" and randomly enqueues new jobs, until empty. Then implement browser history back/forward with two `Stack<string>`s.
4. Write a generic class `Repository<T>` with `Add(T)`, `GetAll()`, and `Count`, backed by a private list. Instantiate it for two different element types. Then add a constrained method `T? FindMax()` requiring `where T : IComparable<T>`.
5. Build a gradebook: `Dictionary<string, List<double>>` mapping student → scores. Support adding scores, printing each student's average, and finding the top student. What goes wrong if you index a student who doesn't exist yet, and which pattern prevents it?
