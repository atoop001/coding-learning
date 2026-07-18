# Chapter 18: Build Tools, JARs, and the Java Ecosystem

## Overview

You can compile a handful of files with `javac`, but real projects have hundreds of source files, dozens of third-party libraries, tests to run, and JARs to ship. **Build tools** automate all of it. This chapter covers Maven (the enterprise standard) and Gradle (its main rival), the standard project layout, dependencies, building runnable JARs — and then zooms out: where Java is actually used (Spring, Android, big data), and what to learn next for employability.

## Definitions & Explanations

### Why build tools

A build tool handles: compiling everything in the right order, **downloading dependencies** (and *their* dependencies, transitively), running tests, packaging JARs, and doing it identically on your machine, a teammate's, and a CI server. It's npm/pip/poetry for Java — plus the compile step those ecosystems don't have.

- **Maven** — XML configuration (`pom.xml`), rigid conventions, boring in the best way. The most common choice in enterprise. **Learn this first.**
- **Gradle** — Groovy/Kotlin script configuration (`build.gradle`), more flexible and faster. Standard for Android. Trivial to pick up once you know Maven concepts.

### The standard directory layout

Maven's convention (Gradle uses the same) — burn it in, every Java repo you'll ever open looks like this:

```
my-app/
├── pom.xml                      ← the build file
└── src/
    ├── main/
    │   ├── java/                ← production code (package dirs under here)
    │   └── resources/           ← non-code files packaged with the app
    └── test/
        └── java/                ← JUnit tests, mirroring main's packages
```

Compiled output lands in `target/` (Gradle: `build/`) — generated, git-ignored, safe to delete.

### A minimal pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Your project's coordinates: how OTHERS would name it -->
    <groupId>dev.yourname</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.release>21</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <!-- JUnit 5 for testing; scope=test keeps it out of the shipped app -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

Dependencies are identified by **coordinates** — `groupId:artifactId:version` — and fetched from **Maven Central** (the ecosystem's npm registry; browse it at central.sonatype.com or mvnrepository.com). Downloads cache in `C:\Users\<you>\.m2\repository`.

### The Maven lifecycle — commands you'll actually run

```powershell
mvn compile        # compile src/main/java  -> target/classes
mvn test           # compile + run all JUnit tests
mvn package        # compile + test + build target/my-app-1.0.0.jar
mvn clean          # delete target/ (combine: mvn clean package)
mvn verify         # package + any integration checks — the "is everything OK" command
```

Each phase runs everything before it — `package` won't build if tests fail (a feature: broken code doesn't ship). IntelliJ runs all of this from the Maven side-panel; opening a folder containing `pom.xml` auto-imports the project, dependencies and all.

Install: download from maven.apache.org, unzip, add its `bin` to PATH — or let IntelliJ use its bundled Maven. Verify with `mvn -version`.

### JARs and running your app

A **JAR** (Java ARchive) is a zip of `.class` files + metadata (`META-INF/MANIFEST.MF`) — the unit of distribution:

```powershell
java -jar target/my-app-1.0.0.jar        # works IF the manifest declares a Main-Class
java -cp target/my-app-1.0.0.jar dev.yourname.Main   # works regardless
```

To make `java -jar` work, configure the `maven-jar-plugin` with your main class; to bundle dependencies *into* one fat/uber JAR (needed once you have any), use the `maven-shade-plugin`. Both are copy-paste pom snippets — the capstone project walks you through it.

### Where Java is used — the map for your job hunt

- **Spring / Spring Boot** — *the* dominant framework for enterprise backends: REST APIs, dependency injection, database access, security. The single highest-value thing to learn after this track. A Boot "hello REST endpoint" is ~15 lines; start at start.spring.io.
- **Android** — mobile apps on the JVM family (nowadays Kotlin-first, but Java-capable, Gradle-built, same core concepts).
- **Big data & infrastructure** — Kafka, Elasticsearch, Hadoop, Spark, Cassandra, Jenkins, Minecraft — all JVM. Java's performance + ecosystem keeps it under everything.
- **Adjacent JVM languages** — Kotlin and Scala interoperate with Java completely; your Java fundamentals transfer 1:1.

Realistic employability path from here: **Spring Boot → SQL/JPA (databases) → Git fluency + Maven → REST API portfolio project → basic Docker.** That stack matches a huge fraction of junior Java postings.

## Code Examples

### Bootstrapping a Maven project from scratch (PowerShell)

```powershell
mvn archetype:generate "-DgroupId=dev.yourname.hello" "-DartifactId=hello-maven" `
    "-DarchetypeArtifactId=maven-archetype-quickstart" "-DarchetypeVersion=1.5" `
    "-DinteractiveMode=false"
cd hello-maven
mvn package
java -cp target/hello-maven-1.0-SNAPSHOT.jar dev.yourname.hello.App
```

