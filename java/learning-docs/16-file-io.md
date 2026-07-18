# Chapter 16: File I/O and try-with-resources

## Overview

Programs that can't read or write files are toys. This chapter covers modern Java file handling: the `java.nio.file` API (`Path`, `Files`) that superseded the old `File`/`FileReader` classes, reading and writing text, walking directories, and the **try-with-resources** statement — Java's equivalent of Python's `with open(...)` — which guarantees files get closed. File I/O is also where Chapter 14's checked exceptions stop being theoretical: nearly every operation throws `IOException`.

## Definitions & Explanations

### Path — a location in the filesystem

```java
import java.nio.file.Path;

Path p = Path.of("data", "notes.txt");         // relative: data/notes.txt
Path abs = Path.of("D:/atoop/data/notes.txt"); // absolute — forward slashes work on Windows!
p.getFileName();      // notes.txt
p.getParent();        // data
p.toAbsolutePath();   // resolved against the current working directory
p.resolve("sub")      // path arithmetic: data/notes.txt/sub
```

A `Path` is just a description — creating one touches no disk. Prefer forward slashes in literals even on Windows; Java translates. (Backslashes require escaping: `"data\\notes.txt"`.)

**Relative paths resolve against the working directory** — where you *ran* the program, not where the `.java` file lives. In IntelliJ that's the project root by default. Most "file not found" confusion is really working-directory confusion; print `Path.of("").toAbsolutePath()` to see where you are.

### Files — the verbs

`java.nio.file.Files` is a static utility class with almost everything you need:

```java
// Reading
String all = Files.readString(path);              // whole file as one String
List<String> lines = Files.readAllLines(path);    // list of lines

// Writing
Files.writeString(path, "content");                             // create/overwrite
Files.writeString(path, "more", StandardOpenOption.APPEND);     // append
Files.write(path, listOfLines);                                 // lines with newlines

// Filesystem
Files.exists(path);  Files.size(path);  Files.createDirectories(dirPath);
Files.copy(src, dst);  Files.move(src, dst);  Files.delete(path);
Files.list(dir)      // Stream<Path> of directory entries (close it! see below)
```

All of these throw **`IOException`** (checked) — catch or declare, per Chapter 14.

### Resources and try-with-resources

Files, network sockets, database connections are **resources**: they hold OS handles that must be *closed*, even when exceptions fly. The old solution was `finally { reader.close(); }` — verbose and error-prone. The modern solution:

```java
try (BufferedReader reader = Files.newBufferedReader(path)) {
    // use reader
} catch (IOException e) {
    // handle
}   // reader.close() called AUTOMATICALLY here — success OR exception
```

