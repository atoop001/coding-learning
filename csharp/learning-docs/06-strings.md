# Chapter 6: Strings & String Interpolation

## Overview

Strings hold text, and real programs spend a surprising amount of time slicing, joining, searching, and formatting them. C# strings are **immutable** objects with a rich method library, and **string interpolation** (`$"..."`) is the everyday tool for building output. This chapter covers creation, common operations, formatting, comparison, and `StringBuilder` for heavy concatenation.

## Definitions & Explanations

### The basics

```csharp
string greeting = "Hello";        // double quotes ONLY for strings
char initial = 'H';               // single quotes ONLY for chars
string empty = "";
string? nothing = null;           // a string variable may be null
```

Unlike Python/JS, single vs double quotes are *different types*: `'a'` is a `char`, `"a"` is a `string`.

### Immutability

Strings can never change after creation. Every "modifying" method returns a **new** string:

```csharp
string s = "hello";
s.ToUpper();                  // ❌ result thrown away — s is still "hello"
s = s.ToUpper();              // ✅ reassign to capture the new string: "HELLO"
```

Python strings behave identically; JS strings too. The trap is *forgetting to use the return value*.

### Interpolation and formatting

```csharp
string name = "Ada";
int age = 36;

string msg = $"{name} is {age} years old.";      // like f"..." / `${...}`
string math = $"Next year: {age + 1}";           // any expression inside {}

// Format specifiers after a colon
double price = 1234.5678;
Console.WriteLine($"{price:F2}");     // 1234.57   (2 decimals)
Console.WriteLine($"{price:C}");      // $1,234.57 (currency, culture-dependent)
Console.WriteLine($"{price:N0}");     // 1,235     (thousands separators)
Console.WriteLine($"{0.847:P1}");     // 84.7 %    (percent)
Console.WriteLine($"{42,8}");         // '      42' (right-align in 8 chars)
Console.WriteLine($"{42,-8}|");       // '42      |' (left-align)
Console.WriteLine($"{DateTime.Now:yyyy-MM-dd}");  // 2026-07-18
```

### Verbatim and raw strings

```csharp
string path = "C:\\Users\\atoop";        // escaped backslashes
string path2 = @"C:\Users\atoop";        // @ = verbatim: backslashes literal
string json = """
    { "name": "Ada" }
    """;                                  // raw string literal (C# 11): quotes ok
```

### Essential string members

```csharp
string s = "  Hello, World!  ";

s.Length                       // 17 — a property, not a method (no parentheses)
s.Trim()                       // "Hello, World!"     (also TrimStart/TrimEnd)
s.ToUpper() / s.ToLower()
s.Contains("World")            // true
s.StartsWith("  He")           // true; also EndsWith
s.IndexOf('W')                 // 9 (or -1 if absent — like JS, unlike Python's ValueError)
s.Replace("World", "C#")       // "  Hello, C#!  "
s.Substring(2, 5)              // "Hello" — (startIndex, LENGTH), not (start, end)!
s[2]                           // 'H' — indexing gives a char
s.Split(',')                   // string[] { "  Hello", " World!  " }
string.Join(", ", parts)       // inverse of Split — note: static method on string
string.IsNullOrWhiteSpace(s)   // the standard "is this blank?" check
```

Slicing: C# has ranges (`s[2..7]` → chars 2 through 6, like Python's `s[2:7]`) and index-from-end (`s[^1]` = last char).

### Comparing strings

```csharp
a == b                                             // value comparison — works fine
a.Equals(b, StringComparison.OrdinalIgnoreCase)    // case-insensitive
string.Compare(a, b)                               // ordering: <0, 0, >0
```

Good news: `==` on strings compares *contents*, not references (unlike Java!).

### StringBuilder — for building big strings in loops

Because strings are immutable, `result += piece` in a loop creates a brand-new string every pass — O(n²) for large n. Use `StringBuilder`:

```csharp
using System.Text;

var sb = new StringBuilder();
for (int i = 1; i <= 1000; i++)
    sb.AppendLine($"Line {i}");     // mutates the builder, no copies

string report = sb.ToString();      // one final string at the end
```

