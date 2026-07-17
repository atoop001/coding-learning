# Chapter 7: The Box Model

## Overview

Every element on a web page is a rectangular box. The **box model** describes that box's layers: content, padding, border, and margin. Layout, spacing, alignment, "why is this wider than I said," "why won't these gaps collapse," — nearly every early CSS frustration traces back to the box model.

This chapter also covers `display` values (block, inline, inline-block, none), `box-sizing` (the fix that makes widths sane), and overflow. Internalize this and the layout chapters (flexbox, grid) will feel easy.

## Definitions & Explanations

### The four layers

From inside out:

```
┌─────────────────────────── margin (transparent, outside) ─┐
│  ┌───────────────────────── border ────────────────────┐  │
│  │  ┌─────────────────────── padding ───────────────┐  │  │
│  │  │  ┌───────────────────── content ───────────┐  │  │  │
│  │  │  │   text, images, child elements          │  │  │  │
│  │  │  └─────────────────────────────────────────┘  │  │  │
│  │  └───────────────────────────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────┘
```

- **Content box** — where text and children live; sized by `width`/`height`.
- **Padding** — space between content and border. Takes the element's background.
- **Border** — the edge line. `border: 2px solid black` (width, style, color).
- **Margin** — space *outside* the border, pushing neighbors away. Always transparent.

Devtools shows this exact diagram: Elements → select an element → the colored box at the bottom of the Styles/Computed pane. Orange = margin, green = padding, blue = content.

### Shorthand: 1–4 values, clockwise

`margin` and `padding` accept up to four values, applied **clockwise from the top**:

```css
padding: 10px;                  /* all four sides */
padding: 10px 20px;             /* vertical 10, horizontal 20 */
padding: 10px 20px 30px;        /* top 10, sides 20, bottom 30 */
padding: 10px 20px 30px 40px;   /* top, right, bottom, left (TRouBLe) */

/* Individual sides: */
margin-top: 1rem;
padding-left: 2rem;
```

Mnemonic: **TRBL — "trouble"** — Top, Right, Bottom, Left.

### `box-sizing`: the sanity switch

By default (`box-sizing: content-box`), `width` sets only the **content** width — padding and border are *added on top*:

```css
.card { width: 300px; padding: 20px; border: 5px solid; }
/* content-box: actual rendered width = 300 + 20 + 20 + 5 + 5 = 350px 😖 */
```

With `border-box`, `width` means the whole visible box — padding and border squeeze *inward*:

```css
.card { box-sizing: border-box; width: 300px; padding: 20px; border: 5px solid; }
/* rendered width = exactly 300px; content shrinks to 250px 😌 */
```

Virtually every professional stylesheet starts with this reset:

```css
*, *::before, *::after {
  box-sizing: border-box;
}
```

Adopt it now and never think about it again.

### `display`: how boxes behave

- `block` — starts on a new line, fills the container's width, respects `width`/`height` and all margins/paddings. (`div`, `p`, `h1`, `section`…)
- `inline` — flows within text, only as wide as its content. **Ignores** `width`/`height`; vertical margins/paddings don't push lines apart. (`span`, `a`, `strong`…)
- `inline-block` — flows inline **but** respects width/height/vertical spacing. Good for badges, buttons in text.
- `none` — removed entirely: takes no space, invisible to screen readers. (Compare `visibility: hidden` — invisible but still occupies its space.)
- `flex`, `grid` — layout superpowers for the *children*; Chapters 9–10.

You can change any element's display: `a { display: block; }` makes links fill their container (common for nav items to enlarge the click target).

### Margin behaviors you must know

**1. `margin: 0 auto` centers block elements horizontally** (element needs a set width/max-width):

```css
.container { max-width: 60rem; margin: 0 auto; }
```

**2. Margin collapsing.** *Vertical* margins of adjacent block elements don't add — they **overlap**, and the larger one wins:

```css
p { margin-top: 20px; margin-bottom: 20px; }
/* Gap between two paragraphs = 20px, NOT 40px */
```

Collapsing also happens between a parent and its first/last child when nothing (border, padding) separates them — margins "escape" through the parent. Horizontal margins never collapse. Margins inside flex/grid containers never collapse.

**3. Negative margins** pull elements closer / overlap them. Occasionally useful; mostly a smell at this stage.

### Height, min/max, and overflow

- Elements are naturally as tall as their content. Prefer leaving `height` alone; use `min-height` when you need "at least this tall, grow if needed."
- `max-width` beats fixed `width` for responsive-friendly boxes: `width: 100%; max-width: 40rem;`.
- When content is bigger than a constrained box, `overflow` decides: `visible` (default — spills out), `hidden` (clipped), `scroll` (always scrollbars), `auto` (scrollbars only when needed).

### Borders and rounding

```css
.box {
  border: 1px solid #ccc;        /* shorthand: width style color */
  border-radius: 8px;            /* rounded corners */
  border-bottom: 3px solid teal; /* single side */
}
.avatar { border-radius: 50%; }  /* perfect circle (on a square element) */
```

`outline` is similar to border but sits *outside* the box without affecting layout — the browser uses it for focus rings. Style it, never just remove it.

## Code Examples

### Example 1: Seeing every layer

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Box model demo</title>
<style>
  *, *::before, *::after { box-sizing: border-box; }

  .demo {
    width: 300px;
    padding: 24px;                 /* space inside — background shows here */
    border: 6px solid steelblue;
    margin: 40px;                  /* space outside — pushes neighbors away */
    background-color: lightsteelblue; /* fills content + padding */
  }
</style>
</head>
<body>
  <div class="demo">Inspect me in devtools and hover the box-model diagram.</div>
  <div class="demo">Second box — note the gap between us is 40px, not 80px (margin collapse).</div>
