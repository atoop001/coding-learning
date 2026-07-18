# Project 7: Sales Data Analyzer

## Description

A data-processing application: read a CSV file of sales records (order id, date, region, product, quantity, unit price), clean and validate the rows, and produce an analytics report — totals, per-region and per-product breakdowns, top performers, monthly trends — computed with the **Stream API**, written to an output report file, and verified by a **JUnit test suite**. This is the shape of an enormous amount of real backend work: ingest, transform, aggregate, emit — with tests proving the numbers.

## Difficulty

**Advanced** — estimated effort: 10–14 hours.

## Chapters used

- 15 (Lambdas & Streams) — the entire analytics layer
- 16 (File I/O & try-with-resources) — reading input, writing reports
- 17 (Unit Testing with JUnit) — tests for parsing and analytics
- (Supporting: 12 collections, 13 generics, 14 exceptions)

## Requirements checklist

### Data layer
- [ ] `record Sale(String orderId, LocalDate date, String region, String product, int quantity, double unitPrice)` with a derived `total()` method
- [ ] A parser turning one CSV line into a `Sale`, throwing a descriptive exception on malformed input (wrong field count, bad number, bad date — `LocalDate.parse`)
- [ ] File loader using try-with-resources that collects good rows, counts and reports skipped bad rows (with line numbers), and never crashes on a partially-bad file
- [ ] A generator utility that writes a sample CSV of 100+ randomized rows (fixed seed!) so the app is runnable out of the box — including a few deliberately corrupt lines

### Analytics (each implemented as a stream pipeline — no manual loops in this layer)
- [ ] Total revenue across all sales
- [ ] Revenue grouped by region (`Collectors.groupingBy` + summing), printed sorted by revenue descending
- [ ] Top 3 products by units sold
- [ ] Average order value, and count of orders above it
- [ ] Best month: group by `YearMonth` (from the date), find the max by revenue — returns an `Optional` handled properly (no bare `.get()`)
- [ ] At least one pipeline using `filter` + `map` + `sorted` + `limit` together, and one using `anyMatch`/`allMatch`

### Output & tests
- [ ] A formatted plain-text report written to `report.txt` (sections, aligned columns) via buffered writer in try-with-resources
- [ ] JUnit tests for the parser: valid line, wrong field count, non-numeric quantity, bad date — using `assertThrows` with message checks
- [ ] JUnit tests for every analytics method against a small hand-computed fixture (5–6 sales you can verify with a calculator) built in `@BeforeEach`
- [ ] Edge tests: empty sale list (totals are 0 / Optionals empty — no exceptions), single sale, tie for best month (define the behavior, then test it)
- [ ] Analytics methods take `List<Sale>` and return values — they never read files or print (that separation is what makes them testable)

## Hints

- Layer it strictly: `SaleParser` (String → Sale), `SaleRepository` (file → List<Sale>), `SalesAnalytics` (List<Sale> → numbers), `ReportWriter` (numbers → file), and a thin `Main`. Test the middle layers; the file layers get one round-trip test each.
- Develop analytics against the tiny fixture first with tests, *then* point them at the generated 100-row file. If the fixture math is wrong, the big file is unverifiable.
- `Collectors.groupingBy(Sale::region, Collectors.summingDouble(Sale::total))` is the archetype — most requirements are variations on it.
- Sorting a map by value: stream its `entrySet()` with `Map.Entry.comparingByValue(Comparator.reverseOrder())`.
- For the fixed seed: `new Random(42)` — tests and reruns stay reproducible.
- Never compare computed doubles to exact literals in tests — use the delta overload of `assertEquals`.

## Stretch goals

- Command-line arguments: `java ... analyze data.csv --region EMEA --from 2025-01-01` filtering the pipeline.
- CSV quirks: support quoted fields containing commas (this is genuinely tricky — write tests first).
- A second output format: the same report as HTML or Markdown, selected by a flag — notice how the layering makes this cheap.
- Performance taste test: run the top-products pipeline with `.parallelStream()` on 1,000,000 generated rows and time both variants.
- Export aggregates as JSON by hand-building the string — then note how much you want Chapter 18's Gson.
