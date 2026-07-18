# Chapter 11: Packages, Access Modifiers, and the Classpath

## Overview

Real Java projects aren't one file — they're hundreds, organized into **packages** (Java's namespaces/folders), guarded by **access modifiers** (who may see what), and located at runtime via the **classpath**. This chapter is less flashy than OOP but it's the plumbing every employer assumes you know: why files start with `package com.company.app;`, what `import` really does, what `protected` actually means, and why "class not found" errors happen.

## Definitions & Explanations

### Packages

A package is a named group of classes, mirroring a folder structure:

```java
package com.acme.billing;      // first line of the file (before imports)

public class Invoice { ... }
```

- File location must match: `com/acme/billing/Invoice.java` (from the source root).
- The **fully qualified name** is `com.acme.billing.Invoice` — that's the class's real name; `Invoice` is shorthand within its own package.
- Naming convention: reverse internet domain (`com.acme.billing`, `org.apache.commons`), all lowercase. For learning projects, something like `dev.yourname.projectname` is fine.
- Files with no `package` line land in the *default package* — acceptable for single-file experiments, forbidden in real projects (classes in named packages can't even import from it).

Analogy: packages are Python modules/packages or JS module paths — but declared inside the file and rigidly tied to directory layout.

### Imports

```java
import java.util.Scanner;        // single class
import java.util.*;              // everything in java.util (NOT subpackages)
import static java.lang.Math.PI; // static import: use PI bare, no Math. prefix
```

- `import` doesn't load or copy anything (unlike Python's `import` executing a module) — it's purely a compile-time abbreviation so you can write `Scanner` instead of `java.util.Scanner`.
- `java.lang.*` (String, Math, System, Integer…) is auto-imported always.
- IntelliJ manages imports for you (Alt+Enter on red names); professional style prefers single-class imports over wildcards.

### The four access levels

Java has four visibility levels — one more than most people realize, because one has no keyword:

| Modifier | Same class | Same package | Subclass (other pkg) | Everyone |
|---------------------|:---:|:---:|:---:|:---:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(none)* — "package-private" | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

- **`private`** — this class only. Default choice for fields.
- **package-private** (no keyword at all) — visible throughout its own package. Great for helper classes that are implementation details of a package.
- **`protected`** — package **plus** subclasses anywhere. Note it's *more* open than package-private, not less — a common misconception.
- **`public`** — the world. Your deliberate API surface.

Guideline: start everything as private/package-private and open up only when needed. Minimizing surface area is what makes large codebases changeable. Python has only conventions (`_name`); Java actually enforces this.

Top-level classes can only be `public` or package-private. One `public` class per file, named like the file.

### The classpath

The **classpath** is the list of places (`directories` and JAR files) where `javac` and `java` look for compiled classes:

```powershell
javac -d out src/com/acme/billing/Invoice.java     # compile INTO out/
java -cp out com.acme.billing.Main                 # run, looking in out/
java -cp "out;lib/gson-2.11.0.jar" com.acme.Main   # Windows separator is ; (Linux/Mac use :)
```

- `-d out` tells javac to mirror the package structure under `out/`.
- `-cp` (or `-classpath`) tells both tools where to search. Default is the current directory.
- `Error: Could not find or load main class X` almost always means: wrong classpath, wrong fully-qualified name, or running from the wrong directory.

In practice, Maven/Gradle and IDEs manage the classpath for you (Chapter 18) — but when they misbehave, this mental model is how you debug it.

### JARs (preview)

A **JAR** is a zip of compiled classes plus metadata — Java's distributable unit, like a Python wheel or npm package. You'll build them in Chapter 18.

## Code Examples

### A minimal multi-package project

Directory layout:

```
src/
└── com/
    └── acme/
        ├── app/
        │   └── Main.java
        └── billing/
            ├── Invoice.java
            └── TaxTable.java      (package-private helper)
```

```java
// src/com/acme/billing/TaxTable.java
package com.acme.billing;

// NO access modifier on the class: package-private.
// Only classes inside com.acme.billing can see this — it's an internal detail.
class TaxTable {
    static double rateFor(String category) {
        return switch (category) {
            case "food" -> 0.05;
            case "books" -> 0.00;
            default -> 0.25;
        };
    }
}
```

```java
// src/com/acme/billing/Invoice.java
package com.acme.billing;

public class Invoice {
    private final String category;   // private: internal state
    private final double subtotal;

    public Invoice(String category, double subtotal) {
        this.category = category;
        this.subtotal = subtotal;
    }

    // public API — the only thing the outside world needs
    public double total() {
        double rate = TaxTable.rateFor(category);   // same package: allowed
        return subtotal * (1 + rate);
    }
}
```

```java
// src/com/acme/app/Main.java
package com.acme.app;

import com.acme.billing.Invoice;    // cross-package use requires import (or FQN)

public class Main {
    public static void main(String[] args) {
        Invoice inv = new Invoice("books", 200.0);
        System.out.printf("Total: %.2f%n", inv.total());

        // TaxTable t = new TaxTable();  // ❌ would not compile:
        // TaxTable is not public in com.acme.billing; cannot be accessed from outside package
    }
}
```

Compile and run from the project root (PowerShell):

```powershell
javac -d out (Get-ChildItem -Recurse src -Filter *.java).FullName
java -cp out com.acme.app.Main
```

### Static imports, judiciously

```java
package com.acme.geometry;

import static java.lang.Math.PI;
import static java.lang.Math.pow;

public class Sphere {
    public static double volume(double r) {
        return 4.0 / 3.0 * PI * pow(r, 3);   // reads like the formula
    }
}
```

Use static imports for math-heavy code and test assertions (Chapter 17); avoid them when they obscure where a name comes from.

## Common Pitfalls

### 1. Package line doesn't match the folder

```java
// file lives at src/com/acme/util/Text.java
package com.acme.tools;     // ❌ mismatch — compiles alone, but nothing can find it
package com.acme.util;      // ✅ path and package must agree
```

IDEs flag this instantly; command-line builds fail with confusing "cannot find symbol" errors at *use* sites.

### 2. Running with the wrong name or from the wrong place

```powershell
java -cp out Main                     # ❌ its real name is com.acme.app.Main
java -cp out com.acme.app.Main        # ✅ always the fully qualified name
java -cp src com.acme.app.Main        # ❌ src has .java files; classpath needs .class output
```

### 3. Thinking "no modifier" means private

```java
class Helper {
    int count;          // ❌ if you MEANT private — this is package-visible
    private int count;  // ✅ say what you mean
}
```

Package-private is a deliberate tool, not a default you fall into.

### 4. `protected` assumed to be "subclass only"

`protected` also grants access to the *whole package*. If you want "subclasses only, period," Java simply can't express that — pick `protected` and know what it includes.

### 5. Windows-specific: `;` vs `:` in classpaths

```powershell
java -cp "out:lib/thing.jar" com.acme.Main    # ❌ ':' is the Linux/Mac separator
java -cp "out;lib/thing.jar" com.acme.Main    # ✅ Windows uses ';'
```

Copy-pasted commands from tutorials (usually written for Mac/Linux) break exactly here.

### 6. Wildcard import illusions

```java
import java.util.*;        // gets List, Map, Scanner...
// ...but NOT java.util.function.Function — wildcards don't include subpackages
import java.util.function.*;   // separate import needed
```

## Practice Exercises

1. **Hand-build the layout.** Recreate the `com.acme` example above exactly — folders, three files, package lines — and compile/run it *from PowerShell only* (no IDE). Then break the package line in `TaxTable.java` and record the error you get.
2. **Access audit.** In the example project, try each of these and record the compiler message: (a) instantiate `TaxTable` from `Main`; (b) read `inv.subtotal` from `Main`; (c) change `Invoice`'s class modifier from `public` to nothing and re-import it from `Main`. Restore everything after.
3. **Your own library.** Create packages `dev.<you>.textkit` (with a public `TextStats` class offering `wordCount` and `longestWord` from Chapter 6) and `dev.<you>.demo` (with a `Main` using it). Make any helper methods package-private and justify each visibility choice in comments.
4. **Classpath scavenger hunt.** Compile your library to `out/`, then `cd` into a *different* directory and successfully run `Main` using `-cp` with a full path. Then deliberately run with a wrong `-cp` and a wrong class name; write down both distinct error messages so you'll recognize them forever.
5. **Protected exploration.** Create `com.zoo.Animal` with a `protected String name`, and `com.safari.Lion extends Animal` in a *different* package. Verify Lion can use `name` on itself, but that a non-subclass in `com.safari` cannot touch `animal.name`. Summarize the `protected` rule in your own words as a comment.
