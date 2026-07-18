# Project 4: Normalize the Messy Spreadsheet

## Description

Every business has one: the giant spreadsheet that runs everything and is quietly full of contradictions. Your job is the classic rescue mission — take a denormalized, dirty, single-table data dump (an event-registration sheet you'll create from the spec below), diagnose exactly which normalization rules it violates, design a proper 3NF schema, migrate the data into it with SQL, clean the dirt along the way, and prove the result is consistent. This is the most "real job interview" project in the track: normalization *applied*, not recited.

## Difficulty

**Intermediate** — estimated effort: 6–10 hours.

## Chapters Used

- Chapter 10 (normalization & ER modeling — the core)
- Chapter 14 (NULL handling & data cleaning)
- Chapter 5 (constraints that prevent regression)
- Chapter 4 (INSERT ... SELECT migration)
- Chapter 8 (subqueries for auditing)

## The Mess (create this yourself)

Build a table `registrations_flat` and hand-enter **at least 30 rows** matching this shape — inventing the data, but deliberately including every listed defect:

Columns: `attendee_name`, `attendee_email`, `attendee_company`, `company_city`, `event_name`, `event_date`, `event_venue`, `venue_capacity`, `ticket_type` ('standard'/'vip'/'student' — with inconsistent casing in the data), `price_paid` (some as `'45'`, some `'45.00'`, some `'FREE'`), `dietary_prefs` (comma-separated lists like `'vegetarian, gluten-free'`), `registered_on`.

Required defects to plant:
- [ ] The same attendee registered for multiple events with their email spelled in different cases (and once misspelled)
- [ ] The same event's venue/capacity repeated on every row — with one contradictory capacity
- [ ] Company city repeated per company — with one contradiction
- [ ] Comma-separated `dietary_prefs` (1NF violation), with inconsistent spacing
- [ ] Sentinels: `'FREE'`, `'N/A'`, empty strings, and true NULLs all meaning "missing" somewhere
- [ ] At least one full duplicate row (double-submitted registration)

## Requirements Checklist

### Diagnosis
- [ ] A written analysis (comments in your script or a short markdown file): for each normal form 1NF→3NF, which columns violate it and the specific functional dependency at fault (e.g. `event_name → venue`, `venue → capacity`)
- [ ] Audit queries proving each planted defect exists: duplicate detection, contradictory-fact detection (same event, two capacities), sentinel census, comma-list detection
- [ ] An ER sketch (text art or photo) of the target design before any DDL is written

### Target schema
- [ ] 3NF tables — expected shape is roughly: `attendees`, `companies`, `venues`, `events`, `ticket_types` (or a CHECK — justify), `registrations` (the junction with price actually paid), `dietary_preferences` + a junction to attendees — but *your* justified design wins over this list
- [ ] Full constraints: PKs, FKs with chosen ON DELETE, UNIQUE on natural keys (email — normalized to lowercase), CHECKs on price and dates, deliberate NULLability per column
- [ ] Money stored as integer cents; `'FREE'` becomes 0 — *decide and document* whether "free" and "unknown price" are the same thing (they aren't)

### Migration (the heart of the project — all in SQL, transactional)
- [ ] Populate every clean table from `registrations_flat` using `INSERT ... SELECT DISTINCT` with normalization functions (LOWER, TRIM, CAST, NULLIF, CASE)
- [ ] Resolve each contradiction with a documented rule (e.g. "take the capacity from the most recent row" or "flag for human review") — no silent coin-flips
- [ ] Split the comma-separated dietary lists into junction rows (hand-mapping the distinct list values to rows is acceptable; a recursive-CTE splitter is a stretch goal)
- [ ] De-duplicate attendees and registrations with an explicit keep-rule
- [ ] The whole migration wrapped in a transaction, preceded by a file backup

### Verification
- [ ] Row-count reconciliation: every flat row accounted for (migrated, merged, or explicitly rejected — with counts that add up, shown by queries)
- [ ] A rebuilt "flat view" (Chapter 10's pattern): one SELECT joining the clean schema that reproduces the original shape — minus the contradictions
- [ ] Prove the update-anomaly is dead: change one venue's capacity with a single-row UPDATE and show every event at that venue sees it
- [ ] Prove regression is impossible: attempted INSERTs of each original defect class into the clean schema, each rejected by a constraint (kept commented with error messages)

## Hints

- Do the diagnosis honestly *before* designing — the design falls out of the list of functional dependencies almost mechanically.
- Migrate parents before children: companies/venues → attendees/events → registrations → dietary junctions. FKs will force this order anyway.
- `GROUP BY event_name HAVING COUNT(DISTINCT venue_capacity) > 1` is your contradiction detector — the same shape works for every repeated-fact column.
- For the misspelled email: dedup by a rule you can defend (same name + near-same email needs a human decision — model that as a `review_queue` table rather than guessing).
- Keep the flat table around untouched. It's your audit trail and your re-run source when (not if) you redo the migration.

## Stretch Goals

- [ ] Write the dietary-list splitter as a recursive CTE over the comma positions — fully automatic 1NF repair
- [ ] Add a `data_issues` table populated by your audit queries (issue type, source row, resolution) — a professional-grade cleanup log
- [ ] Time-travel integrity: add CHECK/trigger logic ensuring `registered_on` is never after `event_date`
- [ ] After Chapter 15: rewrite the whole migration as a Python script with the flat data arriving from a real CSV file
