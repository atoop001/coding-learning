# Chapter 4: Lists & Tables

## Overview

Lists and tables are HTML's tools for structured data. Lists handle sequences and collections — navigation menus, ingredients, steps, FAQs. Tables handle genuinely two-dimensional data — schedules, price comparisons, statistics — where each cell's meaning depends on both its row and its column.

Both are easy to use badly: tables were once abused for page layout (never do this), and lists are often faked with paragraphs and dashes. Used correctly, they're accessible, stylable, and self-documenting.

## Definitions & Explanations

### Unordered lists: `<ul>`

For collections where order doesn't matter (bullets by default):

```html
<ul>
  <li>Milk</li>
  <li>Eggs</li>
  <li>Bread</li>
</ul>
```

- The **only** valid direct children of `<ul>` are `<li>` elements (plus script/template).
- Each `<li>` can contain anything: text, paragraphs, images, even another list.

### Ordered lists: `<ol>`

For sequences where order *is* the point (numbered by default):

```html
<ol>
  <li>Preheat the oven to 200°C.</li>
  <li>Mix the dry ingredients.</li>
  <li>Bake for 25 minutes.</li>
</ol>
```

Useful attributes:

- `start="5"` — begin numbering at 5.
- `reversed` — count down.
- `type="A"` / `"a"` / `"I"` / `"i"` — letters or Roman numerals (usually better done in CSS with `list-style-type`).

Ask: "if I shuffled these items, would the content break?" Yes → `<ol>`. No → `<ul>`.

### Nested lists

A list inside a list item creates sub-levels. The nested list goes **inside the parent `<li>`**, after that item's text:

```html
<ul>
  <li>
    Frontend
    <ul>
      <li>HTML</li>
      <li>CSS</li>
    </ul>
  </li>
  <li>Backend</li>
</ul>
```

### Description lists: `<dl>`

For name/value pairs — glossaries, metadata, FAQs:

```html
<dl>
  <dt>HTML</dt>
  <dd>The language that structures web content.</dd>

  <dt>CSS</dt>
  <dd>The language that styles web content.</dd>
</dl>
```

- `<dt>` = description term, `<dd>` = description details.
- A term can have multiple `<dd>`s, and multiple `<dt>`s can share one `<dd>`.

### Lists as navigation

Menus are lists of links — mark them up that way. Screen readers then announce "list, 4 items," telling users the menu's size up front:

```html
<nav>
  <ul>
    <li><a href="index.html">Home</a></li>
    <li><a href="shop.html">Shop</a></li>
    <li><a href="contact.html">Contact</a></li>
  </ul>
</nav>
```

The bullets and indentation get removed with CSS later (`list-style: none`); structure now, style later.

### Tables: the anatomy

```html
<table>
  <caption>Store hours</caption>
  <thead>
    <tr>
      <th scope="col">Day</th>
      <th scope="col">Open</th>
      <th scope="col">Close</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Mon–Fri</th>
      <td>9:00</td>
      <td>18:00</td>
    </tr>
    <tr>
      <th scope="row">Saturday</th>
      <td>10:00</td>
      <td>16:00</td>
    </tr>
  </tbody>
</table>
```

Element by element:

- `<table>` — the container.
- `<caption>` — the table's title, read first by screen readers. First child of `<table>`.
- `<tr>` — table row.
- `<th>` — **header cell** (bold and centered by default, but its real job is meaning: it labels a row or column).
- `<td>` — data cell.
- `<thead>`, `<tbody>`, `<tfoot>` — group rows into header, body, and footer sections. Optional but good practice; they aid styling and let long printed tables repeat headers.
- `scope="col"` on a `<th>` says "I label the column below me"; `scope="row"` says "I label the row beside me." Screen readers use this to announce the right headers as users move between cells.

### Spanning rows and columns

- `colspan="2"` — this cell is two columns wide.
- `rowspan="3"` — this cell is three rows tall.

When a cell spans, the covered cells are simply *not written* in the affected rows — a frequent source of miscounted rows. Sketch the grid on paper first for complex tables.

### When NOT to use a table

- **Page layout.** Pre-CSS websites used tables to position content. This breaks screen readers, mobile, and maintainability. Layout is CSS's job (flexbox/grid, Chapters 9–10).
- **Lists in disguise.** A single column of items is a list, not a table.

Litmus test: could you meaningfully ask "what's in row 3, column 2?" If the columns don't have distinct meanings, it's not tabular data.

## Code Examples

### Example 1: Recipe combining all three list types

```html
<article>
  <h1>Weeknight Tomato Pasta</h1>

  <h2>At a glance</h2>
  <dl>
    <dt>Prep time</dt>   <dd>10 minutes</dd>
    <dt>Cook time</dt>   <dd>20 minutes</dd>
    <dt>Serves</dt>      <dd>4</dd>
  </dl>

  <h2>Ingredients</h2>
  <ul>
    <li>400g spaghetti</li>
    <li>2 tbsp olive oil</li>
    <li>3 cloves garlic, sliced</li>
    <li>800g canned tomatoes</li>
    <li>Fresh basil</li>
  </ul>

  <h2>Method</h2>
  <ol>
    <li>Boil salted water and cook the spaghetti until al dente.</li>
    <li>Meanwhile, warm the oil and soften the garlic — don't brown it.</li>
    <li>Add tomatoes; simmer 15 minutes, crushing them as they cook.</li>
    <li>Toss pasta with sauce; finish with torn basil.</li>
  </ol>
</article>
```

### Example 2: Nested outline

