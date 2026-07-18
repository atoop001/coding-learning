# C# Learning Track

A self-paced course in C# and .NET for someone with basic coding experience (some Python/JavaScript), aimed at **employability** and **.NET web development**, on Windows. It consists of 18 study chapters (`learning-docs/`) and 8 guided projects (`projects/`) that bundle the chapters into real programs. Project specs contain requirements and hints only — no solution code; the building is the learning.

**Relationship to Java:** C# and Java are close cousins (CLR ↔ JVM, NuGet ↔ Maven, ASP.NET Core ↔ Spring Boot, xUnit ↔ JUnit). This track works as an *alternative to* a Java track — the concepts transfer both ways — or as a fast *follow-up after* Java, in which case the OOP chapters (8–10) will be mostly review and you can move at double speed.

## Chapters (in order)

| # | Chapter | Topics |
|---|---------|--------|
| 01 | [Getting Started](learning-docs/01-getting-started.md) | .NET SDK on Windows, dotnet CLI, VS Code/Visual Studio, first console app, how .NET works |
| 02 | [Variables & Types](learning-docs/02-variables-and-types.md) | Built-in types, `var`, value vs reference types, null & nullables, conversions |
| 03 | [Operators & Control Flow](learning-docs/03-operators-and-control-flow.md) | Operators, `if`/`else`, ternary, `switch` statement & expression, pattern matching |
| 04 | [Loops](learning-docs/04-loops.md) | `while`, `do-while`, `for`, `foreach`, `break`/`continue` |
| 05 | [Methods](learning-docs/05-methods.md) | Parameters, return types, overloading, optional/named args, `out`, tuples, `params` |
| 06 | [Strings & Interpolation](learning-docs/06-strings.md) | Immutability, formatting, common operations, comparison, `StringBuilder` |
| 07 | [Arrays & Lists](learning-docs/07-arrays-and-lists.md) | Fixed arrays, `List<T>`, indexing, 2D data, copying vs aliasing |
| 08 | [Classes & Objects](learning-docs/08-classes-and-objects.md) | Properties, constructors, encapsulation, `static`, records |
| 09 | [Inheritance & Polymorphism](learning-docs/09-inheritance-and-polymorphism.md) | Base/derived classes, `virtual`/`override`, `base`, casting, `sealed` |
| 10 | [Interfaces & Abstract Classes](learning-docs/10-interfaces-and-abstract-classes.md) | Contracts, abstract members, choosing between them, design for testability |
| 11 | [Namespaces & Project Structure](learning-docs/11-namespaces-and-project-structure.md) | `using`, solutions, multi-project layouts, class libraries, `internal` vs `public` |
| 12 | [Collections & Generics](learning-docs/12-collections-and-generics.md) | `Dictionary`, `HashSet`, `Queue`/`Stack`, writing generic classes/methods, constraints |
| 13 | [Exceptions & Error Handling](learning-docs/13-exceptions-and-error-handling.md) | try/catch/finally, throwing, custom exceptions, Try-pattern vs exceptions |
| 14 | [Delegates, Events & Lambdas](learning-docs/14-delegates-events-lambdas.md) | `Func`/`Action`, lambdas & closures, multicast, events, pub/sub |
| 15 | [LINQ](learning-docs/15-linq.md) | Where/Select/OrderBy/GroupBy, aggregation, deferred execution |
| 16 | [File I/O & Async/Await](learning-docs/16-file-io-and-async.md) | Files, paths, `using`/`IDisposable`, JSON, `async`/`await`, `Task.WhenAll`, `HttpClient` |
| 17 | [Unit Testing with xUnit](learning-docs/17-unit-testing-xunit.md) | Test projects, `[Fact]`/`[Theory]`, assertions, AAA, testing with fakes |
| 18 | [The .NET Ecosystem](learning-docs/18-dotnet-ecosystem.md) | NuGet, ASP.NET Core minimal API walkthrough, where C# is used (web, desktop, Unity), career lanes |

Each chapter contains an overview, plain-English explanations (with Python/JS contrasts), compilable code examples, common pitfalls with fixes, and 3–5 practice exercises.

## Projects (in order, easiest → hardest)

| # | Project | After chapter | Bundles |
|---|---------|:---:|---------|
| 1 | [Console Quiz Game](projects/01-console-quiz-game.md) | 05 | I/O, types, control flow, loops, methods |
| 2 | [Text Statistics Tool](projects/02-text-stats-tool.md) | 07 | Strings, arrays/lists, method decomposition |
| 3 | [Contact Manager](projects/03-contact-manager.md) | 08 | Classes, encapsulation, `List<T>` CRUD, menus |
| 4 | [RPG Character & Inventory System](projects/04-rpg-character-system.md) | 11 | Inheritance, polymorphism, interfaces, project structure |
| 5 | [Event-Driven Smart Home Simulation](projects/05-event-driven-simulation.md) | 14 | Delegates, events, lambdas, collections |
| 6 | [Movie Data Query Tool](projects/06-data-query-tool.md) | 16 | LINQ, file I/O, JSON, defensive parsing, async |
| 7 | [Tested Utility Library](projects/07-tested-utility-library.md) | 17 | Class libraries, xUnit, API design, generics |
| 8 | [Capstone: BookNook Web API](projects/08-capstone-web-api.md) | 18 | Everything — ASP.NET Core minimal API + tested core |

Every spec has a description, difficulty & effort estimate, chapters used, a requirements checklist, hints (nudges, not answers), and stretch goals. **Don't skip the checklists** — "done" means every box ticked and the program survives hostile input.

## Suggested Cadence

At roughly **5–8 hours per week**, the track takes about **14–18 weeks**. Chapters alone take 2–4 hours each (reading + exercises); do the exercises — reading alone does not stick.

| Weeks | Do |
|---|---|
| 1–2 | Chapters 01–04 |
| 3 | Chapter 05 → **Project 1** |
| 4 | Chapters 06–07 → **Project 2** |
| 5–6 | Chapter 08 → **Project 3** |
| 7–8 | Chapters 09–11 → **Project 4** |
| 9 | Chapters 12–13 |
| 10 | Chapter 14 → **Project 5** |
| 11–12 | Chapters 15–16 → **Project 6** |
| 13 | Chapter 17 → **Project 7** |
| 14–16 | Chapter 18 → **Capstone (Project 8)** |
| after | Next steps from Chapter 18: EF Core + SQL, auth, Docker, one cloud provider |

Working habits that pay off:

- **Type every example yourself** — don't paste. Compiler errors are the curriculum.
- Keep one scratch console project (`Playground`) alive for experiments.
- Put each project in git from day one (`git init`, small commits) — the habit is a job skill in itself.
- When stuck longer than ~30 minutes, re-read the relevant chapter's pitfalls section first; most beginner blockers are listed there.
- From Project 4 onward, revisit an earlier project after finishing a new chapter and improve it (persistence, LINQ, tests) — refactoring old code teaches more than writing new code.
