# Chapter 11: File I/O & Working with Paths

## Overview

Programs that forget everything when they exit are toys. **File I/O** (input/output) is how programs persist data: reading configuration, saving user records, processing logs, importing CSVs. Nearly every project from here on — the contact book, the expense tracker, the Flask app's data layer — reads and writes files.

This chapter covers opening/reading/writing text files safely with `with`, file modes, line-by-line processing, the modern `pathlib` way of handling file paths (crucial on Windows), and working with CSV files.

## Definitions & Explanations

**Opening a file** — `open(path, mode, encoding="utf-8")` returns a file object. Always pass `encoding="utf-8"` for text files; Windows otherwise defaults to a legacy encoding and mangles special characters.

**Modes:**

| Mode | Meaning | If file exists | If it doesn't |
|---|---|---|---|
| `"r"` | read (default) | reads it | **error** (FileNotFoundError) |
| `"w"` | write | **erases it!** | creates it |
| `"a"` | append | adds to the end | creates it |
| `"x"` | exclusive create | **error** | creates it |
| `"rb"` / `"wb"` | binary read/write | for images, etc. — no encoding | |

**The `with` statement (context manager)** — the *only* way you should open files:

```python
with open("notes.txt", "r", encoding="utf-8") as f:
    content = f.read()
# file is automatically closed here — even if an error occurred inside
```

`with` guarantees the file gets closed. Forgetting to close files causes locked files (very visible on Windows), lost buffered writes, and resource leaks.

**Reading:**

- `f.read()` — the whole file as one string (fine for small files)
- `f.readlines()` — a list of lines (each keeping its trailing `\n`)
- `for line in f:` — **the best way**: streams line by line, works for gigabyte files

**Writing:**

- `f.write(text)` — writes exactly what you give it; **no automatic newline** — add `\n` yourself
- `print(value, file=f)` — print's conveniences (spaces, newline), redirected into a file

**`pathlib.Path`** — the modern, object-oriented way to handle paths. Why not plain strings? Because `"C:\new\data.txt"` has an escape-sequence landmine, `/` vs `\` differs across OSes, and joining with `+` invites missing-separator bugs. `Path` solves all of it:

```python
from pathlib import Path

p = Path("data") / "notes.txt"     # joining with / — works on every OS
p.exists()                          # does it exist?
p.name, p.stem, p.suffix            # 'notes.txt', 'notes', '.txt'
p.parent                            # Path('data')
p.mkdir(parents=True, exist_ok=True)   # make directories, no error if present
p.read_text(encoding="utf-8")       # slurp a file in one call
p.write_text("hi", encoding="utf-8")   # write in one call (overwrites)
Path.cwd()                          # current working directory
p.glob("*.txt")                     # iterate matching files
p.unlink()                          # delete the file
```

`open()` accepts `Path` objects directly.

**Relative vs. absolute paths** — `"data.txt"` is resolved relative to the **current working directory** (where you *ran* Python from — not necessarily where the script lives!). This is the #1 cause of "works in VS Code, fails in the terminal." Robust scripts anchor paths to the script's own location: `Path(__file__).parent / "data.txt"`.

**CSV (comma-separated values)** — the everyday tabular text format. Don't hand-split on commas (quoted fields containing commas will break you) — use the `csv` module: `csv.reader` / `csv.writer` for lists, `csv.DictReader` / `csv.DictWriter` for dicts keyed by the header row. When opening CSV files for the `csv` module, pass `newline=""` to `open()` (prevents blank rows on Windows).

## Code Examples

### Writing, then reading, a text file

```python
from pathlib import Path

path = Path("journal.txt")

# WRITE ("w" creates or ERASES-and-recreates)
with open(path, "w", encoding="utf-8") as f:
    f.write("Day 1: started learning file I/O\n")
    f.write("Day 2: with-blocks are mandatory\n")

# APPEND ("a" adds to the end)
with open(path, "a", encoding="utf-8") as f:
    f.write("Day 3: pathlib is nice\n")

# READ — whole file
with open(path, "r", encoding="utf-8") as f:
    content = f.read()
print(content)

# READ — line by line (preferred for processing)
with open(path, encoding="utf-8") as f:        # "r" is the default mode
    for line_number, line in enumerate(f, 1):
        print(f"{line_number}: {line.strip()}")   # strip() removes the trailing \n
```

### The pathlib shortcuts

```python
from pathlib import Path

notes = Path("quick_note.txt")
notes.write_text("one-liner write\n", encoding="utf-8")
print(notes.read_text(encoding="utf-8"))
print(notes.exists(), notes.suffix, notes.stat().st_size, "bytes")
```

### Anchoring to the script's own folder

```python
from pathlib import Path

# Robust: data lives next to this .py file no matter where you run it from
BASE_DIR = Path(__file__).parent
DATA_FILE = BASE_DIR / "data" / "records.txt"

DATA_FILE.parent.mkdir(parents=True, exist_ok=True)   # ensure data/ exists

