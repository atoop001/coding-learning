# Chapter 6: Strings and Text Handling

## Overview

Text is everywhere — user input, file contents, API payloads, log lines. Java's `String` class is rich but has sharp edges that trip up newcomers from Python/JS: strings are **objects** (so `==` is a trap), they are **immutable** (every "modification" makes a new string), and heavy concatenation calls for a separate class, `StringBuilder`. This chapter covers the essential String API, formatting, comparison done right, and a first taste of text processing.

## Definitions & Explanations

### Strings are immutable objects

A `String` can never change after creation. `toUpperCase()`, `trim()`, `replace()` — all return *new* strings:

```java
String s = "hello";
s.toUpperCase();               // creates "HELLO"... and throws it away
s = s.toUpperCase();           // reassign to keep it
```

Same as Python strings; different from thinking of them as mutable buffers.

### Comparison: `.equals()`, not `==`

`==` on objects compares *references* (are these the same object in memory?). `.equals()` compares *content*. String literals are sometimes shared (the "string pool"), which makes `==` work *just often enough* to fool you:

```java
String a = "java";
String b = "java";
String c = new String("java");
a == b            // true (both point at the pooled literal) — misleading!
a == c            // false — different objects
a.equals(c)       // true — same characters ✅
a.equalsIgnoreCase("JAVA")  // true
"x".equals(input) // ✅ null-safe ordering: literal first never NPEs
```

**Rule: always use `.equals()` for strings.** For ordering, `a.compareTo(b)` returns negative/zero/positive (lexicographic).

### Essential String methods

| Method | What it does | Python/JS analogue |
|--------|--------------|--------------------|
| `length()` | number of chars | `len(s)` / `s.length` |
| `charAt(i)` | char at index i | `s[i]` |
| `substring(a, b)` | chars from a (incl.) to b (excl.) | `s[a:b]` |
| `indexOf(x)` | first position of x, or -1 | `s.find` / `s.indexOf` |
| `contains(x)` | true if x occurs | `x in s` / `s.includes` |
| `startsWith(x)` / `endsWith(x)` | prefix/suffix test | same names |
| `toUpperCase()` / `toLowerCase()` | case change | `upper()` / `lower()` |
| `trim()` / `strip()` | remove surrounding whitespace | `strip()` |
| `replace(a, b)` | replace ALL occurrences | `replace` (Python yes, JS no!) |
| `split(regex)` | split into `String[]` | `split` — but takes a REGEX |
| `isEmpty()` / `isBlank()` | "" / only whitespace | falsy check |
| `String.join(sep, parts)` | join pieces | `sep.join(...)` |
| `repeat(n)` | n copies | `s * n` / `s.repeat(n)` |

**No slicing, no negative indices.** `s.substring(s.length() - 3)` is Java for `s[-3:]`.

### chars, Strings, and conversion

