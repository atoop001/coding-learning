# Chapter 1: Getting Started with Java

## Overview

Java is one of the most widely used programming languages in the world, especially in **enterprise software** — banking systems, insurance platforms, e-commerce backends, Android apps, and huge server-side applications. If your goal is employability, Java is a strong bet: it has been in the top 3 most in-demand languages for over two decades.

Coming from Python or JavaScript, the biggest mental shifts are:

1. **Java is compiled** — you can't just run a `.java` file the way you run a `.py` file (well, modern Java can single-file run, but the normal workflow is compile-then-run).
2. **Java is statically typed** — every variable has a declared type, checked *before* the program runs.
3. **Everything lives in a class** — there's no top-level script code.

This chapter gets Java installed on your Windows machine, gets your first program running, and explains what actually happens when Java code runs.

## Definitions & Explanations

### JDK, JRE, and JVM — the three-letter soup

- **JVM (Java Virtual Machine)** — the program that actually *runs* Java programs. Java code doesn't compile to Windows machine code; it compiles to **bytecode**, a portable intermediate format. The JVM reads bytecode and executes it. This is why the same compiled Java program runs on Windows, Mac, and Linux ("write once, run anywhere").
- **JRE (Java Runtime Environment)** — the JVM plus the standard libraries. Enough to *run* Java programs, not to write them. (Modern JDKs no longer ship a separate JRE — you just use the JDK.)
- **JDK (Java Development Kit)** — everything: the JVM, the standard library, and developer tools like `javac` (the compiler). **This is what you install.**

Analogy for Python folks: the JVM is like the Python interpreter, and bytecode is like `.pyc` files — except in Java, compiling to bytecode is an explicit, separate step, and the compiler enforces types.

### Compilation: source → bytecode → execution

```
HelloWorld.java  --(javac)-->  HelloWorld.class  --(java)-->  program runs on JVM
   (source code)                  (bytecode)
```

- `javac` is the **compiler**: it reads `.java` source files, checks them for errors (including type errors), and produces `.class` files.
- `java` is the **launcher**: it starts a JVM and runs the bytecode.

In JavaScript, errors like calling a function that doesn't exist blow up *at runtime*. In Java, most such errors are caught *at compile time* — the program won't even build. This feels strict at first but is a huge safety net in large codebases.

### LTS versions

Java releases a new version every six months, but **LTS (Long-Term Support)** versions are what companies use: Java 8, 11, 17, 21. Install the newest LTS (21 or later). Everything in this track works on Java 17+.

## Installing the JDK on Windows

1. Download an OpenJDK build. Two good free options:
   - **Eclipse Temurin** from adoptium.net (recommended)
   - **Microsoft Build of OpenJDK** from microsoft.com/openjdk
2. Run the `.msi` installer. **Check the options** to:
   - Set `JAVA_HOME` environment variable
   - Add Java to `PATH`
3. Verify in a new PowerShell window:

```powershell
java -version
javac -version
```

Both should print a version number (e.g., `openjdk 21.0.x`). If PowerShell says the command isn't recognized, the `PATH` wasn't set — re-run the installer with the PATH option checked, or add `C:\Program Files\Eclipse Adoptium\jdk-21...\bin` to your PATH manually via *System Properties → Environment Variables*.

### Choosing an editor

- **IntelliJ IDEA Community Edition** (free) — the industry-standard Java IDE. Best autocomplete, refactoring, and error highlighting. Recommended for this track.
- **VS Code** with the "Extension Pack for Java" — lighter weight, fine if you already live in VS Code.

Either works. IntelliJ will feel heavier than what you're used to from Python/JS, but professional Java development almost universally happens in IntelliJ, so it's worth learning.

## Code Examples

### Your first program

Create a file named `HelloWorld.java` (the filename **must** match the class name, including capitalization):

