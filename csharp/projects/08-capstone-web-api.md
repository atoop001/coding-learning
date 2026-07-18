# Project 8 (Capstone): "BookNook" — ASP.NET Core Web API with Tests

## Description

Build a complete minimal Web API for a small book-tracking service: books can be created, listed, searched, updated, rated, and deleted over HTTP with JSON, backed by a JSON file for persistence and covered by a unit-tested core library. This capstone assembles the entire track — OOP design, interfaces for testability, LINQ, async file I/O, exceptions, xUnit — into the shape of a real backend service, the kind of artifact you can show in a portfolio and discuss in interviews.

## Difficulty

**Advanced (capstone)** — estimated effort: 12–20 hours across multiple sessions.

## Chapters Used

All of them, most directly: 10 (interfaces), 12 (collections), 13 (exceptions), 15 (LINQ), 16 (file I/O & async), 17 (xUnit), 18 (NuGet & ASP.NET Core minimal APIs).

## Requirements Checklist

### Architecture
- [ ] Solution with three projects: `src/BookNook.Core` (classlib: domain + services), `src/BookNook.Api` (the `dotnet new web` app), `tests/BookNook.Core.Tests` (xunit) — references wired appropriately; the API project stays thin
- [ ] A `Book` model: `Id` (int), `Title`, `Author`, `Year`, `Rating` (double 0–5, nullable until first rated), `Status` (an enum: `Wishlist`, `Reading`, `Finished`)
- [ ] An `IBookRepository` interface in Core (`GetAllAsync`, `GetByIdAsync`, `AddAsync`, `UpdateAsync`, `DeleteAsync`) with **two implementations**: `InMemoryBookRepository` (for tests) and `JsonFileBookRepository` (persists to `books.json` with async file I/O)
- [ ] A `BookService` in Core containing all business rules, depending only on `IBookRepository` (constructor injection)
- [ ] The API registers the JSON repository and the service in the DI container (`builder.Services.AddSingleton<...>`) and route handlers take the service as a parameter

### Business rules (in `BookService`, not in route handlers)
- [ ] Title and author are required, trimmed, max 200 chars — invalid input is rejected with a descriptive error
- [ ] Duplicate (title + author, case-insensitive) is rejected
- [ ] Rating only allowed when `Status` is `Finished`; must be 0–5 in 0.5 steps
- [ ] Ids are assigned by the repository, never by the client

### HTTP endpoints
- [ ] `GET /books` — all books; optional query parameters `status`, `author`, `minRating` combine as filters (LINQ)
- [ ] `GET /books/{id}` — one book or **404**
- [ ] `POST /books` — create; **201** with Location header on success, **400** with the validation message on rule violations
- [ ] `PUT /books/{id}` — update title/author/year/status; proper 404/400 handling
- [ ] `POST /books/{id}/rating` — set the rating (enforcing the Finished rule)
- [ ] `DELETE /books/{id}` — **204** or 404
- [ ] `GET /books/stats` — total count, count per status, average rating of finished books, most-read author (LINQ aggregation)
- [ ] Data survives an API restart (JSON file persistence, written asynchronously)

### Tests
- [ ] `BookService` is tested **against the in-memory repository** — no file system in unit tests
- [ ] At least 20 tests covering: every business rule (accept and reject paths), duplicate detection, rating constraints, filter combinations, stats math, and not-found behavior
- [ ] `[Theory]` used for validation input tables (bad titles, bad ratings)
- [ ] `dotnet test` green; API manually verified with curl/REST client for every endpoint (keep a `requests.http` or a curl cheat-sheet file in the repo as your manual test script)

## Hints

- Build inside-out and keep the API for last: model → `IBookRepository` + in-memory impl → `BookService` with tests → JSON repository → wire up endpoints. The API layer should end up being ~60 lines of routing.
- Let service methods return a result type rather than throwing for expected failures (invalid input, duplicates) — your Project 7 `Result<T>` (or a simple version of it) maps beautifully onto 400 vs 201 decisions in handlers. Reserve exceptions for genuinely broken states.
- For the JSON repository, load once at startup, hold the list in memory, and rewrite the file after each mutation — simple and adequate here. Note in a comment why this wouldn't survive multiple server instances (Chapter 18 pitfall).
- Enum serialization: add `JsonStringEnumConverter` to the JSON options so statuses read as `"Reading"`, not `1`, over the wire.
- Minimal-API handlers can take `(int id, BookService svc)` — DI fills the service automatically. Return `Results.Ok/Created/NotFound/BadRequest`.
- Test the filter combinations by seeding the in-memory repo with a known cast of ~8 books in the test constructor.
- When something behaves oddly, `dotnet run` with a breakpoint-friendly IDE (VS/VS Code debugger) will teach you more than console spam — this project is worth learning the debugger on.

## Stretch Goals

- Pagination on `GET /books` (`page`, `pageSize` query params) with sane defaults and caps.
- Integration tests with `WebApplicationFactory` (`Microsoft.AspNetCore.Mvc.Testing` NuGet package) hitting real HTTP endpoints against the in-memory repository.
- Swap `JsonFileBookRepository` for SQLite via `Microsoft.Data.Sqlite` or EF Core — your interface means nothing above it changes; doing this swap is the single best proof the architecture worked.
- Add OpenAPI/Swagger UI (`AddEndpointsApiExplorer` + `Swashbuckle` or built-in `MapOpenApi`) and explore your API in the browser.
- Deploy it: publish with `dotnet publish -c Release`, containerize with a Dockerfile, or push to a free Azure App Service tier — and add the URL to your portfolio README.
