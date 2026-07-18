# Chapter 15: Lambdas and Streams

## Overview

Java 8 (2014) bolted functional programming onto Java, and modern codebases lean on it heavily: **lambdas** are inline anonymous functions, and the **Stream API** is a pipeline for transforming collections declaratively — Java's answer to Python comprehensions/`map`/`filter` and JS's `array.map().filter().reduce()`. Interviewers now expect fluency here, and code review culture prefers a clean stream over a five-line loop.

You already have the mental model from JS arrow functions and Python lambdas; this chapter maps it onto Java's typed world: functional interfaces, method references, and the stream pipeline lifecycle.

## Definitions & Explanations

### Lambdas — anonymous functions

```java
(a, b) -> a + b                    // two params, expression body
name -> name.toUpperCase()         // one param: parens optional
() -> System.out.println("hi")     // no params
s -> {                             // block body needs braces AND return
    String t = s.strip();
    return t.toLowerCase();
}
```

JS arrow functions with `->` instead of `=>`. Parameter types are usually inferred.

### Functional interfaces — what a lambda *is*

Java has no standalone function type. A lambda is an instance of a **functional interface** — any interface with exactly one abstract method (Chapter 10). The lambda supplies that method's body:

```java
Comparator<String> byLength = (a, b) -> Integer.compare(a.length(), b.length());
Runnable task = () -> System.out.println("running");
```

The core toolbox lives in `java.util.function` — learn these four cold:

| Interface | Method | Shape | Typical use |
|-----------|--------|-------|-------------|
| `Predicate<T>` | `test` | `T -> boolean` | filtering |
| `Function<T,R>` | `apply` | `T -> R` | mapping/transforming |
| `Consumer<T>` | `accept` | `T -> void` | doing something per element |
| `Supplier<T>` | `get` | `() -> T` | lazy production/factories |

Plus `Comparator<T>` for ordering — the everyday lambda target since forever.

### Method references — lambda shorthand

When a lambda just calls one existing method, name the method instead:

| Form | Example | Equivalent lambda |
|------|---------|-------------------|
| static | `Integer::parseInt` | `s -> Integer.parseInt(s)` |
| instance method of the element | `String::toUpperCase` | `s -> s.toUpperCase()` |
| instance method of a captured object | `System.out::println` | `x -> System.out.println(x)` |
| constructor | `ArrayList::new` | `() -> new ArrayList<>()` |

### Streams — pipelines over data

A stream pipeline has three phases:

```java
List<String> result = names.stream()          // 1. SOURCE
        .filter(n -> n.length() > 3)          // 2. INTERMEDIATE ops (lazy, chainable)
        .map(String::toUpperCase)             //    ...
        .sorted()                             //    ...
        .toList();                            // 3. TERMINAL op — triggers everything
```

Key intermediate ops: `filter` (keep matching), `map` (transform), `sorted` (optionally with a Comparator), `distinct`, `limit(n)`, `skip(n)`, `flatMap` (flatten nested).

Key terminal ops: `toList()`, `forEach`, `count()`, `sum()` (numeric streams), `anyMatch`/`allMatch`/`noneMatch`, `findFirst`, `min`/`max`, `reduce`, and `collect(Collectors...)` for grouping/joining.

Crucial properties:

- **Lazy**: intermediate ops do nothing until a terminal op runs.
- **Non-mutating**: the source collection is untouched; results are new.
- **Single-use**: a consumed stream cannot be reused — build a fresh one.
- Streams *replace* loops for transform/filter/aggregate jobs; loops remain right for complex stateful logic and index-driven work.

### Optional — maybe-a-value

Some terminal ops can't promise a result (`findFirst` on an empty stream), so they return `Optional<T>` — a box holding a value or nothing, replacing null-returns:

```java
Optional<String> first = names.stream().filter(n -> n.startsWith("Z")).findFirst();
String result = first.orElse("none found");           // safe unwrap with default
first.ifPresent(System.out::println);                 // act only if present
```

Never call `.get()` without checking — that's just an NPE in a fancier hat.

### Numeric streams

`IntStream`/`LongStream`/`DoubleStream` avoid boxing and add `sum`, `average`, `range`:

```java
IntStream.rangeClosed(1, 5).sum();                    // 15 — Python's sum(range(1,6))
people.stream().mapToInt(Person::getAge).average();   // OptionalDouble
```

## Code Examples

### From loops to streams

```java
import java.util.List;

public class LoopVsStream {
    public static void main(String[] args) {
        List<String> words = List.of("stream", "of", "consciousness", "in", "java");

        // OLD WAY: accumulate with a loop
        int count = 0;
        for (String w : words) {
            if (w.length() > 2) count++;
        }
        System.out.println(count);                            // 3

        // STREAM WAY: say WHAT, not HOW
        long count2 = words.stream()
                .filter(w -> w.length() > 2)
                .count();
        System.out.println(count2);                           // 3

        // Transform + sort + collect
        List<String> shouty = words.stream()
                .filter(w -> w.length() > 2)
                .map(String::toUpperCase)
                .sorted()
                .toList();
        System.out.println(shouty);          // [CONSCIOUSNESS, JAVA, STREAM]
    }
}
```

### Working with objects: the full toolkit