```html
<h2>Course syllabus</h2>
<ol>
  <li>
    HTML
    <ol>
      <li>Document structure</li>
      <li>
        Content elements
        <ul>
          <li>Text</li>
          <li>Images</li>
          <li>Media</li>
        </ul>
      </li>
    </ol>
  </li>
  <li>
    CSS
    <ol>
      <li>Selectors</li>
      <li>Layout</li>
    </ol>
  </li>
</ol>
```

### Example 3: A real comparison table

```html
<table>
  <caption>Hosting plan comparison (prices per month)</caption>
  <thead>
    <tr>
      <th scope="col">Feature</th>
      <th scope="col">Starter</th>
      <th scope="col">Pro</th>
      <th scope="col">Business</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Price</th>
      <td>$5</td>
      <td>$12</td>
      <td>$29</td>
    </tr>
    <tr>
      <th scope="row">Storage</th>
      <td>10 GB</td>
      <td>50 GB</td>
      <td>200 GB</td>
    </tr>
    <tr>
      <th scope="row">Custom domain</th>
      <td>No</td>
      <td>Yes</td>
      <td>Yes</td>
    </tr>
    <tr>
      <th scope="row">Support</th>
      <td colspan="2">Email only</td>  <!-- spans Starter AND Pro columns -->
      <td>24/7 phone</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <td colspan="4">All plans include free SSL certificates.</td>
    </tr>
  </tfoot>
</table>
```

### Example 4: rowspan — a class schedule

```html
<table>
  <caption>Monday schedule</caption>
  <thead>
    <tr>
      <th scope="col">Time</th>
      <th scope="col">Activity</th>
      <th scope="col">Room</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">9:00</th>
      <td rowspan="2">Workshop: Intro to CSS</td>  <!-- covers 9:00 and 10:00 rows -->
      <td rowspan="2">Lab A</td>
    </tr>
    <tr>
      <th scope="row">10:00</th>
      <!-- No td here: both remaining cells are covered by rowspans above -->
    </tr>
    <tr>
      <th scope="row">11:00</th>
      <td>Q&amp;A session</td>
      <td>Auditorium</td>
    </tr>
  </tbody>
</table>
```

### Example 5: Minimal table styling preview

Tables look cramped by default. A taste of the CSS you'll fully understand by Chapter 8:

```html
<style>
  table {
    border-collapse: collapse; /* merge the double borders into single lines */
  }
  th, td {
    border: 1px solid #999;
    padding: 8px 12px;         /* breathing room inside every cell */
    text-align: left;
  }
  thead {
    background-color: #eee;    /* distinguish the header row */
  }
</style>
```

## Common Pitfalls

1. **Fake lists.**
   ```html
   <!-- ❌ visually a list, semantically three unrelated paragraphs -->
   <p>- Milk</p>
   <p>- Eggs</p>

   <!-- ✅ -->
   <ul>
     <li>Milk</li>
     <li>Eggs</li>
   </ul>
   ```

2. **Content directly inside `<ul>`/`<ol>`.**
   ```html
   <!-- ❌ text and other elements can't be direct children of ul -->
   <ul>
     Shopping list:
     <li>Milk</li>
   </ul>

   <!-- ✅ the label goes outside (or in a heading) -->
   <p>Shopping list:</p>
   <ul><li>Milk</li></ul>
   ```

3. **Nesting a list as a *sibling* of `<li>` instead of inside it.**
   ```html
   <!-- ❌ invalid: ul can't be a direct child of ul -->
   <ul>
     <li>Frontend</li>
     <ul><li>HTML</li></ul>
   </ul>

   <!-- ✅ nested list lives INSIDE the parent li -->
   <ul>
     <li>Frontend
       <ul><li>HTML</li></ul>
     </li>
   </ul>
   ```

4. **All-`<td>` tables.** Using `<td>` for header cells loses the row/column labeling that screen readers depend on, and loses the default styling hint. Headers are `<th>` with a `scope`.

5. **Miscounted cells with spans.** After adding `colspan`/`rowspan`, rows end up with too many or too few cells and the grid shears sideways. Count: every row's cells plus incoming rowspans must total the same number of columns.

6. **Tables for layout.** If the "table" is really a photo next to a paragraph, that's a flexbox job (Chapter 9). Tables are for data.

7. **Forgetting `<caption>`.** Sighted users infer a table's purpose from surrounding text; screen-reader users hearing "table, 5 rows, 4 columns" get no such context. The caption fixes that in one line.

## Practice Exercises

1. **Ordered vs. unordered judgment.** For each, decide `<ul>`, `<ol>`, or `<dl>` and write the markup: (a) the ingredients of a sandwich; (b) instructions for assembling flat-pack furniture; (c) a glossary of five web-dev terms; (d) your top-5 favorite films *ranked*; (e) links in a site footer.

2. **Three-level outline.** Mark up the table of contents of a textbook (real or invented) with chapters (`<ol>`), sections nested inside chapters, and at least one third level. Validate the nesting (nested lists inside `<li>`).

3. **Data table from the wild.** Find a small real dataset (weather forecast, sports standings, nutritional info) and build a proper table: `caption`, `thead`/`tbody`, `<th scope>` on both the header row and the first column.

4. **Span challenge.** Build a weekly timetable (days as columns, hours as rows) where at least one activity spans two hours (`rowspan`) and one spans two days (`colspan`). Verify no row has missing or extra cells by checking it renders as a clean grid.

5. **FAQ page.** Create a FAQ using a `<dl>` where each `<dt>` is a question and each `<dd>` an answer; include at least one question with two answer paragraphs and two questions that share a single answer.