(The quotes around `-D` flags matter in PowerShell.) Or skip the archetype: create the folder layout and `pom.xml` by hand — it's genuinely just those files — then open the folder in IntelliJ.

### Using a real third-party library

Add Gson (Google's JSON library) to the pom:

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.11.0</version>
</dependency>
```

```java
package dev.yourname.hello;

import com.google.gson.Gson;
import java.util.List;

record Task(String title, boolean done) { }

public class JsonDemo {
    public static void main(String[] args) {
        Gson gson = new Gson();

        // Object -> JSON string (like JSON.stringify / json.dumps)
        List<Task> tasks = List.of(new Task("learn maven", true),
                                   new Task("learn spring", false));
        String json = gson.toJson(tasks);
        System.out.println(json);
        // [{"title":"learn maven","done":true},{"title":"learn spring","done":false}]

        // JSON string -> object (like JSON.parse / json.loads)
        Task parsed = gson.fromJson("{\"title\":\"ship it\",\"done\":false}", Task.class);
        System.out.println(parsed.title() + " -> done=" + parsed.done());
    }
}
```

The magic to appreciate: you declared three lines of XML, and Maven fetched the library, put it on the compile classpath *and* the test classpath, and will warn about version conflicts. No manual JAR downloads, ever.

### Manifest config for `java -jar`

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.4.1</version>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>dev.yourname.hello.JsonDemo</mainClass>
                    </manifest>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Then: `mvn package` → `java -jar target/hello-maven-1.0-SNAPSHOT.jar` (add the shade plugin once dependencies like Gson must ride along).

## Common Pitfalls

### 1. Code outside the sacred layout

```
src/MyClass.java              # ❌ Maven can't see it
src/main/java/MyClass.java    # ✅ (ideally inside a package folder too)
```

Convention over configuration: Maven doesn't *find* your code, it *expects* it.

### 2. Test classes in main (or vice versa)

Tests in `src/main/java` ship JUnit with your app and clutter the JAR; production code in `src/test/java` silently never gets packaged. The split is meaningful — respect it.

### 3. Dependency declared but "cannot find symbol" anyway

Usually: (a) IntelliJ hasn't re-imported — click the Maven refresh icon (Load Maven Changes); (b) typo'd coordinates — check mvnrepository.com for the exact groupId/artifactId; (c) `scope=test` on something main code uses.

### 4. `java -jar` says "no main manifest attribute"

The JAR has no `Main-Class` in its manifest. Add the jar-plugin config above, or run with `-cp jar + full class name`. And once you use external libraries, a plain JAR won't contain them — that's the shade/fat-jar step, not an error in your code.

### 5. SNAPSHOT confusion

`1.0-SNAPSHOT` means "unreleased, still moving." Fine during development; never depend on someone else's SNAPSHOT for anything you care about.

### 6. Fighting the tool instead of the convention

Renaming `target/`, hand-editing files inside it, committing it to git, or restructuring the layout "to be cleaner" all end in pain. `target/`/`build/` are disposable outputs — add them to `.gitignore` and never look inside except to grab your JAR.

## Practice Exercises

1. **Hand-rolled Maven.** Without the archetype, create the full layout (folders + minimal `pom.xml` + a `Main` in package `dev.<you>.first` + one JUnit test) using only a text editor and PowerShell. Run `mvn test` and `mvn package`, then run the JAR both ways (`-jar` after adding the manifest config, `-cp` before). Keep this as your template.
2. **Dependency field trip.** Find Apache Commons Lang (`commons-lang3`) on mvnrepository.com, add it, and use `StringUtils.capitalize` and `StringUtils.isBlank` in a small program. Then run `mvn dependency:tree` and read the output. How many JARs is your one-dependency project actually using?
3. **Migrate a chapter.** Take your Chapter 17 tests (MathKit or Temperature) out of IntelliJ's ad-hoc setup and into a proper Maven project: code in `src/main/java`, tests in `src/test/java`, JUnit via the pom. Confirm `mvn test` runs them from the command line — this exact flow is what CI servers do.
4. **JSON inventory.** Using Gson, extend Chapter 12's inventory: save the `Map`/records to `inventory.json` on exit and load it on startup (combine with Chapter 16's file I/O). You now have persistence with a real third-party library — the everyday texture of professional Java.
5. **Recon for what's next.** Visit start.spring.io and generate (don't build yet — just read) a Maven project with the "Spring Web" dependency. Open its pom.xml and its main class. Write down: three things you recognize from this track, and three new things you'd need to learn. That second list is your post-track syllabus.