- Anything implementing `AutoCloseable` can go in the parentheses (that's the functional-interface-style contract: one method, `close()`).
- Multiple resources: separate with `;` — they close in reverse order.
- This is Java's `with open(f) as fh:`. **Always** use it for resources; a bare `new FileReader(...)` with manual close is a code-review flag.

### Streaming large files

`Files.readString` loads everything into memory — fine for configs, wrong for gigabytes. For big files, stream line by line:

```java
try (Stream<String> lines = Files.lines(path)) {     // lazy — reads as you consume
    long errors = lines.filter(l -> l.contains("ERROR")).count();
}
```

`Files.lines` and `Files.list` hold the file open — they belong in try-with-resources too (easy to forget since other streams don't need closing).

### Scanner and console vs file

`Scanner` (Chapter 2) works on files as well: `new Scanner(Files.newBufferedReader(path))` — handy for token-by-token parsing. For structured formats in the real world (JSON, CSV with quoting), you'd use a library (Jackson, OpenCSV) via Maven — Chapter 18; hand-rolling `split(",")` is fine for learning and simple data.

## Code Examples

### Write, read, append — round trip

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.List;

public class FileRoundTrip {
    public static void main(String[] args) {
        Path notes = Path.of("notes.txt");

        try {
            // Write (creates or overwrites)
            Files.writeString(notes, "line one\nline two\n");

            // Append
            Files.writeString(notes, "line three\n", StandardOpenOption.APPEND);

            // Read back — whole file
            System.out.println("--- readString ---");
            System.out.print(Files.readString(notes));

            // Read back — as lines
            System.out.println("--- readAllLines ---");
            List<String> lines = Files.readAllLines(notes);
            for (int i = 0; i < lines.size(); i++) {
                System.out.println((i + 1) + ": " + lines.get(i));
            }

            System.out.println("size in bytes: " + Files.size(notes));
        } catch (IOException e) {
            System.err.println("File trouble: " + e.getMessage());
        }
    }
}
```

### CSV parsing into objects (chapters 8 + 12 + 14 + 16 combined)

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.ArrayList;
import java.util.List;

record Student(String name, int score) { }

public class CsvLoader {

    // Parse one line, with defensive error handling
    static Student parseLine(String line) {
        String[] parts = line.split(",");
        if (parts.length != 2) {
            throw new IllegalArgumentException("expected 2 fields: " + line);
        }
        return new Student(parts[0].strip(), Integer.parseInt(parts[1].strip()));
    }

    public static void main(String[] args) throws IOException {
        Path csv = Path.of("students.csv");
        // Create sample data if missing, so the demo is self-contained
        if (!Files.exists(csv)) {
            Files.writeString(csv, "ada, 91\nbob, 78\ncy, 84\nBAD LINE\ndee, 67\n");
        }

        List<Student> students = new ArrayList<>();
        int skipped = 0;

        for (String line : Files.readAllLines(csv)) {
            if (line.isBlank()) continue;
            try {
                students.add(parseLine(line));
            } catch (IllegalArgumentException e) {     // covers NumberFormatException too? No —
                skipped++;                              // NFE is a subclass of IAE! (nice trivia)
                System.err.println("Skipping bad row: " + e.getMessage());
            }
        }

        double avg = students.stream().mapToInt(Student::score).average().orElse(0);
        System.out.printf("Loaded %d students (skipped %d). Average: %.1f%n",
                students.size(), skipped, avg);
    }
}
```

### Buffered writing and directory walking

```java
import java.io.BufferedWriter;
import java.io.IOException;
import java.nio.file.*;
import java.util.stream.Stream;

public class ReportWriter {
    public static void main(String[] args) {
        Path outDir = Path.of("reports");
        Path report = outDir.resolve("summary.txt");

        try {
            Files.createDirectories(outDir);            // mkdir -p; fine if it exists

            // Buffered writer for many small writes; auto-closed by try-with-resources
            try (BufferedWriter w = Files.newBufferedWriter(report)) {
                w.write("Report generated");
                w.newLine();                            // platform-correct newline
                for (int i = 1; i <= 5; i++) {
                    w.write("item " + i);
                    w.newLine();
                }
            }                                           // <- closed & flushed HERE

            // List .txt files in the directory — stream needs closing, so: try-with-resources
            try (Stream<Path> entries = Files.list(outDir)) {
                entries.filter(p -> p.toString().endsWith(".txt"))
                       .forEach(p -> System.out.println("found: " + p));
            }
        } catch (IOException e) {
            System.err.println("Report failed: " + e.getMessage());
        }
    }
}
```

## Common Pitfalls

### 1. Not closing resources (or closing them manually)

```java
BufferedReader r = Files.newBufferedReader(path);   // ❌ leak if an exception skips close()
r.close();

try (BufferedReader r = Files.newBufferedReader(path)) { ... }   // ✅ always
```

Leaked handles exhaust OS limits and, on Windows specifically, keep files *locked* — your own program can't delete or rewrite a file it forgot to close.

### 2. Wrong working directory assumptions

```java
Files.readString(Path.of("data.txt"));   // ❌ NoSuchFileException — where IS data.txt?
System.out.println(Path.of("").toAbsolutePath());   // ✅ debug: print the working dir first
```

Run configuration in IntelliJ ("Working directory") and your PowerShell `cd` location both change what "data.txt" means.

### 3. Hardcoded backslashes

```java
Path.of("data\notes.txt")     // ❌ \n is a NEWLINE — instant bad path
Path.of("data\\notes.txt")    // works, Windows-only style
Path.of("data/notes.txt")     // ✅ portable, works on Windows too
Path.of("data", "notes.txt")  // ✅ best: let Path join
```

### 4. Ignoring IOException because the compiler nagged

```java
try { Files.writeString(p, s); } catch (IOException e) { }   // ❌ silent data loss
```

Chapter 14's rules apply doubly here — I/O fails in real life (disk full, file locked by Excel, permissions). Log it, tell the user, or rethrow wrapped.

### 5. readString on huge files

```java
Files.readString(Path.of("10gb.log"));               // ❌ OutOfMemoryError
try (Stream<String> lines = Files.lines(path)) { }   // ✅ stream it
```

### 6. Forgetting `Files.lines`/`Files.list` need closing

```java
long n = Files.lines(path).count();          // ❌ file handle leaks
try (var lines = Files.lines(path)) {        // ✅
    long n = lines.count();
}
```

## Practice Exercises

1. **Journal.** A console app: each run asks for a journal entry and appends it to `journal.txt` with a timestamp (`java.time.LocalDateTime.now()`). A `read` command prints all past entries numbered. Confirm entries survive across runs — your first persistent program.
2. **Word-count utility.** Read any text file (path from `args[0]`) and print: line count, word count, character count, and the 5 most frequent words (combine Chapter 12's frequency map with sorting by value — research `Map.Entry.comparingByValue`). Handle a missing file with a clean one-line error, not a stack trace.
3. **CSV round trip.** Extend `CsvLoader`: after loading students, add a computed letter grade to each and write `report.csv` with a header row (`name,score,grade`). Re-open the file you wrote and verify it parses back to the same students. Where do bad rows in the *input* end up — and should they appear in the output? (Decide and document.)
4. **File organizer (careful: operates on real files!).** In a scratch directory you create, generate 10 empty files with mixed extensions (`.txt`, `.jpg`, `.pdf`) using `Files.createFile`, then write a program that sorts them into subfolders by extension (`Files.move` + `createDirectories`). Print each move. Run it twice — the second run should do nothing gracefully.
5. **Try-with-resources mechanics.** Write a class `Noisy implements AutoCloseable` whose constructor and `close()` both print. Open three in one try-with-resources statement and observe construction and close order (close is reverse!). Then throw an exception mid-block and confirm all three still close. Summarize the guarantees in a comment.
