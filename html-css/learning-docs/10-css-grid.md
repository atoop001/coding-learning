# Chapter 10: CSS Grid

## Overview

CSS Grid is the first CSS system designed for **two-dimensional layout** — rows *and* columns at once. Where flexbox flows content along one line, grid lets you draw the whole page structure first (columns, rows, named regions) and then place content into it. Magazine layouts, dashboards, photo walls, page shells with sidebars — grid handles in a few lines what once required frameworks.

Like flexbox: a **grid container** (`display: grid`) controls **grid items** (direct children). Learn the track model, the `fr` unit, placement by line numbers, `auto-fit`/`minmax` for automatic responsiveness, named template areas, and subgrid for aligning nested grids to the parent's tracks.

## Definitions & Explanations

### Defining a grid

```css
.layout {
  display: grid;
  grid-template-columns: 200px 1fr 1fr;  /* three columns */
  grid-template-rows: auto 300px;         /* two rows */
  gap: 1rem;                              /* same gap property as flexbox */
}
```

- **Tracks** — the columns and rows themselves.
- **Lines** — the numbered boundaries between (and around) tracks. A 3-column grid has column lines 1,2,3,4. Negative numbers count from the end (−1 = last line).
- **Cells** — single track intersections; **areas** — rectangles of cells.

Track sizing values:

- Fixed: `200px`, `10rem`
- `auto` — sized by content
- **`fr`** — a *fraction of the remaining space*. `1fr 2fr` = leftover space split 1:2. The signature grid unit.
- `minmax(min, max)` — track stays within a range: `minmax(150px, 1fr)`
- `repeat()` — shorthand: `repeat(4, 1fr)` = `1fr 1fr 1fr 1fr`; `repeat(3, 100px 1fr)` repeats the *pattern*.

Rows you don't define but content flows into are **implicit** — sized by `grid-auto-rows`:

```css
grid-auto-rows: minmax(120px, auto);  /* every auto-created row: at least 120px */
```

### Placing items

By default items auto-flow into cells left-to-right, top-to-bottom. Explicit placement uses lines:

```css
.item {
  grid-column: 1 / 3;      /* from column line 1 to line 3 (spans 2 columns) */
  grid-row: 2 / 4;         /* rows 2 through 3 */
}
.item-b { grid-column: 1 / -1; }      /* full width: first line to last */
.item-c { grid-column: span 2; }      /* just "span 2 tracks from wherever I land" */
```

Items can overlap (place two items in the same cells) — layering is controlled with `z-index` (Chapter 12).

### Named template areas — layout you can read

```css
.page {
  display: grid;
  grid-template-columns: 220px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  min-height: 100vh;
  gap: 1rem;
}
header { grid-area: header; }
nav    { grid-area: sidebar; }
main   { grid-area: main; }
footer { grid-area: footer; }
```

The ASCII-art strings *are* the layout. Each string is a row; repeated names merge cells into one area; a `.` marks an intentionally empty cell. Rearranging the page = rearranging the strings (which becomes superpowered with media queries in Chapter 11).

### Alignment

Grid uses the same alignment vocabulary as flexbox, doubled for two axes:

- `justify-*` = **inline (row) axis**, `align-*` = **block (column) axis**.
- `justify-items` / `align-items` — align each item *within its own cell* (default `stretch`).
- `justify-content` / `align-content` — align *the whole grid* within the container (matters when tracks don't fill it).
- `justify-self` / `align-self` — per-item override.
- `place-items: center;` — shorthand for both item alignments; `display: grid; place-items: center;` is the shortest perfect-centering recipe in CSS.

### Subgrid: aligning nested grids to the parent

Normally a grid item that's itself `display: grid` starts a brand-new, independent track layout — its rows/columns have no relationship to the parent grid's. That breaks alignment when, say, cards in a row each need their title/body/footer to line up across cards even though each card's content is a different length. `grid-template-columns: subgrid` (or `-rows`) fixes this: the nested grid reuses the parent's tracks instead of defining its own, so cell edges line up perfectly. It's well-supported in current browsers now.

```css
.row { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1rem; }
.card { display: grid; grid-row: span 3; grid-template-rows: subgrid; }
/* each .card's rows now align to the SAME row tracks as its siblings */
```

### Auto-responsive grids: `auto-fit` + `minmax`

The most valuable grid recipe in practice:

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
}
```

Read it as: "create as many columns as fit, each at least 220px, sharing leftover space equally." Columns add/remove themselves as the viewport changes — responsive with **zero media queries**.

- `auto-fill` keeps empty track slots reserved; `auto-fit` collapses them so real items stretch. With a full container they behave identically; with few items, `auto-fit` lets them grow.

### `grid-auto-flow` and dense packing

```css
grid-auto-flow: row;          /* default */
grid-auto-flow: column;       /* flow down columns instead */
grid-auto-flow: dense;        /* backfill holes left by spanning items
                                 (visual order diverges from source order — caution) */
