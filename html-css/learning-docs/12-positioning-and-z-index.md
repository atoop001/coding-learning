# Chapter 12: Positioning & Z-Index

## Overview

Flexbox and grid handle elements *in the flow* — each taking space, pushing neighbors. But some UI lives *outside* normal flow: sticky headers, dropdown menus, modals, badges pinned to a card's corner, cookie banners, tooltips. That's the **positioning** system: `position` with its five values, the inset properties (`top`/`right`/`bottom`/`left`), and `z-index` for stacking order.

Positioning is powerful and precise — and the most misused tool in beginner CSS. The golden rule up front: **flexbox/grid for layout; positioning for overlays and pinning.** If you're positioning paragraphs to build a page column, back up a chapter.

## Definitions & Explanations

### `position: static` (the default)

Normal document flow. `top`/`left`/etc. and `z-index` do nothing. Every element starts here.

### `position: relative`

The element stays exactly where it was in flow (its space is preserved), but:

1. You can nudge its *rendering* with `top/right/bottom/left` (rarely needed).
2. **Far more importantly: it becomes a positioning anchor** — a "containing block" — for absolutely-positioned descendants.

```css
.card { position: relative; }   /* usually added ONLY to serve as an anchor */
```

### `position: absolute`

The element is **removed from normal flow** (takes no space; siblings act as if it's gone) and positioned relative to its **nearest positioned ancestor** (any ancestor whose position isn't static). No positioned ancestor? It anchors to the document.

```css
.badge {
  position: absolute;
  top: 8px;
  right: 8px;
}
```

The essential pairing — memorize it as one unit:

```css
.parent { position: relative; }  /* anchor */
.child  { position: absolute; top: 0; right: 0; }  /* pinned to parent's corner */
```

Inset property notes:

- Offsets measure from the corresponding edge of the containing block *inward*.
- Setting opposite edges stretches the element: `top: 0; bottom: 0;` = full height of the anchor. `inset: 0;` (shorthand for all four = 0) = cover the anchor completely — the standard overlay recipe.
- Absolutely-positioned elements shrink-wrap their content unless sized or stretched.

### `position: fixed`

Removed from flow and pinned to the **viewport** — it stays put while the page scrolls. Cookie banners, floating action buttons, old-school fixed headers.

```css
.cookie-banner {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
}
```

Caveat: an ancestor with a `transform` (or `filter`, `will-change`) hijacks `fixed` to be relative to *that ancestor* instead of the viewport — a notorious "why is my fixed element not fixed" bug.

### `position: sticky`

A hybrid: the element scrolls normally **until** it reaches the offset you set, then sticks there — but only *within its parent*; it un-sticks when the parent's end scrolls past. Section headers in long lists, in-page nav, table headers.

```css
.section-header {
  position: sticky;
  top: 0;          /* REQUIRED: sticky without an offset does nothing */
}
```

Sticky gotchas: it needs a scrollable run of parent below it (a parent no taller than the sticky element gives it nowhere to stick), and any ancestor with `overflow: hidden`/`auto` can silently disable it.

### `z-index` and stacking

When elements overlap, who's on top? By default: later-in-source paints above earlier. `z-index` overrides this — **but only on positioned elements** (or flex/grid children):

```css
.dropdown { position: absolute; z-index: 10; }
```

**Stacking contexts** — the concept that explains every "z-index: 999999 isn't working" mystery. Certain properties create a *stacking context*: `position` + `z-index`, `opacity < 1`, `transform`, `filter`, `position: fixed`, and others. Within a context, children stack among themselves; **the whole context stacks as one unit against outsiders.** A child with `z-index: 9999` inside a context that itself sits at `z-index: 1` can never rise above an outside element at `z-index: 2`.

Debugging recipe: when z-index seems ignored, walk *up* the ancestors of both elements and find which two ancestors actually overlap-compete; fix the z-index at *that* level (or remove the accidental context creator, often a stray `transform` or `opacity`).

Sanity practice: define a small scale (1 = raised card, 10 = dropdown, 100 = fixed header, 1000 = modal, 1010 = toast) instead of escalating 99999s.

### Layout-adjacent properties

