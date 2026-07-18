# Project 2: Tutoring Business Gradebook

## Description

Design a multi-table relational schema for a small tutoring business (a domain you may know well): students, tutors, subjects, scheduled sessions, and assessment scores. This is your first real *relational* design — foreign keys, a junction-style structure, and queries that join three and four tables to answer the questions an actual tutoring business asks every week: Who is teaching whom? What does each student's schedule look like? How are scores trending?

## Difficulty

**Beginner–Intermediate** — estimated effort: 5–8 hours.

## Chapters Used

- Chapter 5 (table creation, constraints, primary/foreign keys)
- Chapter 6 (joins — the core of this project)
- Chapter 2–4 (querying and data changes throughout)
- Chapter 3 (date functions for schedules)

## Requirements Checklist

### Schema
- [ ] Tables: `students`, `tutors`, `subjects`, `sessions`, and `scores` — each with a surrogate primary key (except where a composite key is clearly better; justify your choice in a comment)
- [ ] `sessions` links one student, one tutor, and one subject, with a date, start time, and duration in minutes (positive, CHECKed)
- [ ] `scores` records assessment results: which student, which subject, the date, the score, and the maximum possible score
- [ ] Each tutor has an hourly rate stored safely for money arithmetic
- [ ] A tutor can teach multiple subjects: model this properly (which table shape does "can teach" require?) and enforce that a session's tutor actually teaches that session's subject is NOT required — but note in a comment how you *would* approach it
- [ ] Every foreign key has a deliberate ON DELETE behavior, each justified in a one-line comment
- [ ] `PRAGMA foreign_keys = ON;` at the top of every script
- [ ] Seed data: at least 6 students, 3 tutors, 4 subjects, 20 sessions spread over several weeks, and 15 scores — enough that queries return interesting results, including at least one student with no sessions and one subject nobody studies

### Queries (save each with a comment stating the business question)
- [ ] The full schedule for one named week: date, time, student name, tutor name, subject — sorted chronologically
- [ ] One student's complete history: every session with tutor and subject names
- [ ] Every student paired with the subjects they've ever had a session in (no duplicates)
- [ ] Students who have had sessions with more than one tutor (think before reaching for tools you haven't learned — a self-join on sessions can do it)
- [ ] All students with **no** sessions at all, and separately, all subjects with no sessions (the LEFT JOIN / IS NULL pattern)
- [ ] Each score shown as a percentage alongside student and subject names
- [ ] A tutor's earnings statement: every session they taught with the computed cost (duration × rate), itemized
- [ ] A "same classroom" report: pairs of students who share at least one subject (self-join through sessions, no duplicate or self-pairs)

### Integrity demonstrations
- [ ] Attempt to schedule a session for a nonexistent student — show the FK rejection
- [ ] Delete (or attempt to delete) a tutor who has sessions — show and explain the outcome your ON DELETE choice produces
- [ ] One CHECK violation attempt per CHECK constraint, kept commented in the script with the observed errors

## Hints

- Sketch the ER diagram on paper first: five boxes, arrows with crow's feet. Every arrow becomes a FK column; every M:N becomes a table.
- "Tutor can teach subjects" and "session happened" are *different relationships* between the same entities — they need different tables.
- For the multiple-tutors query: join sessions to sessions on the same student with different tutors, or count distinct tutors per student if you've peeked at Chapter 7.
- Store times as ISO text (`'14:30'`) and dates separately, or a single `'2026-07-20 14:30'` — either works; be consistent and comment your choice.
- If a join returns more rows than expected, you've probably joined two "many" tables without going through their shared parent.

## Stretch Goals

- [ ] Add `session_status` ('scheduled', 'completed', 'cancelled', 'no_show') with a CHECK, and rewrite the earnings statement to count only completed sessions
- [ ] Add a `parents` table (a student has one or two linked parents) and a query producing an emergency-contact sheet for one day's schedule
- [ ] Enforce "tutor must teach the subject" with the technique of your choice once you reach Chapter 13 (a BEFORE INSERT trigger) — come back and add it
- [ ] After Chapter 7: score averages per student per subject, and tutor monthly earnings summaries
