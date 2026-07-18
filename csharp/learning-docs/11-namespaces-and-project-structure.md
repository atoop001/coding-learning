# Chapter 11: Namespaces & Project Structure

## Overview

Until now you've probably kept everything in one `Program.cs`. Real projects split code across many files, folders, and even multiple projects, and **namespaces** keep names organized and collision-free — like Python packages/modules or JS modules, but decoupled from the file system. This chapter covers namespaces, `using` directives, multi-file and multi-project solutions, and the standard layout of a professional C# codebase.

## Definitions & Explanations

### Namespaces

A namespace is a named container for types:

```csharp
// File: Geometry/Circle.cs
namespace MyApp.Geometry;          // file-scoped namespace (C# 10+) — applies to whole file

public class Circle
{
    public double Radius { get; set; }
}
```

Elsewhere, use the type by fully qualified name or a `using`:

```csharp
// File: Program.cs
using MyApp.Geometry;              // like Python's `from ... import *` for one namespace

var c = new Circle { Radius = 2 };
// without the using: var c = new MyApp.Geometry.Circle { Radius = 2 };
```

Key contrasts with Python/JS:
- **No per-type imports needed within the same namespace.** Two files in `MyApp.Geometry` see each other automatically.
- `using` imports a *namespace* (all its types), not a file. There is no file path in the import.
- Namespaces are conventionally `CompanyOrApp.Area.Subarea` and usually mirror the folder structure (`MyApp/Geometry/Circle.cs` → `MyApp.Geometry`), but that's convention, not law.

The older braced style means the same thing — you'll see it in existing code:

```csharp
namespace MyApp.Geometry
{
    public class Circle { /* ... */ }
}
```

### using directives, global usings, aliases

```csharp
using System.Text;                        // ordinary using
using static System.Math;                 // now: Sqrt(2) instead of Math.Sqrt(2)
using Db = MyApp.Data.DatabaseConnection; // alias to shorten/disambiguate
```

Modern templates enable **implicit usings**: common namespaces (`System`, `System.Collections.Generic`, `System.Linq`, ...) are auto-imported project-wide. That's why `List<T>` worked without any `using` line. You can add your own in any file:

```csharp
global using MyApp.Geometry;   // applies to the entire project
```

### Access modifiers at the type level

- `public class` — usable from other projects.
- `internal class` — usable only within *this* project (this is the **default** if you write no modifier!). A frequent surprise: your class "can't be found" from another project because it's implicitly internal.

### One project vs many: solutions

- **Project (`.csproj`)** = one build output: an executable (`console`, `web`) or a **class library** (`classlib`, a `.dll` others reference).
- **Solution (`.sln`)** = a set of projects developed together.

Typical professional layout:

```
MySolution/
├─ MySolution.sln
├─ src/
│  ├─ MyApp/                  (console or web app)
│  │  ├─ MyApp.csproj
│  │  └─ Program.cs
│  └─ MyApp.Core/             (class library: the domain logic)
│     ├─ MyApp.Core.csproj
│     └─ Models/
│        └─ Customer.cs
└─ tests/
   └─ MyApp.Core.Tests/       (xUnit test project — Chapter 17)
```

Why split? The library holds reusable, UI-free logic; the app is a thin shell; tests target the library. This is *the* shape you'll see in workplaces.

### Building a multi-project solution with the CLI

```powershell
mkdir MySolution; cd MySolution
dotnet new sln -n MySolution

dotnet new console  -o src/MyApp
dotnet new classlib -o src/MyApp.Core

dotnet sln add src/MyApp src/MyApp.Core

# Let the app USE the library:
dotnet add src/MyApp reference src/MyApp.Core

dotnet build          # builds everything in dependency order
dotnet run --project src/MyApp
```

After the reference is added, `using MyApp.Core;` in the app gives access to the library's **public** types.

## Code Examples

### A small app split across files and namespaces

```csharp
// src/MyApp.Core/Models/Contact.cs
namespace MyApp.Core.Models;

public class Contact                         // public: visible to the app project
{
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
    public override string ToString() => $"{Name} <{Email}>";
}
```

```csharp
// src/MyApp.Core/Services/ContactBook.cs
namespace MyApp.Core.Services;

using MyApp.Core.Models;                     // usings can sit below the namespace too

public class ContactBook
{
    private readonly List<Contact> _contacts = new();   // private state stays hidden

    public void Add(Contact c) => _contacts.Add(c);

    public IReadOnlyList<Contact> All => _contacts;     // expose read-only view

    public Contact? FindByName(string name) =>
        _contacts.Find(c => c.Name.Equals(name, StringComparison.OrdinalIgnoreCase));
}
```

```csharp
// src/MyApp/Program.cs
using MyApp.Core.Models;
using MyApp.Core.Services;

var book = new ContactBook();
book.Add(new Contact { Name = "Ada Lovelace", Email = "ada@math.org" });
book.Add(new Contact { Name = "Grace Hopper", Email = "grace@navy.mil" });

Console.WriteLine($"Contacts: {book.All.Count}");
Console.WriteLine(book.FindByName("ada lovelace") ?? new Contact { Name = "(not found)" });
```

### Resolving a name collision with an alias

```csharp
using WinTimer = System.Windows.Forms.Timer;    // two libraries both define "Timer"
using ThreadTimer = System.Threading.Timer;

// Now each is unambiguous:
// var t = new ThreadTimer(...);
```

## Common Pitfalls

**1. "Type or namespace could not be found" (CS0246).** Checklist, in order: (a) missing `using` line? (b) type is `internal` in another project? Make it `public`. (c) missing project reference? `dotnet add reference ...`. (d) typo/case.

**2. Forgetting classes default to internal.**

```csharp
// In MyApp.Core:
class Contact { }          // ❌ internal by default — invisible to MyApp
public class Contact { }   // ✅
```

**3. Namespace/folder drift.** Nothing *forces* `Services/ContactBook.cs` to use `...Services`, but tooling and humans assume it. When you move a file, update its namespace (IDEs offer to do this).

**4. Circular project references.** `A` referencing `B` while `B` references `A` won't build. Push the shared types down into a third project both can reference (commonly `X.Core` or `X.Shared`).

**5. One giant file.** The compiler is fine with 3,000-line `Program.cs`; your future self is not. Convention: **one public type per file, file named after the type** (`Contact.cs` contains `Contact`).

**6. Two classes with the same name in one namespace.** CS0101. Either rename or move one to a different namespace — this is exactly the collision namespaces exist to prevent.

## Practice Exercises

1. Take any previous multi-class exercise (e.g., the shapes from Chapter 9) and refactor it into one-file-per-type under a `Shapes/` folder with namespace `MyApp.Shapes`. Confirm it still runs.
2. Build the solution layout from this chapter with the CLI: a solution, a console app, and a class library, with the reference wired up. Put a `MathUtils` class with a `Clamp` method in the library and call it from the app.
3. Deliberately break things to learn the errors: (a) remove a `using` line, (b) change a library class to `internal`, (c) remove the project reference from the `.csproj`. Record each compiler error code and message in a notes comment, then fix all three.
4. Add a second class library `MyApp.Reporting` that references `MyApp.Core` and provides a `ContactReport` class producing a formatted string from a `ContactBook`. Wire it into the app. Draw the dependency arrows as an ASCII diagram in a comment.
5. Create a name collision on purpose: define `class Logger` in two different namespaces, reference both namespaces from `Program.cs`, and resolve the ambiguity twice — once with a fully qualified name and once with a `using` alias.