- **`overflow: hidden`** on the anchor clips absolutely-positioned children that stick out (sometimes wanted, often the reason your badge vanished).
- **`inset`** shorthand: `inset: 0;` = `top:0; right:0; bottom:0; left:0;`
- **Centering an absolute element**: `inset: 0; margin: auto;` (with a width/height), or the classic `top: 50%; left: 50%; transform: translate(-50%, -50%);` (transform preview — Chapter 13).

## Code Examples

### Example 1: Badge pinned to a card corner

```html
<style>
  .product {
    position: relative;       /* the anchor */
    width: 240px;
    border: 1px solid #e5e7eb;
    border-radius: 10px;
    padding: 1rem;
  }
  .sale-badge {
    position: absolute;
    top: -10px;               /* negative offsets poke outside the anchor */
    right: -10px;
    background: crimson;
    color: white;
    padding: 0.25rem 0.6rem;
    border-radius: 999px;
    font-size: 0.8rem;
    font-weight: 700;
  }
</style>

<article class="product">
  <span class="sale-badge">-30%</span>
  <h3>Trail Boots</h3>
  <p>$89.00</p>
</article>
```

### Example 2: Image with a caption overlay

```html
<style>
  .figure-overlay {
    position: relative;
    max-width: 480px;
  }
  .figure-overlay img { width: 100%; display: block; border-radius: 8px; }
  .figure-overlay figcaption {
    position: absolute;
    left: 0; right: 0; bottom: 0;        /* stretch across the bottom */
    padding: 1.5rem 1rem 0.75rem;
    color: white;
    /* gradient scrim keeps text readable over any photo */
    background: linear-gradient(transparent, rgb(0 0 0 / 0.75));
    border-radius: 0 0 8px 8px;
  }
</style>

<figure class="figure-overlay">
  <img src="https://picsum.photos/960/640" alt="Fog rolling over forested hills" />
  <figcaption>Morning fog, Cascade foothills</figcaption>
</figure>
```

### Example 3: Sticky header + sticky section labels

```html
<style>
  * { margin: 0; box-sizing: border-box; }
  .site-header {
    position: sticky;
    top: 0;
    z-index: 100;                    /* stay above page content while stuck */
    background: white;
    border-bottom: 1px solid #e5e7eb;
    padding: 0.75rem 1rem;
  }
  .letter {
    position: sticky;
    top: 3.2rem;                     /* sticks BELOW the sticky header */
    background: #f3f4f6;
    padding: 0.25rem 1rem;
    font-weight: 700;
    z-index: 50;
  }
  .contact { padding: 1rem; border-bottom: 1px solid #f3f4f6; }
</style>

<header class="site-header">Contacts</header>

<section>
  <h2 class="letter">A</h2>
  <div class="contact">Alice Anders</div>
  <div class="contact">Ana Alves</div>
  <!-- more … -->
</section>
<section>
  <h2 class="letter">B</h2>
  <div class="contact">Ben Brook</div>
  <!-- Scroll: each letter sticks under the header, then is pushed away
       by the next section's letter — classic sticky behavior. -->
</section>
```

### Example 4: Modal dialog overlay (CSS side)

```html
<style>
  .modal-backdrop {
    position: fixed;
    inset: 0;                              /* cover the whole viewport */
    background: rgb(0 0 0 / 0.5);
    z-index: 1000;
    display: grid;
    place-items: center;                   /* flex/grid still work INSIDE positioned elements */
  }
  .modal {
    background: white;
    border-radius: 12px;
    padding: 2rem;
    width: min(90%, 28rem);
    box-shadow: 0 20px 50px rgb(0 0 0 / 0.3);
  }
</style>

<div class="modal-backdrop">
  <div class="modal" role="dialog" aria-modal="true" aria-labelledby="mtitle">
    <h2 id="mtitle">Delete file?</h2>
    <p>This cannot be undone.</p>
    <button>Cancel</button> <button>Delete</button>
  </div>
</div>
<!-- Showing/hiding needs JavaScript (or the native <dialog> element —
     worth reading about); the positioning pattern is pure CSS. -->
```

### Example 5: Stacking-context trap, reproduced

