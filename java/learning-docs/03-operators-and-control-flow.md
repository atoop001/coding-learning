# Chapter 3: Operators and Control Flow (if / switch)

## Overview

Programs make decisions. This chapter covers Java's operators (arithmetic, comparison, logical) and its branching constructs: `if`/`else if`/`else`, the ternary operator, and `switch` — including the modern *switch expression* syntax that professional Java code increasingly uses.

Coming from Python: braces replace indentation, conditions must be in parentheses, and conditions must be actual `boolean`s — no "truthy" values. Coming from JS: no `===` needed (but `==` on objects is a trap, covered later), and `switch` is much more capable than the one you know.

## Definitions & Explanations

### Arithmetic operators

`+  -  *  /  %` work as expected, with two caveats from Chapter 2: integer `/` truncates, and `%` is remainder (can be negative: `-7 % 3` is `-1`, unlike Python's `2`).

Compound assignment and increment:

```java
count += 5;    // count = count + 5   (also -=, *=, /=, %=)
count++;       // increment by 1
count--;       // decrement by 1
```

There is no `**` power operator — use `Math.pow(base, exp)` (returns `double`).

### Comparison operators

`==  !=  <  <=  >  >=` — all return `boolean`.

**Critical rule:** `==` compares primitive *values* but object *references*. For `String`s and other objects, use `.equals()`:

```java
int a = 5;
a == 5                     // fine — primitives compare by value
"hello".equals(userInput)  // correct way to compare strings
userInput == "hello"       // ❌ compiles, but compares references — usually wrong
```

### Logical operators

| Java | Python | Meaning |
|------|--------|---------|
| `&&` | `and`  | logical AND (short-circuits) |
| `\|\|` | `or` | logical OR (short-circuits) |
| `!`  | `not`  | logical NOT |

Short-circuiting matters: in `x != null && x.length() > 0`, the second half never runs if `x` is null — this ordering prevents `NullPointerException`.

**No truthiness.** In Python, `if my_list:` works. In Java, conditions must be `boolean`:

```java
if (name) { }             // ❌ compile error — name is a String
if (name != null && !name.isEmpty()) { }   // ✅ say what you mean
```

### if / else if / else

```java
if (condition) {
    // ...
} else if (otherCondition) {
    // ...
} else {
    // ...
}
```

Braces are technically optional for single statements, but **always use braces** — omitting them is a notorious source of bugs (see pitfalls).

### Ternary operator

Java's inline conditional, equivalent to Python's `a if cond else b`:

```java
String label = (score >= 60) ? "pass" : "fail";
```

### switch — classic statement form

```java
switch (dayNumber) {
    case 1:
        System.out.println("Monday");
        break;                       // without break, execution FALLS THROUGH
    case 6:
    case 7:                          // stacking cases = OR
        System.out.println("Weekend");
        break;
    default:
        System.out.println("Some other weekday");
}
```

Works on: `int` (and smaller integer types), `char`, `String`, and enums. **Not** on `long`, `double`, or `boolean`.

### switch — modern expression form (Java 14+)

The arrow form has no fall-through, and can *return a value*:

```java
String dayType = switch (dayNumber) {
    case 1, 2, 3, 4, 5 -> "Weekday";
    case 6, 7          -> "Weekend";
    default            -> "Invalid";
};
```

Prefer this form in new code — it's safer (no forgotten `break`) and more concise. When a case needs multiple statements, use a block with `yield`:

```java
int fee = switch (memberLevel) {
    case "gold" -> 0;
    case "silver" -> {
        int base = 5;
        yield base + 2;    // yield supplies the value from a block
    }
    default -> 10;
};
```

## Code Examples

### Grading with if/else chains

```java
public class Grader {
    public static void main(String[] args) {
        int score = 87;

        String grade;
        if (score >= 90) {
            grade = "A";
        } else if (score >= 80) {
            grade = "B";
        } else if (score >= 70) {
            grade = "C";
        } else if (score >= 60) {
            grade = "D";
        } else {
            grade = "F";
        }

        // Ternary for a compact yes/no
        String status = (score >= 60) ? "PASS" : "FAIL";

        System.out.println("Score " + score + " -> grade " + grade + " (" + status + ")");
    }
}
```

### Realistic input validation

```java
import java.util.Scanner;

public class LoginCheck {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        System.out.print("Username: ");
        String user = in.nextLine();
        System.out.print("PIN: ");
        String pinText = in.nextLine();

        // Guard clauses: check bad cases first, exit early
        if (user == null || user.isBlank()) {
            System.out.println("Username required.");
            return;                       // return exits main immediately
        }
        if (!pinText.matches("\\d{4}")) {  // exactly four digits
            System.out.println("PIN must be 4 digits.");
            return;
        }

        int pin = Integer.parseInt(pinText);
        // note the && short-circuit protecting nothing here, but the order of
        // checks above protected parseInt from non-numeric input
        boolean ok = user.equals("admin") && pin == 1234;
        System.out.println(ok ? "Welcome!" : "Access denied.");
        in.close();
    }
}
```

### Modern switch expression

```java
public class Seasons {
    public static void main(String[] args) {
        int month = 7;

        String season = switch (month) {
            case 12, 1, 2 -> "Winter";
            case 3, 4, 5  -> "Spring";
            case 6, 7, 8  -> "Summer";
            case 9, 10, 11 -> "Autumn";
            default -> "Unknown month";
        };

        System.out.println("Month " + month + " is in " + season);

        // switch on a String
        String command = "start";
        switch (command) {
            case "start" -> System.out.println("Starting...");
            case "stop"  -> System.out.println("Stopping...");
            default      -> System.out.println("Unknown command: " + command);
        }
    }
}
```

## Common Pitfalls

### 1. Forgotten `break` in classic switch (fall-through)

```java
switch (n) {
    case 1:
        System.out.println("one");   // ❌ no break: if n==1 this prints "one" AND "two"
    case 2:
        System.out.println("two");
        break;
}
```

**Fix:** add `break` to every case — or better, use the arrow form, which never falls through.

### 2. Comparing Strings with `==`

```java
if (command == "start") { }        // ❌ may be false even when text matches
if (command.equals("start")) { }   // ✅
if ("start".equals(command)) { }   // ✅ and also null-safe
```

### 3. Braceless if bodies

```java
if (isAdmin)
    grantAccess();
    logAccess();        // ❌ ALWAYS runs — indentation lies, it's not inside the if
```

Java isn't Python: indentation means nothing. **Fix:** always write braces:

```java
if (isAdmin) {
    grantAccess();
    logAccess();
}
```

### 4. Semicolon after the condition

```java
if (x > 0);            // ❌ the semicolon IS the (empty) body
{
    System.out.println("positive");   // runs unconditionally
}
```

### 5. Null check on the wrong side of `&&`

```java
if (name.length() > 0 && name != null) { }   // ❌ NPE before null check runs
if (name != null && name.length() > 0) { }   // ✅ short-circuit saves you
```

### 6. Assuming `%` behaves like Python's

```java
int wrapped = -1 % 5;              // -1 in Java (2 in Python)
int fixed = ((-1 % 5) + 5) % 5;    // 4 — the "always positive" idiom
```

## Practice Exercises

1. **FizzBuzz decisions.** For a single hardcoded `int n`, print "Fizz" if divisible by 3, "Buzz" if divisible by 5, "FizzBuzz" if both, otherwise the number itself. Get the order of checks right — test with 15.
2. **BMI classifier.** Read weight (kg) and height (m) with `Scanner`, compute BMI (`weight / height²`), then use an if/else chain to print the WHO category (underweight < 18.5, normal < 25, overweight < 30, obese otherwise). Guard against zero or negative height before dividing.
3. **Switch calculator.** Read two doubles and an operator string (`+`, `-`, `*`, `/`). Use a modern switch expression to compute the result. Handle division by zero and unknown operators with sensible messages instead of crashes.
4. **Leap year.** Read a year and print whether it's a leap year: divisible by 4, except centuries, unless divisible by 400. Write it once with nested ifs and once as a single boolean expression with `&&`/`||` — verify both agree on 1900, 2000, 2024, and 2025.
5. **Fall-through on purpose.** Using a classic `switch` with deliberate fall-through, print the lyrics pattern of "The Twelve Days of Christmas" style accumulation for a day number 1–5 (day 3 prints gifts 3, 2, and 1). Add a comment marking each intentional fall-through.
