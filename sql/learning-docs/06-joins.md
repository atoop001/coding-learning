# Chapter 6: Joins — Combining Tables

## Overview

Relational databases spread data across multiple tables on purpose (customers here, orders there) — and **joins** put it back together at query time. Joins are the single most important skill separating "knows some SQL" from "can actually use a database." Interviews test them; every real web app query uses them.

This chapter covers `INNER JOIN`, `LEFT JOIN`, self-joins, and the **junction table** pattern for many-to-many relationships.

## Sample Schema for This Chapter

A tiny school. Note the three relationship shapes: one-to-many (teacher→course), many-to-many (student↔course via `enrollments`), and self-referencing (student→mentor who is also a student).

```sql
PRAGMA foreign_keys = ON;

CREATE TABLE teachers (
    id   INTEGER PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE courses (
    id         INTEGER PRIMARY KEY,
    title      TEXT NOT NULL,
    teacher_id INTEGER REFERENCES teachers(id)   -- nullable: course may be unstaffed
);

CREATE TABLE students (
    id        INTEGER PRIMARY KEY,
    name      TEXT NOT NULL,
    mentor_id INTEGER REFERENCES students(id)    -- self-reference! senior student mentor
);

-- Junction table: one row per (student, course) pairing
CREATE TABLE enrollments (
    student_id INTEGER NOT NULL REFERENCES students(id) ON DELETE CASCADE,
    course_id  INTEGER NOT NULL REFERENCES courses(id)  ON DELETE CASCADE,
    grade      TEXT,                              -- NULL until graded
    PRIMARY KEY (student_id, course_id)           -- composite PK: no duplicate pairs
);

INSERT INTO teachers (name) VALUES ('Ms. Rivera'), ('Mr. Okafor');

INSERT INTO courses (title, teacher_id) VALUES
    ('Algebra',        1),
    ('Chemistry',      2),
    ('Creative Writing', 1),
    ('Astronomy',      NULL);          -- unstaffed course

INSERT INTO students (name, mentor_id) VALUES
    ('Ada',   NULL),                   -- Ada has no mentor
    ('Ben',   1),                      -- mentored by Ada
    ('Carla', 1),
    ('Dev',   2);                      -- mentored by Ben

INSERT INTO enrollments (student_id, course_id, grade) VALUES
    (1, 1, 'A'), (1, 2, 'B+'),
    (2, 1, 'B'),
    (3, 3, NULL),
    (4, 1, 'A-'), (4, 3, 'B');
-- Note: nobody is enrolled in Astronomy; Dev has no Chemistry.
```

## Definitions & Explanations

### How a join works, mentally

`FROM a JOIN b ON condition` conceptually pairs **every row of a with every row of b**, then keeps only the pairs where the `ON` condition is true. The result is a wider table containing columns from both. In practice the condition is almost always *foreign key = primary key*.

### INNER JOIN — matches only

```sql
SELECT ... FROM courses
INNER JOIN teachers ON courses.teacher_id = teachers.id;
```

Keeps only rows that match on both sides. Unstaffed Astronomy (teacher_id NULL) disappears, as would any teacher with no courses. `JOIN` alone means `INNER JOIN`.

### LEFT JOIN — keep everything on the left

```sql
SELECT ... FROM courses
LEFT JOIN teachers ON courses.teacher_id = teachers.id;
```

Keeps **every row of the left table**; where no right-side match exists, the right table's columns come back as NULL. This is how you answer "show all X, *with* their Y if any" and — crucially — "which X have *no* Y" (filter `WHERE right.id IS NULL`).

`RIGHT JOIN` is the mirror image (supported in SQLite 3.39+; historically people just swapped table order and used LEFT). `FULL OUTER JOIN` keeps unmatched rows from both sides (also 3.39+). `CROSS JOIN` is the raw all-pairs combination — rarely wanted, occasionally useful for generating combinations.

### Table aliases and qualifying columns

When two tables share column names (`id`, `name`), you must qualify: `students.name` vs `teachers.name`. Aliases keep this readable:

```sql
FROM enrollments e
JOIN students s ON e.student_id = s.id
```

After aliasing, use the alias everywhere (`s.name`), including SELECT.

### Self-join — a table joined to itself

To pair students with their mentors (also students), join `students` to `students` under **two different aliases** — think of them as two copies:

```sql
FROM students s
LEFT JOIN students m ON s.mentor_id = m.id
```

### Many-to-many and junction tables

A student takes many courses; a course has many students. You cannot model this with a single FK in either table. The solution is a **junction table** (`enrollments`) with one row per pairing, FKs to both sides, and a composite PK to forbid duplicates. Querying across it means **two joins**: students → enrollments → courses. Relationship attributes (the `grade`) live on the junction row itself — a grade belongs to the *pairing*, not to the student or the course alone.

### Multi-table joins
Chain as many joins as needed; each `JOIN ... ON ...` clause connects one more table. Read them top to bottom as a path through your schema diagram.

## Code Examples

