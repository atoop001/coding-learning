# Chapter 3: Strings & F-Strings

## Overview

Text is everywhere in programming: user input, file contents, web pages, error messages, database records. Python's string type (`str`) is powerful and pleasant to use, and mastering it pays off immediately — nearly every program you write in this track manipulates text somewhere.

This chapter covers creating strings, indexing and slicing, the most useful string methods, and **f-strings** — Python's modern way to build text from values, which you will use in virtually every script from now on.

## Definitions & Explanations

**String (`str`)** — an ordered sequence of characters. Create one with single quotes `'hi'`, double quotes `"hi"` (identical meaning — pick one and be consistent), or triple quotes `"""..."""` for text spanning multiple lines.

**Immutability** — strings cannot be changed in place. Every "modification" (uppercasing, replacing, concatenating) creates a *new* string. `name[0] = "X"` is an error. If you want a changed string, assign the result: `name = name.upper()`.

**Index** — each character has a position, starting at **0**. `"cat"[0]` is `"c"`. Negative indexes count from the end: `[-1]` is the last character.

**Slice** — a substring extracted with `s[start:stop]`. It includes `start`, **excludes** `stop`. `"python"[0:2]` → `"py"`. Omit either bound to mean "from the beginning" / "to the end": `s[:3]`, `s[3:]`. A third number is the step: `s[::2]` takes every other character; `s[::-1]` reverses the string.

**Method** — a function attached to a value, called with dot syntax: `"Hi".lower()`. Strings have dozens; the essential ones:

| Method | What it does | Example → result |
|---|---|---|
| `.lower()` / `.upper()` | change case | `"Hey".lower()` → `"hey"` |
| `.strip()` | remove whitespace at both ends | `"  hi \n".strip()` → `"hi"` |
| `.replace(old, new)` | swap substrings | `"a-b".replace("-", "_")` → `"a_b"` |
| `.split(sep)` | break into a list | `"a,b,c".split(",")` → `["a","b","c"]` |
| `sep.join(list)` | glue a list into one string | `", ".join(["a","b"])` → `"a, b"` |
| `.startswith(x)` / `.endswith(x)` | boolean checks | `"file.txt".endswith(".txt")` → `True` |
| `.find(x)` | index of first match, `-1` if absent | `"hello".find("ll")` → `2` |
| `.count(x)` | how many times x appears | `"banana".count("a")` → `3` |
| `.isdigit()` | all characters are digits? | `"123".isdigit()` → `True` |
| `.title()` / `.capitalize()` | capitalization helpers | `"bob smith".title()` → `"Bob Smith"` |

**f-string (formatted string literal)** — a string prefixed with `f` where `{expression}` placeholders are evaluated and inserted: `f"Hello, {name}!"`. Inside the braces you can put any expression — variables, arithmetic, method calls — and add **format specs** after a colon:

- `{value:.2f}` — float with 2 decimal places
- `{value:,}` — thousands separators (`1,234,567`)
- `{value:>10}` — right-align in a field 10 wide (`<` left, `^` center)
- `{value:05d}` — pad an int with zeros to width 5 (`00042`)

**Escape sequences** — special characters written with a backslash: `\n` (newline), `\t` (tab), `\"` (a literal quote inside a double-quoted string), `\\` (a literal backslash). A **raw string** `r"C:\new\folder"` disables escapes — essential for Windows paths and regular expressions.

**`len()`** — built-in function returning the number of characters: `len("hello")` → `5`.

**`in` operator** — membership test: `"py" in "python"` → `True`.

## Code Examples

### Creating and combining strings

```python
first = "Ada"
last = 'Lovelace'                 # single or double quotes — same thing

# Concatenation with + (works, but f-strings are usually better)
full = first + " " + last
print(full)                       # Ada Lovelace

# Repetition with *
divider = "-" * 30
print(divider)                    # ------------------------------

# Multi-line text with triple quotes
letter = """Dear reader,
Welcome to Python.
Sincerely, the author"""
print(letter)
```

### Indexing and slicing

```python
word = "programming"

print(word[0])        # p        (first character)
print(word[-1])       # g        (last character)
print(word[0:3])      # pro      (indexes 0,1,2 — stop is excluded)
print(word[3:])       # gramming (from index 3 to the end)
print(word[:4])       # prog     (start to index 3)
print(word[::2])      # pormig   (every 2nd character)
print(word[::-1])     # gnimmargorp (reversed)
print(len(word))      # 11
```

### F-strings — the workhorse

