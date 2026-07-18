# Project 6: Movie Data Query Tool (LINQ + File I/O)

## Description

Build a console tool that loads a dataset of movies from a CSV file, then offers an interactive query menu powered end-to-end by LINQ: filter by genre/year/rating, sort, group, aggregate, and export results to JSON. You'll create the dataset file yourself (a starter block is specified below), parse it defensively, and treat LINQ as your query engine — exactly how professional C# code slices data every day.

## Difficulty

**Intermediate–Advanced** — estimated effort: 7–10 hours.

## Chapters Used

- 15 LINQ
- 16 File I/O & Async/Await Basics
- 12 Collections & Generics
- 13 Exceptions & Error Handling
- 08 Classes (records)

## Requirements Checklist

- [ ] A `Movie` record: `Title`, `Genre`, `Year`, `Rating` (0.0–10.0), `Minutes`
- [ ] A `movies.csv` data file you author with **at least 25 rows** spanning ≥5 genres and ≥3 decades, plus a header line, plus (deliberately) 2 malformed rows for testing
- [ ] Loading: read the file, skip the header, parse each line into a `Movie`; malformed lines are skipped with a warning naming the line number — the program never crashes on bad data
- [ ] A missing data file produces a clear message and a graceful exit, not an unhandled exception
- [ ] Menu of queries, each implemented as a LINQ chain:
  - [ ] All movies of a genre (user-supplied, case-insensitive), sorted by rating descending
  - [ ] Top N movies overall (N from user input, validated)
  - [ ] Movies from a year range (from–to), sorted by year then title
  - [ ] Average rating and total runtime **per genre**, one line each (GroupBy)
  - [ ] The best movie of each decade (GroupBy on `Year / 10 * 10`)
  - [ ] Search: titles containing a substring, case-insensitive
  - [ ] Stats dashboard: count, mean rating, longest movie, shortest movie, most common genre — in one aligned display
- [ ] Every query result prints via one shared display method taking `IEnumerable<Movie>` (write it once)
- [ ] Empty results print "no matches" rather than nothing or an exception (`FirstOrDefault`/`Any` where appropriate)
- [ ] An export option writes the **most recent query's results** to `results.json`, indented, using `System.Text.Json` — and reports the absolute path written
- [ ] File writes use async APIs (`WriteAllTextAsync`) with an async `Main`

## Hints

- Author the CSV by hand or generate it once with a throwaway script — either way, commit to a fixed schema: `Title,Genre,Year,Rating,Minutes`.
- Parsing: `line.Split(',')` then `int.TryParse`/`double.TryParse` each numeric field; a single `TryParseMovie(string line, out Movie?)` method concentrates all the defensiveness in one place.
- Titles containing commas will break naive splitting — either forbid them in your data (document it) or handle quoted fields as a stretch goal.
- "Most recent query's results" means each menu handler should materialize its chain (`.ToList()`) and stash it in one shared variable — a natural demonstration of deferred execution vs snapshots.
- Decade key: `m.Year / 10 * 10` (integer division doing something useful for once).
- For per-genre lines, project groups into an anonymous type or tuple `(Genre, Avg, TotalMin)` before printing.
- Keep each menu case to ~5 lines by pushing chains into named methods like `TopN(int n)` returning `List<Movie>`.

## Stretch Goals

- Add `Directors` as a semicolon-separated field and a query "all movies by director X" (`SelectMany` or `Split` inside `Where`).
- Support combined filters (genre AND year range AND minimum rating) built up conditionally — start from the full set and apply `Where` clauses only for criteria the user supplied.
- Load the CSV with `File.ReadAllLinesAsync` and time it against the sync version on a 100k-row generated file.
- Add a `sort by` free choice (title/year/rating/runtime, asc/desc) implemented by selecting a `Func<Movie, object>` key selector from a dictionary.
- Round-trip: an import option that reads back your exported `results.json` and merges (no duplicate titles) into the in-memory list.
