# Chapter 9: Inheritance & Polymorphism

## Overview

Inheritance lets one class build on another: a `Dog` **is an** `Animal`, so it gets everything `Animal` has and can add or change behavior. Polymorphism is the payoff: code written against the base type (`Animal`) automatically works with every subclass, each behaving its own way. C# is stricter than Python here — methods must be explicitly marked `virtual` to be overridable, and `override` to override — which makes intent visible and prevents accidents.

## Definitions & Explanations

### Basic inheritance

```csharp
class Animal                              // base class (parent / superclass)
{
    public string Name { get; set; } = "";
    public void Eat() => Console.WriteLine($"{Name} is eating.");
}

class Dog : Animal                        // Dog derives from Animal (Python: class Dog(Animal))
{
    public void Fetch() => Console.WriteLine($"{Name} fetches the ball!");
}

var rex = new Dog { Name = "Rex" };
rex.Eat();      // inherited from Animal
rex.Fetch();    // Dog's own method
```

- A class can inherit from **one** base class only (no multiple inheritance — interfaces fill that role, Chapter 10).
- Everything `public`/`protected` is inherited; `private` members exist in the subclass's objects but aren't directly accessible from subclass code.

### virtual / override — opting in to polymorphism

In Python every method is overridable. In C#, the base class must say `virtual` ("subclasses may replace this"), and the subclass must say `override` ("I am replacing it deliberately"):

```csharp
class Animal
{
    public string Name { get; set; } = "";
    public virtual string Speak() => "...";           // virtual: replaceable
}

class Dog : Animal
{
    public override string Speak() => "Woof!";        // override: replacement
}

class Cat : Animal
{
    public override string Speak() => "Meow.";
}
```

### Polymorphism in action

A base-type variable can hold any derived object, and **virtual calls dispatch to the actual runtime type**:

```csharp
List<Animal> zoo = new() { new Dog { Name = "Rex" }, new Cat { Name = "Mia" } };

foreach (Animal a in zoo)
    Console.WriteLine($"{a.Name}: {a.Speak()}");
// Rex: Woof!
// Mia: Meow.       <- each object used ITS OWN Speak, through an Animal variable
```

This is the whole point: the loop doesn't know or care which species it's holding.

### base — calling up the chain

```csharp
class Puppy : Dog
{
    public override string Speak() => base.Speak() + " (but squeaky)";   // like super()
}
```

### Constructors and inheritance

Constructors are **not** inherited. A subclass constructor must ensure the base constructor runs — via `base(...)`:

```csharp
class Animal
{
    public string Name { get; }
    public Animal(string name) => Name = name;
}

class Dog : Animal
{
    public string Breed { get; }

    public Dog(string name, string breed) : base(name)   // call Animal's constructor
    {
        Breed = breed;
    }
}
```

If the base class has no parameterless constructor, `: base(...)` is mandatory — a common compile error for newcomers.

### Type checks and casting

```csharp
Animal a = new Dog("Rex", "Lab");

if (a is Dog d)                 // pattern matching: test AND cast in one step
    d.Fetch();                  // 'd' is a Dog-typed view of the same object

Dog? maybe = a as Dog;          // 'as': cast or null (never throws)
Dog forced = (Dog)a;            // hard cast: InvalidCastException if wrong
```

Prefer `is` patterns. But note: *frequent* type-checking is a design smell — usually a virtual method should be doing that work.

### sealed and object

- `sealed class` — cannot be inherited from. `sealed override` — stops further overriding.
- Every class ultimately inherits from `object`, which supplies `ToString()`, `Equals()`, `GetHashCode()` — all virtual, all overridable.

## Code Examples

### A complete shapes hierarchy

```csharp
// Shapes — the canonical polymorphism example, done properly
class Shape
{
    public string Name { get; }
    protected Shape(string name) => Name = name;    // protected: only subclasses call it

    public virtual double Area() => 0;

    public override string ToString() => $"{Name}: area {Area():F2}";
}

class Circle : Shape
{
    public double Radius { get; }
    public Circle(double radius) : base("Circle") => Radius = radius;
    public override double Area() => Math.PI * Radius * Radius;
}

class Rectangle : Shape
{
    public double Width { get; }
    public double Height { get; }
    public Rectangle(double w, double h) : base("Rectangle") { Width = w; Height = h; }
    public override double Area() => Width * Height;
}

class Square : Rectangle                       // a Square IS a Rectangle
{
    public Square(double side) : base(side, side) { }
}

class Program
{
    static void Main()
    {
        List<Shape> shapes = new()
        {
            new Circle(1),
            new Rectangle(3, 4),
            new Square(2),
        };

        double total = 0;
        foreach (Shape s in shapes)
        {
            Console.WriteLine(s);        // uses overridden ToString -> overridden Area
            total += s.Area();           // polymorphic dispatch
        }
        Console.WriteLine($"Total area: {total:F2}");

        // Find the largest — works no matter what shapes are added later
        Shape biggest = shapes[0];
        foreach (Shape s in shapes)
            if (s.Area() > biggest.Area()) biggest = s;
        Console.WriteLine($"Biggest: {biggest.Name}");
    }
}
```

## Common Pitfalls

**1. Forgetting `virtual`/`override` — method hiding.** If you redeclare a method without `override`, you *hide* it, and polymorphism silently breaks:

```csharp
class Dog : Animal
{
    public string Speak() => "Woof!";      // ⚠ CS0108 warning: hides Animal.Speak
}
Animal a = new Dog();
a.Speak();                                  // "..." — the BASE version ran!
```

Fix: `virtual` on the base method, `override` on the derived one. Never ignore the CS0108 warning.

**2. Missing base constructor call.**

```csharp
class Dog : Animal      // Animal has only Animal(string name)
{
    public Dog() { }    // ❌ CS7036: no argument for 'name'
    public Dog(string name) : base(name) { }   // ✅
}
```

**3. Overusing inheritance.** Inheritance means *is-a*, permanently. A `Car` is not an `Engine` (it *has* one — use a property: composition). Deep hierarchies (4+ levels) are usually a mistake; prefer shallow trees plus interfaces.

**4. Downcasting on faith.**

```csharp
Dog d = (Dog)someAnimal;         // ❌ crashes if it's actually a Cat
if (someAnimal is Dog d2) { }    // ✅ test first
```

**5. Calling virtual methods from a constructor.** The override runs before the subclass's constructor body has initialized its fields — subtle bugs. Avoid it.

## Practice Exercises

1. Build an `Employee` base class (Name, virtual `MonthlyPay()`) with `SalariedEmployee` (annual salary / 12) and `HourlyEmployee` (rate × hours) subclasses. Put a mixed list through one payroll loop that prints each pay and the total.
2. Add a `Manager : SalariedEmployee` with a bonus, whose `MonthlyPay()` calls `base.MonthlyPay()` and adds the bonus. Verify the whole hierarchy still works through the same payroll loop unchanged.
3. Create a `Vehicle` hierarchy (`Car`, `Motorcycle`, `Truck`) where a virtual `Describe()` builds on the base description with `base.Describe()`. Each constructor must chain with `: base(...)`.
4. Demonstrate the method-hiding bug deliberately: write a base/derived pair *without* `virtual`/`override`, show through a base-typed variable that the wrong method runs, then fix it. Keep both versions in comments with an explanation.
5. Design question (answer in comments, then implement the better option): you need `Bird`, `Penguin`, and `Plane` — all can `Describe()`, only some can fly. Why does putting `Fly()` in a common base class fail? Sketch two alternatives. (You'll meet the clean answer — interfaces — next chapter.)
