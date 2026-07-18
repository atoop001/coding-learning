# Java Learning Track

A self-paced, employability-focused path from zero Java to building tested, Maven-packaged applications. Designed for a learner with some Python/JavaScript background, on Windows. Chapters are the study material; projects are where it becomes real — **no solutions are provided anywhere, on purpose**.

## How to use this track

1. Read chapters in order; type out and run every code example (don't copy-paste — typing builds syntax memory).
2. Do the practice exercises at the end of each chapter — at least 3 of 5.
3. When you reach a project's "gate" chapter (see plan below), stop and build the project before reading on. Projects deliberately bundle several chapters.
4. Keep everything in git from Project 3 onward; the capstone belongs on GitHub.

## Chapters (`learning-docs/`)

| # | File | Topic |
|---|------|-------|
| 01 | `01-getting-started.md` | JDK install on Windows, javac/java, IntelliJ vs VS Code, first program, how compilation works |
| 02 | `02-variables-and-types.md` | Static typing, the 8 primitives, objects & references, wrappers, casting |
| 03 | `03-operators-and-control-flow.md` | Operators, if/else, ternary, classic & modern switch |
| 04 | `04-loops.md` | while, do-while, for, for-each, break/continue/labels |
| 05 | `05-methods-and-static.md` | Methods, overloading, varargs, pass-by-value, what `static` means |
| 06 | `06-strings-and-text.md` | String API, immutability, equals vs ==, formatting, StringBuilder |
| 07 | `07-arrays.md` | 1D & 2D arrays, Arrays utility class, classic algorithms |
| 08 | `08-classes-and-objects.md` | Fields, constructors, `this`, encapsulation, getters/setters, toString |
| 09 | `09-inheritance-and-polymorphism.md` | extends, super, @Override, dynamic dispatch, instanceof/casting |
| 10 | `10-interfaces-and-abstract-classes.md` | Contracts, implements, default methods, Comparable, choosing between them |
| 11 | `11-packages-access-modifiers-classpath.md` | Packages, imports, the four access levels, classpath, JAR preview |
| 12 | `12-collections-framework.md` | List/Set/Map, ArrayList, HashMap, iteration, equals/hashCode, records |
| 13 | `13-generics.md` | Generic classes & methods, bounds, wildcards & PECS, erasure |
| 14 | `14-exceptions.md` | try/catch/finally, checked vs unchecked, custom exceptions, stack traces |
| 15 | `15-lambdas-and-streams.md` | Lambdas, functional interfaces, method references, Stream API, Optional |
| 16 | `16-file-io.md` | Path/Files, reading & writing, try-with-resources, CSV parsing |
| 17 | `17-unit-testing-junit.md` | JUnit 5, assertions, fixtures, parameterized tests, testing exceptions |
| 18 | `18-build-tools-and-ecosystem.md` | Maven/Gradle, dependencies, building JARs, Spring/Android and what to learn next |

## Projects (`projects/`)

| # | File | Builds on chapters | Difficulty |
|---|------|--------------------|------------|
| 1 | `01-number-guessing-game.md` | 01–04 | Beginner |
| 2 | `02-unit-converter.md` | 02–06 | Beginner |
| 3 | `03-student-gradebook.md` | 04–07 | Beginner-plus |
| 4 | `04-zoo-hierarchy.md` | 08–10 | Intermediate |
| 5 | `05-library-management.md` | 08–13 | Intermediate |
| 6 | `06-bank-system.md` | 09–14 | Intermediate-plus |
| 7 | `07-sales-data-analyzer.md` | 12–17 | Advanced |
| 8 | `08-capstone-task-manager.md` | Everything (esp. 17–18) | Capstone |

## Suggested cadence (~14 weeks at 8–10 hrs/week)

| Week | Study | Build |
|------|-------|-------|
| 1 | Ch. 01–02 | exercises |
| 2 | Ch. 03–04 | **Project 1: Guessing Game** |
| 3 | Ch. 05–06 | **Project 2: Unit Converter** |
| 4 | Ch. 07 | **Project 3: Gradebook** |
| 5 | Ch. 08–09 | exercises (OOP needs soak time) |
| 6 | Ch. 10 | **Project 4: Zoo Hierarchy** |
| 7 | Ch. 11–12 | exercises |
| 8 | Ch. 13 | **Project 5: Library System** |
| 9 | Ch. 14 | **Project 6: Bank System** |
| 10 | Ch. 15 | exercises (streams reward repetition) |
| 11 | Ch. 16–17 | start **Project 7: Sales Analyzer** |
| 12 | finish Project 7 | Ch. 18 |
| 13–14 | — | **Project 8: Capstone (TaskForge)** |

Going faster is fine; skipping projects is not — they are the actual learning.

## After the track

For enterprise employability, the standard next steps in rough order: **Spring Boot** (start.spring.io), **SQL + JDBC/JPA**, Git fluency, one deployed portfolio REST API, then basic Docker. Chapter 18 ends with a concrete recon exercise pointing you there.
