# Chapter 5: Methods and `static`

## Overview

Methods are Java's functions — but in Java, *every* function belongs to a class; there are no free-floating functions like Python's `def` at module level or JS's top-level `function`. This chapter covers defining and calling methods, parameters and return types, overloading, and finally demystifies that `static` keyword you've been typing in `public static void main` since Chapter 1.

Well-factored methods are the first big step from "scripts" to "software": small, named, testable units.

## Definitions & Explanations

### Anatomy of a method

```java
public static int addTwo(int a, int b) {
    return a + b;
}
//  │      │    │   │        │
//  │      │    │   │        └─ parameters: each needs a TYPE and a name
//  │      │    │   └─ method name (camelCase by convention)
//  │      │    └─ return type: what the method gives back (or void for nothing)
//  │      └─ static: belongs to the class, not an instance (below)
//  └─ access modifier (Ch. 11); public = callable from anywhere
```

Key contrasts with Python/JS:

- **Return type is declared and enforced.** A method declared `int` *must* return an `int` on every path, or it won't compile. `void` methods return nothing (`return;` alone is allowed to exit early).
- **Parameter types are declared.** Passing a `String` to an `int` parameter is a compile error.
- **No default parameter values, no keyword arguments.** Java's answer is *overloading* (below).

### Calling methods

```java
int sum = addTwo(3, 4);                 // same-class static method
double r = Math.sqrt(2.0);              // static method on another class: ClassName.method
String up = "hello".toUpperCase();      // instance method: object.method
```

### `static` vs instance — the key idea

- A **static** method belongs to the *class itself*. You call it as `ClassName.method(...)`. It cannot touch instance fields because there is no instance. `Math.max`, `Integer.parseInt`, and `main` are static.
- An **instance** method belongs to *an object*. You call it as `object.method(...)`, and inside it can use that object's data. `"abc".length()` is an instance method — it needs a particular string to measure.

Until we build classes with state (Chapter 8), everything you write will be `static` — that's normal. Rule of thumb: `static` = "a pure function or utility that doesn't need per-object data."

Static also applies to variables: a `static` field is shared by the whole class (one copy total), while instance fields exist once per object. More in Chapter 8.

### Method overloading

Java allows several methods with the **same name but different parameter lists**. The compiler picks the right one from the argument types:

```java
static double area(double radius) { return Math.PI * radius * radius; }
static double area(double width, double height) { return width * height; }
```

This is how Java compensates for missing default/keyword arguments. Note: you *cannot* overload on return type alone.

### Varargs

A method can accept a variable number of arguments; inside, they arrive as an array:

```java
static int sumAll(int... numbers) {      // like Python *args
    int total = 0;
    for (int n : numbers) total += n;
    return total;
}
// sumAll(), sumAll(1), sumAll(1, 2, 3) all work
```

Varargs must be the last parameter.

### Pass-by-value (important!)

Java is strictly **pass-by-value** — but what gets copied differs:

- Primitives: the *value* is copied. The method cannot change the caller's variable.
- Objects: the *reference* is copied. The method can mutate the object it points to, but reassigning the parameter doesn't affect the caller.

This is exactly how Python and JS behave too, so your intuition transfers — Java just makes people argue about the terminology.

### Scope

Variables live from their declaration to the end of their enclosing `{ }` block. Method parameters and locals vanish when the method returns. There are no closures over locals for ordinary methods (lambdas, Chapter 15, get close).

## Code Examples

### A small utility class

```java
public class TempUtils {

    // Convert Celsius to Fahrenheit. Pure function: input -> output, no side effects.
    public static double toFahrenheit(double celsius) {
        return celsius * 9.0 / 5.0 + 32.0;
    }

    // void method: performs an action, returns nothing
    public static void printReport(String city, double celsius) {
        double f = toFahrenheit(celsius);            // methods calling methods
        System.out.printf("%s: %.1f C = %.1f F%n", city, celsius, f);
    }

    // Early return / guard clause pattern
    public static String describe(double celsius) {
        if (celsius < -50 || celsius > 60) {
            return "sensor error";                    // exit early on bad data
        }
        if (celsius <= 0) return "freezing";
        if (celsius < 15) return "cold";
        if (celsius < 25) return "mild";
        return "hot";
    }

    public static void main(String[] args) {
        printReport("Oslo", -3.5);
        printReport("Cairo", 34.0);
        System.out.println(describe(20.0));           // mild
        System.out.println(describe(999.0));          // sensor error
    }
}
```

