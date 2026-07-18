# Chapter 10: Interfaces and Abstract Classes

## Overview

Chapter 9's `Shape.area()` returning `0.0` was a smell: a "generic shape" has no meaningful area, and nothing *forced* subclasses to override it. Java has two tools for saying "here's a contract; concrete classes must fill in the blanks":

- **Abstract classes** — partial implementations that cannot be instantiated.
- **Interfaces** — pure contracts a class promises to fulfill, and Java's answer to multiple inheritance.

Interfaces especially are the backbone of professional Java: `List`, `Comparable`, `Runnable` are all interfaces, and "program to the interface" is the single most-repeated design maxim in enterprise code.

## Definitions & Explanations

### Abstract classes

```java
public abstract class Shape {
    public abstract double area();     // no body — subclasses MUST implement
    public String describe() {         // regular method — inherited as usual
        return "a shape with area " + area();
    }
}
```

- `abstract class` cannot be instantiated: `new Shape()` is a compile error.
- An `abstract` method has no body; every concrete subclass **must** override it (or itself be abstract).
- Otherwise it's a normal class: fields, constructors (called via `super`), implemented methods, the works.
- Use one when subclasses share **state and code** but the base concept alone is incomplete.

Python analogue: `abc.ABC` with `@abstractmethod` — but Java enforces it at compile time, not first-call time.

### Interfaces

```java
public interface Payable {
    double amountDue();                          // implicitly public abstract
}

public class Invoice implements Payable {
    private final double total;
    public Invoice(double total) { this.total = total; }

    @Override
    public double amountDue() { return total; }  // fulfill the contract
}
```

- An interface declares *what* a class can do, with no per-object state (no instance fields; only `public static final` constants).
- A class `implements` an interface and must provide bodies for all its abstract methods.
- **A class can implement many interfaces** (comma-separated) while extending only one class — this is how Java does multiple inheritance of *type*.
- An interface is a type: `Payable p = new Invoice(500);` — full polymorphism, exactly like Chapter 9.
- Since Java 8, interfaces may also carry `default` methods (with bodies, inherited but overridable) and `static` helper methods. This keeps interfaces evolvable without breaking implementors.

TS folks: like TypeScript `interface`, but checked *nominally* (you must declare `implements`), not structurally.

### Choosing between them

| Question | Abstract class | Interface |
|----------|----------------|-----------|
| Shared fields/state? | ✅ yes | ❌ no instance state |
| Shared method code? | ✅ yes | ✅ default methods (limited) |
| How many can a class take? | one (`extends`) | many (`implements`) |
| Constructors? | ✅ | ❌ |
| Relationship it models | "is-a, with shared machinery" | "can-do / behaves-like" |

Rules of thumb: model *capabilities* (`Comparable`, `Printable`, `Payable`) as interfaces; use an abstract class only when subclasses genuinely share code and fields. When in doubt, start with an interface — you can add an abstract base class later.

### Two interfaces you must know

**`Comparable<T>`** — "objects of this class have a natural order" (used by `Arrays.sort`, `Collections.sort`):

```java
public class Player implements Comparable<Player> {
    private final int score;
    // negative if this < other, 0 if equal, positive if this > other
    @Override
    public int compareTo(Player other) {
        return Integer.compare(this.score, other.score);
    }
}
```

**`Runnable`** — "has a `void run()`" — the classic single-method callback, and a preview of lambdas (Chapter 15). Interfaces with exactly one abstract method are called **functional interfaces**.

## Code Examples

### Abstract base + interface together

```java
// Payable.java — a capability
public interface Payable {
    double amountDue();

    // default method: free behavior for all implementors, still overridable
    default String paymentSummary() {
        return String.format("Amount due: %.2f", amountDue());
    }
}
```

```java
// StaffMember.java — shared state & code for people on staff
public abstract class StaffMember implements Payable {
    private final String name;

    protected StaffMember(String name) {          // constructor for subclasses' use
        this.name = name;
    }

    public String getName() { return name; }

    // amountDue() is NOT implemented here — still abstract, pushed to children

    @Override
    public String toString() {
        return name + " -> " + paymentSummary();
    }
}
```

```java
// SalariedStaff.java
public class SalariedStaff extends StaffMember {
    private final double monthlySalary;

    public SalariedStaff(String name, double monthlySalary) {
        super(name);
        this.monthlySalary = monthlySalary;
    }

    @Override
    public double amountDue() { return monthlySalary; }
}
```

```java
// HourlyStaff.java
public class HourlyStaff extends StaffMember {
    private final double rate;
    private int hoursWorked;

    public HourlyStaff(String name, double rate) {
        super(name);
        this.rate = rate;
    }

    public void logHours(int hours) { hoursWorked += hours; }

    @Override
    public double amountDue() { return rate * hoursWorked; }
}
```

