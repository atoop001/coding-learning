# Project 4: Zoo Simulator — an OOP Hierarchy

## Description

Model a zoo in objects: an `Animal` base class, a family of subclasses (mammals, birds, reptiles — and concrete species under those), and a `Zoo` class that owns and operates on the whole population polymorphically. A daily-routine simulation (feed everything, hear every sound, do species-specific behaviors) demonstrates that one loop over `Animal[]` drives wildly different behavior per object. This is your first *designed* program: the class diagram matters more than any single method.

## Difficulty

**Intermediate** — estimated effort: 5–8 hours.

## Chapters used

- 08 (Classes & Objects) — fields, constructors, encapsulation, `toString`
- 09 (Inheritance & Polymorphism) — `extends`, `super`, `@Override`, dynamic dispatch, `instanceof`
- 10 (Interfaces & Abstract Classes) — abstract `Animal`, capability interfaces, `enum` (stretch goal `Diet`)
- (Supporting: 05 methods, 07 arrays for the zoo's internal storage)

## Requirements checklist

- [ ] Abstract class `Animal` with private fields (at least: name, age, weight), a constructor validating them, getters, and a `toString`
- [ ] `Animal` declares abstract `String makeSound()` and abstract `String favoriteFood()`
- [ ] Intermediate layer: at least two of `Mammal`, `Bird`, `Reptile` extending `Animal`, each adding one shared field or behavior (e.g., `Bird` has `wingspanCm`)
- [ ] At least five concrete species classes (e.g., `Lion`, `Elephant`, `Parrot`, `Penguin`, `Snake`) with species-appropriate overrides
- [ ] At least one subclass constructor uses `super(...)` with arguments, and at least one override calls `super.method()` to extend (not replace) parent behavior
- [ ] Capability interfaces `Swimmer` (`swim()`) and `Flyer` (`fly()`) implemented by the right species — including at least one bird that swims but does not fly
- [ ] `Zoo` class encapsulating an `Animal[]` (private) with `addAnimal`, capacity handling, and no direct external access to the array
- [ ] `Zoo.dailyRoutine()` loops the population **once**, printing each animal's sound and feeding line — no `instanceof` needed for this part (that's the point of polymorphism)
- [ ] `Zoo.swimShow()` uses pattern-matching `instanceof` (or a smarter design) to make only the swimmers swim
- [ ] `Zoo.heaviestAnimal()` and `Zoo.averageAge()` computed over the population
- [ ] A demo `main` that builds a zoo of 8+ mixed animals and exercises every feature
- [ ] Every override is annotated `@Override`; `new Animal(...)` is impossible (verify, then note it in a comment)

## Hints

- Design before typing: sketch the tree (Animal → Mammal/Bird/Reptile → species) and decide *for each field and method which level it belongs to*. Weight belongs to Animal; wingspan to Bird; "Woof" to Dog. If you find a field only one subclass uses sitting in the base class, move it down.
- The "is-a" test guards the hierarchy; the "can-do" test picks out interfaces. Penguins are the canonical proof that `fly()` doesn't belong on `Bird`.
- Make `makeSound()` return a `String` rather than printing — the caller decides presentation, and later (Chapter 17) it's testable.
- For `swimShow`, first make it work with `instanceof`. Then, as a design meditation: could `Zoo` keep a separate `Swimmer[]` filled at add-time? Which is better here, and why? Write your conclusion as a comment.
- If two species share code (e.g., all big cats roar), that's a hint an intermediate class (`BigCat`) wants to exist.

## Stretch goals

- `Comparable<Animal>` by weight, so `Arrays.sort` orders the population; print a weigh-in board.
- An `enum Diet { CARNIVORE, HERBIVORE, OMNIVORE }` field, and a feeding-cost report per diet.
- Hunger simulation: feeding sets `hungry = false`; every `dailyRoutine` ends by making everyone hungry again; sounds differ when hungry.
- A `Nocturnal` marker interface and a night-shift routine that only disturbs nocturnal animals.
- Breeding: `Animal reproduce()` returning a new animal of the same species (research covariant return types).
