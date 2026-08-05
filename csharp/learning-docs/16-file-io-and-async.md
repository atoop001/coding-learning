# Chapter 16: File I/O & Async/Await Basics

## Overview

Programs need to persist data and talk to the outside world. This chapter covers reading and writing files (text, lines, and JSON), paths and directories, the `using` statement for reliable cleanup, and then introduces **async/await** — C#'s way to wait on slow operations (disk, network) without freezing the program. If you've seen JS's `async/await`, C#'s version will look nearly identical; it should, since JavaScript borrowed it *from* C#.

## Definitions & Explanations

### The File class — one-liners for common cases

```csharp
using System.IO;    // implicit in modern templates

// Whole-file operations (fine for small/medium files)
File.WriteAllText("notes.txt", "Hello file!");     // create/overwrite
File.AppendAllText("notes.txt", "\nMore text");
string content = File.ReadAllText("notes.txt");

File.WriteAllLines("list.txt", new[] { "alpha", "beta" });
string[] lines = File.ReadAllLines("list.txt");    // one string per line

File.Exists("notes.txt")     // ALWAYS check before reading
File.Delete("notes.txt");
File.Copy("a.txt", "b.txt", overwrite: true);
```

### Paths — never concatenate strings

```csharp
string path = Path.Combine("data", "2026", "log.txt");   // data\2026\log.txt on Windows
Path.GetFileName(path);        // log.txt
Path.GetExtension(path);       // .txt
Path.GetFullPath(path);        // absolute version

Directory.CreateDirectory("data/2026");     // mkdir -p behavior (no error if exists)
string[] files = Directory.GetFiles("data", "*.txt");
```

`Path.Combine` handles separators and platform differences — string `+` with `"\\"` does not.

### Streams and `using` — for big files and guaranteed cleanup

Files are OS resources that must be released. Types that hold such resources implement `IDisposable`, and the **`using` statement** guarantees `Dispose()` runs — like Python's `with open(...)`:

```csharp
// Read line-by-line without loading the whole file (huge-file friendly)
using (StreamReader reader = new StreamReader("big.log"))
{
    string? line;
    while ((line = reader.ReadLine()) != null)
        Console.WriteLine(line);
}   // file handle closed here, even if an exception occurred

// Modern "using declaration": disposed at end of enclosing scope
using var writer = new StreamWriter("out.txt", append: true);
writer.WriteLine("appended line");
```

### JSON — the standard persistence format

```csharp
using System.Text.Json;

record Contact(string Name, string Email, int Age);

var contacts = new List<Contact> { new("Ada", "ada@x.org", 36) };

// Serialize: objects -> JSON text
string json = JsonSerializer.Serialize(contacts,
    new JsonSerializerOptions { WriteIndented = true });
File.WriteAllText("contacts.json", json);

// Deserialize: JSON text -> objects (returns null on literal "null" json)
var loaded = JsonSerializer.Deserialize<List<Contact>>(
    File.ReadAllText("contacts.json")) ?? new List<Contact>();
```

Note: `JsonSerializer` uses **public properties** (records qualify); private fields are ignored.

By default an `enum` property serializes as its underlying number (e.g. `1`), which is unreadable over the wire. Add a `JsonStringEnumConverter` to the options to serialize (and accept) enums as their name instead:

```csharp
enum Status { Wishlist, Reading, Finished }
record Book(string Title, Status Status);

var options = new JsonSerializerOptions { Converters = { new JsonStringEnumConverter() } };
string json = JsonSerializer.Serialize(new Book("Dune", Status.Reading), options);
// {"Title":"Dune","Status":"Reading"}  <- not "Status":1
```

### Async/await — the 80% you need now

Reading a large file or calling a web API takes time. A **synchronous** call blocks the thread until done. An **`async` method returns a `Task`** — a promise of a future result (JS: literally `Promise`) — and **`await`** pauses *this method* (not the whole program) until the task completes:

```csharp
static async Task Main()                     // Main can be async
{
    Console.WriteLine("Starting download...");
    string text = await File.ReadAllTextAsync("big.txt");   // yields while waiting
    Console.WriteLine($"Read {text.Length} chars");
}
```

Rules of thumb:
- `async` methods return `Task` (no result) or `Task<T>` (a `T` result); `await` unwraps them.
- Awaiting is contagious by design: a method that awaits must be `async`, and its callers typically await it — "async all the way up".
- Most I/O APIs ship an `...Async` twin (`ReadAllTextAsync`, `GetStringAsync`). Prefer them in anything beyond toy programs, and always in web apps (Chapter 18) where blocked threads are wasted capacity.

```csharp
// Concurrency: start several tasks, then await them together
static async Task<int> TotalSizeAsync(string[] paths)
{
    Task<string>[] reads = paths.Select(p => File.ReadAllTextAsync(p)).ToArray();
    string[] contents = await Task.WhenAll(reads);     // all run CONCURRENTLY
    return contents.Sum(c => c.Length);
}
```

```csharp
// The classic demo: an HTTP call (HttpClient is the built-in fetch)
using System.Net.Http;

static async Task Main()
{
    using var http = new HttpClient();
    string body = await http.GetStringAsync("https://api.github.com/zen");
    Console.WriteLine(body);
}
```

