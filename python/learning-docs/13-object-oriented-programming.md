# Chapter 13: Object-Oriented Programming

## Overview

You've been *using* objects all along: strings, lists, dicts, file handles, `Path`s — each bundles data with the methods that operate on it. **Object-oriented programming (OOP)** lets you define your own such bundles: a `BankAccount` that knows its balance and how to withdraw, a `Player` with a name and a score, a `Product` that knows how to display itself.

OOP is the organizing principle of most large Python codebases and frameworks — Flask views, Django models, pytest fixtures, exceptions (you subclassed `Exception` last chapter!). This chapter covers classes, instances, `__init__`, methods, attributes, inheritance, and the "dunder" methods that plug your classes into Python's syntax.

**Hints in practice** — Chapter 6 mentioned type hints as a preview; from here on, the track's own examples start wearing them on their main signatures (`self` stays unhinted — its type is always the class). They cost nothing at runtime and pay off fast on methods with several parameters.

## Definitions & Explanations

**Class** — a blueprint defining what data (attributes) and behavior (methods) a kind of object has. Defined with `class Name:` — names use `CapWords` by convention.

**Instance (object)** — a concrete thing built from the blueprint: `account = BankAccount("Ada", 100)`. Each instance has its own attribute values. Creating one is called *instantiation*; it happens by calling the class like a function.

**`__init__`** — the *initializer*, run automatically when an instance is created. Its job: receive constructor arguments and store them as attributes. It returns nothing.

**`self`** — the instance a method was called on. Every regular method's first parameter is `self` (the name is convention, near-universal). When you write `account.deposit(50)`, Python calls `BankAccount.deposit(account, 50)` — `self` *is* `account`. Inside the class, all access to the instance's own data goes through `self`: `self.balance`.

**Attributes** — variables attached to an object.

- *Instance attributes* — set via `self.x = ...` (usually in `__init__`); each object gets its own.
- *Class attributes* — set directly in the class body; shared by all instances (good for constants and defaults, dangerous for mutable values).

**Method** — a function defined inside a class. Called as `obj.method(args)`.

**Encapsulation & the underscore convention** — Python has no `private` keyword. A single leading underscore (`self._balance`) signals "internal — don't touch from outside." It's a convention, honored socially and by tools, not enforced.

**`@property`** — turns a method into a computed, read-only attribute: define `def full_name(self):` decorated with `@property`, then use `person.full_name` (no parentheses). Great for derived values and for controlled access to `_underscore` internals.

**Inheritance** — a class can extend another: `class SavingsAccount(BankAccount):`. The *child* (subclass) inherits all methods and attributes of the *parent* (superclass) and may **add** new ones or **override** existing ones. `super()` calls the parent's version — most importantly `super().__init__(...)` inside the child's `__init__` so parent setup still happens. Guideline: inheritance models "**is a**" (a `SavingsAccount` *is a* `BankAccount`); prefer *composition* ("has a" — storing another object as an attribute) when the relationship is looser.

**Dunder (double-underscore) methods** — hooks that let your objects participate in Python's built-in syntax:

| Dunder | Powers | Example trigger |
|---|---|---|
| `__init__` | construction | `Point(3, 4)` |
| `__repr__` | developer string — unambiguous | `repr(p)`, REPL echo, printing lists of objects |
| `__str__` | user-facing string (falls back to `__repr__`) | `print(p)`, `f"{p}"` |
| `__eq__` | equality | `p1 == p2` (default compares identity!) |
| `__lt__` | ordering | `sorted(points)` |
| `__len__` | length | `len(playlist)` |
| `__add__` | `+` operator | `p1 + p2` |
| `__getitem__` | indexing | `playlist[0]`, and thereby `for` loops |
| `__contains__` | membership | `song in playlist` |

You never call these directly; you implement them and Python's syntax invokes them.

**`isinstance(obj, Class)`** — is `obj` an instance of `Class` (or a subclass)? Preferred over `type(obj) == Class`.

## Code Examples

### A first class

```python
class Player:
    """A player in a game."""

    def __init__(self, name: str, score: int = 0):
        self.name = name              # instance attributes: this player's own data
        self.score = score

    def add_points(self, points: int) -> None:
        """Increase this player's score."""
        self.score += points

    def describe(self) -> str:
        return f"{self.name} has {self.score} points"


# Instantiation — __init__ runs with self bound to the new object
p1 = Player("Nia")
p2 = Player("Omar", score=50)

p1.add_points(30)
p1.add_points(20)
print(p1.describe())      # Nia has 50 points
print(p2.describe())      # Omar has 50 points
print(p1.name, p2.name)   # separate objects, separate data
```

### Class attributes vs instance attributes

```python
class Circle:
    PI = 3.14159            # class attribute — shared, effectively a constant

    def __init__(self, radius):
        self.radius = radius    # instance attribute — per-circle

    def area(self):
        return Circle.PI * self.radius ** 2

print(Circle(2).area())     # 12.56636
print(Circle.PI)            # accessible on the class itself
```

### Encapsulation with `_underscore` and `@property`

```python
class BankAccount:
    def __init__(self, owner: str, balance: float = 0.0):
        self.owner = owner
        self._balance = float(balance)      # _ = "internal, use the methods"

    @property
    def balance(self) -> float:
        """Read-only view of the balance."""
        return self._balance

    def deposit(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("deposit must be positive")
        self._balance += amount

    def withdraw(self, amount: float) -> None:
        if amount <= 0:
            raise ValueError("withdrawal must be positive")
        if amount > self._balance:
            raise ValueError("insufficient funds")
        self._balance -= amount

acct = BankAccount("Ada", 100)
acct.deposit(50)
acct.withdraw(30)
print(acct.balance)          # 120.0 — looks like an attribute, backed by a method
# acct.balance = 999         # AttributeError — no setter defined: protected!
```

