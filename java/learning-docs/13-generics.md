# Chapter 13: Generics

## Overview

You've been *using* generics since `List<String>` — now you'll understand and *write* them. Generics let classes and methods be parameterized by type: write a `Box<T>` once, get a type-safe `Box<String>`, `Box<Integer>`, `Box<Invoice>` for free, with the compiler guaranteeing you never put a `String` where an `Integer` belongs. Coming from Python (no static types) or plain JS, this is new machinery; if you know TypeScript generics, Java's will look familiar, minus structural typing and with a couple of JVM-specific quirks (erasure).

## Definitions & Explanations

### The problem generics solve

Before generics (pre-2004 Java), collections held bare `Object`s:

```java
List box = new ArrayList();       // "raw type" — still legal, never write it
box.add("hello");
box.add(42);                       // nothing stops mixed junk
String s = (String) box.get(1);    // compiles... ClassCastException at RUNTIME
```

With generics, the mistake is caught **at compile time**:

```java
List<String> box = new ArrayList<>();
box.add("hello");
box.add(42);                       // ❌ compile error — incompatible types
String s = box.get(0);             // no cast needed; the compiler knows
```

### Generic classes

Declare type parameters in angle brackets after the class name; use them like ordinary types inside:

```java
public class Box<T> {              // T = "some type, decided by the user of Box"
    private T content;

    public void put(T item) { this.content = item; }
    public T get() { return content; }
    public boolean isEmpty() { return content == null; }
}

Box<String> b = new Box<>();       // T = String for this instance
b.put("hi");
String s = b.get();                // typed! no cast
```

Conventions: `T` (type), `E` (element), `K`/`V` (key/value), `R` (result). They're just names — `Box<Thing>` would compile — but everyone uses the conventions.

Multiple parameters work too: `public class Pair<A, B> { ... }`.

### Generic methods

A method can have its own type parameter, declared **before the return type**:

```java
public static <T> T firstOrDefault(List<T> list, T fallback) {
    return list.isEmpty() ? fallback : list.get(0);
}

String s = firstOrDefault(names, "nobody");   // T inferred as String — no <> needed at call site
```

The `<T>` up front says "this method works for any type T"; the compiler infers T from the arguments.

### Bounded type parameters

Constrain what T can be with `extends` (which here means "is or extends/implements"):

```java
// T must be Comparable to itself, or max() couldn't call compareTo
public static <T extends Comparable<T>> T max(List<T> list) {
    T best = list.get(0);
    for (T item : list) {
        if (item.compareTo(best) > 0) best = item;
    }
    return best;
}
```

Inside the method you may use `Comparable`'s methods on T. `max(List<Circle>)` won't compile unless `Circle implements Comparable<Circle>` — the constraint is enforced.

### Wildcards: `?`, `? extends`, `? super`

Here's the counterintuitive part: `List<Dog>` is **not** a subtype of `List<Animal>` (if it were, you could add a Cat to it through the Animal view). Wildcards express flexible relationships:

- `List<?>` — a list of *something unknown*. You can read `Object`s and check size; you can't add (except null).
- `List<? extends Animal>` — a list of Animal *or any subtype* (maybe `List<Dog>`). Safe to **read** Animals **out**; can't add.
- `List<? super Dog>` — a list of Dog *or any supertype* (maybe `List<Animal>`, `List<Object>`). Safe to **add** Dogs **in**; reads come out as `Object`.

Mnemonic — **PECS**: *Producer Extends, Consumer Super*. A parameter you only read from → `? extends`; one you only write into → `? super`. When a method both reads and writes, use an exact type parameter `<T>` instead.

```java
static double totalArea(List<? extends Shape> shapes) {   // accepts List<Circle>, List<Rectangle>...
    double sum = 0;
    for (Shape s : shapes) sum += s.area();
    return sum;
}
```

### Type erasure (why some things don't work)

