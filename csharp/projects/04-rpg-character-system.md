# Project 4: RPG Character & Inventory System

## Description

Design and build the object model for a small role-playing game: a character hierarchy (warrior, mage, healer) with distinct abilities, an inventory of typed items, and a turn-based combat simulation between two parties printed to the console. There is no graphics and minimal user input — the point is **object-oriented design**: inheritance, polymorphism, interfaces, and abstract classes making a system that's easy to extend.

## Difficulty

**Intermediate** — estimated effort: 6–10 hours.

## Chapters Used

- 08 Classes & Objects
- 09 Inheritance & Polymorphism
- 10 Interfaces & Abstract Classes
- 11 Namespaces & Project Structure
- 07 Arrays & Lists

## Requirements Checklist

- [ ] An abstract `Character` base class: `Name`, `Health`, `MaxHealth`, `IsAlive` (computed), an abstract `Attack(Character target)` method, and a virtual `TakeDamage(int amount)` that clamps health at 0
- [ ] At least three concrete classes (`Warrior`, `Mage`, `Healer`) with genuinely different behavior — not just different numbers (e.g., mage spends mana, healer's "attack" can heal an ally instead, warrior builds rage)
- [ ] `Health` can never be set negative or above `MaxHealth` from outside — encapsulation enforced with private setters/validation
- [ ] An `Item` hierarchy or an interface-based design for items: at least `Weapon` (boosts attack), `Potion` (consumable, heals), and one item of your own invention
- [ ] An `IUsable` interface (`Use(Character target)`) implemented by consumable items
- [ ] Each character has an `Inventory` (a class wrapping a `List<Item>`, with a capacity limit and methods to add/use/list — not a raw public list)
- [ ] A combat loop: two teams (`List<Character>`) alternate turns until one team has no living members; each turn is reported in readable text
- [ ] The combat loop works entirely through the `Character` base type — adding a fourth class must require **zero changes** to the combat code (prove it by adding one at the end)
- [ ] Types are organized into namespaces/folders (e.g., `Rpg.Characters`, `Rpg.Items`, `Rpg.Combat`), one type per file
- [ ] Overridden `ToString()` everywhere it helps the battle log read well
- [ ] No public fields anywhere; no downcasting (`(Warrior)c`) in the combat loop

## Hints

- Start on paper: list each class, what it *is* (inheritance), what it *can do* (interfaces), what it *has* (composition). The Chapter 9 "is-a vs has-a" test resolves most design doubts.
- `protected` is the right visibility for things like a `SetHealth` helper subclasses need but outsiders shouldn't touch.
- Let the base constructor take name/maxHealth and chain with `: base(...)` — each subclass adds its own stats.
- For target selection keep it simple: attack the first living enemy (`team.Find(c => c.IsAlive)`).
- If subclass special abilities tempt you into `if (attacker is Mage)` inside combat — stop; that logic belongs *inside* the subclass's own `Attack` override.
- Randomness (`Random.Shared.Next(min, max)`) makes damage ranges and battles interesting; pass a seed for repeatable testing.
- A `BattleLog` or plain `Console.WriteLine` narration from within character methods is fine at this scale.

## Stretch Goals

- Add status effects (poison, shield) as their own small class hierarchy, ticked each turn.
- Add an `IComparable<Character>` implementation (by a speed stat) and sort the turn order each round.
- Equipment slots: equipping a `Weapon` changes attack output; only one weapon equipped at a time.
- A simple character-creation menu letting the player assemble a team before the battle runs.
- Extract the battle-narration into an interface (`ILogger`) injected into the combat system — silent tests, chatty console — foreshadowing Chapters 10/17 testing patterns.
