# Chapter 2: Variables, Primitive Types, and Objects

## Overview

In Python or JavaScript, a variable is just a label you can stick on any value: `x = 5`, then `x = "hello"` — no complaints. Java is different: **every variable has a declared type, fixed at compile time**, and the compiler refuses to build code that violates it. This is *static typing*, and it's the single biggest adjustment coming from dynamic languages.

Java also splits its values into two fundamentally different kinds — **primitives** (raw numbers, booleans, characters stored directly) and **objects** (everything else, accessed through references). Understanding this split explains half of Java's quirks, so we cover it carefully here.

## Definitions & Explanations

### Declaring variables

```java
int age = 30;          // type name, variable name, initial value
String city = "Oslo";  // String is a class (object type), so it's capitalized
double price;          // declared but not yet initialized
price = 9.99;          // assigned later — fine, but must be assigned before use
```

Once declared as `int`, a variable can *only* hold ints. `age = "thirty";` is a compile error, not a runtime surprise.

### The eight primitive types

Primitives are the raw, built-in value types. They are not objects: no methods, stored directly, very fast.

| Type      | Size    | Range / values                     | Typical use |
|-----------|---------|------------------------------------|-------------|
| `byte`    | 8-bit   | -128 to 127                        | raw binary data |
| `short`   | 16-bit  | -32,768 to 32,767                  | rare |
| `int`     | 32-bit  | ±2.1 billion                       | **default integer type** |
| `long`    | 64-bit  | ±9.2 quintillion                   | big counts, timestamps |
| `float`   | 32-bit  | ~7 decimal digits precision        | rare (graphics) |
| `double`  | 64-bit  | ~15 decimal digits precision       | **default decimal type** |
| `char`    | 16-bit  | a single UTF-16 character, `'A'`   | single characters |
| `boolean` | 1 value | `true` or `false`                  | conditions |

Notes for Python/JS people:

- Python's `int` is unlimited size; Java's `int` **overflows** silently past ±2,147,483,647. Use `long` for big numbers.
- JS has only one number type (`number`, a double). Java makes you choose, and integer division behaves differently (see pitfalls).
- `char` uses **single quotes** (`'A'`); `String` uses **double quotes** (`"A"`). They are not interchangeable.
- Literals: `long` needs an `L` suffix (`5_000_000_000L`), `float` needs `f` (`3.14f`). Underscores in numeric literals are legal separators: `1_000_000`.

### Objects and references

Everything that isn't a primitive is an **object**: `String`, arrays, everything you build with classes. Object variables don't hold the object itself — they hold a **reference** (think: an arrow pointing at the object, like Python names or JS object references).

```java
String a = "hello";
String b = a;        // b points at the SAME object; no copy is made
```

An object variable can also hold `null` — the absence of an object, like Python's `None` or JS's `null`. Calling anything on `null` throws the infamous `NullPointerException` at runtime.

### Wrapper classes and autoboxing

Each primitive has an object twin: `int` ↔ `Integer`, `double` ↔ `Double`, `boolean` ↔ `Boolean`, `char` ↔ `Character`, etc. You need wrappers when a context requires objects (collections, Chapter 12, can hold `Integer` but not `int`). Java converts automatically:

```java
Integer boxed = 42;      // autoboxing: int → Integer
int unboxed = boxed;     // unboxing: Integer → int
```

Wrappers also carry useful statics: `Integer.parseInt("42")`, `Integer.MAX_VALUE`, `Double.parseDouble("3.14")`.

### `var` — local type inference (Java 10+)

```java
var count = 10;            // compiler infers int
var name = "Ada";          // compiler infers String
```

`var` is **not** dynamic typing — the type is still fixed, just inferred. You can't later do `count = "ten"`. Use `var` when the type is obvious from the right-hand side; write the explicit type when it aids readability. `var` only works for local variables with initializers.

### Constants

```java
final double TAX_RATE = 0.25;   // final = cannot be reassigned
```

`final` is Java's `const`. Convention: constants in ALL_CAPS.

### Casting between numeric types

- **Widening** (small → big) is automatic: `int` → `long` → `double`.
- **Narrowing** (big → small) requires an explicit cast and can lose data:

```java
double d = 9.99;
int i = (int) d;      // i == 9 — truncates toward zero, does NOT round
long big = 10_000_000_000L;
int oops = (int) big; // overflow: garbage value, no error!
```

## Code Examples

### Tour of the type system