```java
import java.util.*;
import java.util.stream.Collectors;

record Employee(String name, String dept, double salary) { }

public class StreamToolkit {
    public static void main(String[] args) {
        List<Employee> staff = List.of(
            new Employee("Ada", "eng", 9500),
            new Employee("Bob", "sales", 6100),
            new Employee("Cy", "eng", 8200),
            new Employee("Dee", "sales", 7000),
            new Employee("Eli", "hr", 5800)
        );

        // Sorting with a Comparator built from method references
        List<Employee> bySalaryDesc = staff.stream()
                .sorted(Comparator.comparingDouble(Employee::salary).reversed())
                .toList();
        System.out.println(bySalaryDesc.get(0).name());       // Ada

        // Aggregation
        double payroll = staff.stream()
                .mapToDouble(Employee::salary)
                .sum();
        System.out.println("payroll: " + payroll);            // 36600.0

        // Matching
        boolean anyoneUnder6k = staff.stream()
                .anyMatch(e -> e.salary() < 6000);
        System.out.println(anyoneUnder6k);                     // true

        // Grouping — THE interview favorite
        Map<String, List<Employee>> byDept = staff.stream()
                .collect(Collectors.groupingBy(Employee::dept));
        System.out.println(byDept.keySet());                   // [eng, sales, hr] (order varies)

        // Grouping with downstream aggregation
        Map<String, Double> avgByDept = staff.stream()
                .collect(Collectors.groupingBy(
                        Employee::dept,
                        Collectors.averagingDouble(Employee::salary)));
        System.out.println(avgByDept);         // {eng=8850.0, sales=6550.0, hr=5800.0}

        // Joining strings
        String roster = staff.stream()
                .map(Employee::name)
                .collect(Collectors.joining(", ", "Team: ", "."));
        System.out.println(roster);            // Team: Ada, Bob, Cy, Dee, Eli.

        // Optional handling done right
        Optional<Employee> topEarner = staff.stream()
                .max(Comparator.comparingDouble(Employee::salary));
        topEarner.ifPresent(e -> System.out.println("top: " + e.name()));
    }
}
```

### Lambdas beyond streams

```java
import java.util.*;
import java.util.function.*;

public class LambdaUses {
    public static void main(String[] args) {
        List<String> tasks = new ArrayList<>(List.of("email", "", "deploy", " "));

        tasks.removeIf(String::isBlank);              // Predicate — Ch. 12's cliffhanger resolved
        tasks.replaceAll(String::toUpperCase);        // UnaryOperator
        tasks.forEach(System.out::println);           // Consumer
        tasks.sort(Comparator.naturalOrder());        // Comparator

        Map<String, Integer> counts = new HashMap<>();
        counts.merge("clicks", 1, Integer::sum);      // BinaryOperator
        counts.computeIfAbsent("views", k -> 0);      // Function

        // Storing behavior in variables and passing it around
        Function<Double, Double> vat = price -> price * 1.25;
        Function<Double, Double> rounded = price -> Math.round(price * 100) / 100.0;
        Function<Double, Double> checkout = vat.andThen(rounded);   // composition!
        System.out.println(checkout.apply(19.99));                  // 24.99
    }
}
```

## Common Pitfalls

### 1. Forgetting the terminal operation

```java
words.stream().filter(w -> w.length() > 2);    // ❌ does NOTHING — lazy pipeline never ran
words.stream().filter(w -> w.length() > 2).count();   // ✅ terminal op executes it
```

### 2. Reusing a consumed stream

```java
var s = words.stream();
s.count();
s.findFirst();     // ❌ IllegalStateException: stream has already been operated upon or closed
```

Build a new stream from the collection each time — streams are cheap.

### 3. Mutating external state from inside a stream

```java
List<String> out = new ArrayList<>();
words.stream().map(String::toUpperCase).forEach(out::add);   // ❌ works, but defeats the model
List<String> out = words.stream().map(String::toUpperCase).toList();   // ✅ collect, don't leak
```

### 4. Modifying (effectively) captured variables

```java
int total = 0;
prices.forEach(p -> total += p);   // ❌ variables captured by lambdas must be effectively final
double total = prices.stream().mapToDouble(Double::doubleValue).sum();  // ✅ use reduction
```

### 5. `Optional.get()` without checking

```java
list.stream().findFirst().get();               // ❌ NoSuchElementException on empty
list.stream().findFirst().orElse(fallback);    // ✅ orElse / orElseThrow / ifPresent
```

### 6. Streaming when a loop is clearer

A three-branch stateful process with early exit and index math shoehorned into `reduce` is worse than the loop it replaced. Streams shine on filter/map/aggregate shapes. Readability decides — a rule stated in most companies' style guides.

## Practice Exercises

1. **Rewrite with streams.** Take Chapter 7's exercises — max, average, count-above-average over an `int[]` — and redo them with `IntStream`/`Arrays.stream(arr)` in as few lines as each allows. Compare line counts and readability in a comment.
2. **Product pipeline.** Given `record Product(String name, String category, double price, int stock)` and 8+ sample products, produce with one stream each: (a) names of in-stock products under 50, alphabetically; (b) total inventory value (`price * stock` summed); (c) the most expensive product per category (`groupingBy` + a downstream max — research `Collectors.maxBy`); (d) a single `String` of all category names, distinct, joined with " | ".
3. **Comparator combinators.** Sort a list of `Employee` by department, then salary descending within department, then name — one `Comparator` chain using `comparing`, `thenComparing`, and `reversed()`. Print it. Then sort *null-safely* when some names may be null (research `Comparator.nullsLast`).
4. **Functional interfaces by hand.** Without streams: write a method `static <T> List<T> keep(List<T> items, Predicate<T> rule)` implementing filter with a plain loop, and `static <T, R> List<R> transform(List<T> items, Function<T, R> f)` implementing map. Use them with three different lambdas each. You've just re-derived the stream core.
5. **FizzBuzz, final form.** `IntStream.rangeClosed(1, 100)` mapped to the correct FizzBuzz strings and printed — no loops, no if statements (a nested ternary or a `mapToObj` with a switch expression is fair game). Compare against your Chapter 4 version and reflect: which would you rather maintain, and why? (Comment.)