Rule of thumb: a handful of `+` concatenations or interpolation — fine. Concatenating in a loop with many iterations — `StringBuilder`.

## Code Examples

### Name formatter

```csharp
// NameTagMaker — trims, validates, formats
Console.Write("Enter full name: ");
string raw = Console.ReadLine() ?? "";
string cleaned = raw.Trim();

if (string.IsNullOrWhiteSpace(cleaned))
{
    Console.WriteLine("No name entered.");
    return;
}

string[] parts = cleaned.Split(' ', StringSplitOptions.RemoveEmptyEntries);
string first = parts[0];
string last = parts.Length > 1 ? parts[^1] : "";   // ^1 = last element

// Capitalize: first letter upper, rest lower
string Cap(string word) =>
    word.Length == 0 ? word : char.ToUpper(word[0]) + word[1..].ToLower();

Console.WriteLine($"Name tag : {Cap(first)} {Cap(last)}".TrimEnd());
Console.WriteLine($"Username : {(first + last).ToLower()}");
Console.WriteLine($"Initials : {char.ToUpper(first[0])}.{(last.Length > 0 ? char.ToUpper(last[0]) + "." : "")}");
```

### Simple word statistics

```csharp
string text = "the quick brown fox jumps over the lazy dog the end";

string[] words = text.Split(' ', StringSplitOptions.RemoveEmptyEntries);
Console.WriteLine($"Words: {words.Length}");
Console.WriteLine($"Chars (no spaces): {text.Replace(" ", "").Length}");

// Longest word
string longest = "";
foreach (string w in words)
    if (w.Length > longest.Length) longest = w;
Console.WriteLine($"Longest word: '{longest}' ({longest.Length} letters)");

// Count occurrences of "the"
int theCount = 0;
foreach (string w in words)
    if (w == "the") theCount++;
Console.WriteLine($"'the' appears {theCount} times");
```

## Common Pitfalls

**1. Forgetting strings are immutable.**

```csharp
name.Trim();               // ❌ does nothing visible
name = name.Trim();        // ✅
```

**2. `Substring(start, length)` — the second argument is a LENGTH.** Python's `s[2:5]` takes an *end index*; C#'s `Substring(2, 5)` takes *five characters* starting at 2. Prefer range syntax `s[2..5]`, which matches Python's semantics.

**3. Indexing a possibly-empty string.**

```csharp
char first = input[0];                    // ❌ crashes if input is ""
char? first = input.Length > 0 ? input[0] : null;   // ✅ guard first
```

**4. Comparing user input without normalizing.**

```csharp
if (answer == "yes")                            // misses "Yes ", "YES"
if (answer?.Trim().ToLower() == "yes")          // ✅ normalize first
```

**5. Concatenating numbers accidentally.** `"Total: " + 1 + 2` is `"Total: 12"` (left-to-right, like JS). Use interpolation or parentheses: `$"Total: {1 + 2}"`.

**6. Single vs double quotes.**

```csharp
string s = 'hello';     // ❌ CS1012: too many characters in character literal
string s = "hello";     // ✅
```

## Practice Exercises

1. Write a program that reads a sentence and prints it reversed, in all caps, and with vowels removed (three lines of output).
2. Write `static bool IsPalindrome(string s)` that ignores case, spaces, and punctuation ("A man, a plan, a canal: Panama" → true). Hint: build a cleaned string first, `char.IsLetterOrDigit` helps.
3. Read a full name and produce an email address `first.last@company.com`, lowercased and with a fallback when only one name is given. Handle extra spaces gracefully.
4. Write a simple Caesar cipher: shift each letter by 3 positions (wrapping z→c), leaving non-letters untouched. Encrypt and then decrypt to verify round-tripping. Use `StringBuilder`.
5. Benchmark intuition: build a 50,000-piece string once with `+=` in a loop and once with `StringBuilder`, timing each with `System.Diagnostics.Stopwatch`. Record the difference in a comment.
