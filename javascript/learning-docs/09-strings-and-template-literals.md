# Chapter 9: Strings & Template Literals

## Overview

Text is everywhere in web development: usernames, search queries, URLs, error messages, HTML itself. JavaScript **strings** come with a large toolkit of built-in methods for searching, slicing, cleaning, and transforming text, and **template literals** (backtick strings) make building text from data pleasant instead of painful.

You've been using strings since Chapter 1; this chapter makes you fluent: methods you'll reach for daily, how strings interact with arrays (`split`/`join`), the immutability rule that trips up beginners, and a working-minimum introduction to regular expressions.

## Definitions & Explanations

### Three kinds of quotes

```js
const a = "double quotes";
const b = 'single quotes';
const c = `backticks — template literal`;
```

Double and single quotes are equivalent — pick one style and stay consistent. Backticks create **template literals**, which add two superpowers:

1. **Interpolation** — embed any expression with `${...}`:
   ```js
   const name = "Ada";
   console.log(`Hello, ${name}! Next year you'll be ${36 + 1}.`);
   ```
2. **Multi-line strings** — line breaks inside backticks are kept literally.

### Strings are immutable

**You cannot change a string in place.** Every string method returns a **new** string; the original is untouched:

```js
let word = "hello";
word.toUpperCase();     // returns "HELLO"...
console.log(word);      // ...but word is still "hello"
word = word.toUpperCase(); // reassign to keep the result
```

Also, `word[0] = "H"` silently does nothing. To "modify" a string, build a new one.

### Indexing and length

- `str.length` — number of characters.
- `str[0]` or `str.at(0)` — character at an index (0-based).
- `str.at(-1)` — last character (negative indexes work with `.at`, not with brackets).

### The essential method toolkit

**Case & whitespace**
- `toUpperCase()` / `toLowerCase()`
- `trim()` — removes whitespace from both ends (also `trimStart`, `trimEnd`)

**Searching**
- `includes(sub)` — boolean, is `sub` anywhere inside?
- `startsWith(sub)` / `endsWith(sub)`
- `indexOf(sub)` — first position, or `-1` if absent

**Extracting**
- `slice(start, end)` — substring from `start` up to but **not including** `end`; negative indexes count from the end.

**Replacing**
- `replace(a, b)` — replaces the **first** occurrence of `a`
- `replaceAll(a, b)` — replaces **every** occurrence

**Splitting & joining**
- `split(separator)` — string → array (`"a,b,c".split(",")` → `["a","b","c"]`)
- `array.join(separator)` — array → string (the reverse)

**Padding & repeating**
- `padStart(len, ch)` / `padEnd(len, ch)` — pad to a minimum length
- `repeat(n)` — the string, n times

### Comparing strings

`===` compares exactly, case-sensitively: `"Apple" === "apple"` is `false`. For case-insensitive comparison, lowercase both sides first. Alphabetical ordering uses `<` / `>` (by character code) or, better for human-friendly sorting, `a.localeCompare(b)`.

### Regular expressions: a working minimum

A **regular expression** (regex) is a pattern for matching text — useful for validation, searching, and replacing when plain string methods aren't precise enough. This is a *working minimum*: enough to read and write simple patterns confidently. For anything more elaborate, treat MDN's regex guide as the reference — don't try to memorize the whole syntax up front.

**Literal syntax**: a regex is written between slashes, `/pattern/flags`. Common flags: `g` (global — find/replace *every* match, not just the first) and `i` (case-insensitive).

**Character classes** — match one character from a set:
- `[abc]` — any of a, b, or c; `[^abc]` — any character *except* those
- `[a-z]`, `[0-9]` — ranges
- Shorthands: `\d` (digit), `\w` (word character: letter/digit/underscore), `\s` (whitespace) — capitalized forms (`\D`, `\W`, `\S`) negate them
- `.` — any character except a newline

**Quantifiers** — how many times the preceding token repeats:
- `*` — 0 or more, `+` — 1 or more, `?` — 0 or 1
- `{3}` — exactly 3, `{2,4}` — between 2 and 4, `{2,}` — 2 or more

**Anchors** — a position, not a character: `^` start of string, `$` end of string.

**Groups**: `(...)` captures part of a match for reuse or extraction; `(?:...)` groups without capturing.

**The methods you'll actually use**:
- `regex.test(str)` — boolean: does the pattern match anywhere in `str`?
- `str.match(regex)` — match details (or `null`); with the `g` flag, returns *all* matches as an array of strings
- `str.replace(regex, replacement)` — like the string method, but pattern-based; needs the `g` flag to replace every match, not just the first

```js
console.log(/^\d{3}-\d{4}$/.test("555-1234"));      // true — simple phone pattern
console.log("Order #4521 and #99".match(/#\d+/g));  // ["#4521", "#99"]
console.log("2026-08-05".replace(/-/g, "/"));        // "2026/08/05"
```

## Code Examples

### 1. Template literals in practice

```js
const product = "Desk Lamp";
const price = 34.5;
const qty = 2;

// Interpolation with expressions inside ${}
const line = `${qty} × ${product} = $${(qty * price).toFixed(2)}`;
console.log(line); // "2 × Desk Lamp = $69.00"

// Multi-line: great for small templates
const receipt = `
=== RECEIPT ===
Item: ${product}
Qty:  ${qty}
Total: $${(qty * price).toFixed(2)}
===============`;
console.log(receipt);

// Conditionals inside interpolation:
const stock = 0;
console.log(`Status: ${stock > 0 ? "In stock" : "Sold out"}`);
```

### 2. Cleaning user input

```js
const rawEmail = "  Ada.Lovelace@Example.COM \n";

const email = rawEmail.trim().toLowerCase();
console.log(email); // "ada.lovelace@example.com"

// Simple validations with the search methods:
console.log(email.includes("@"));           // true
console.log(email.endsWith(".com"));        // true
console.log(email.startsWith("admin"));     // false
```

### 3. Slicing and dicing

```js
const filename = "vacation-photo-2025.jpeg";

// Extension: everything after the last dot
const dot = filename.lastIndexOf(".");
console.log(filename.slice(dot + 1));   // "jpeg"

// First 8 characters, and last 4:
console.log(filename.slice(0, 8));      // "vacation"
console.log(filename.slice(-4));        // "jpeg"

// Replace:
console.log(filename.replace("jpeg", "png"));       // first (only) occurrence
console.log("a-b-c-d".replaceAll("-", "/"));        // "a/b/c/d"
```

### 4. `split` and `join` — strings ⇄ arrays

```js
const csv = "milk,eggs,bread,butter";
const items = csv.split(",");
console.log(items);            // ["milk", "eggs", "bread", "butter"]

// Now the full array toolkit applies:
const shouted = items.map((i) => i.toUpperCase()).join(" | ");
console.log(shouted);          // "MILK | EGGS | BREAD | BUTTER"

// Words in a sentence:
const sentence = "the quick brown fox";
const words = sentence.split(" ");
console.log(words.length);     // 4

// Characters:
console.log("abc".split(""));  // ["a", "b", "c"]

// Classic trick — capitalize every word:
const title = sentence
  .split(" ")
  .map((w) => w[0].toUpperCase() + w.slice(1))
  .join(" ");
console.log(title);            // "The Quick Brown Fox"
```

### 5. Formatting output

```js
// Padding aligns columns:
const inventory = [["apples", 12], ["kiwis", 3], ["bananas", 140]];
for (const [name, count] of inventory) {
  console.log(`${name.padEnd(10, ".")} ${String(count).padStart(4)}`);
}
// apples....   12
// kiwis.....    3
// bananas...  140

// repeat() for separators and simple bars:
console.log("=".repeat(20));
console.log("█".repeat(7) + "░".repeat(3) + " 70%");
```

### 6. A realistic helper: slugify

```js
// Turn a title into a URL-friendly "slug"
function slugify(title) {
  return title
    .trim()
    .toLowerCase()
    .replaceAll(" ", "-")
    .replace(/[^a-z0-9-]/g, ""); // regex: strip anything not a letter/digit/dash
}

console.log(slugify("  10 Tips for Learning JavaScript!  "));
// "10-tips-for-learning-javascript"
```

That `/[^a-z0-9-]/g` is a regex: "any character that's NOT a lowercase letter, digit, or dash, globally" — see "Regular expressions: a working minimum" above for how to read patterns like this.

## Common Pitfalls

### 1. Expecting methods to modify the string

```js
let name = "ada";
name.toUpperCase();          // ❌ result discarded
console.log(name);           // "ada"

name = name.toUpperCase();   // ✅ keep the returned value
console.log(name);           // "ADA"
```

### 2. `replace` only replaces the first match

```js
console.log("a.b.c".replace(".", "-"));     // ❌ "a-b.c" — only the first!
console.log("a.b.c".replaceAll(".", "-"));  // ✅ "a-b-c"
```

### 3. Case-sensitive comparisons

```js
const answer = "Paris";
console.log(answer === "paris");                          // ❌ false
console.log(answer.toLowerCase() === "paris".toLowerCase()); // ✅ true
```

### 4. Confusing `slice` boundaries

```js
const s = "JavaScript";
console.log(s.slice(0, 4));  // "Java" — index 4 NOT included
console.log(s.slice(4));     // "Script" — to the end
console.log(s.slice(4, 4));  // "" — empty when start === end
// Think: slice(start, end) has length end - start.
```

### 5. Building HTML/strings with + instead of templates

```js
const user = "Sam";
const unread = 3;

// ❌ Fiddly, error-prone quotes and plus signs:
const msg1 = "Hi " + user + ", you have " + unread + " new message" + (unread === 1 ? "" : "s") + ".";

// ✅ Template literal — readable at a glance:
const msg2 = `Hi ${user}, you have ${unread} new message${unread === 1 ? "" : "s"}.`;
```

### 6. Numbers hiding in strings

```js
const zip = "07030";
console.log(zip + 1);         // ❌ "070301"
// Remember Chapter 2: convert before math — but note leading zeros
// are LOST as numbers, which is why zip codes should STAY strings.
```

### 7. Greedy matching grabs more than you want

```js
const html = "<b>bold</b> and <i>italic</i>";
console.log(html.match(/<.+>/)[0]);   // ❌ "<b>bold</b> and <i>italic</i>" — greedy: grabs from the FIRST < to the LAST >
console.log(html.match(/<.+?>/)[0]);  // ✅ "<b>" — the `?` makes the quantifier lazy, stopping at the first match
```

Quantifiers (`+`, `*`, `{n,m}`) are greedy by default — they consume as much text as possible before backtracking to satisfy the rest of the pattern. Append `?` to a quantifier (`+?`, `*?`) to make it lazy instead.

## Practice Exercises

1. **Mad libs.** Declare variables `adjective`, `noun`, and `verb`, then use one multi-line template literal to print a short 3-line story using all three at least once each.

2. **Initials machine.** Write `initials(fullName)` that returns uppercase initials: `initials("ada lovelace king")` → `"A.L.K."`. Handle extra spaces around the input with `trim()`. (Hint: `split`, `map`, `join`.)

3. **Password masker.** Write `mask(cardNumber)` that takes a string like `"4242424242424242"` and returns `"************4242"` — all but the last 4 characters replaced by `*`. Use `slice`, `repeat`, and `length`; it should work for any length.

4. **Word counter.** Given a paragraph string of your choosing, print: total word count, the longest word, how many words contain the letter "e", and the paragraph in "Title Case". Use `split`, array methods, and string methods together.

5. **CSV parser.** Given `const data = "name,score\nada,92\nlinus,88\ngrace,95"`, split it into lines, then split each line by commas, and print a formatted table with padded columns and a total/average score row. (Hint: `"\n"` is the newline character; skip the header row when computing numbers.)

6. **Validity checks.** Write three regexes and test them with `.test()`: a valid-looking US zip code (`12345` or `12345-6789`), a string containing only letters and spaces, and a hex color code like `#a1b2c3` (3 or 6 hex digits after `#`). Try each against at least one string that should pass and one that should fail.

7. **Find and replace.** Given `const messy = "Contact:  john@x.com,   jane@y.com  ,bob@z.com"`, use a regex with `match` (global flag) to extract all email addresses into an array, then use a separate regex with `replace` to collapse every run of whitespace into a single space.