```html
<style>
  .trap {
    opacity: 0.99;            /* innocent-looking; creates a stacking context! */
    position: relative;
    z-index: 1;
  }
  .trap .tooltip {
    position: absolute;
    z-index: 99999;           /* huge… but trapped inside .trap's context */
    top: 100%; left: 0;
    background: #111; color: white; padding: 0.5rem;
  }
  .rival {
    position: relative;
    z-index: 2;               /* beats .trap's ENTIRE context, tooltip included */
    background: gold; padding: 2rem;
    margin-top: -1rem;
  }
</style>

<div class="trap">
  Hover target
  <div class="tooltip">z-index 99999, still underneath!</div>
</div>
<div class="rival">z-index 2 wins — remove .trap's opacity line and watch it flip.</div>
```

## Common Pitfalls

1. **Building layouts with absolute positioning.** Pixel-pinning every box "works" at your window size and shatters at any other — plus removed-from-flow elements can't push the footer down, so content overlaps. Layout = flexbox/grid; positioning = overlays, badges, pinning.

2. **Forgetting the `relative` anchor.** Your absolute element flies to the top-left of the *page* instead of its card. Nearest positioned ancestor — add `position: relative` to the intended parent. This one mistake explains most beginner positioning chaos.

3. **`z-index` on a static element.** Silently ignored. The element needs `position` (any non-static) — or to be a flex/grid child.

4. **The stacking-context ambush.** `z-index: 999999` loses because an ancestor with `transform`, `opacity`, or `filter` created a context that caps it (Example 5). Fix at the ancestor level; don't keep adding nines.

5. **Sticky that never sticks.** Missing `top` offset; a parent with no scroll run left; or an ancestor with `overflow: hidden/auto` eating the scroll. Check all three in that order.

6. **Fixed elements covering content.** A fixed header overlaps the top of the page; anchor links scroll targets underneath it. Compensate: `body { padding-top: <header height>; }` (fixed) or prefer `sticky` (which keeps its space), and `html { scroll-padding-top: <height>; }` for anchor jumps.

7. **Fixed inside transformed ancestors.** A `transform` anywhere up the tree makes `fixed` behave like `absolute` in that subtree. Portals/modals belong near `<body>` in the markup partly for this reason.

8. **Content overlap after removing elements from flow.** Absolutely positioning something means siblings no longer make room; text flows under your element. If neighbors should make room, the element belongs in flow (padding/margin/flex/grid), not positioned.

9. **Overlay without a scrim blocking interaction visibility.** A modal without a backdrop leaves the page visually "live" and confusing. `position: fixed; inset: 0; background: rgb(0 0 0 / .5)` communicates modality (and gives a click-to-close target).

## Practice Exercises

1. **Anchor drill.** Build a 3×2 grid of cards; give each card one absolutely-positioned decoration in a different spot (top-left badge, bottom-right icon, full-bleed corner ribbon, centered stamp using `inset: 0; margin: auto`, a bar stretched along the top via `left:0; right:0`, and one *deliberately* missing its relative anchor). Explain in a comment where the sixth one went and why, then fix it.

2. **Sticky notes app UI.** Create a long page with a sticky site header, sticky sub-navigation *below* the header (correct cumulative offset), and a right sidebar whose "Related" box is sticky within the sidebar only. Verify each sticks and releases at the right scroll positions.

3. **Overlay gallery.** Make an image grid where every image has (a) a gradient-scrim caption overlay pinned to its bottom and (b) a hover-revealed full-cover overlay (`inset: 0`) with a centered button. No layout shifting on hover.

4. **The modal stack.** Build (statically visible, no JS needed) a page containing: a sticky header (z 100), an open dropdown menu under a nav item (absolute, z 10), a fixed cookie banner (z 500), and a modal with backdrop (z 1000) — all layered correctly per that scale. Then sabotage it: add `transform: translateZ(0)` to the header and document in comments everything that breaks and why.

5. **Stacking-context forensics.** Recreate Example 5 from memory, then produce three different one-line fixes that put the tooltip on top (removing the context creator; raising the trapped ancestor; restructuring the HTML). Comment the trade-offs of each.