</body>
</html>
```

### Example 2: content-box vs border-box, side by side

```html
<style>
  .a, .b {
    width: 200px;
    padding: 20px;
    border: 10px solid coral;
    margin-bottom: 1rem;
    background: papayawhip;
  }
  .a { box-sizing: content-box; } /* renders 260px wide */
  .b { box-sizing: border-box;  } /* renders 200px wide */
</style>

<div class="a">content-box: I'm secretly 260px wide.</div>
<div class="b">border-box: I'm exactly the 200px you asked for.</div>
```

### Example 3: A card component — the box model earning its keep

```html
<style>
  *, *::before, *::after { box-sizing: border-box; }
  body {
    font-family: system-ui, sans-serif;
    background: #f0f2f5;
    margin: 0;
  }

  .card {
    max-width: 20rem;
    margin: 2rem auto;              /* auto = horizontally centered */
    padding: 1.5rem;
    background: white;
    border: 1px solid #e0e0e0;
    border-radius: 12px;
  }

  .card h2 {
    margin: 0 0 0.5rem;             /* kill default top margin; tidy bottom gap */
    font-size: 1.25rem;
  }

  .card p  { margin: 0 0 1rem; color: #555; }

  .card .tag {
    display: inline-block;          /* inline flow + padding that works */
    padding: 0.25em 0.75em;
    background: #e8f0fe;
    color: #1a56db;
    border-radius: 999px;           /* pill shape */
    font-size: 0.8rem;
  }
</style>

<div class="card">
  <h2>Box Model Mastery</h2>
  <p>Padding for inner breathing room, margin for outer distance, border-box so the math never surprises you.</p>
  <span class="tag">CSS</span>
  <span class="tag">Fundamentals</span>
</div>
```

### Example 4: Centered page container — the universal pattern

```css
.container {
  max-width: 70rem;   /* never wider than this */
  margin: 0 auto;     /* centered when the viewport is wider */
  padding: 0 1rem;    /* breathing room so content never touches screen edges */
}
```

```html
<header><div class="container">Header content</div></header>
<main class="container">Page content</main>
<footer><div class="container">Footer content</div></footer>
```

### Example 5: Display values and overflow

```html
<style>
  .badge {
    display: inline-block;   /* inline would ignore these paddings vertically */
    padding: 0.5rem 1rem;
    background: gold;
  }
  nav a {
    display: block;          /* whole row is clickable */
    padding: 0.75rem 1rem;
    border-bottom: 1px solid #ddd;
  }
  .clip {
    width: 200px;
    height: 100px;
    overflow: auto;          /* scrollbar appears only if needed */
    border: 1px solid #999;
  }
  .gone   { display: none; }        /* removed: no space kept */
  .hidden { visibility: hidden; }   /* invisible: space kept */
</style>
```

## Common Pitfalls

1. **Skipping the `box-sizing` reset.** Widths mysteriously overshoot, elements set to `width: 50%` don't fit side by side once padding is added. Add the universal `border-box` reset to every project, first thing.

2. **Padding vs. margin confusion.** Background/borders wrap padding, not margin. Want space *inside* the visible box → padding. Want distance *between* boxes → margin. If your background color extends further than you want, you used padding where you meant margin.

3. **Fighting default margins.** Browsers give `body`, headings, paragraphs, and lists default margins. That unexplained ~8px gap around the page is `body { margin: 8px }`. Resets like `body { margin: 0 }` and setting your own heading margins put you in control.

4. **Surprise margin collapse.**
   ```css
   /* You expect 30px + 20px = 50px gap; you get 30px. */
   .top    { margin-bottom: 30px; }
   .bottom { margin-top: 20px;   }
   ```
   Also: a child's `margin-top` moving the *parent* down instead of creating space inside — that's parent/child collapse; give the parent `padding-top`, a border, or make it a flex container.

5. **Setting fixed `height` on text containers.** Content grows (longer text, larger user font) and overflows or gets clipped. Use `min-height` or let content size the box.

6. **Trying to give a `span` width/height.** Inline elements ignore them. Switch to `inline-block` or `block` first.

7. **`width: 100%` plus padding on content-box** — the classic full-width input that overflows its container by its padding amount. `border-box` fixes it (see pitfall 1 — it always comes back to that).

8. **Removing focus outlines.** `:focus { outline: none }` makes keyboard navigation invisible. If the default ring clashes with your design, replace it (`outline: 2px solid …; outline-offset: 2px`), don't delete it.

## Practice Exercises

1. **Box-model math.** An element has `width: 400px; padding: 25px; border: 5px solid; margin: 50px`. Compute its total rendered width (a) with `content-box` and (b) with `border-box`. Then build it and verify both answers with the devtools box-model diagram.

2. **Card row.** Build three `.card` boxes styled like Example 3, displayed side by side using `display: inline-block`, each `width: 30%`. Observe the mystery gaps between them (whitespace between inline elements!) and the effect of padding with and without the border-box reset.

3. **Collapse lab.** Create two stacked `<div>`s with `margin-bottom: 40px` and `margin-top: 30px` respectively; measure the real gap in devtools. Then place a paragraph with `margin-top: 2rem` as the first child of a colored-background section and explain (in a comment) where its margin went; fix it two different ways.

4. **Centered layout shell.** Build a page with header, main, footer, each using the `.container` pattern from Example 4. The header and footer backgrounds must span the full viewport width while their *content* stays aligned with `main`. Shrink the window to confirm side padding keeps text off the edges.

5. **Overflow gallery.** Make a fixed-size box (300×150) containing a long paragraph, and show it four times — once with each `overflow` value (`visible`, `hidden`, `scroll`, `auto`) — labeled. In comments, note when you'd use each.