with open(DATA_FILE, "a", encoding="utf-8") as f:
    f.write("a record\n")
```

### Processing a file: word count

```python
from pathlib import Path

def word_frequencies(path):
    """Return a dict of word -> count for a text file."""
    counts = {}
    with open(path, encoding="utf-8") as f:
        for line in f:
            for word in line.lower().split():
                word = word.strip(".,!?;:\"'()")     # crude punctuation trim
                if word:
                    counts[word] = counts.get(word, 0) + 1
    return counts

if __name__ == "__main__":
    freqs = word_frequencies("journal.txt")
    for word, n in sorted(freqs.items(), key=lambda kv: kv[1], reverse=True)[:10]:
        print(f"{word:<15} {n}")
```

### CSV — reading and writing properly

```python
import csv
from pathlib import Path

path = Path("expenses.csv")

# WRITE with a header row
rows = [
    {"date": "2026-07-01", "category": "food", "amount": 12.50},
    {"date": "2026-07-02", "category": "travel", "amount": 40.00},
    {"date": "2026-07-02", "category": "food", "amount": 7.25},
]
with open(path, "w", encoding="utf-8", newline="") as f:      # newline="" matters!
    writer = csv.DictWriter(f, fieldnames=["date", "category", "amount"])
    writer.writeheader()
    writer.writerows(rows)

# READ back as dicts
total = 0.0
with open(path, encoding="utf-8", newline="") as f:
    reader = csv.DictReader(f)
    for row in reader:                       # each row is a dict of strings
        total += float(row["amount"])        # CSV values are ALWAYS strings
        print(f"{row['date']}  {row['category']:<8} ${float(row['amount']):>7.2f}")
print(f"{'TOTAL':>18} ${total:>7.2f}")
```

### Listing files in a folder

```python
from pathlib import Path

folder = Path(".")
for p in sorted(folder.glob("*.py")):
    size_kb = p.stat().st_size / 1024
    print(f"{p.name:<30} {size_kb:>6.1f} KB")

# Recursive: every .txt anywhere below this folder
for p in folder.rglob("*.txt"):
    print(p)
```

## Common Pitfalls

**1. Opening with `"w"` when you meant `"a"`** — `"w"` truncates the file to nothing the instant it opens. If your saved data keeps vanishing, this is why. Append (`"a"`) or read-modify-rewrite deliberately.

**2. Not using `with`**

```python
f = open("data.txt", "w", encoding="utf-8")
f.write("hello")
# forgot f.close() — data may sit unflushed; file stays locked on Windows
```

Always `with`. No exceptions in this track.

**3. Backslash escapes in Windows path strings**

```python
# path = "C:\temp\new.txt"       # \t is a TAB, \n is a NEWLINE — broken path
path = r"C:\temp\new.txt"        # raw string, or:
from pathlib import Path
path = Path("C:/temp/new.txt")   # forward slashes work fine on Windows via pathlib
```

**4. Forgetting `\n` on writes**

```python
f.write("line 1")
f.write("line 2")     # file contains: line 1line 2
f.write("line 1\n")   # RIGHT
```

**5. Forgetting to strip `\n` on reads** — lines from a file keep their newline, so `line == "quit"` fails when the file contains `"quit\n"`. Use `line.strip()` (or `rstrip("\n")`) before comparing.

**6. Relative-path confusion** — `FileNotFoundError` for a file you can see in the folder usually means the *working directory* isn't what you think. Print `Path.cwd()` to check; anchor with `Path(__file__).parent` to fix.

**7. Reading a file that may not exist**

```python
from pathlib import Path
path = Path("save.txt")
if path.exists():
    data = path.read_text(encoding="utf-8")
else:
    data = ""            # sensible default for a first run
# (Chapter 12 shows the try/except alternative, which avoids a race condition.)
```

**8. Treating CSV values as numbers** — everything read from CSV is a string. Convert explicitly: `float(row["amount"])`, `int(row["qty"])`.

## Practice Exercises

1. **Log keeper.** Write a script that appends a timestamped line (use `datetime.now()`) to `activity.log` each time it runs, then prints the last 5 lines of the file. Run it several times to watch the log grow.
2. **Line numberer.** Read any `.py` file of yours and write a copy named `<original>_numbered.txt` where every line is prefixed with its right-aligned line number and a colon. Use `pathlib` to build the output name from the input's stem.
3. **Simple todo persistence.** Extend Chapter 5's menu-loop todo program: load tasks from `todo.txt` at startup (one per line, if the file exists) and save them back on quit. Verify tasks survive a restart.
4. **CSV report.** Hand-create `grades.csv` with columns `name,assignment,score` and ~10 rows. Write a script using `csv.DictReader` that prints each student's average and writes a new `summary.csv` with columns `name,average` via `DictWriter`.
5. **Folder inventory.** Write a script that takes a folder path, and for each file in it (non-recursive) prints name, extension, and size — then prints totals: file count and combined size in KB. Bonus: group counts by extension using a dict.
