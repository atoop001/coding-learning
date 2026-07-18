# Chapter 9: Inheritance and Polymorphism

## Overview

Inheritance lets one class build on another: a `SavingsAccount` **is a** `BankAccount` with extra rules. Polymorphism — "many forms" — lets code written against the parent type work seamlessly with any child: a loop over `Animal[]` calls each animal's *own* `makeSound()`. Together they're the heart of classic OOP design, they're everywhere in Java frameworks (Spring, Android), and they're guaranteed interview material.

Python and JS both have class inheritance, so the concept isn't new — but Java's static typing makes the rules stricter and more explicit: `extends`, `super`, `@Override`, and compile-time checks on what you're allowed to call.

## Definitions & Explanations

### `extends` — creating a subclass

```java
public class Animal { ... }
public class Dog extends Animal { ... }   // Dog inherits Animal's fields & methods
```

- `Dog` gets everything non-private from `Animal` and can add its own members.
- Java has **single inheritance**: one parent only (no Python-style multiple inheritance; interfaces fill that gap in Chapter 10).
- Every class implicitly extends `java.lang.Object`, which is where `toString()`, `equals()`, and `hashCode()` come from.
- The vocabulary: *superclass/parent/base* vs *subclass/child/derived*. "Dog **is-a** Animal" is the design test — if "is-a" sounds wrong, don't inherit (a `Car` is not an `Engine`; it *has* one — use a field).

### `super` — talking to the parent

- `super(...)` calls the parent constructor. **Constructors are not inherited**; every subclass constructor must ensure the parent's constructor runs, either explicitly (`super(name)`) or implicitly (Java inserts `super()` if the parent has a no-arg constructor). It must be the first statement.
- `super.method()` calls the parent's version of an overridden method (like Python's `super().method()`).

### Overriding

A subclass can **override** an inherited method — replace its behavior — by redeclaring it with the same signature:

```java
@Override
public String makeSound() { return "Woof"; }
```

Always write `@Override`. It's optional but priceless: if you typo the name or signature, the compiler errors instead of silently creating an unrelated new method.

Rules: same name and parameters; return type same (or a subtype); can't *reduce* visibility (`public` can't become `private`); can't override `final` methods.

**Overriding ≠ overloading.** Overloading (Chapter 5) = same name, different parameters, same class. Overriding = same signature, subclass replaces parent behavior.

### Polymorphism — the payoff

A variable of the parent type can hold any subclass object:

```java
Animal a = new Dog("Rex");     // legal: a Dog IS an Animal
a.makeSound();                  // "Woof" — DOG's version runs!
```

Which method body runs is decided **at runtime by the object's actual class** (dynamic dispatch), not by the variable's declared type. This is what lets one method accept `Animal` and work for every animal ever written — including ones that don't exist yet.

The flip side: through an `Animal` variable you can only call methods *declared on `Animal`*. `a.fetch()` won't compile even if the object is really a `Dog` — the compiler only trusts the declared type.

### Casting and `instanceof`

When you must access subclass-specific members, check-and-cast:

```java
if (a instanceof Dog) {
    Dog d = (Dog) a;      // classic downcast
    d.fetch();
}
// Modern pattern matching (Java 16+): test + cast + name in one step
if (a instanceof Dog d) {
    d.fetch();
}
```

A wrong cast compiles but throws `ClassCastException` at runtime — always guard with `instanceof`. (If you find yourself doing this a lot, your design probably wants polymorphism or interfaces instead.)

### `final` and `protected`

- `final class` — cannot be extended (e.g., `String`).
- `final` method — cannot be overridden.
- `protected` member — visible to subclasses (and same package). Sits between `private` and `public`; full story in Chapter 11.

## Code Examples

### A shape hierarchy

```java
// Shape.java — the general concept
public class Shape {
    private final String name;

    public Shape(String name) {
        this.name = name;
    }

    public String getName() { return name; }

    // A default implementation subclasses are expected to override
    public double area() {
        return 0.0;
    }

    @Override
    public String toString() {
        return String.format("%s(area=%.2f)", name, area());
    }
}
```

```java
// Circle.java
public class Circle extends Shape {
    private final double radius;

    public Circle(double radius) {
        super("Circle");              // MUST run the parent constructor first
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
```

```java
// Rectangle.java
public class Rectangle extends Shape {
    private final double width, height;

    public Rectangle(double width, double height) {
        super("Rectangle");
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() {
        return width * height;
    }

    // Subclass-specific extra method — not on Shape
    public boolean isSquare() {
        return width == height;
    }
}
```

### Polymorphism in action

