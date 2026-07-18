# Chapter 4: Loops

## Overview

Loops repeat work. C# gives you four: `while`, `do-while`, the classic three-part `for`, and `foreach` (the closest cousin of Python's `for ... in` and JS's `for...of`). You'll use `foreach` most of the time for walking collections and `for` when you need index arithmetic. This chapter also covers `break`, `continue`, and how to avoid the classic infinite-loop and off-by-one mistakes.

## Definitions & Explanations

### while — check first, then run

```csharp
int countdown = 5;
while (countdown > 0)          // condition checked BEFORE each iteration
{
    Console.WriteLine(countdown);
    countdown--;               // forget this and you loop forever
}
```

Same shape as Python/JS `while`, just with parentheses and braces.

### do-while — run first, then check

Runs the body **at least once**. Python has no equivalent; JS's `do...while` is identical. Perfect for "ask until valid" input loops:

```csharp
int number;
do
{
    Console.Write("Enter a number from 1-10: ");
}
while (!int.TryParse(Console.ReadLine(), out number) || number < 1 || number > 10);
```

### for — counter-controlled

```csharp
//   initializer ; condition ; iterator
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"i = {i}");    // prints 0,1,2,3,4
}
```

- Runs the initializer once, checks the condition before each pass, runs the iterator after each pass.
- Python's `for i in range(5)` ≈ `for (int i = 0; i < 5; i++)`.
- Counting down: `for (int i = 10; i >= 1; i--)`. Stepping by 2: `i += 2`.
- The loop variable `i` only exists inside the loop.

### foreach — iterate a collection

```csharp
string[] names = { "Ada", "Linus", "Grace" };

foreach (string name in names)     // like Python's: for name in names
{
    Console.WriteLine(name);
}
```

`foreach` works on anything *enumerable*: arrays, `List<T>`, `Dictionary<K,V>`, strings (yields `char`s), LINQ results. Two rules:

1. The loop variable is **read-only** — you can't assign to `name`.
2. You may **not add/remove items** of the collection you're iterating (runtime exception). Mutate a copy, or use a `for` loop with an index.

### break and continue

```csharp
foreach (int n in numbers)
{
    if (n < 0) continue;   // skip this iteration, go to the next
    if (n == 0) break;     // exit the loop entirely
    Console.WriteLine(n);
}
```

Same as Python/JS. C# has no `for...else` (Python) — use a `bool found` flag instead.

## Code Examples

### FizzBuzz — the classic

```csharp
for (int i = 1; i <= 30; i++)
{
    string output = (i % 3, i % 5) switch    // tuple pattern — elegant!
    {
        (0, 0) => "FizzBuzz",
        (0, _) => "Fizz",
        (_, 0) => "Buzz",
        _      => i.ToString(),
    };
    Console.WriteLine(output);
}
```

### Input validation loop + running total

```csharp
// SumUntilDone — keep reading numbers until the user types "done"
double total = 0;
int count = 0;

while (true)                                     // deliberate infinite loop...
{
    Console.Write("Enter a number (or 'done'): ");
    string? line = Console.ReadLine();

    if (line?.Trim().ToLower() == "done")
        break;                                   // ...with an explicit exit

    if (double.TryParse(line, out double value))
    {
        total += value;
        count++;
    }
    else
    {
        Console.WriteLine("  Not a number — try again.");
    }
}

if (count > 0)
    Console.WriteLine($"Sum: {total}, Average: {total / count:F2}");
else
    Console.WriteLine("No numbers entered.");
```

### Nested loops — a multiplication table

```csharp
for (int row = 1; row <= 9; row++)
{
    for (int col = 1; col <= 9; col++)
    {
        // {row * col,4} right-aligns each product in 4 characters
        Console.Write($"{row * col,4}");
    }
    Console.WriteLine();     // newline after each row
}
```

### Searching with an early exit

```csharp
int[] primes = { 2, 3, 5, 7, 11, 13 };
int target = 7;
bool found = false;

for (int i = 0; i < primes.Length; i++)   // note: .Length for arrays
{
    if (primes[i] == target)
    {
        Console.WriteLine($"Found {target} at index {i}");
        found = true;
        break;                            // no need to keep looking
    }
}

if (!found)
    Console.WriteLine($"{target} not in the array.");
```

## Common Pitfalls

**1. The infinite loop — forgetting to change the loop variable.**

```csharp
int i = 0;
while (i < 10)
{
    Console.WriteLine(i);      // ❌ i never changes — runs forever
}

while (i < 10)
{
    Console.WriteLine(i);
    i++;                       // ✅
}
```

**2. Off-by-one: `<=` with `.Length`.** Arrays are 0-indexed; the last valid index is `Length - 1`.

```csharp
for (int i = 0; i <= items.Length; i++)  // ❌ IndexOutOfRangeException on last pass
for (int i = 0; i < items.Length; i++)   // ✅
```

**3. Modifying a collection inside `foreach`.**

```csharp
foreach (var item in list)
    if (item.IsExpired) list.Remove(item);       // ❌ InvalidOperationException

list.RemoveAll(item => item.IsExpired);          // ✅ built-in, safe
// or iterate a copy: foreach (var item in list.ToList()) ...
```

**4. Assigning to the `foreach` variable expecting to change the collection.**

```csharp
foreach (int n in numbers) { n = n * 2; }        // ❌ compile error — read-only
for (int i = 0; i < numbers.Length; i++)         // ✅ use an index to write back
    numbers[i] *= 2;
```

**5. Semicolon right after the loop header.**

```csharp
for (int i = 0; i < 3; i++);       // ❌ that ; IS the (empty) body
{
    Console.WriteLine(i);          // runs once, and i doesn't even exist here
}
```

**6. Confusing `.Length` (arrays, strings) with `.Count` (Lists).** Both exist, on different types; IntelliSense will steer you.

## Practice Exercises

1. Print the numbers 1–100, but for multiples of 7 print `LUCKY` instead. Then modify it to also stop entirely (with a message) at the first number greater than 50 that ends in 3.
2. Write a guessing game: pick a fixed secret number (hardcode 42 for now), and loop asking the user to guess, printing "higher"/"lower" until correct. Count and report the number of guesses.
3. Using nested loops, print a right triangle of `*` with height read from the user (height 4 = rows of 1, 2, 3, 4 stars). Then invert it.
4. Read 5 integers into an array using a `for` loop, then use a `foreach` to compute and print the minimum, maximum, and average without using built-in Min/Max methods.
5. Write a loop that computes the factorial of a user-supplied n (use `long`!). Report the first n at which the result overflows to a negative number — what does that tell you about `long`?
