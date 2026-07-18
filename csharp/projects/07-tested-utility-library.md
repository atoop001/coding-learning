# Project 7: Tested Utility Library ("SharpKit")

## Description

Build a reusable class library of utility functions — string helpers, validation, date math, and a generic result type — with a full xUnit test suite as a first-class deliverable. The library has no console UI at all; the tests *are* the way you run it (plus a small demo app). This mirrors real library development and is the project most directly rehearsing job-interview skills: clean public APIs, edge-case thinking, and meaningful test coverage.

## Difficulty

**Advanced** — estimated effort: 8–12 hours.

## Chapters Used

- 17 Unit Testing with xUnit
- 11 Namespaces & Project Structure
- 12 Collections & Generics
- 13 Exceptions & Error Handling
- 05/06 Methods & Strings

## Requirements Checklist

### Structure
- [ ] A solution with three projects: `src/SharpKit` (classlib), `tests/SharpKit.Tests` (xunit), `src/SharpKit.Demo` (console app referencing the library), all added to the `.sln` with references wired via the CLI
- [ ] Public types organized in namespaces: `SharpKit.Text`, `SharpKit.Validation`, `SharpKit.Time`, `SharpKit.Core`
- [ ] `dotnet test` runs green from the solution root; `dotnet run --project src/SharpKit.Demo` shows a few features working

### Library features (each fully unit-tested)
- [ ] `Text.Slugify(string)` — "Hello, World!" → "hello-world" (lowercase, spaces/punctuation to single hyphens, no leading/trailing hyphens)
- [ ] `Text.Truncate(string, int max, string suffix = "…")` — never exceeds `max` *including* the suffix; documents and tests behavior when `max` is smaller than the suffix
- [ ] `Text.ToTitleCase(string)` with a small stop-word list ("a", "of", "the" stay lowercase unless first word)
- [ ] `Validation.IsValidEmail(string)` — simple, documented rules (one "@", non-empty parts, a "." in the domain); returns bool, never throws
- [ ] `Validation.RequireInRange(int value, int min, int max, string paramName)` — returns the value or throws `ArgumentOutOfRangeException` with the right parameter name
- [ ] `Time.Age(DateTime birthDate, DateTime asOf)` — correct age in years including the day-before-birthday edge case
- [ ] `Time.NextOccurrence(DayOfWeek day, DateTime from)` — the next date falling on that weekday (never returns `from` itself)
- [ ] `Core.Result<T>` — a generic success/failure type: static `Ok(T)` / `Fail(string error)`, properties `IsSuccess`, `Value` (throws `InvalidOperationException` if accessed on failure), `Error`; plus `Map<TOut>(Func<T,TOut>)` that short-circuits on failure

### Tests
- [ ] At least 30 test cases total; every public method has tests
- [ ] `[Theory]`/`[InlineData]` used for all input-table cases (slugify, email, ranges)
- [ ] Every `throw` in the library has a test asserting the exception type
- [ ] Null/empty/whitespace inputs have explicit, tested behavior for every string-taking method (decide: throw or return sentinel — be consistent and document it)
- [ ] Test names follow `Method_Scenario_Expectation`
- [ ] No test touches the file system, network, console, or current time — `Age`/`NextOccurrence` take the reference time as a parameter for exactly this reason

## Hints

- Work feature-by-feature, test-first where you can: write the `[Theory]` table from the spec above, watch it fail, implement until green. It's genuinely faster here.
- `Slugify` is a chain of small transformations — build it stepwise with tests for each rule before combining.
- The `Age` edge cases: birthday today, birthday tomorrow, Feb 29 birthdate in a non-leap year. Write those three tests before the implementation and your logic will come out right.
- `Result<T>` is your first taste of the "railway" pattern; `Map` should return `Result<TOut>.Fail(Error)` without invoking the function when already failed — test that with a lambda that would throw if called.
- Keep XML doc comments (`/// <summary>`) on every public member — hover-docs in the IDE are part of a library's UX.
- When a test is awkward to write, treat it as API feedback: too many parameters, unclear behavior, or hidden dependencies. Redesign, don't force it.

## Stretch Goals

- Add `Text.WordWrap(string, int width)` returning `IReadOnlyList<string>` — a deceptively deep function; test long words exceeding the width.
- Add `Core.Retry.Run<T>(Func<T> action, int attempts)` that retries on exception and returns `Result<T>` — test with a lambda that fails N times then succeeds (closure counter!).
- Measure coverage (`dotnet test --collect:"XPlat Code Coverage"` + a report tool) and close any untested branch.
- Pack the library into an actual NuGet package locally (`dotnet pack`) and consume the `.nupkg` from a fresh console project via a local package source.
- Property-style tests: for any input, `Slugify` output never contains uppercase, spaces, or double hyphens — implement by looping over a batch of random strings inside one test.