```python
name = "Sam"
score = 87.4567
balance = 1234567.891

print(f"Player {name} scored {score} points")
print(f"Rounded: {score:.1f}")            # Rounded: 87.5
print(f"Balance: ${balance:,.2f}")        # Balance: $1,234,567.89
print(f"Next year the score doubles to {score * 2:.0f}")   # expressions work
print(f"Shout: {name.upper()}!")          # method calls work too

# Alignment — handy for simple tables
for item, price in [("apple", 0.5), ("watermelon", 4.25)]:
    print(f"{item:<12} ${price:>6.2f}")
# apple        $  0.50
# watermelon   $  4.25

# Debugging shortcut: {var=} prints both the name and the value
print(f"{score=}")                        # score=87.4567
```

### Cleaning and validating user input

```python
# A very common real-world pattern:
raw = input("Enter your email: ")         # e.g. "  Alice@Example.COM "

email = raw.strip().lower()               # methods chain left to right
print(f"Normalized: {email}")             # alice@example.com

if "@" in email and email.endswith(".com"):
    print("Looks plausible.")
else:
    print("That doesn't look like a .com email.")
```

### Splitting and joining

```python
csv_line = "2026-07-17,groceries,45.90"

parts = csv_line.split(",")               # ['2026-07-17', 'groceries', '45.90']
date, category, amount = parts            # unpack into three variables
print(f"{category} on {date}: ${float(amount):.2f}")

# join is the reverse — note the separator comes FIRST
tags = ["python", "web", "backend"]
print(" | ".join(tags))                   # python | web | backend
```

### A realistic mini-program

```python
# initials.py — build initials and a username from a full name

full_name = input("Full name: ").strip()          # "  grace brewster hopper "
words = full_name.split()                          # split() with no arg splits on any whitespace

initials = ""
for w in words:                                    # loops are formally Chapter 5
    initials += w[0].upper()

username = (words[0][0] + words[-1]).lower()

print(f"Initials: {initials}")                     # GBH
print(f"Suggested username: {username}")           # ghopper
```

## Common Pitfalls

**1. Trying to change a string in place**

```python
s = "cat"
# s[0] = "b"           # TypeError: 'str' object does not support item assignment
s = "b" + s[1:]        # RIGHT: build a new string
```

**2. Forgetting methods return new strings**

```python
name = "alice"
name.upper()           # computes "ALICE"... and throws it away
print(name)            # alice — unchanged!

name = name.upper()    # RIGHT: capture the result
```

**3. Concatenating strings with numbers**

```python
age = 30
# print("Age: " + age)         # TypeError
print("Age: " + str(age))      # works
print(f"Age: {age}")           # better — f-strings convert automatically
```

**4. Off-by-one confusion with slices**

`s[2:5]` gives characters at indexes 2, 3, 4 — *not* 5. Remember: the stop index is where the slice *stops before*. Helpful check: `s[a:b]` always has length `b - a` (when in range).

**5. Forgetting the `f` prefix**

```python
name = "Kim"
print("Hello, {name}")     # prints literally: Hello, {name}
print(f"Hello, {name}")    # prints: Hello, Kim
```

**6. Windows paths without raw strings**

```python
# path = "C:\new\test.txt"    # \n becomes a newline, \t becomes a tab!
path = r"C:\new\test.txt"     # raw string: backslashes kept literally
```

**7. Comparing without normalizing**

```python
answer = input("Continue? (yes/no) ")
if answer == "yes":                    # fails for "Yes", "YES", " yes "
    ...
if answer.strip().lower() == "yes":    # robust
    ...
```

## Practice Exercises

1. **Name formatter.** Ask for a full name in any messy capitalization (e.g. `"  jOHN roNALD reuel TOLKIEN "`). Print it cleaned and in title case, plus the number of characters in the cleaned name (excluding spaces).
2. **Slice practice.** Given `s = "abcdefghij"`, use slicing to produce: the first three characters, the last three, every third character, the string reversed, and the middle four characters. Print each.
3. **Simple censor.** Ask the user for a sentence and a word to censor. Print the sentence with every occurrence of that word replaced by `***`, case-insensitively is a bonus.
4. **Receipt line.** Using one f-string per line, print a three-line receipt where item names are left-aligned in 15 characters and prices right-aligned in 8 with two decimals — the decimal points should line up vertically.
5. **Email splitter.** Ask for an email address and print the username part (before `@`) and the domain part (after `@`) separately. Handle extra spaces around the input. What does your program do if there's no `@` at all? (Just observe — proper error handling comes in Chapter 12.)
