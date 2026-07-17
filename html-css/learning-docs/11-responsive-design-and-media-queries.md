# Chapter 11: Responsive Design & Media Queries

## Overview

Your pages will be viewed on phones held sideways, tablets, laptops, ultrawide monitors, and screens that don't exist yet. **Responsive design** means building one page that adapts to all of them — not separate "mobile sites," not fixed 960px layouts that force pinch-zooming.

The toolkit: fluid layouts (percentages, `fr`, flex/grid wrap), flexible media, the viewport meta tag, **media queries** for the adjustments fluidity can't handle, responsive images, and modern fluid sizing (`clamp()`). You already own most of the pieces — this chapter assembles them into a methodology: **mobile-first**.

## Definitions & Explanations

### The viewport meta tag (the prerequisite)

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

Without it, mobile browsers pretend to be ~980px wide desktop screens and shrink the result — your media queries never even fire. It's been in every skeleton since Chapter 2; now you know why.

### Fluid foundations before any media query

Most responsiveness comes from *not fixing sizes* in the first place:

```css
.container { width: 100%; max-width: 70rem; margin: 0 auto; padding: 0 1rem; }
img, video { max-width: 100%; height: auto; }   /* media never overflows its container */
```

Plus everything from Chapters 9–10: `flex-wrap`, `flex: 1 1 <basis>`, `repeat(auto-fit, minmax(...))`. A layout built on those is 80% responsive before the first media query exists.

### Media queries

A media query applies CSS conditionally:

```css
/* When the viewport is at least 48em wide… */
@media (min-width: 48em) {
  .sidebar { display: block; }
}

/* Modern range syntax (all current browsers): */
@media (width >= 48em) { … }
@media (30em <= width < 64em) { … }   /* between two sizes */
```

Other useful conditions:

```css
@media (orientation: landscape) { … }
@media (hover: hover) { … }               /* device has a real hover (mouse) —
                                             gate hover-only effects behind this */
@media (prefers-reduced-motion: reduce) { … }   /* Chapter 13 */
@media (prefers-color-scheme: dark) { … }       /* user's OS dark mode */
@media print { nav, footer { display: none; } } /* print stylesheets */
```

Media queries add no specificity — rules inside them win over earlier equal-specificity rules by *source order*, which is why query blocks go **after** the base styles they adjust.

### Mobile-first methodology

Write base styles for the narrowest screens; layer enhancements at wider **breakpoints** with `min-width` queries:

```css
/* Base = mobile: single column, everything stacked */
.features { display: grid; gap: 1rem; }

/* Tablet and up */
@media (min-width: 40em) {
  .features { grid-template-columns: 1fr 1fr; }
}

/* Desktop and up */
@media (min-width: 64em) {
  .features { grid-template-columns: repeat(4, 1fr); }
}
```

Why mobile-first beats desktop-first (`max-width` queries):

1. The simple layout is the default; complexity is *added*, not unwound.
2. Small screens (often on slower devices) parse the least CSS.
3. Forgetting a breakpoint degrades gracefully to the stacked layout instead of a broken desktop squeeze.

**Choose breakpoints from your content, not devices.** Drag the window narrower until the design breaks — that's a breakpoint. Common ballparks: ~40em (640px), ~48em (768px), ~64em (1024px). Use `em` in queries; they respect user font-size settings.

### Testing responsively

Devtools → device toolbar (`Ctrl+Shift+M`): preview arbitrary sizes, device presets, and DPR. Also just drag the window edge — smooth resizing exposes breakage between breakpoints that presets hide. Always also check: 320px wide (small phones) and text zoomed to 200% (`Ctrl` + `+`).

### Fluid sizing with `min()`, `max()`, `clamp()`

```css
.container { width: min(100% - 2rem, 70rem); margin-inline: auto; }  /* container in one line */

h1 { font-size: clamp(1.75rem, 1.2rem + 2.5vw, 3rem); }
/* clamp(minimum, preferred, maximum):
   never smaller than 1.75rem, never bigger than 3rem,
   scales smoothly with the viewport in between.
   Mixing rem into the middle value keeps it responsive to user font settings. */

section { padding-block: clamp(2rem, 8vw, 6rem); }  /* fluid section spacing */
```

`clamp()` often *replaces* font-size media queries entirely.

### Responsive images

Serving one 2000px image to every phone wastes bandwidth. Two tools:

```html
<!-- Resolution switching: same picture, multiple sizes; browser picks -->
<img
  src="photo-800.jpg"
  srcset="photo-400.jpg 400w, photo-800.jpg 800w, photo-1600.jpg 1600w"
  sizes="(min-width: 64em) 33vw, (min-width: 40em) 50vw, 100vw"
  alt="Hiker on a ridge at sunrise"
/>
<!-- sizes tells the browser how wide the image will DISPLAY at each breakpoint,
     so it can pick the smallest sufficient file BEFORE layout happens. -->

<!-- Art direction: different crops per breakpoint -->
<picture>
  <source media="(min-width: 48em)" srcset="hero-wide.jpg" />
  <source media="(min-width: 30em)" srcset="hero-square.jpg" />
  <img src="hero-tall.jpg" alt="Chef plating a dish" />
</picture>
```

And in CSS, `object-fit` controls how an `<img>` fills a sized box:

```css
.card img { width: 100%; height: 200px; object-fit: cover; }  /* crop, don't squish */
```

### Responsive patterns cheat sheet

- **Stack → columns**: grid with 1 column base, more at breakpoints (or `auto-fit`/`minmax`, zero queries).
- **Nav collapse**: full menu at wide sizes; at narrow sizes a toggle button (the toggle needs JavaScript; the CSS side is `display` swaps in a query).
- **Table rescue**: wide tables get `overflow-x: auto` on a wrapper so *they* scroll, not the page.
- **Hide with care**: `display: none` at some sizes is fine for redundant decoration; hiding *content* mobile users still need is a design failure.

## Code Examples

### Example 1: Complete mobile-first page

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Responsive demo</title>
<style>
  * { box-sizing: border-box; margin: 0; }
  body { font-family: system-ui, sans-serif; line-height: 1.6; }
  img { max-width: 100%; height: auto; display: block; }

  .container { width: min(100% - 2rem, 70rem); margin-inline: auto; }

  /* ---------- Base (mobile): everything stacks ---------- */
  .site-header { background: #0f172a; color: white; padding: 1rem 0; }
  .site-header .container { display: flex; flex-wrap: wrap; gap: 0.75rem;
                            justify-content: space-between; align-items: center; }
  .site-header ul { display: flex; flex-wrap: wrap; gap: 1rem; list-style: none; padding: 0; }
  .site-header a { color: #cbd5e1; text-decoration: none; }

  .hero { padding: clamp(2rem, 8vw, 6rem) 0; background: #eef2ff; }
  .hero h1 { font-size: clamp(1.9rem, 1.2rem + 3vw, 3.25rem); line-height: 1.15; }

  .columns { display: grid; gap: 1.5rem; padding: 2rem 0; }

  /* ---------- Tablet ---------- */
  @media (min-width: 40em) {
    .columns { grid-template-columns: 1fr 1fr; }
  }

  /* ---------- Desktop ---------- */
  @media (min-width: 64em) {
    .columns { grid-template-columns: repeat(3, 1fr); }
    .hero .container { display: grid; grid-template-columns: 3fr 2fr; gap: 2rem; align-items: center; }
  }
</style>
</head>
<body>
  <header class="site-header">
    <div class="container">
      <strong>Trailhead Co.</strong>
      <ul><li><a href="#">Tours</a></li><li><a href="#">Gear</a></li><li><a href="#">Contact</a></li></ul>
    </div>
  </header>

  <section class="hero">
    <div class="container">
      <div>
        <h1>Hike further, carry less</h1>
        <p>Fluid type via clamp(), fluid padding, and a hero that becomes two columns only when there's room.</p>
      </div>
      <img src="https://picsum.photos/800/600" alt="Backpacker crossing an alpine meadow" />
    </div>
  </section>

  <div class="container columns">
    <article><h2>Plan</h2><p>1 column on phones, 2 on tablets, 3 on desktops.</p></article>
    <article><h2>Pack</h2><p>Resize the window and watch each breakpoint engage.</p></article>
    <article><h2>Go</h2><p>The base styles never had to be undone — that's mobile-first.</p></article>
  </div>
</body>
</html>
```

### Example 2: Content-driven breakpoint discovery

```css
/* Start with no queries: */
.team { display: flex; flex-wrap: wrap; gap: 1rem; }
.team .member { flex: 1 1 16rem; }

/* Drag the window: cards wrap by themselves. Only when you find something
   that actually breaks (e.g. the bio text gets too cramped under 22rem)
   do you add a query — at THAT width: */
@media (max-width: 22em) {
  .team .member { flex-basis: 100%; }
}
```

### Example 3: Dark mode and reduced data respect

```css
:root {
  --paper: #ffffff;
  --ink: #1f2937;
}
@media (prefers-color-scheme: dark) {
  :root {
    --paper: #0f172a;
    --ink: #e5e7eb;
  }
}
body { background: var(--paper); color: var(--ink); }
```

### Example 4: Responsive table and nav patterns

```html
<style>
  /* Table: scrolls inside its wrapper instead of stretching the page */
  .table-wrap { overflow-x: auto; }
  .table-wrap table { border-collapse: collapse; min-width: 40rem; }

  /* Nav: hamburger button only exists on small screens */
  .menu-toggle { display: block; }
  .site-nav ul { display: none; }         /* hidden until toggled by JS… */
  .site-nav ul.open { display: flex; flex-direction: column; }

  @media (min-width: 48em) {
    .menu-toggle { display: none; }        /* desktop: no button… */
    .site-nav ul { display: flex; gap: 1.5rem; }  /* …menu always visible */
  }
</style>

<div class="table-wrap">
  <table><!-- wide data table --></table>
</div>
```

### Example 5: srcset in a card grid

```html
<style>
  .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr)); gap: 1rem; }
  .grid img { width: 100%; aspect-ratio: 3 / 2; object-fit: cover; }
