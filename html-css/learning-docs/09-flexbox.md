# Chapter 9: Flexbox

## Overview

Flexbox (Flexible Box Layout) is CSS's tool for arranging items **along one axis** — a row or a column — with effortless alignment, spacing, and reordering. Navigation bars, button groups, card rows, centering anything, media objects (avatar + text), sticky footers: all one-liners with flexbox that were genuinely painful before it existed.

The mental model: a **flex container** (the parent you set `display: flex` on) controls the layout of its **flex items** (its direct children). Container properties distribute and align; item properties size and override.

## Definitions & Explanations

### Activating flexbox and the two axes

```css
.container { display: flex; }
```

Immediately, the children line up in a row. Two axes now exist:

- **Main axis** — the direction items flow. Default: horizontal (row).
- **Cross axis** — perpendicular to main. Default: vertical.

`flex-direction` sets the main axis:

```css
flex-direction: row;            /* default: left → right */
flex-direction: row-reverse;    /* right → left */
flex-direction: column;         /* top → bottom (axes swap!) */
flex-direction: column-reverse;
```

Everything else is defined *relative to these axes*, which is why "justify vs align" only clicks once you internalize: **justify-content = main axis; align-items = cross axis.** In a column, `justify-content` is vertical.

### Container properties

**`justify-content`** — distribute items along the **main** axis:

- `flex-start` (default) | `flex-end` | `center`
- `space-between` — first and last flush to edges, equal gaps between
- `space-around` — equal space around each item (half-gaps at edges)
- `space-evenly` — perfectly equal gaps everywhere

**`align-items`** — align items along the **cross** axis:

- `stretch` (default — items fill the container's cross size; why flex children in a row are all equal height!)
- `flex-start` | `flex-end` | `center`
- `baseline` — align text baselines (great for mixed-size text in a row)

**`flex-wrap`** — by default items squeeze onto one line, shrinking as needed:

```css
flex-wrap: nowrap;   /* default: one line, shrink or overflow */
flex-wrap: wrap;     /* overflowing items move to new lines */
```

**`align-content`** — when wrapping creates multiple lines, distributes the *lines* along the cross axis (same values as justify-content). No effect on single-line containers — a classic confusion.

**`gap`** — space between items (not at the edges). The modern replacement for margin hacks:

```css
gap: 1rem;          /* both directions */
gap: 1rem 2rem;     /* row-gap column-gap */
```

### Item properties

**`flex-grow`** — how much of the container's *spare* space this item absorbs (default 0 = none). Numbers are proportions: grow 2 takes twice the extra space of grow 1.

**`flex-shrink`** — how readily the item shrinks below its natural size when space is tight (default 1). `flex-shrink: 0` = "never squash me" (vital for icons/avatars).

**`flex-basis`** — the item's starting size along the main axis before growing/shrinking (default `auto` = content size). Think of it as a smarter `width` (or height, in columns).

**The `flex` shorthand** — grow, shrink, basis:

```css
flex: 0 1 auto;   /* default: don't grow, may shrink, natural size */
flex: 1;          /* = 1 1 0 — grow from zero: items share space EQUALLY */
flex: auto;       /* = 1 1 auto — grow from natural size: bigger content stays bigger */
flex: none;       /* = 0 0 auto — rigid */
flex: 0 0 200px;  /* fixed 200px, no flexing */
```

Prefer the shorthand; single-word values cover nearly every real case.

**`align-self`** — override `align-items` for one item: `align-self: flex-end;`

**`order`** — reorder visually without touching HTML (default 0, lower first). Use sparingly: screen readers and tab order still follow the HTML, so big reorders confuse keyboard users.

### The centering one-liner

```css
.parent {
  display: flex;
  justify-content: center;  /* horizontal (in a row) */
  align-items: center;      /* vertical */
}
```

Perfect centering — the problem that plagued CSS for 15 years — solved in three lines.

### When flexbox vs grid?

- **Flexbox**: one dimension (a row *or* a column), content-driven sizing, small-scale components.
- **Grid** (next chapter): two dimensions (rows *and* columns together), layout-driven, page-scale structure.
They complement each other; pages typically use grid for the macro layout and flexbox inside components.

## Code Examples

### Example 1: Navigation bar — flexbox's poster child

```html
<style>
  * { box-sizing: border-box; margin: 0; }

  .navbar {
    display: flex;
    justify-content: space-between; /* logo left, links right */
    align-items: center;            /* vertically centered regardless of heights */
    padding: 0.75rem 1.5rem;
    background: #111827;
    color: white;
  }
  .navbar .logo { font-weight: 700; font-size: 1.25rem; }

  .navbar ul {
    display: flex;                  /* nested flex container for the links */
    gap: 1.5rem;
    list-style: none;
    padding: 0;
  }
  .navbar a { color: #d1d5db; text-decoration: none; }
  .navbar a:hover { color: white; }
</style>

<nav class="navbar">
  <span class="logo">Acme</span>
  <ul>
    <li><a href="#">Products</a></li>
    <li><a href="#">Pricing</a></li>
    <li><a href="#">Docs</a></li>
    <li><a href="#">Sign in</a></li>
  </ul>
</nav>
```

### Example 2: Media object — avatar that never squashes

```html
<style>
  .comment {
    display: flex;
    gap: 1rem;
    align-items: flex-start;   /* avatar hugs the top, not stretched */
    max-width: 40rem;
    padding: 1rem;
  }
  .comment img {
    width: 48px;
    height: 48px;
    border-radius: 50%;
    flex-shrink: 0;            /* long text can never crush the avatar */
  }
  .comment .body { flex: 1; }  /* text takes all remaining width */
</style>

<article class="comment">
  <img src="https://picsum.photos/96" alt="" />
  <div class="body">
    <p><strong>Sam Rivera</strong> · 2 hours ago</p>
    <p>This flexbox pattern — fixed item plus flex:1 item — is probably the
       single most reused layout on the internet.</p>
  </div>
</article>
```

### Example 3: Equal-width card row that wraps

```html
<style>
  .cards {
    display: flex;
    flex-wrap: wrap;
    gap: 1.25rem;
  }
  .card {
    flex: 1 1 220px;   /* grow & shrink from a 220px basis:
                          wide screen → 4 across; narrow → wraps to fewer */
    padding: 1.25rem;
    background: white;
    border: 1px solid #e5e7eb;
    border-radius: 10px;
  }
</style>

<div class="cards">
  <div class="card"><h3>Fast</h3><p>Short text.</p></div>
  <div class="card"><h3>Flexible</h3><p>Somewhat longer text that wraps to multiple lines.</p></div>
  <div class="card"><h3>Fluid</h3><p>Text.</p></div>
  <div class="card"><h3>Fun</h3><p>All cards in a row share the same height automatically — that's align-items: stretch.</p></div>
</div>
```

### Example 4: Sticky footer with a flex column

```html
<style>
  html, body { height: 100%; margin: 0; }
  body {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
  }
  main { flex: 1; }   /* main absorbs all spare vertical space,
                         pushing the footer to the bottom even on short pages */
  footer { background: #111827; color: white; padding: 1rem; }
</style>

<body>
  <header>Header</header>
  <main>Short content — footer still sits at the viewport bottom.</main>
  <footer>© 2026</footer>
</body>
```

### Example 5: Proportional growth and self-alignment

```html
<style>
  .toolbar {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    height: 72px;
    background: #f3f4f6;
    padding: 0 1rem;
  }
  .search { flex: 2; }           /* search bar gets twice the spare space… */
  .status { flex: 1; }           /* …of the status area */
  .help   { align-self: flex-start; }  /* this one item pins to the top */
  .toolbar input { width: 100%; padding: 0.5rem; }
</style>

<div class="toolbar">
  <button>Menu</button>
  <div class="search"><input type="search" placeholder="Search…" /></div>
  <div class="status">3 items selected</div>
  <button class="help">?</button>
</div>
```

## Common Pitfalls

1. **Setting flex properties on the wrong element.** `justify-content` on an *item* does nothing; `flex-grow` on the *container* does nothing. Container properties: display, direction, wrap, justify-content, align-items/content, gap. Item properties: flex/grow/shrink/basis, align-self, order.

2. **Expecting `display: flex` to affect grandchildren.** Flexbox reaches only **direct children**. A wrapper `<div>` between container and the things you want to lay out swallows the flexing — either remove the wrapper or make it a flex container too.

3. **Justify/align amnesia in columns.** In `flex-direction: column`, `justify-content` is *vertical* and `align-items` horizontal. When alignment "does nothing," check the direction first — you're probably aligning the wrong axis.

4. **Fighting shrink instead of disabling it.** Images and fixed-purpose items getting crushed by long siblings → `flex-shrink: 0`, not width hacks.

5. **`width` vs `flex-basis` confusion.** In a flex row, `flex-basis` (when not `auto`) wins over `width`. Mixing both invites surprises — inside flex layouts, prefer the `flex` shorthand and drop `width`.

6. **Margin hacks instead of `gap`.** `margin-right` on every item plus a `:last-child` reset is the old way; it breaks on wrap. `gap` handles all of it.

7. **Text overflowing a `flex: 1` item.** Flex items refuse to shrink below their content's minimum size by default (`min-width: auto`). Long unbreakable strings (URLs) blow out layouts. Fix: `min-width: 0` on the item (plus `overflow-wrap: break-word` on text).

8. **Using `order` to fix source-order problems.** If the visual order you want is *always* wanted, reorder the HTML. `order` is for exceptional cases (e.g., responsive rearranging), because keyboard/screen-reader order won't follow it.

9. **`align-content` on a single line.** It only affects multi-line (wrapped) containers. For one line, you want `align-items`.

## Practice Exercises

1. **Alignment sampler.** Build one row of five small boxes and duplicate it six times, each with a different `justify-content` value (all six values), labeled. Then a 300px-tall container demonstrating all four `align-items` values on differently-sized boxes.

2. **Navbar, three states.** Recreate Example 1's navbar from scratch, then extend it: add a centered group of links *between* logo and a "Sign in" button (three groups: left, center, right — hint: three children, or clever grow values), and make links wrap gracefully on narrow windows.

3. **Pricing row.** Build three pricing cards side by side, equal width and equal height, with a "Buy" button pinned to each card's bottom regardless of feature-list length (hint: card itself is a flex column; something gets `margin-top: auto` or `flex: 1`). Cards wrap on narrow screens.

4. **Holy-grail-lite.** Using only flexbox: full-height page with header, footer, and a middle row containing a 200px sidebar plus fluid main content. Footer sticks to the bottom on short content. Sidebar never shrinks.

5. **Chat bubbles.** Build a chat log where received messages align left and sent messages align right (same HTML structure, different class), each bubble at most 70% wide, with the timestamp baseline-aligned next to the sender name. Hint: `align-self` or auto margins on items.