```java
// Invoice.java — Payable but NOT a StaffMember: interfaces cross hierarchies!
public class Invoice implements Payable {
    private final String vendor;
    private final double total;

    public Invoice(String vendor, double total) {
        this.vendor = vendor;
        this.total = total;
    }

    @Override
    public double amountDue() { return total; }

    @Override
    public String toString() { return "Invoice from " + vendor + ": " + paymentSummary(); }
}
```

```java
// PayrollDemo.java — the polymorphic payoff
public class PayrollDemo {
    public static void main(String[] args) {
        HourlyStaff dev = new HourlyStaff("Devon", 75.0);
        dev.logHours(120);

        // One array of the INTERFACE type mixes unrelated classes
        Payable[] toPay = {
            new SalariedStaff("Sam", 5200.0),
            dev,
            new Invoice("Cloud Hosting Inc", 89.99)
        };

        double total = 0;
        for (Payable p : toPay) {
            System.out.println(p);
            total += p.amountDue();       // each class's own implementation runs
        }
        System.out.printf("Total outgoing: %.2f%n", total);

        // new StaffMember("X");   // ❌ compile error: StaffMember is abstract
    }
}
```

### Comparable in practice

```java
import java.util.Arrays;

public class Player implements Comparable<Player> {
    private final String name;
    private final int score;

    public Player(String name, int score) {
        this.name = name;
        this.score = score;
    }

    @Override
    public int compareTo(Player other) {
        return Integer.compare(this.score, other.score);  // ascending by score
    }

    @Override
    public String toString() { return name + ":" + score; }

    public static void main(String[] args) {
        Player[] board = {
            new Player("ada", 310), new Player("bob", 150), new Player("cy", 275)
        };
        Arrays.sort(board);                        // works BECAUSE of Comparable
        System.out.println(Arrays.toString(board)); // [bob:150, cy:275, ada:310]
    }
}
```

## Common Pitfalls

### 1. Instantiating an abstract class

```java
Shape s = new Shape();          // ❌ Shape is abstract; cannot be instantiated
Shape s = new Circle(2.0);      // ✅ abstract types hold concrete objects
```

### 2. Forgetting to implement an interface method

```java
public class Invoice implements Payable { }
// ❌ "Invoice is not abstract and does not override abstract method amountDue()"
```

The fix is exactly what the message says. IntelliJ's quick-fix (Alt+Enter → Implement methods) generates the stubs.

### 3. Adding instance state to an interface

```java
public interface Payable {
    double total = 0;      // ❌ this is implicitly public static FINAL — a constant,
}                          //    shared and unchangeable, not a per-object field
```

Per-object state belongs in classes (or an abstract base class).

### 4. `implements` vs `extends` mix-ups

```java
class A extends SomeInterface { }     // ❌ interfaces are implemented, not extended (by classes)
class B implements SomeClass { }      // ❌ classes are extended
interface C extends OtherInterface { } // ✅ interfaces DO extend each other
```

### 5. compareTo subtraction overflow

```java
public int compareTo(Player o) { return this.score - o.score; }   // ❌ overflows on extreme values
public int compareTo(Player o) { return Integer.compare(score, o.score); }  // ✅
```

### 6. Fat interfaces

An interface with 15 methods forces every implementor to write 15 bodies. Keep interfaces small and focused (often 1–3 methods) — the Interface Segregation Principle. Notice the JDK's own style: `Comparable` has one method.

## Practice Exercises

1. **Abstract Shape, done right.** Rebuild Chapter 9's hierarchy with `Shape` abstract: `abstract double area()` and `abstract double perimeter()`, plus a concrete `String report()` that uses both. Implement `Circle`, `Rectangle`, and `Triangle` (Heron's formula). Verify `new Shape(...)` no longer compiles.
2. **Capability interfaces.** Define `Flyer` (`fly()`), `Swimmer` (`swim()`). Create `Duck` (both), `Penguin` (swims only), `Eagle` (flies only), each extending a common `Bird` class holding the name. Write `static void airShow(Flyer[] flyers)` and pass it only the fliers. What happens if you try to slip a Penguin in? (Record the compile error.)
3. **Comparable Products.** A `Product` (name, price) implementing `Comparable<Product>` by price. Sort an array and print it. Then change the natural order to name (alphabetical) — one line — and observe the new sort. Bonus thought: what if some callers want price order and others name order? (One sentence; Chapter 15's `Comparator` is the answer.)
4. **Default method evolution.** Add a default method `boolean isOverdue()` to `Payable` returning `false`, then override it in `Invoice` only. Confirm staff members inherit the default while invoices use their own. Explain in a comment why default methods exist (hint: what breaks if you add a new *abstract* method to a published interface?).
5. **Design call.** For each of these, decide interface, abstract class, or plain class, and justify in one sentence each: (a) `Exportable` — things that can render themselves as CSV; (b) `AbstractRepository` — database access classes sharing connection-handling code; (c) `Money` — an amount plus currency; (d) `Clickable` — UI elements responding to clicks. No code required — this judgment *is* the skill.