</style>

<div class="grid">
  <article>
    <img
      src="trip-640.jpg"
      srcset="trip-320.jpg 320w, trip-640.jpg 640w, trip-1280.jpg 1280w"
      sizes="(min-width: 64em) 25vw, (min-width: 40em) 50vw, 100vw"
      alt="Kayaks lined up on a pebble beach"
      loading="lazy"
    />
    <h3>Coastal paddle</h3>
  </article>
  <!-- more cards -->
</div>
```

## Common Pitfalls

1. **Missing viewport meta.** The #1 cause of "my media queries don't work on my phone." Everything renders tiny; queries written for 400px widths never match because the browser claims ~980px.

2. **Desktop-first then cramming.** Building the wide layout first and bolting on `max-width` overrides produces twice the CSS, half of it un-doing the other half. Start narrow.

3. **Device-name breakpoints.** "iPhone breakpoint," "iPad breakpoint" — devices change yearly, and your content doesn't care. Break where *your design* breaks.

4. **Fixed widths smuggled back in.** One `width: 600px` on a wrapper undoes everything downstream. Search your CSS for `width:` followed by px and justify each one (usually the fix is `max-width`).

5. **Overflow from forgotten elements.** A 100vw section, an unwrapped table, a long URL, or an image without `max-width: 100%` causes phantom horizontal scrolling on mobile. Devtools tip: `* { outline: 1px solid red }` temporarily, or scroll-check at 320px width.

6. **Hover-dependent functionality.** Touch screens have no hover — menus that only open on `:hover` are unusable there. Gate hover embellishments behind `@media (hover: hover)` and give touch users click/tap paths.

7. **`vw`-only font sizes.** `font-size: 4vw` becomes microscopic on phones, gigantic on ultrawides, and — worse — ignores browser zoom. Always `clamp()` with rem bounds and a rem term in the middle.

8. **Queries before base styles.** Because queries don't add specificity, a query block written *above* the base rule it should override loses on source order. Query blocks come after their base styles.

9. **Testing only at presets.** The layout works at exactly 375px and 768px and shatters at 500px. Drag-resize through the whole range.

## Practice Exercises

1. **Breakpoint autopsy.** Take any fixed-width page you've built (or build a deliberately rigid 3-column 960px one). Convert it mobile-first: strip fixed widths, stack everything, then reintroduce columns at content-chosen breakpoints. Document (in CSS comments) why you chose each breakpoint value.

2. **Zero-query challenge.** Build a page with header, card grid, and footer that adapts respectably from 320px to 1440px using *no media queries at all* — only fluid techniques (`min()`, `clamp()`, `auto-fit`/`minmax`, `flex-wrap`). Then add at most two queries for refinements fluidity couldn't achieve, and note what they were.

3. **Fluid type scale.** Create a `clamp()`-based scale for h1–h4 and body text that you can verify: at 320px each heading equals its minimum, at 1280px its maximum. Show the same article at three window widths; text zoom to 200% must not break anything.

4. **The gauntlet.** Build a page containing all four classic overflow hazards: a data table, a long unbroken URL in text, a full-bleed image, and a code block. Make the page pass a 320px-width test with zero horizontal page scrolling — each hazard handled with the appropriate pattern.

5. **Preference queries.** Extend any previous exercise with: automatic dark mode via `prefers-color-scheme` (custom-property swap), a print stylesheet hiding nav/footer and forcing dark-on-white text, and hover effects that only apply under `(hover: hover)`. Test dark mode via devtools' rendering emulation.