```java
public class ShapeDemo {
    public static void main(String[] args) {
        // One array of the PARENT type holds mixed children
        Shape[] shapes = {
            new Circle(1.0),
            new Rectangle(3.0, 4.0),
            new Rectangle(2.0, 2.0)
        };

        double total = 0;
        for (Shape s : shapes) {
            System.out.println(s);       // each object's own toString/area runs
            total += s.area();           // dynamic dispatch: right area() every time
        }
        System.out.printf("Total area: %.2f%n", total);

        // Accessing subclass-only members needs a guarded cast
        for (Shape s : shapes) {
            if (s instanceof Rectangle r && r.isSquare()) {
                System.out.println("Found a square!");
            }
        }
    }
}
```

### `super.method()` — extend, don't replace

```java
public class LoggedAccount extends BankAccount {   // BankAccount from Chapter 8
    public LoggedAccount(String owner, double opening) {
        super(owner, opening);
    }

    @Override
    public void deposit(double amount) {
        super.deposit(amount);          // reuse the parent's validation & logic...
        System.out.println("LOG: deposited " + amount);   // ...then add behavior
    }
}
```

## Common Pitfalls

### 1. Forgetting the parent constructor call

```java
public class Circle extends Shape {
    public Circle(double r) {
        this.radius = r;     // ❌ compile error if Shape has no no-arg constructor:
    }                        //    "constructor Shape in class Shape cannot be applied"
}
// ✅ first line must be super("Circle"); when the parent requires arguments
```

### 2. Typo-overriding without `@Override`

```java
public String tostring() { ... }        // ❌ new unrelated method; println still prints garbage
@Override
public String tostring() { ... }        // ✅ now it's a COMPILE ERROR — the typo is caught
@Override
public String toString() { ... }        // ✅✅ correct
```

This is why `@Override` is non-negotiable.

### 3. Calling child-only methods through a parent variable

```java
Animal a = new Dog("Rex");
a.fetch();                    // ❌ cannot find symbol — Animal has no fetch()
((Dog) a).fetch();            // works, but smells; prefer instanceof guard or redesign
```

### 4. Blind downcasting

```java
Shape s = new Circle(1.0);
Rectangle r = (Rectangle) s;  // ❌ compiles; ClassCastException at runtime
if (s instanceof Rectangle r2) { ... }   // ✅ guard first
```

### 5. Fields don't polymorphize

```java
class Parent { String label = "parent"; }
class Child extends Parent { String label = "child"; }
Parent p = new Child();
System.out.println(p.label);   // "parent"! — fields resolve by DECLARED type
```

Only *methods* dispatch dynamically. Moral: keep fields private; expose via (overridable) methods.

### 6. Inheriting when composition fits better

```java
class ArrayListWithLogging extends ArrayList<String> { ... }   // ❌ fragile — you inherit 30+ methods you didn't design for
class LoggingList { private final ArrayList<String> items;  }  // ✅ has-a: wrap and expose only what you mean
```

"Favor composition over inheritance" is a mantra you'll hear in every code review. Inherit for genuine is-a relationships with shared behavior; wrap otherwise.

## Practice Exercises

1. **Vehicle hierarchy.** `Vehicle` (fields: make, model, year; method `describe()`), with subclasses `Car` (adds `numDoors`), `Motorcycle` (adds `hasSidecar`), and `Truck` (adds `payloadKg`, overrides `describe()` to append payload). Put one of each in a `Vehicle[]` and print all descriptions from one loop.
2. **Employee pay.** `Employee` (name, base monthly salary, method `monthlyPay()`), `Manager extends Employee` (adds bonus, overrides `monthlyPay()` using `super.monthlyPay() + bonus`), `Contractor extends Employee` (hourly rate × hours, ignores base). Compute a total payroll from an `Employee[]`. Which class made you bend the model, and what does that suggest? (Comment.)
3. **Override equals-of-behavior.** Add a `SavingsAccount extends BankAccount` whose `withdraw` refuses to let balance drop below a minimum reserve of 100. Reuse the parent's logic where possible with `super`. Demonstrate one plain account and one savings account behaving differently through a `BankAccount`-typed variable.
4. **instanceof census.** Given a mixed `Shape[]`, write `static int countSquares(Shape[] shapes)` using pattern-matching `instanceof`. Then explain in a comment why adding a `boolean isSquare()` to `Shape` itself might be a *worse* design (hint: does "is square" mean anything for a Circle?).
5. **Break it to learn it.** In your Vehicle hierarchy: (a) remove `@Override` from one override and misspell the method — observe what happens and write it down; (b) make `describe()` `final` in `Vehicle` and try overriding — record the error; (c) try `extends` from two classes at once — record the error. Restore everything after.