```sql
-- 1. INNER JOIN: courses with their teachers (Astronomy absent)
SELECT c.title, t.name AS teacher
FROM courses c
JOIN teachers t ON c.teacher_id = t.id;

-- 2. LEFT JOIN: ALL courses, teacher shown when there is one
SELECT c.title, t.name AS teacher       -- Astronomy row appears, teacher = NULL
FROM courses c
LEFT JOIN teachers t ON c.teacher_id = t.id;

-- 3. The "find the missing" pattern: courses with NO teacher
SELECT c.title
FROM courses c
LEFT JOIN teachers t ON c.teacher_id = t.id
WHERE t.id IS NULL;

-- 4. Junction traversal: who takes what, with grades
SELECT s.name AS student, c.title AS course, e.grade
FROM enrollments e
JOIN students s ON e.student_id = s.id
JOIN courses  c ON e.course_id  = c.id
ORDER BY s.name, c.title;

-- 5. All students, including any not enrolled in anything
SELECT s.name, c.title
FROM students s
LEFT JOIN enrollments e ON e.student_id = s.id
LEFT JOIN courses c     ON c.id = e.course_id
ORDER BY s.name;

-- 6. Everyone in Algebra (filter on the far table)
SELECT s.name, e.grade
FROM students s
JOIN enrollments e ON e.student_id = s.id
JOIN courses c     ON c.id = e.course_id
WHERE c.title = 'Algebra';

-- 7. Self-join: each student with their mentor's name
SELECT s.name AS student,
       m.name AS mentor              -- NULL for Ada
FROM students s
LEFT JOIN students m ON s.mentor_id = m.id;

-- 8. Four tables at once: student, course, grade, AND the course's teacher
SELECT s.name AS student, c.title, e.grade, t.name AS teacher
FROM enrollments e
JOIN students s      ON s.id = e.student_id
JOIN courses  c      ON c.id = e.course_id
LEFT JOIN teachers t ON t.id = c.teacher_id
ORDER BY s.name;

-- 9. Classmate pairs: students who share a course (self-join on the junction)
SELECT c.title,
       a.name AS student_1,
       b.name AS student_2
FROM enrollments e1
JOIN enrollments e2 ON e1.course_id = e2.course_id
                   AND e1.student_id < e2.student_id   -- avoids self-pairs & duplicates
JOIN students a ON a.id = e1.student_id
JOIN students b ON b.id = e2.student_id
JOIN courses  c ON c.id = e1.course_id;

-- 10. LEFT JOIN + condition placement (see Pitfall 3):
--     all students, showing ONLY their Algebra grade if any
SELECT s.name, e.grade
FROM students s
LEFT JOIN enrollments e ON e.student_id = s.id AND e.course_id = 1
ORDER BY s.name;
```

## Common Pitfalls

**1. Forgetting the ON condition (accidental cross join).**

```sql
-- ❌ Every student paired with EVERY course — 4 × 4 = 16 nonsense rows:
SELECT s.name, c.title FROM students s JOIN courses c;

-- ✅ Join through the junction table with conditions:
SELECT s.name, c.title
FROM students s
JOIN enrollments e ON e.student_id = s.id
JOIN courses c ON c.id = e.course_id;
```

If a join result is mysteriously huge, you almost certainly lost a join condition.

**2. INNER JOIN silently dropping rows you needed.**
"List all courses and enrollment counts" written with `JOIN` omits Astronomy entirely. When the question contains the word **all**, reach for LEFT JOIN and check whether unmatched rows should appear.

**3. Putting a left-table filter in WHERE vs the right-table filter.**
With a LEFT JOIN, a condition on the *right* table in `WHERE` turns the join back into an INNER JOIN (the NULL rows fail the condition and vanish):

```sql
-- ❌ Meant "all students, with Algebra grade if any" — but unenrolled students disappear:
SELECT s.name, e.grade
FROM students s
LEFT JOIN enrollments e ON e.student_id = s.id
WHERE e.course_id = 1;

-- ✅ Right-table conditions go in ON to preserve left rows:
SELECT s.name, e.grade
FROM students s
LEFT JOIN enrollments e ON e.student_id = s.id AND e.course_id = 1;
```

Rule of thumb: filters on the *left/preserved* table → WHERE; filters on the *right/optional* table → ON.

**4. Ambiguous column names.**
`SELECT name FROM students s JOIN teachers t ...` errors with "ambiguous column name". Qualify every column in multi-table queries, even unambiguous ones — future-you will thank you.

**5. Duplicate rows from one-to-many joins.**
Joining customers to orders repeats each customer once per order — that's correct join behavior, not a bug. If you want one row per customer, you want aggregation (Chapter 7), not `DISTINCT` slapped on to hide the symptom.

**6. Missing the junction table and putting an FK on the wrong side.**
A `course_id` column on `students` would mean each student takes exactly one course. If both sides can have many, you need the junction table — no exceptions.

## Practice Exercises

Use the school schema above.

1. List every teacher together with the titles of the courses they teach, including teachers who currently teach nothing (add one such teacher first to make it interesting).
2. Produce a report of students who are **not** enrolled in any course (add a new student to test it). Then write the opposite: courses with no enrolled students, which should find Astronomy.
3. Show each enrollment as one line: `student — course — teacher — grade`, sorted by course then student. Ungraded enrollments and unstaffed courses must still appear correctly.
4. Using a self-join, list every mentor together with **how they relate**: each row should show mentor name and mentee name. Then extend it one level: show "grand-mentor" chains (Dev → Ben → Ada) with three name columns.
5. The school adds classrooms: each course meets in one room, each room hosts many courses, and you must also record which period (1–6) each course uses its room. Decide which table(s) and column(s) to add, write the CREATE TABLE / ALTER TABLE statements with proper keys, seed a few rows, and write one query joining students all the way through to room numbers.