```

### Grid vs flexbox, restated

Ask: "am I designing the *tracks* (grid) or letting content *flow* (flexbox)?" Page shell, image wall, dashboard → grid. Nav links, button row, media object, centering → flexbox. Combining them — grid page, flex components — is the norm, not a compromise.

## Code Examples

### Example 1: First grid — see the tracks

```html
<style>
  .demo {
    display: grid;
    grid-template-columns: 100px 1fr 2fr;
    grid-template-rows: 80px 80px;
    gap: 10px;
  }
  .demo > div {
    background: #dbeafe;
    border: 1px solid #60a5fa;
    display: grid; place-items: center;  /* center the label */
  }
</style>

<div class="demo">
  <div>1</div><div>2</div><div>3</div>
  <div>4</div><div>5</div><div>6</div>
</div>
<!-- Open devtools → Elements → click the "grid" badge next to .demo:
     the browser overlays track lines and numbers. Best grid-learning tool there is. -->
```

### Example 2: Full page shell with named areas

```html
<style>
  * { box-sizing: border-box; margin: 0; }
  body {
    display: grid;
    grid-template-columns: 240px 1fr;
    grid-template-rows: auto 1fr auto;
    grid-template-areas:
      "header header"
      "nav    main"
      "footer footer";
    min-height: 100vh;
    font-family: system-ui, sans-serif;
  }
  .site-header { grid-area: header; background: #0f172a; color: white; padding: 1rem; }
  .site-nav    { grid-area: nav;    background: #f1f5f9; padding: 1rem; }
  .site-main   { grid-area: main;   padding: 1.5rem; }
  .site-footer { grid-area: footer; background: #0f172a; color: white; padding: 1rem; }
</style>

<header class="site-header">Dashboard</header>
<nav class="site-nav">
  <ul><li>Overview</li><li>Reports</li><li>Settings</li></ul>
</nav>
<main class="site-main">
  <h1>Welcome back</h1>
  <p>The main area is the flexible 1fr track in both directions.</p>
</main>
<footer class="site-footer">© 2026</footer>
```

### Example 3: Responsive card gallery — no media queries

```html
<style>
  .gallery {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 1.25rem;
    padding: 1.25rem;
  }
  .gallery article {
    background: white;
    border: 1px solid #e5e7eb;
    border-radius: 12px;
    overflow: hidden;              /* clip image corners to the radius */
  }
  .gallery img { width: 100%; height: 160px; object-fit: cover; display: block; }
  .gallery .text { padding: 1rem; }
</style>

<div class="gallery">
  <!-- repeat this article 6–8 times; resize the window and watch columns reflow -->
  <article>
    <img src="https://picsum.photos/480/320" alt="" />
    <div class="text"><h3>Card title</h3><p>Cards per row adapts automatically.</p></div>
  </article>
</div>
```

### Example 4: Featured-item mosaic with spans

```html
<style>
  .mosaic {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-auto-rows: 140px;
    gap: 0.75rem;
  }
  .mosaic > div {
    background: #ddd6fe;
    border-radius: 8px;
    display: grid; place-items: center;
  }
  .hero-tile { grid-column: 1 / 3;  grid-row: 1 / 3; }  /* 2×2 feature */
  .wide-tile { grid-column: 3 / -1; }                    /* spans to the last line */
  .tall-tile { grid-row: span 2; }
</style>

<div class="mosaic">
  <div class="hero-tile">Featured</div>
  <div class="wide-tile">Wide</div>
  <div>1</div>
  <div class="tall-tile">Tall</div>
  <div>2</div>
  <div>3</div>
  <div>4</div>
</div>
```

### Example 5: Form layout with grid

```html
<style>
  .form-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    max-width: 40rem;
  }
  .form-grid .full { grid-column: 1 / -1; }   /* fields that need the whole row */
  .form-grid label { display: block; margin-bottom: 0.25rem; font-weight: 600; }
  .form-grid input, .form-grid textarea {
    width: 100%; padding: 0.5rem; box-sizing: border-box;
  }
</style>

<form class="form-grid">
  <div><label for="fn">First name</label><input id="fn" name="fn" /></div>
  <div><label for="ln">Last name</label><input id="ln" name="ln" /></div>
  <div class="full"><label for="em">Email</label><input id="em" name="em" type="email" /></div>
  <div class="full"><label for="msg">Message</label><textarea id="msg" name="msg" rows="4"></textarea></div>
  <div class="full"><button type="submit">Send</button></div>
</form>
```

## Common Pitfalls

1. **Grid properties on items, item properties on the container.** Same trap as flexbox: `grid-template-columns` belongs on the container; `grid-column` on items. Nothing errors — it just silently does nothing.

2. **Line numbers vs track counts.** `grid-column: 1 / 3` spans **two** tracks (between lines 1 and 3), not three. Off-by-one placements almost always mean you counted tracks where lines were needed. The devtools grid overlay shows the line numbers — use it.

3. **Malformed `grid-template-areas`.** Every row string must have the same number of cells, and each named area must form a rectangle. `"header header" "sidebar main footer"` (2 vs 3 columns) invalidates the whole declaration silently.

4. **Expecting grid to reach nested elements.** Only direct children become grid items. Wrapping cards in an extra `<div>` inside the container flattens everything into one cell.

5. **`height: 100%` disappointment.** For the classic full-page shell, put `min-height: 100vh` on the grid container itself (as in Example 2) rather than chaining percentage heights up the tree.

6. **`auto-fill` vs `auto-fit` mix-ups.** Three items in a wide `auto-fill` grid huddle at their minimum size next to invisible empty tracks. If you want few items to stretch across, you want `auto-fit`.

7. **Content overflowing `1fr` tracks.** Like flexbox, grid items have `min-width: auto` — a long URL or wide table refuses to shrink and blows the track out. Fix with `min-width: 0` on the item or `overflow-wrap: break-word` / `overflow-x: auto` on the content.

8. **Using grid for simple one-direction rows.** A nav bar as a 5-column grid means editing CSS every time a link is added. Content-count-dependent tracks are a sign flexbox fit better.

9. **`dense` flow with interactive content.** `grid-auto-flow: dense` visually reorders items away from source order; keyboard users tab in source order. Keep it to purely decorative galleries.

## Practice Exercises

1. **Track reading drill.** Without a browser, sketch (paper/ASCII) the grids produced by: (a) `grid-template-columns: 100px repeat(2, 1fr) 2fr` with 8 items; (b) `repeat(auto-fit, minmax(200px, 1fr))` in a 650px container with 5 items; (c) Example 4's mosaic. Then build each and check with the devtools grid overlay.

2. **Magazine front page.** Build a 12-column grid (like real design systems use) containing: a full-width masthead, a lead story spanning 8 columns with a 4-column sidebar beside it, then three equal stories, then a footer. Use line-number placement, at least one negative line number, and one `span`.

3. **Areas rebuild.** Recreate Exercise 2's layout using *only* `grid-template-areas` — no line numbers. Then produce a second version where the sidebar moves to the left side by editing only the area strings.

4. **Auto-fit photo wall.** Build a gallery of 12 images with `repeat(auto-fit, minmax(180px, 1fr))`, square-ish tiles (`aspect-ratio: 1` on items, `object-fit: cover` on images), and make every 5th image span 2 columns and 2 rows (`:nth-child` + `span`). Resize the window; confirm no gaps at any width (try adding `dense` and note the difference).

5. **Dashboard.** Combine everything: a full-viewport app shell (named areas: header / sidebar / main / footer) where `main` itself contains an auto-fit grid of stat cards, one chart placeholder spanning two columns, and a table area with `overflow-x: auto` that cannot blow out its track. Use flexbox inside at least two components (header content, a card's footer) to prove the grid-outside/flex-inside pattern.