### Inheritance and `super()`

```python
class SavingsAccount(BankAccount):
    """A BankAccount that earns interest and limits withdrawals."""

    def __init__(self, owner, balance=0.0, rate=0.02):
        super().__init__(owner, balance)     # let the parent set up owner/_balance
        self.rate = rate
        self._withdrawals_this_month = 0

    def add_interest(self):                  # NEW behavior
        self._balance += self._balance * self.rate

    def withdraw(self, amount):              # OVERRIDDEN behavior
        if self._withdrawals_this_month >= 3:
            raise ValueError("savings accounts allow 3 withdrawals per month")
        super().withdraw(amount)             # reuse the parent's checks & logic
        self._withdrawals_this_month += 1

s = SavingsAccount("Grace", 1000)
s.add_interest()
print(f"{s.balance:.2f}")                    # 1020.00
print(isinstance(s, BankAccount))            # True — a SavingsAccount IS a BankAccount
```

### Dunder methods: a Point that feels built-in

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point({self.x}, {self.y})"          # unambiguous, ideally eval-able

    def __eq__(self, other):
        if not isinstance(other, Point):
            return NotImplemented                    # let Python try other options
        return self.x == other.x and self.y == other.y

    def __add__(self, other):
        return Point(self.x + other.x, self.y + other.y)

    def __lt__(self, other):                         # enables sorted()
        return (self.x, self.y) < (other.x, other.y)

p, q = Point(1, 2), Point(3, 4)
print(p + q)                     # Point(4, 6)
print(p == Point(1, 2))          # True (default would be False — different objects!)
print(sorted([q, p]))            # [Point(1, 2), Point(3, 4)]
```

### A container class

```python
class Playlist:
    def __init__(self, name):
        self.name = name
        self._songs = []                 # composition: a Playlist HAS a list

    def add(self, title):
        self._songs.append(title)

    def __len__(self):
        return len(self._songs)

    def __getitem__(self, index):        # enables pl[0] AND `for song in pl`
        return self._songs[index]

    def __contains__(self, title):
        return title in self._songs

pl = Playlist("Focus")
pl.add("Weightless")
pl.add("Divenire")
print(len(pl), pl[0], "Divenire" in pl)   # 2 Weightless True
for song in pl:
    print("-", song)
```

## Common Pitfalls

**1. Forgetting `self`**

```python
class Dog:
    def bark():                     # WRONG — missing self
        print("woof")
Dog().bark()                        # TypeError: bark() takes 0 positional arguments but 1 was given
```

That error message is *the* signature of a missing `self`. Also inside methods: writing `balance` when you mean `self.balance` creates a useless local.

**2. Mutable class attributes**

```python
class Team:
    members = []                    # SHARED by every team!
    def add(self, name):
        self.members.append(name)

a, b = Team(), Team()
a.add("Ana")
print(b.members)                    # ['Ana'] — leaked across instances

class Team:                         # RIGHT
    def __init__(self):
        self.members = []
```

**3. Forgetting `super().__init__()` in a subclass** — the parent's attributes never get created, and you'll hit `AttributeError: 'Child' object has no attribute '_balance'` in inherited methods.

**4. Forgetting to instantiate / calling on the class**

```python
acct = BankAccount        # no parens — acct is the CLASS
# acct.deposit(50)        # TypeError
acct = BankAccount("Ada") # RIGHT
```

**5. No `__repr__`** — `print(my_objects)` showing `[<__main__.Point object at 0x000001>...]` is self-inflicted. Implement `__repr__` early on every class; debugging improves instantly.

**6. Relying on default `==`** — without `__eq__`, `Point(1,2) == Point(1,2)` is `False` (identity comparison). If your objects represent values, define `__eq__`.

**7. Classes for everything** — a bundle of related functions and a dict may be simpler than a class. Reach for a class when data + behavior genuinely belong together, when you need many instances, or when a framework expects one. (For pure data records, look up `dataclasses` — a standard-library shortcut worth knowing.)

## Practice Exercises

1. **Book & Library.** Write a `Book` class (title, author, year, `__repr__`, `__eq__`) and a `Library` class holding books with methods `add`, `remove`, `find_by_author(name)` (returns a list), plus `__len__` and `__contains__`. Demo every feature.
2. **Temperature class.** Build `Temperature` storing degrees Celsius internally, with a `@property` `fahrenheit` computed on the fly, validation that rejects values below −273.15 (raise `ValueError`), and a friendly `__str__` like `21.5°C (70.7°F)`.
3. **Shapes hierarchy.** Create a base `Shape` with method `area()` that raises `NotImplementedError`, then subclasses `Rectangle(w, h)` and `Circle(r)` overriding it. Put mixed shapes in a list and print each one's area from a single loop — that's polymorphism in action.
4. **Inventory item with operators.** Write `Money(amount, currency)` supporting `+` and `==` (raise `ValueError` when currencies differ), `__repr__`, and ordering with `<` so a list of same-currency Money sorts correctly. Comment on why mixing currencies in `<` should fail loudly.
5. **Refactor to OOP.** Take your Chapter 12 `withdraw` function exercise (or the chapter's bank example) and rebuild it as classes: `BankAccount` plus a `CheckingAccount` subclass allowing an overdraft up to a limit. Keep the custom exceptions from Chapter 12 as the error mechanism.