## Code Examples

### A persistent high-score table (the full read-modify-write cycle)

```csharp
using System.Text.Json;

record Score(string Player, int Points);

class ScoreBoard
{
    private const string SavePath = "scores.json";
    private List<Score> _scores = new();

    public void Load()
    {
        if (!File.Exists(SavePath)) return;            // first run: nothing saved yet
        try
        {
            string json = File.ReadAllText(SavePath);
            _scores = JsonSerializer.Deserialize<List<Score>>(json) ?? new();
        }
        catch (JsonException)
        {
            Console.WriteLine("Save file corrupted — starting fresh.");
            _scores = new();
        }
    }

    public void Save() =>
        File.WriteAllText(SavePath,
            JsonSerializer.Serialize(_scores, new JsonSerializerOptions { WriteIndented = true }));

    public void Add(string player, int points)
    {
        _scores.Add(new Score(player, points));
        _scores = _scores.OrderByDescending(s => s.Points).Take(10).ToList();  // top 10
    }

    public void Print()
    {
        foreach (var (s, i) in _scores.Select((s, i) => (s, i)))
            Console.WriteLine($"{i + 1,2}. {s.Player,-12} {s.Points,6}");
    }
}

class Program
{
    static void Main()
    {
        var board = new ScoreBoard();
        board.Load();
        board.Add("Ada", 4200);
        board.Add("Linus", 3100);
        board.Print();
        board.Save();          // survives the next run
    }
}
```

### Log analyzer — streaming a big file

```csharp
static async Task Main()
{
    int errors = 0, warnings = 0, total = 0;

    using var reader = new StreamReader("app.log");
    string? line;
    while ((line = await reader.ReadLineAsync()) != null)
    {
        total++;
        if (line.Contains("ERROR")) errors++;
        else if (line.Contains("WARN")) warnings++;
    }

    Console.WriteLine($"{total} lines: {errors} errors, {warnings} warnings");
}
```

### Sequential vs concurrent awaiting

```csharp
static async Task Main()
{
    var sw = System.Diagnostics.Stopwatch.StartNew();

    // Sequential: ~2s total
    await Task.Delay(1000);                   // stand-in for a slow API call
    await Task.Delay(1000);
    Console.WriteLine($"Sequential: {sw.ElapsedMilliseconds}ms");

    sw.Restart();
    // Concurrent: ~1s total — start both, THEN await
    Task t1 = Task.Delay(1000);
    Task t2 = Task.Delay(1000);
    await Task.WhenAll(t1, t2);
    Console.WriteLine($"Concurrent: {sw.ElapsedMilliseconds}ms");
}
```

## Common Pitfalls

**1. Reading a file that doesn't exist.** `FileNotFoundException`. Check `File.Exists` first (or catch it) — especially for the "first run, no save file yet" case.

**2. Forgetting to dispose.** Opening a `StreamWriter` without `using` keeps the file locked — a later read (or a second run) fails with "file in use". Wrap every Stream/Reader/Writer/HttpClient-per-use in `using`.

**3. Backslash strings as paths.**

```csharp
string p = "data\notes.txt";               // ❌ \n is a NEWLINE escape!
string p = Path.Combine("data", "notes.txt");   // ✅ (or @"data\notes.txt")
```

**4. `async void`.** Un-awaitable, exceptions vanish, crashes ensue. The only legitimate use is event handlers. Everything else: `async Task`.

```csharp
static async void SaveAsync() { ... }      // ❌
static async Task SaveAsync() { ... }      // ✅ callers can await it
```

**5. Blocking on async with `.Result` or `.Wait()`.** Mixing styles invites deadlocks (and defeats the purpose). If you're in async code, await; if a method returns `Task`, make your method async too.

**6. Awaiting in a loop when tasks are independent.**

```csharp
foreach (var url in urls)
    results.Add(await http.GetStringAsync(url));       // ❌ one at a time

var tasks = urls.Select(u => http.GetStringAsync(u));  // ✅ concurrent
var results = await Task.WhenAll(tasks);
```

**7. Relative paths point at the working directory** — which for `dotnet run` is the project folder, but for a double-clicked exe may differ. For real apps anchor with `AppContext.BaseDirectory` or `Environment.GetFolderPath(...)`.

## Practice Exercises

1. Write a journal app: each run asks for an entry and appends it to `journal.txt` with a timestamp; a `read` command prints all entries numbered. Handle the missing-file first run gracefully.
2. Build a CSV importer: given `name,score` lines in a file (create sample data by hand, including one malformed line), parse into a record list, skip and report bad lines, then print stats via LINQ (top scorer, average).
3. Extend Chapter 7's to-do list with JSON persistence: load at startup, save after every change. Corrupt the JSON file manually and make sure your program survives with a clear message.
4. Write an async program that "fetches" three resources with `Task.Delay` stand-ins of different durations (500/1000/1500 ms), first sequentially and then with `Task.WhenAll`, printing elapsed time for each strategy. Predict both numbers before running.
5. (Real network) Use `HttpClient` to `await` a GET of `https://api.github.com/zen` five times concurrently and print all responses. Add a try/catch for `HttpRequestException` and test it by breaking the URL.