### Overloading and varargs in action

```java
public class Formatter {

    // Overload 1: greet with just a name
    public static String greet(String name) {
        return "Hello, " + name + "!";
    }

    // Overload 2: greet with a custom greeting word
    public static String greet(String greeting, String name) {
        return greeting + ", " + name + "!";
    }

    // Varargs: join any number of words with a separator
    public static String join(String separator, String... words) {
        if (words.length == 0) return "";
        String result = words[0];
        for (int i = 1; i < words.length; i++) {
            result += separator + words[i];   // fine for small cases; see Ch. 6 for StringBuilder
        }
        return result;
    }

    public static void main(String[] args) {
        System.out.println(greet("Ada"));                     // picks overload 1
        System.out.println(greet("Good morning", "Ada"));     // picks overload 2
        System.out.println(join(" | ", "one", "two", "three"));
    }
}
```

### Pass-by-value demonstration

```java
public class PassByValue {
    public static void tryToChange(int n, int[] arr) {
        n = 999;            // only changes the local copy
        arr[0] = 999;       // mutates the SHARED array object — caller sees this
        arr = new int[]{7}; // reassigning the parameter — caller does NOT see this
    }

    public static void main(String[] args) {
        int x = 1;
        int[] data = {1, 2, 3};
        tryToChange(x, data);
        System.out.println(x);        // 1      — primitive unchanged
        System.out.println(data[0]);  // 999    — object mutation visible
        System.out.println(data.length); // 3   — reassignment not visible
    }
}
```

## Common Pitfalls

### 1. Missing return on some path

```java
static String sign(int n) {
    if (n > 0) return "positive";
    if (n < 0) return "negative";
}   // ❌ compile error: missing return statement (what if n == 0?)
```

**Fix:** add a final `return "zero";`. The compiler checks *every* path returns.

### 2. Ignoring the return value

```java
name.toUpperCase();               // ❌ does nothing useful — Strings are immutable
name = name.toUpperCase();        // ✅ capture the result
```

Methods that "transform" usually *return* the new value rather than modifying in place.

### 3. Calling an instance method from `main` without an object

```java
public class App {
    public int helper() { return 42; }
    public static void main(String[] args) {
        System.out.println(helper());   // ❌ non-static method cannot be referenced from a static context
    }
}
```

**Fix (for now):** make `helper` static too. (Later, Chapter 8: create an object and call `new App().helper()`.) This error message will haunt you until `static` clicks — reread the definitions section when it appears.

### 4. Shadowing

```java
static int count = 10;
static void demo() {
    int count = 5;                    // shadows the static field
    System.out.println(count);        // 5 — the local wins
}
```

Legal but confusing. Avoid reusing names across scopes.

### 5. Confusing overloading with what Python defaults do

```java
static void log(String msg, boolean timestamp) { ... }
log("hi");     // ❌ no matching overload — Java has no default arguments
```

**Fix:** write a second overload `static void log(String msg) { log(msg, true); }` that delegates.

## Practice Exercises

1. **Math toolkit.** Write a class `MathKit` with static methods `max3(int, int, int)`, `isEven(int)`, `factorial(int)` (return `long`; guard against negatives), and `clamp(int value, int min, int max)`. Exercise each from `main` with printed test cases.
2. **Refactor FizzBuzz.** Take your Chapter 4 FizzBuzz and extract a method `static String fizzBuzzOf(int n)` that returns the string for one number. The loop in `main` should shrink to two lines. Notice how the method is now testable in isolation.
3. **Overloaded describe.** Write three overloads of `describe`: `describe(int age)` returns a life-stage string; `describe(double price)` returns "cheap"/"expensive"; `describe(String word)` returns the word plus its length. Call all three and confirm the compiler dispatches correctly.
4. **Varargs average.** Write `static double average(double... values)` returning the mean, and returning `Double.NaN` for zero arguments (document why in a comment). Then write `static double spread(double... values)` (max minus min) that reuses logic sensibly.
5. **Predict pass-by-value.** Without running it, write down what the `PassByValue` example prints and why, in your own words, as comments. Then run it and reconcile any differences. Explain in one sentence why "Java is pass-by-reference for objects" is technically wrong.