- `charAt` returns a `char` (primitive). Compare chars with `==` (they're primitives): `c == 'a'` is fine.
- Number → String: `String.valueOf(42)` or `"" + 42`.
- String → number: `Integer.parseInt(s)`, `Double.parseDouble(s)` — these throw `NumberFormatException` on bad input (Chapter 14 covers handling that).

### Formatting

```java
String msg = String.format("%s scored %d points (%.1f%%)", name, points, pct);
System.out.printf("%-10s | %5d%n", label, amount);   // %-10s left-align width 10
```

Java 15+ **text blocks** give you multi-line strings like Python's triple quotes:

```java
String html = """
        <html>
          <body>Hello</body>
        </html>
        """;
```

### StringBuilder — for building strings in loops

Because strings are immutable, `result += piece` in a loop creates a new string every pass — O(n²) for big inputs. `StringBuilder` is a *mutable* text buffer:

```java
StringBuilder sb = new StringBuilder();
for (String word : words) {
    sb.append(word).append(", ");     // append returns the builder: chainable
}
String result = sb.toString();
```

Rule of thumb: a few `+` concatenations are fine (the compiler optimizes simple cases); building text in a loop → `StringBuilder`.

## Code Examples

### String API tour

```java
public class StringTour {
    public static void main(String[] args) {
        String raw = "  Java Programming  ";

        String s = raw.strip();                        // "Java Programming"
        System.out.println(s.length());                // 16
        System.out.println(s.toUpperCase());           // JAVA PROGRAMMING
        System.out.println(s.charAt(0));               // J
        System.out.println(s.substring(5));            // Programming
        System.out.println(s.substring(0, 4));         // Java
        System.out.println(s.indexOf("Pro"));          // 5
        System.out.println(s.contains("gram"));        // true
        System.out.println(s.replace("a", "@"));       // J@v@ Progr@mming
        System.out.println("=".repeat(20));            // ====================

        // split returns an array; note the regex argument
        String csv = "alice,bob,carol";
        String[] names = csv.split(",");
        for (String name : names) {
            System.out.println("-> " + name);
        }
        System.out.println(String.join(" & ", names)); // alice & bob & carol
    }
}
```

### Character-by-character processing

```java
public class VowelCounter {
    // Count vowels the classic way: walk the string
    public static int countVowels(String text) {
        int count = 0;
        String lower = text.toLowerCase();
        for (int i = 0; i < lower.length(); i++) {
            char c = lower.charAt(i);
            if (c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u') {
                count++;
            }
        }
        return count;
    }

    // Reverse a string using StringBuilder's built-in
    public static String reversed(String text) {
        return new StringBuilder(text).reverse().toString();
    }

    public static void main(String[] args) {
        System.out.println(countVowels("Strings in Java"));   // 4
        System.out.println(reversed("stressed"));             // desserts

        // Useful char classification helpers:
        System.out.println(Character.isDigit('7'));           // true
        System.out.println(Character.isLetter('!'));          // false
        System.out.println(Character.toUpperCase('q'));       // Q
    }
}
```

### StringBuilder vs concatenation

```java
public class BuilderDemo {
    public static void main(String[] args) {
        // Build a receipt line by line
        String[] items = {"Coffee", "Bagel", "Juice"};
        double[] prices = {3.50, 2.25, 4.00};

        StringBuilder receipt = new StringBuilder();
        receipt.append("RECEIPT\n");
        double total = 0;
        for (int i = 0; i < items.length; i++) {
            receipt.append(String.format("%-10s %6.2f%n", items[i], prices[i]));
            total += prices[i];
        }
        receipt.append("-".repeat(17)).append("\n");
        receipt.append(String.format("%-10s %6.2f%n", "TOTAL", total));

        System.out.print(receipt);   // println/print accept a StringBuilder directly
    }
}
```

## Common Pitfalls

### 1. `==` instead of `.equals()` — the classic

```java
if (input == "yes") { }          // ❌ false for user-typed "yes"
if ("yes".equals(input)) { }     // ✅ correct AND null-safe
```

This bug is invisible in small tests (literal pooling) and appears the moment real input arrives. Burn the rule in now.

### 2. Forgetting immutability

```java
String s = "hello";
s.concat(" world");
System.out.println(s);           // ❌ "hello" — the result was discarded
s = s.concat(" world");          // ✅
```

### 3. `split` takes a regex

```java
"3.14.159".split(".")            // ❌ returns EMPTY array — "." is regex "any char"
"3.14.159".split("\\.")          // ✅ escape it: {"3", "14", "159"}
"a|b".split("\\|")               // same story for | + * ? ( ) [ ] etc.
```

### 4. Index out of bounds on substring/charAt

```java
String s = "abc";
s.charAt(3);                     // ❌ StringIndexOutOfBoundsException (last index is 2)
s.substring(1, 10);              // ❌ same exception
s.substring(3);                  // ✅ legal, returns "" (the empty tail)
```

### 5. Comparing case-sensitively by accident

```java
if (answer.equals("Yes")) { }               // misses "yes", "YES"
if (answer.equalsIgnoreCase("yes")) { }     // ✅
// or normalize first: answer = answer.strip().toLowerCase();
```

### 6. Concatenation order surprises

```java
System.out.println("Sum: " + 1 + 2);        // Sum: 12  — left-to-right, string wins
System.out.println("Sum: " + (1 + 2));      // Sum: 3   ✅ parenthesize the math
```

## Practice Exercises

1. **Initials.** Read a full name (e.g., "ada byron lovelace") and print initials, capitalized and dotted: `A.B.L.` — handle any number of name parts and stray extra spaces.
2. **Palindrome check.** Write `static boolean isPalindrome(String s)` ignoring case and non-letters, so "A man, a plan, a canal: Panama" returns true. (Hints: build a cleaned string first with `StringBuilder` + `Character.isLetter`, then compare with its reverse — or walk two indexes inward.)
3. **Word statistics.** Read a sentence and report: word count, longest word, average word length (one decimal), and the sentence in Title Case. Use `split("\\s+")` and explain in a comment why `\\s+` beats `" "`.
4. **CSV cleanup.** Given the line `"  alice , 42 ,oslo  "`, split on commas and strip each field, then print each field as `field[i]='value'` with no stray spaces. Then rejoin with `";"` using `String.join`.
5. **Caesar cipher.** Write `static String encrypt(String text, int shift)` that shifts letters by `shift` positions (wrapping z→a), preserving case and leaving non-letters alone. Write the matching `decrypt` and verify `decrypt(encrypt(s, 5), 5).equals(s)` for a test sentence.
