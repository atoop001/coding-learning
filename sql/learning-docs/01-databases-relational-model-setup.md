# Chapter 1: What Databases Are, the Relational Model, and Setting Up SQLite

## Overview

Almost every application you use — a web store, a school gradebook, a banking app — needs to store data so it survives after the program closes. You *could* store data in plain files (text, JSON, CSV), but as soon as you need to search millions of records, let multiple users write at once, or guarantee nothing gets half-saved during a crash, files fall apart. Databases solve these problems.

This chapter explains what a database actually is, introduces the **relational model** (the idea behind SQL databases), and gets a working SQL environment running on your Windows machine using **SQLite** — a tiny, free, zero-configuration database engine that is perfect for learning and is also the most widely deployed database in the world (it's inside your phone, your browser, and most apps you own).

By the end you will have SQLite installed, a graphical browser to look at data, and your first database created.

## Definitions & Explanations

### Database
A **database** is an organized collection of data, stored so it can be efficiently retrieved, updated, and protected. "Organized" is the key word — unlike a folder of random files, a database enforces structure.

### Database Management System (DBMS)
The **DBMS** is the software that manages the database: it reads and writes the files on disk, answers queries, checks rules, and handles concurrent access. Examples: SQLite, PostgreSQL, MySQL, SQL Server, Oracle. When people say "a SQLite database" they usually mean a database file managed by the SQLite engine.

### The Relational Model
Invented by E.F. Codd in 1970, the relational model organizes data into **tables** (formally called *relations*):

- A **table** is a named grid of data about one kind of thing (e.g., `students`, `orders`).
- Each **row** (also called a *record* or *tuple*) is one instance of that thing — one student, one order.
- Each **column** (also called a *field* or *attribute*) is one property of that thing — `name`, `email`, `order_date`. Every column has a **data type** (text, number, date...).
- A **primary key** is a column (or set of columns) whose value uniquely identifies each row — like a student ID. No two rows may share a primary key value.
- A **foreign key** is a column in one table that stores the primary key of a row in another table, creating a **relationship**. An `orders` table might have a `customer_id` column pointing at the `customers` table. This is what "relational" enables: instead of copying the customer's name onto every order, you store it once and *reference* it.

Example, conceptually:

```
customers                        orders
+----+------------+             +----+-------------+------------+
| id | name       |             | id | customer_id | total      |
+----+------------+             +----+-------------+------------+
| 1  | Ada Chen   |  <--------  | 10 | 1           | 42.50      |
| 2  | Ben Ortiz  |  <--------  | 11 | 2           | 19.99      |
+----+------------+             | 12 | 1           | 7.25       |
                                +----+-------------+------------+
```

Order 10 and order 12 both "belong to" Ada because their `customer_id` is `1`.

### SQL
**SQL** (Structured Query Language, pronounced "sequel" or "ess-cue-ell") is the standard language for talking to relational databases. You write *declarative* statements — you describe **what** data you want, and the DBMS figures out **how** to get it. SQL statements fall into a few families you'll learn across this track:

- **Queries**: `SELECT` — read data.
- **Data manipulation (DML)**: `INSERT`, `UPDATE`, `DELETE` — change data.
- **Data definition (DDL)**: `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE` — change structure.
- **Transaction control**: `BEGIN`, `COMMIT`, `ROLLBACK` — group changes safely.

SQL keywords are conventionally written in UPPERCASE (`SELECT`, `FROM`) but the language is case-insensitive for keywords. Statements end with a semicolon `;`.

### Why SQLite for learning?
- **Zero setup**: a whole database is just one file on disk (e.g., `mydata.db`). No server, no accounts, no ports.
- **Standard SQL**: nearly everything you learn transfers directly to PostgreSQL/MySQL (Chapter 16 covers the differences).
- **Everywhere**: Python ships with SQLite support built in (Chapter 15), and it's a legitimate production choice for many apps.

### Other terms you'll meet
- **Schema**: the overall structure of a database — its tables, columns, types, and relationships.
- **Query**: any request to the database, most often a `SELECT`.
- **Result set**: the table-shaped answer a query returns.
- **Client / shell**: a program you type SQL into. We'll use the `sqlite3` command-line shell and DB Browser for SQLite.

## Setting Up on Windows

### Option A (recommended): install everything with winget

Open **PowerShell** and run:

```powershell
winget install SQLite.SQLite          # the sqlite3 command-line shell
winget install DBBrowserForSQLite.DBBrowserForSQLite   # graphical browser
```

Close and reopen PowerShell, then verify:

```powershell
sqlite3 --version
```

You should see a version number like `3.45.0 ...`. If `sqlite3` is not recognized, log out/in or add the install folder to your PATH (winget usually handles this).

### Option B: manual download
1. Go to https://www.sqlite.org/download.html and download the **"sqlite-tools" bundle for Windows** (a zip containing `sqlite3.exe`).
2. Unzip to a folder like `C:\sqlite` and add that folder to your PATH (Settings → System → About → Advanced system settings → Environment Variables).
3. Download **DB Browser for SQLite** from https://sqlitebrowser.org/ and run the installer.

### VS Code setup (optional but nice)
1. Install the **SQLite** extension by alexcvzz, or **SQLite Viewer** — either lets you open `.db` files and run queries inside VS Code.
2. Create a folder for this track, e.g. `D:\atoop\coding-projects\sql\practice`, and open it in VS Code. You can keep `.sql` script files there and run them against your databases.

### DB Browser for SQLite — quick tour
- **New Database** creates a `.db` file.
- The **Browse Data** tab shows table contents like a spreadsheet.
- The **Execute SQL** tab is where you type queries — this is where you'll spend most time.
- **Write Changes** saves your edits to the file (DB Browser holds changes in a pending transaction until you click this).

## Code Examples

### Your first database session (command line)

Open PowerShell in any folder and run:

```powershell
sqlite3 first.db
```

This creates (or opens) the file `first.db` and drops you into the SQLite shell, showing a `sqlite>` prompt. Now type SQL:

```sql
-- Create a table to hold books.
-- 'id' is the primary key; 'INTEGER PRIMARY KEY' auto-numbers rows in SQLite.
CREATE TABLE books (
    id     INTEGER PRIMARY KEY,
    title  TEXT NOT NULL,      -- NOT NULL means this column is required
    author TEXT NOT NULL,
    year   INTEGER
);

-- Insert some rows. Text values use single quotes.
INSERT INTO books (title, author, year) VALUES ('Dune', 'Frank Herbert', 1965);
INSERT INTO books (title, author, year) VALUES ('Kindred', 'Octavia Butler', 1979);
INSERT INTO books (title, author, year) VALUES ('Project Hail Mary', 'Andy Weir', 2021);

-- Read everything back. * means "all columns".
SELECT * FROM books;
```

Expected output:

```
1|Dune|Frank Herbert|1965
2|Kindred|Octavia Butler|1979
3|Project Hail Mary|Andy Weir|2021
```

### Useful shell dot-commands (SQLite shell only, not SQL)

```sql
.tables          -- list tables in the current database
.schema books    -- show the CREATE TABLE statement for a table
.mode column     -- pretty column output
.headers on      -- show column names in results
.quit            -- exit the shell
```

After `.mode column` and `.headers on`, `SELECT * FROM books;` prints:

```
id  title              author          year
--  -----------------  --------------  ----
1   Dune               Frank Herbert   1965
2   Kindred            Octavia Butler  1979
3   Project Hail Mary  Andy Weir       2021
```

### The same thing in DB Browser
1. Open DB Browser → **Open Database** → pick `first.db`.
2. Go to **Execute SQL**, paste `SELECT * FROM books;`, press F5 or the ▶ button.
3. Browse the table visually under **Browse Data**.

### Running a .sql script file

Save queries in a file, e.g. `setup.sql`, then run it against a database:

```powershell
sqlite3 first.db ".read setup.sql"
```

This is how you'll load the sample schemas used throughout this track.

## Common Pitfalls

**1. Using double quotes for text values.**
In SQL, *strings use single quotes*. Double quotes mean "this is a column or table name."

```sql
-- ❌ Wrong (SQLite may tolerate it, other databases will not):
SELECT * FROM books WHERE title = "Dune";

-- ✅ Correct:
SELECT * FROM books WHERE title = 'Dune';
```

**2. Forgetting the semicolon in the shell.**
If you press Enter and see `...>` instead of `sqlite>`, the shell is waiting for the rest of your statement. Type `;` and press Enter to finish it.

**3. Creating the database in the wrong folder.**
`sqlite3 first.db` creates the file *in the current directory*. If you can't find your database later, you probably ran the shell from a different folder. Use full paths when unsure: `sqlite3 D:\atoop\coding-projects\sql\practice\first.db`.

**4. Editing in DB Browser and losing changes.**
DB Browser doesn't save until you click **Write Changes**. If you close without writing, edits are discarded (it will warn you — read the dialog).

**5. Confusing dot-commands with SQL.**
`.tables` and `.schema` only work in the sqlite3 shell. They are not SQL and will fail in DB Browser or Python. The SQL-standard-ish alternative in SQLite is `SELECT name FROM sqlite_master WHERE type = 'table';`.

## Practice Exercises

1. Install SQLite and DB Browser using the steps above. Verify `sqlite3 --version` works in PowerShell, and take note of which version you have.
2. Create a database file called `practice.db`. In it, create a table `movies` with columns: `id` (integer primary key), `title` (required text), `director` (text), `release_year` (integer), and `rating` (a number like 8.5). Insert at least five movies you know.
3. Use `.mode column`, `.headers on`, and `SELECT * FROM movies;` to display your table neatly. Then open the same file in DB Browser and view it in the Browse Data tab.
4. On paper (or in a text file), sketch the relational design for a simple to-do app: what tables would you need for *users* and *tasks*, what columns would each have, and which column links a task to its user? Label the primary keys and the foreign key.
5. In your `practice.db`, run `.schema movies` and read the output. Then run `DROP TABLE movies;` followed by `.tables` to confirm it's gone — and recreate it from the schema you saved. (Getting comfortable destroying and rebuilding throwaway databases is a learning superpower.)