Generics exist only at compile time; the JVM erases them (a `List<String>` is just a `List` at runtime — unlike C#). Consequences you'll hit:

- `new T()` and `new T[10]` — illegal (T unknown at runtime).
- `obj instanceof List<String>` — illegal; only `instanceof List<?>`.
- No primitives as type arguments: `List<int>` ❌ → `List<Integer>` ✅ (autoboxing bridges the gap).
- Two overloads differing only in type argument (`f(List<String>)` vs `f(List<Integer>)`) — clash.

You don't need deep erasure knowledge to *use* generics — just recognize these limits when the compiler complains.

## Code Examples

### A generic Pair and a generic method together

```java
public class Pair<A, B> {
    private final A first;
    private final B second;

    public Pair(A first, B second) {
        this.first = first;
        this.second = second;
    }

    public A getFirst() { return first; }
    public B getSecond() { return second; }

    // A generic STATIC method on a generic class declares its own parameters
    public static <A, B> Pair<B, A> swap(Pair<A, B> p) {
        return new Pair<>(p.getSecond(), p.getFirst());
    }

    @Override
    public String toString() { return "(" + first + ", " + second + ")"; }

    public static void main(String[] args) {
        Pair<String, Integer> entry = new Pair<>("apples", 12);
        System.out.println(entry);                  // (apples, 12)
        Pair<Integer, String> flipped = swap(entry);
        System.out.println(flipped);                // (12, apples)

        // Type safety in action:
        // Integer x = entry.getFirst();            // ❌ compile error — it's a String
        String name = entry.getFirst();             // ✅
        System.out.println(name.toUpperCase());
    }
}
```

### A bounded generic in practice

```java
import java.util.List;

public class Stats {

    // Works for Integer, Double, String, your Comparable Player from Ch. 10 — anything Comparable
    public static <T extends Comparable<T>> T max(List<T> items) {
        if (items.isEmpty()) {
            throw new IllegalArgumentException("empty list has no max");
        }
        T best = items.get(0);
        for (T item : items) {
            if (item.compareTo(best) > 0) best = item;
        }
        return best;
    }

    public static void main(String[] args) {
        System.out.println(max(List.of(3, 9, 4)));                 // 9
        System.out.println(max(List.of("pear", "apple", "fig")));  // pear
        System.out.println(max(List.of(2.5, 2.7, 1.1)));           // 2.7
        // max(List.of(new Object()));   // ❌ Object isn't Comparable — caught at compile time
    }
}
```

### A typed stack — generics + your own data structure

```java
import java.util.ArrayList;
import java.util.List;

public class Stack<E> {
    private final List<E> items = new ArrayList<>();

    public void push(E item) { items.add(item); }

    public E pop() {
        if (isEmpty()) throw new IllegalStateException("stack is empty");
        return items.remove(items.size() - 1);
    }

    public E peek() {
        if (isEmpty()) throw new IllegalStateException("stack is empty");
        return items.get(items.size() - 1);
    }

    public boolean isEmpty() { return items.isEmpty(); }
    public int size() { return items.size(); }

    public static void main(String[] args) {
        Stack<String> history = new Stack<>();
        history.push("page1");
        history.push("page2");
        history.push("page3");
        System.out.println(history.pop());    // page3 — LIFO
        System.out.println(history.peek());   // page2
        System.out.println(history.size());   // 2
    }
}
```

## Common Pitfalls

### 1. Raw types

```java
List names = new ArrayList();          // ❌ raw type: compiler warnings, runtime casts, 2003 vibes
List<String> names = new ArrayList<>(); // ✅ always parameterize
```

### 2. Primitives as type arguments

```java
List<int> nums;          // ❌ won't compile
List<Integer> nums;      // ✅ — and mind the null-unboxing NPE from Ch. 12
```

### 3. Assuming `List<Dog>` is a `List<Animal>`

```java
List<Dog> dogs = new ArrayList<>();
List<Animal> animals = dogs;              // ❌ incompatible types — by design!
List<? extends Animal> animals = dogs;    // ✅ read-only Animal view
```

If the first line were legal, `animals.add(new Cat())` would poison the dog list.

### 4. Trying to instantiate a type parameter

```java
class Factory<T> {
    T create() { return new T(); }        // ❌ erasure: T doesn't exist at runtime
}
// ✅ standard workaround: accept a Supplier<T> (Ch. 15) or a Class<T> and call a passed-in creator
```

### 5. Forgetting the method-level `<T>` declaration

```java
public static T identity(T x) { return x; }        // ❌ "cannot find symbol T"
public static <T> T identity(T x) { return x; }    // ✅ declare it before the return type
```

### 6. Overusing wildcards

```java
static <T> void copy(List<? extends T> src, List<? super T> dst) { ... }  // library-grade
static void printAll(List<?> items) { ... }                                // fine
```

If wildcard soup is confusing you when *writing* code, use plain `<T>` parameters — wildcards earn their keep mostly in widely-reused library APIs. Reading them, however, is mandatory: the JDK docs are full of them.

## Practice Exercises

1. **Generic Box, extended.** Implement `Box<T>` with `put`, `get`, `isEmpty`, and `map`-free transfer method `static <T> void transfer(Box<T> from, Box<T> to)`. Show that `transfer(stringBox, integerBox)` fails to compile, and explain the error message in a comment.
2. **Pair utilities.** Using the `Pair<A, B>` class above, write generic static methods `static <A, B> List<A> firsts(List<Pair<A, B>> pairs)` and `static <T> Pair<T, T> minMax(List<T extends Comparable<T> — adjust the signature correctly!> items)` returning the smallest and largest element. (Getting the second signature to compile *is* the exercise.)
3. **Bounded sum.** Write `static double sumAll(List<? extends Number> nums)` that totals any list of numeric wrappers (`Number` has `doubleValue()`). Verify it accepts `List<Integer>`, `List<Double>`, and `List<Long>` — then explain in a comment why the parameter can't just be `List<Number>`.
4. **Generic Queue.** Build `Queue<E>` (FIFO: `enqueue`, `dequeue`, `peek`, `isEmpty`, `size`) backed by a `LinkedList<E>` or `ArrayList<E>`. Demonstrate it with two different element types. Then make it iterable-friendly by adding `List<E> toList()` returning a defensive copy — explain why returning the internal list directly would break encapsulation.
5. **PECS drill.** Write `static <T> void copyAll(List<? extends T> source, List<? super T> target)` that appends every source element to target. Build `List<Dog>`, `List<Animal>`, `List<Object>` and determine by experiment (and record as comments) exactly which combinations of source/target compile. Map each result back to the PECS mnemonic.
