# Chapter 14: Exceptions and Error Handling

## Overview

Things go wrong: files are missing, users type "banana" where a number belongs, networks drop. Java handles failures with **exceptions** — objects that are *thrown* at the failure point and *caught* wherever it makes sense to respond. You've already met a few uninvited: `NullPointerException`, `ArrayIndexOutOfBoundsException`, `NumberFormatException`.

Python (`try/except/raise`) and JS (`try/catch/throw`) prepare you well; Java adds one big novelty — **checked exceptions**, which the compiler *forces* you to deal with — plus `finally`, exception hierarchies, and conventions employers care deeply about (error handling quality is a classic code-review topic).

## Definitions & Explanations

### try / catch / finally

```java
try {
    int n = Integer.parseInt(input);         // may throw
    System.out.println(100 / n);             // may also throw
} catch (NumberFormatException e) {          // runs only if that exception occurred
    System.out.println("Not a number: " + input);
} catch (ArithmeticException e) {            // multiple catch blocks: first match wins
    System.out.println("Can't divide by zero");
} finally {
    System.out.println("Runs ALWAYS — exception or not");   // cleanup goes here
}
```

- Execution jumps from the throw point straight to the first matching `catch`; code between is skipped.
- The exception object `e` carries details: `e.getMessage()`, `e.printStackTrace()`.
- `catch (IOException | SQLException e)` handles multiple types in one block (Java's `except (A, B)`).
- `finally` runs on success, on exception, even on `return` — for cleanup. (Modern code often replaces it with try-with-resources; Chapter 16.)

### Throwing

```java
if (amount <= 0) {
    throw new IllegalArgumentException("amount must be positive, got " + amount);
}
```

`throw` (one object) — like Python's `raise`. Write **specific, message-rich** exceptions: the message is what a future you sees in a log at 3 a.m.

### The hierarchy — and checked vs unchecked

```
Throwable
├── Error                    (JVM disasters: OutOfMemoryError — don't catch)
└── Exception
    ├── RuntimeException     → UNCHECKED (NPE, IllegalArgumentException,
    │                          IndexOutOfBounds, NumberFormatException...)
    └── everything else      → CHECKED   (IOException, SQLException...)
```

**Unchecked** (subclasses of `RuntimeException`): usually programming bugs. The compiler doesn't make you handle them — fix the code instead.

**Checked** (everything else under `Exception`): anticipated external failures (missing file, dead network). The compiler enforces the **catch-or-declare rule**: any method calling checked-throwing code must either catch it or declare it onward:

```java
public String readConfig() throws IOException {   // "declare": pass responsibility to my caller
    return Files.readString(Path.of("config.txt"));
}
```

This has no Python/JS equivalent — nothing there forces error handling. It's controversial even among Java veterans, but you must be fluent in it: the standard library's I/O is checked-exception territory.

**Choosing when you throw:** invalid arguments/state → unchecked (`IllegalArgumentException`, `IllegalStateException`). Recoverable environmental failure the caller should plan for → checked (or, common modern style, wrap in an unchecked domain exception).

### Custom exceptions

```java
public class InsufficientFundsException extends Exception {   // checked (extend RuntimeException for unchecked)
    private final double shortfall;

    public InsufficientFundsException(double shortfall) {
        super("Insufficient funds: short by " + shortfall);
        this.shortfall = shortfall;
    }

    public double getShortfall() { return shortfall; }
}
```

Custom exceptions make failures *typed and specific* — callers can catch exactly what they can handle, and carry structured data about what went wrong.

### Reading a stack trace

```
Exception in thread "main" java.lang.NumberFormatException: For input string: "abc"
        at java.base/java.lang.Integer.parseInt(Integer.java:652)
        at PaymentApp.parseAmount(PaymentApp.java:31)
        at PaymentApp.main(PaymentApp.java:12)
```

Read top line for *what*, then scan down to the **first line in YOUR code** (`PaymentApp.java:31`) for *where*. The chain below shows who-called-whom. Learning to jump straight to the relevant line is a daily-use skill.

## Code Examples

### Robust user input (finally closing the loop from Chapter 4)

```java
import java.util.Scanner;

public class SafeInput {
    // Keep asking until we get a valid int in range
    public static int askInt(Scanner in, String prompt, int min, int max) {
        while (true) {
            System.out.print(prompt);
            String line = in.nextLine();
            try {
                int value = Integer.parseInt(line.strip());
                if (value < min || value > max) {
                    System.out.printf("Must be between %d and %d.%n", min, max);
                    continue;
                }
                return value;                       // success exits the loop
            } catch (NumberFormatException e) {     // "abc" lands here — no crash
                System.out.println("'" + line + "' is not a whole number.");
            }
        }
    }

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int age = askInt(in, "Age (0-120): ", 0, 120);
        System.out.println("Recorded: " + age);
    }
}
```

### Custom exception in a domain model

```java
public class Account {
    private double balance;

    public Account(double opening) { this.balance = opening; }

    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount <= 0) {
            // unchecked: caller made a programming error
            throw new IllegalArgumentException("amount must be positive: " + amount);
        }
        if (amount > balance) {
            // checked: legitimate business condition callers must plan for
            throw new InsufficientFundsException(amount - balance);
        }
        balance -= amount;
    }

    public double getBalance() { return balance; }

    public static void main(String[] args) {
        Account acct = new Account(100.0);
        try {
            acct.withdraw(250.0);
        } catch (InsufficientFundsException e) {
            // typed data on the exception enables a precise response:
            System.out.println(e.getMessage());
            System.out.printf("Consider depositing %.2f first.%n", e.getShortfall());
        }
        System.out.println("Balance intact: " + acct.getBalance());   // 100.0 — state not corrupted
    }
}
```

### Catch-or-declare, wrapping, and finally

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

public class ConfigLoader {

    // Option A: declare — push the checked exception to the caller
    public static String load(String file) throws IOException {
        return Files.readString(Path.of(file));
    }

    // Option B: catch and wrap in an unchecked exception with context.
    // Common in real apps: callers deal with ONE domain exception type.
    public static String loadOrFail(String file) {
        try {
            return Files.readString(Path.of(file));
        } catch (IOException e) {
            throw new IllegalStateException("Cannot load config: " + file, e);
            //                                                    cause ─┘ keeps the original stack trace!
        }
    }

    public static void main(String[] args) {
        try {
            System.out.println(load("settings.txt"));
        } catch (IOException e) {
            System.out.println("Using defaults (" + e.getMessage() + ")");
        } finally {
            System.out.println("Startup phase complete.");
        }
    }
}
```

Note the two-argument constructor `new IllegalStateException(msg, e)` — always pass the original as the **cause** when wrapping, or you amputate the stack trace.

## Common Pitfalls

### 1. Swallowing exceptions

```java
try {
    process(order);
} catch (Exception e) { }            // ❌ silent failure — the WORST habit in Java
```

An empty catch block hides bugs indefinitely. At minimum log it; ideally handle it or rethrow. If you truly must ignore, comment why.

### 2. Catching `Exception` (or `Throwable`) too broadly

```java
catch (Exception e) { ... }          // ❌ also traps NPEs and bugs you wanted to see
catch (NumberFormatException e) { }  // ✅ catch the narrowest type you can handle
```

Broad catches are occasionally right at top-level boundaries (a server keeping alive); everywhere else, be specific.

### 3. Catch blocks ordered wrong

```java
try { ... }
catch (Exception e) { ... }                 // ❌ this matches everything...
catch (NumberFormatException e) { ... }     // ❌ compile error: already caught above
```

Subclass catches must come **before** superclass catches.

### 4. Using exceptions for normal control flow

```java
try {                                          // ❌ slow and unreadable
    while (true) sum += values[i++];
} catch (ArrayIndexOutOfBoundsException e) { }
for (int v : values) sum += v;                 // ✅ conditions for expected cases
```

Exceptions are for *exceptional* situations; ordinary logic uses ifs and loops.

### 5. Losing the cause when wrapping

```java
catch (IOException e) {
    throw new IllegalStateException("load failed");        // ❌ original trace gone
    throw new IllegalStateException("load failed", e);     // ✅ chained cause
}
```

### 6. Believing catch "fixes" the problem

After catching, execution continues *after* the try/catch — with whatever half-done state exists. Design so state stays valid (validate *before* mutating, as `Account.withdraw` does), and only catch where you can actually do something meaningful: retry, substitute a default, inform the user, or rethrow with context.

## Practice Exercises

1. **Crash tour.** Write code that deliberately triggers, one at a time: `ArithmeticException`, `NullPointerException`, `ArrayIndexOutOfBoundsException`, `NumberFormatException`, and `ClassCastException`. For each, record the top line of the stack trace in a comment, then wrap it in a specific catch that prints a friendly message instead.
2. **Bulletproof calculator.** Extend Chapter 3's switch calculator so no input can crash it: non-numeric operands, unknown operators, and division by zero each produce a clear message and re-prompt. Centralize number-reading in a `askDouble(...)` helper modeled on `askInt` above.
3. **Custom exception pair.** For a `Thermostat` class (`setTarget(double celsius)`), create `TemperatureOutOfRangeException` (checked, carrying the offending value and the legal range) thrown outside 5–35°C, and use `IllegalArgumentException` for NaN input. Write a caller that catches the custom exception and clamps to the nearest legal value, printing what it did.
4. **Declare vs handle.** Write `static int[] parseAll(String[] tokens)` two ways: version A throws `NumberFormatException` up to the caller (document with a comment where it's ultimately caught); version B catches per-token, skips bad tokens, counts them, and returns the good ones (hint: parse into a `List<Integer>` first). In comments, name one scenario where each version is the better design.
5. **Finally semantics.** Predict the output of a method whose try block returns `"A"`, whose catch returns `"B"`, and whose finally prints `"F"` — in three variants: no exception, an exception caught, an exception *in the catch block itself*. Write the code, run all three, and reconcile your predictions. (Bonus: discover what happens if finally itself contains a `return` — then never do it again.)
