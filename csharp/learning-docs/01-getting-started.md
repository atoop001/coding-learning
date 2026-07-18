# Chapter 1: Getting Started with C# and .NET

## Overview

C# (pronounced "C sharp") is a modern, statically typed, general-purpose programming language created by Microsoft. It runs on **.NET**, a free, open-source, cross-platform runtime and framework. C# is one of the most in-demand languages for professional software jobs: it powers enterprise web backends (ASP.NET Core), desktop apps, cloud services on Azure, and most games built with Unity.

If you know some Python or JavaScript, the biggest shift is that C# is **compiled** and **statically typed**: the compiler checks your types before the program ever runs, catching a whole class of bugs early. In exchange for a bit more ceremony, you get speed, tooling that can autocomplete almost everything, and refactoring safety that dynamic languages can't match.

By the end of this chapter you will have the .NET SDK installed on Windows, a working editor, and your first running console app.

## Definitions & Explanations

- **.NET SDK** — The Software Development Kit: the compiler, the `dotnet` command-line tool, and the base libraries. You need this to *build* C# programs.
- **.NET Runtime** — The part that *runs* compiled programs. Installing the SDK includes the runtime.
- **CLR (Common Language Runtime)** — The virtual machine at the heart of .NET. Like Python's interpreter or the JavaScript engine in Node, it manages memory (garbage collection), safety checks, and execution — but it runs *compiled* code, not source text.
- **IL (Intermediate Language)** — When you build a C# program, the compiler produces IL, a portable low-level bytecode. At run time the CLR's **JIT (Just-In-Time) compiler** turns IL into native machine code for your CPU. This is why the same build runs on Windows, Linux, and macOS.
- **Solution (`.sln`)** — A container that groups one or more projects (like a workspace).
- **Project (`.csproj`)** — One buildable unit: an app or a library. An XML file lists its settings and package dependencies. Unlike Python, you don't just run loose `.py` files — code lives inside a project.
- **`dotnet` CLI** — The command-line entry point for everything: creating projects, building, running, testing, and managing packages.

### How a C# program differs from Python/JavaScript

| Concept | Python/JS | C# |
|---|---|---|
| Execution | Interpreter runs source directly | Compiler → IL → JIT → machine code |
| Types | Checked at run time (dynamic) | Checked at compile time (static) |
| Entry point | Top of the file just runs | A `Main` method (or top-level statements in one file) |
| Statements end with | Newline | Semicolon `;` |
| Blocks | Indentation (Python) / braces (JS) | Braces `{ }` — indentation is style only |
| Naming | `snake_case` (Py) / `camelCase` (JS) | `PascalCase` for types & methods, `camelCase` for locals |

## Installing the .NET SDK on Windows

1. Go to <https://dotnet.microsoft.com/download> and download the latest **.NET SDK** (LTS version recommended) for Windows x64.
2. Run the installer with default options.
3. Open a new terminal (PowerShell or Windows Terminal) and verify:

```powershell
dotnet --version
# Expected output: something like 8.0.404 (any recent version is fine)
```

Alternatively, with winget: `winget install Microsoft.DotNet.SDK.8`

## Choosing an Editor

- **Visual Studio Code** (recommended to start): lightweight. Install the **C# Dev Kit** extension from the Extensions panel. Free.
- **Visual Studio Community**: the full IDE — heavier, but the best debugger and designer tools. Free for individuals. Worth installing later, especially for ASP.NET Core work.

Either works for this whole track; the chapters assume VS Code + terminal.

## Code Examples

### Your first console app

```powershell
# Create a folder, then a new console project inside it
mkdir HelloCSharp
cd HelloCSharp
dotnet new console

# Run it
dotnet run
```

`dotnet new console` generates `Program.cs`:

```csharp
// Program.cs — top-level statements: no class/Main boilerplate needed
// (the compiler generates the Main method for you behind the scenes)
Console.WriteLine("Hello, World!");
```

`Console.WriteLine` is C#'s `print()` / `console.log()`.

### The classic explicit form

Before .NET 6, every program looked like this — and you will still see it everywhere, so learn to read it:

```csharp
using System;                      // import the System namespace (like Python's `import`)

namespace HelloCSharp              // a namespace groups related code (Chapter 11)
{
    class Program                  // all C# code lives inside a class
    {
        // Main is the entry point: execution starts here
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, World!");   // every statement ends with ;
        }
    }
}
```

### Reading input and using variables

```csharp
// A slightly more interesting program
Console.Write("What is your name? ");      // Write = no newline; WriteLine = newline
string? name = Console.ReadLine();         // ReadLine returns the typed text (or null)

Console.Write("What year were you born? ");
string? input = Console.ReadLine();
int year = int.Parse(input!);              // convert text to a number (like Python's int())

int age = 2026 - year;
Console.WriteLine($"Hello {name}, you turn {age} this year.");  // $"..." = interpolation
```

Note the types written *before* the variable names (`string name`, `int year`). That's static typing: `name` can only ever hold a string.

### Useful CLI commands

```powershell
dotnet new console -n MyApp   # create project in a new folder MyApp
dotnet build                  # compile without running (output goes to bin/)
dotnet run                    # build + run
dotnet new list               # see all project templates (console, classlib, webapi...)
```

## Common Pitfalls

**1. Forgetting semicolons.** Every statement needs one.

```csharp
Console.WriteLine("Hi")     // ❌ error CS1002: ; expected
Console.WriteLine("Hi");    // ✅
```

**2. Case sensitivity — including built-ins.** `console.writeline` does not exist.

```csharp
console.writeline("Hi");    // ❌ CS0103: 'console' does not exist
Console.WriteLine("Hi");    // ✅ C# is case-sensitive and uses PascalCase
```

**3. Running `dotnet run` in the wrong folder.** The command looks for a `.csproj` in the current directory. If you see "Couldn't find a project to run", `cd` into the project folder first.

**4. Expecting Python-style loose scripts.** You can't run a lone `.cs` file with `dotnet run`; code must be part of a project. (Create one in seconds with `dotnet new console`.)

**5. Treating `ReadLine` input as a number.** Input is always a `string`; convert it explicitly.

```csharp
int year = Console.ReadLine();              // ❌ cannot convert string to int
int year = int.Parse(Console.ReadLine()!);  // ✅ explicit conversion
```

## Practice Exercises

1. Install the .NET SDK, verify with `dotnet --version`, and create and run a console app named `Playground`.
2. Modify `Program.cs` to print your name, your favorite language so far, and one reason you're learning C# — on three separate lines.
3. Write a program that asks for the user's first and last name separately, then greets them with both in one sentence using string interpolation.
4. Write a program that asks for two numbers and prints their sum, difference, and product. (Use `int.Parse` on each input.)
5. Explore: run `dotnet new list` and identify which template you would use for (a) a reusable library, (b) a web API, (c) unit tests. Write your answers as comments in `Program.cs`.