```java
public class TypeTour {
    public static void main(String[] args) {
        // --- primitives ---
        int apples = 12;
        long worldPopulation = 8_100_000_000L;   // needs L suffix
        double temperature = -4.5;
        boolean isRaining = false;
        char grade = 'A';

        // --- objects ---
        String message = "Java is strongly typed";
        Integer boxedCount = 42;                  // wrapper object

        // widening happens automatically
        double applesAsDouble = apples;           // 12.0

        // narrowing needs a cast
        int roundedDown = (int) temperature;      // -4 (truncation)

        System.out.println("apples: " + apples);
        System.out.println("as double: " + applesAsDouble);
        System.out.println("truncated temp: " + roundedDown);
        System.out.println("grade char: " + grade);
        System.out.println(message + " — " + boxedCount + " " + isRaining);
        System.out.println(worldPopulation + " people");
    }
}
```

### Overflow and precision demo

```java
public class NumberGotchas {
    public static void main(String[] args) {
        // Integer overflow wraps silently — Python users beware!
        int max = Integer.MAX_VALUE;              // 2,147,483,647
        System.out.println(max + 1);              // -2147483648 (wrapped!)

        // Integer division truncates (like Python's //, unlike Python's /)
        System.out.println(7 / 2);                // 3, not 3.5
        System.out.println(7 / 2.0);              // 3.5 — one double makes it double math
        System.out.println(7 % 2);                // 1 (remainder)

        // Floating point is binary — same 0.1 + 0.2 issue as JS/Python
        System.out.println(0.1 + 0.2);            // 0.30000000000000004
        // For money, use long cents or java.math.BigDecimal — never double.

        // Parsing strings to numbers
        int parsed = Integer.parseInt("123");
        double parsedD = Double.parseDouble("3.14");
        System.out.println(parsed + parsedD);     // 126.14
    }
}
```

### Reading user input

`Scanner` is the standard beginner tool for console input (an object from the standard library):

```java
import java.util.Scanner;   // import brings a class into scope (Ch. 11)

public class InputDemo {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);  // 'new' creates an object

        System.out.print("Your name: ");
        String name = scanner.nextLine();          // reads a whole line

        System.out.print("Your age: ");
        int age = scanner.nextInt();               // reads an int token

        System.out.println("Hi " + name + ", next year you'll be " + (age + 1));
        scanner.close();
    }
}
```

## Common Pitfalls

### 1. Integer division surprise

```java
double average = (5 + 6) / 2;      // ❌ 5.0 — int division happened FIRST
double average = (5 + 6) / 2.0;    // ✅ 5.5
```

If both operands are ints, `/` is integer division. Make one operand a double (or cast) to get decimal math.

### 2. Comparing with `=` instead of `==`

```java
if (x = 5) { }    // ❌ assignment, compile error (unless x is boolean!)
if (x == 5) { }   // ✅ comparison
```

Java catches this at compile time for non-booleans — one advantage over old-school JS.

### 3. Using `double` for money

```java
double total = 0.1 + 0.2;              // ❌ 0.30000000000000004
long totalCents = 10 + 20;             // ✅ store cents as integers
// or: BigDecimal total = new BigDecimal("0.10").add(new BigDecimal("0.20"));
```

### 4. `char` vs `String` quotes

```java
char c = "A";     // ❌ incompatible types
char c = 'A';     // ✅
String s = 'A';   // ❌
String s = "A";   // ✅
```

### 5. Uninitialized local variables

```java
int count;
System.out.println(count);   // ❌ compile error: variable might not have been initialized
```

Unlike fields (Chapter 8), local variables have **no default value** — you must assign before reading. JS would give you `undefined`; Java refuses to compile.

### 6. Comparing wrapper objects with `==`

```java
Integer a = 1000, b = 1000;
System.out.println(a == b);        // ❌ often false! compares references
System.out.println(a.equals(b));   // ✅ true — compares values
```

More on `==` vs `.equals()` in Chapters 6 and 8 — it's Java's most classic trap.

## Practice Exercises

1. **Type census.** Declare one variable of each of the eight primitive types plus a `String`, give each a sensible value, and print them all with labels. Try assigning a wrong-typed value to each and note the compile errors.
2. **Temperature converter.** Read a Celsius value with `Scanner` (as `double`) and print Fahrenheit (`F = C * 9 / 5 + 32`) with exactly one decimal place using `printf`. Test that `100` prints `212.0` — if it doesn't, you've hit the integer-division pitfall.
3. **Overflow explorer.** Print `Integer.MAX_VALUE`, then `Integer.MAX_VALUE + 1`. Then do the same computation using `long` variables so it comes out correct. Explain in a comment why the two differ.
4. **Truncation vs rounding.** Given `double x = 7.75;`, produce: the truncated int (7), the rounded int (8, hint: `Math.round`), and the ceiling (8, hint: `Math.ceil`). Watch the return types — `Math.round(double)` returns `long`.
5. **Swap.** Read two ints, then swap the values of the two variables using a third temporary variable, and print them before and after.