```java
// HelloWorld.java
// Every Java program lives inside a class.
public class HelloWorld {

    // main is the entry point. The JVM looks for exactly this signature.
    // - public: callable from outside the class
    // - static: no object needs to be created first (explained in Ch. 5)
    // - void:   returns nothing
    // - String[] args: command-line arguments as an array of strings
    public static void main(String[] args) {
        System.out.println("Hello, world!");   // note the semicolon — required!
    }
}
```

Compile and run from PowerShell:

```powershell
javac HelloWorld.java    # produces HelloWorld.class
java HelloWorld          # note: no .class extension here
```

Output:

```
Hello, world!
```

Modern shortcut (Java 11+) — run a single source file directly:

```powershell
java HelloWorld.java     # compiles in memory and runs — handy for small experiments
```

### A slightly bigger example

```java
// Greeter.java
public class Greeter {
    public static void main(String[] args) {
        String name = "Ada";          // typed variable: String, not just "var x = ..."
        int year = 2026;              // int is a primitive type (Ch. 2)

        // String concatenation with + (like JS)
        System.out.println("Hello, " + name + "! It is " + year + ".");

        // printf-style formatting, like Python's f-strings but with format codes
        System.out.printf("Hello, %s! It is %d.%n", name, year);

        // Command-line arguments
        if (args.length > 0) {
            System.out.println("First argument: " + args[0]);
        }
    }
}
```

```powershell
javac Greeter.java
java Greeter Bob
```

### Anatomy checklist

Every beginner Java file follows this shape:

```java
public class ClassName {          // filename must be ClassName.java
    public static void main(String[] args) {
        // statements end with semicolons;
        // blocks are delimited by { braces }, not indentation
    }
}
```

Unlike Python, **indentation is not significant** — braces define structure. Indent consistently anyway (4 spaces is the Java convention); the compiler doesn't care but every human reader does.

## Common Pitfalls

### 1. Filename doesn't match the class name

```java
// File saved as hello.java  ❌
public class HelloWorld { ... }
```

Error: `class HelloWorld is public, should be declared in a file named HelloWorld.java`.
**Fix:** rename the file to `HelloWorld.java`. Java is case-sensitive: `helloworld.java` is also wrong.

### 2. Running with the `.class` extension

```powershell
java HelloWorld.class      # ❌  Error: Could not find or load main class HelloWorld.class
java HelloWorld            # ✅
```

`java` takes a *class name*, not a filename.

### 3. Forgetting semicolons

```java
System.out.println("hi")    // ❌ compile error: ';' expected
System.out.println("hi");   // ✅
```

Coming from Python (no semicolons) or JS (optional semicolons), this bites everyone for a week. The compiler error message points at the exact line.

### 4. Wrong `main` signature

```java
public void main(String[] args) { }        // ❌ missing static — JVM won't find it
public static void main(String args) { }   // ❌ String, not String[]
public static void main(String[] args) { } // ✅
```

The program compiles but fails at launch with `Error: Main method not found`.

### 5. `println` vs `printf` confusion

`System.out.println(x)` prints anything and adds a newline. `System.out.printf(format, args...)` uses format specifiers (`%s` string, `%d` integer, `%f` float, `%n` newline) and does **not** add a newline automatically.

## Practice Exercises

1. **Install & verify.** Install a JDK (Temurin 21 LTS), then run `java -version` and `javac -version` in PowerShell. Then install IntelliJ IDEA Community and create a new project.
2. **Hello, you.** Write, compile, and run a program `AboutMe.java` that prints your name, your goal for learning Java, and today's date on three separate lines.
3. **Break it on purpose.** Take your working `AboutMe.java` and, one at a time: remove a semicolon, rename the class (but not the file), and remove `static` from `main`. Compile/run after each change and write down the exact error message you get. Learning to read compiler errors is a core skill.
4. **Arguments.** Write `Echo.java` that prints how many command-line arguments it received, then prints each one on its own line. Run it with `java Echo apple banana cherry`.
5. **Formatted output.** Using `printf`, print a small "receipt": an item name left-padded to look aligned, and a price with exactly 2 decimal places (hint: `%.2f`).
