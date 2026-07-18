# Chapter 18: The .NET Ecosystem — NuGet, ASP.NET Core & Where C# Takes You

## Overview

You now know the language. This final chapter maps the territory around it: **NuGet** (the package manager), **ASP.NET Core** (the web framework, with a working minimal API you'll build), and a tour of where C# is used professionally — web backends, desktop, games, cloud — so you can pick a direction. For employability, the single highest-value next step after this track is ASP.NET Core web APIs, which is why this chapter gives it the most space and the capstone project targets it.

## Definitions & Explanations

### NuGet — .NET's package manager

NuGet is to .NET what pip is to Python and npm to JavaScript. Packages live at nuget.org; dependencies are recorded in your `.csproj` (no separate requirements.txt/package.json).

```powershell
dotnet add package Newtonsoft.Json            # install latest
dotnet add package Serilog --version 3.1.1    # pin a version
dotnet list package                            # what's installed
dotnet remove package Serilog
```

The `.csproj` gains:

```xml
<ItemGroup>
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
</ItemGroup>
```

`dotnet restore` (run automatically by build) downloads everything. Packages you'll meet constantly: `Serilog` (logging), `Dapper`/`EntityFrameworkCore` (databases), `FluentValidation`, `xunit`, `Moq`/`NSubstitute` (test fakes), `Spectre.Console` (rich console UIs).

### ASP.NET Core — the web framework

ASP.NET Core is Microsoft's open-source, cross-platform web framework — consistently among the fastest mainstream frameworks in industry benchmarks (far faster than Flask/Express in throughput terms). It covers:

- **Minimal APIs** — tiny Express-like route handlers (below).
- **MVC / Web API controllers** — the classic structured style, common in enterprises.
- **Razor Pages / Blazor** — server-rendered pages / C# running in the browser via WebAssembly.

**Kestrel** is its built-in web server; **middleware** is its request pipeline (same concept as Express middleware); **dependency injection** is built into the framework core — the interface habits from Chapter 10 are exactly what it expects.

### A working minimal API in ~30 lines

```powershell
dotnet new web -o TaskApi
cd TaskApi
dotnet run          # then browse to the printed http://localhost:PORT
```

Replace `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// In-memory "database" for the demo
var tasks = new List<TaskItem>
{
    new(1, "Learn C#", true),
    new(2, "Build an API", false),
};

// Routes — compare to Express: app.get("/", (req, res) => ...)
app.MapGet("/", () => "TaskApi is running");

app.MapGet("/tasks", () => tasks);                     // auto-serialized to JSON

app.MapGet("/tasks/{id}", (int id) =>                  // route parameter, typed!
    tasks.FirstOrDefault(t => t.Id == id) is TaskItem task
        ? Results.Ok(task)
        : Results.NotFound());                          // proper 404

app.MapPost("/tasks", (NewTask input) =>               // JSON body -> object, automatically
{
    int nextId = tasks.Count == 0 ? 1 : tasks.Max(t => t.Id) + 1;
    var task = new TaskItem(nextId, input.Title, false);
    tasks.Add(task);
    return Results.Created($"/tasks/{task.Id}", task); // 201 + Location header
});

app.MapDelete("/tasks/{id}", (int id) =>
    tasks.RemoveAll(t => t.Id == id) > 0 ? Results.NoContent() : Results.NotFound());

app.Run();

// Records as data transfer objects (DTOs)
record TaskItem(int Id, string Title, bool Done);
record NewTask(string Title);
```

Try it (second terminal):

```powershell
curl http://localhost:5000/tasks
curl http://localhost:5000/tasks/1
curl -X POST http://localhost:5000/tasks -H "Content-Type: application/json" -d '{"title":"Ship it"}'
curl -X DELETE http://localhost:5000/tasks/2
```

Notice how much you already understand: records, LINQ (`FirstOrDefault`, `Max`), lambdas, pattern matching, generic lists — a web API is your existing skills plus routing.

### Where C# is used — picking your lane

| Domain | Tech | Notes |
|---|---|---|
| **Web APIs & backends** | ASP.NET Core | Biggest job market; enterprise, fintech, SaaS |
| **Full-stack web** | Blazor | C# instead of JS in the browser |
| **Databases** | Entity Framework Core, Dapper | The standard ORMs; learn EF Core early for jobs |
| **Cloud** | Azure (Functions, App Service) | C# is Azure's first-class citizen; AWS supports it well too |
| **Games** | **Unity** | C# is *the* Unity language — the indie/game path |
| **Desktop** | WinUI 3, WPF, WinForms, MAUI, Avalonia | Windows LOB apps everywhere; MAUI/Avalonia go cross-platform |
| **Mobile** | .NET MAUI | iOS/Android from one C# codebase |

A realistic employability path from here: **ASP.NET Core Web API → Entity Framework Core + SQL → auth (JWT) → Docker + one cloud provider → integration testing**. Each step has first-party tutorials at <https://learn.microsoft.com/aspnet/core>.

### C# and Java — cousins

C# began (2000) as Microsoft's answer to Java, and the languages remain close: JVM ↔ CLR, bytecode ↔ IL, Maven/Gradle ↔ NuGet, Spring Boot ↔ ASP.NET Core, JUnit ↔ xUnit. If you ever learned Java, C# reads as "Java with nicer ergonomics" (properties, LINQ, events, async/await); if you learn C# first, Java will feel familiar going the other way. Knowing one gets you 70% of the way to the other — which is why this track works as an alternative to, or follow-up to, a Java track.

## Code Examples

### Consuming a NuGet package

```powershell
dotnet new console -o PrettyDemo && cd PrettyDemo
dotnet add package Spectre.Console
```

```csharp
using Spectre.Console;      // namespace from the PACKAGE — same using syntax

var table = new Table();
table.AddColumn("Chapter");
table.AddColumn("Status");
table.AddRow("LINQ", "[green]done[/]");
table.AddRow("ASP.NET Core", "[yellow]in progress[/]");
AnsiConsole.Write(table);   // renders a styled table in the terminal
```

### Environment awareness (common in real apps)

```csharp
// Configuration & environment — how real apps avoid hardcoding
string env = Environment.GetEnvironmentVariable("APP_ENV") ?? "development";
string dataDir = env == "production"
    ? Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData), "MyApp")
    : "dev-data";
Directory.CreateDirectory(dataDir);
Console.WriteLine($"[{env}] data at {Path.GetFullPath(dataDir)}");
```

## Common Pitfalls

**1. NuGet package installed into the wrong project.** In a multi-project solution, `dotnet add package` applies to the project in the current directory (or the one you name). If the compiler can't find the namespace, check *which* `.csproj` got the `PackageReference`.

**2. Blocking calls in web handlers.** In ASP.NET Core, use `async` handlers and `await` for anything slow (database, HTTP). A blocked thread serves nobody; this is the Chapter 16 material becoming a job skill.

**3. Storing state in server memory and expecting it to persist.** The `List<TaskItem>` above resets on every restart and breaks with multiple server instances. Real APIs persist to a database — that's the EF Core step of your path, not a flaw in your understanding.

**4. Grabbing unmaintained packages.** Before adopting a package check downloads, last update, and license on nuget.org. Prefer packages from known publishers; every dependency is code you now effectively own.

**5. Tutorial paralysis.** The ecosystem is huge — Blazor! MAUI! Unity! — and it's tempting to sample everything shallowly. Employability comes from depth in one lane (recommendation: web APIs) with breadth added later.

## Practice Exercises

1. Create a console app, add `Spectre.Console` from NuGet, and render your Chapter 15 per-customer order summary as a styled table. Then run `dotnet list package` and inspect what changed in the `.csproj`.
2. Build the TaskApi from this chapter and exercise every route with curl (or a REST client). Add two features: `GET /tasks/pending` (LINQ filter) and `PUT /tasks/{id}/done` marking a task complete (think: which status codes for success and missing id?).
3. Add input validation to the POST route: reject empty/whitespace titles and titles over 100 chars with `Results.BadRequest("...")`. Verify with curl that valid input still works and both invalid cases return 400.
4. Write a small console client for your own API: use `HttpClient` + `System.Text.Json` to GET `/tasks`, deserialize into records, and print them — your Chapter 16 skills talking to your Chapter 18 API.
5. Research exercise (write answers as a markdown file in the project): pick your lane. Compare the job listings in your area for "C# web developer" vs "Unity developer" vs ".NET desktop" — note 5 recurring skills employers ask for, and map each to either a chapter of this track or a named next-step technology.
