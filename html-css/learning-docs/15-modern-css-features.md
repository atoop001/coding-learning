# Chapter 15: Modern CSS — Custom Properties, Nesting, Container Queries & Friends

## Overview

CSS has evolved faster in the last few years than in the previous fifteen. Features that once required preprocessors (Sass) or JavaScript are now native: **custom properties** (variables), **nesting**, **container queries**, powerful selectors like `:has()` and `:is()`, `aspect-ratio`, logical properties, and native `<dialog>`/popover behaviors. This closing chapter tours the modern features you'll actually meet in codebases and job postings, with an emphasis on the two that change how you architect styles: custom properties and container queries.

A habit to carry forward: before relying on any newer feature, check support at **caniuse.com**. Everything in this chapter is supported in all current major browsers (as of this track's writing) unless flagged.

## Definitions & Explanations

### Custom properties (CSS variables)

Declare with a `--` prefix; read with `var()`:

```css
:root {                       /* :root = <html>, so these are global */
  --brand: hsl(255 60% 55%);
  --space: 1rem;
  --radius: 10px;
}
.card {
  border-radius: var(--radius);
  padding: var(--space);
  border: 2px solid var(--brand);
}
```

What makes them far more than find-and-replace constants:

1. **They cascade and inherit like normal properties.** Redefine a variable on any element and everything inside sees the new value:

   ```css
   .dark-section { --ink: white; --paper: #111; }
   /* every rule using var(--ink)/var(--paper) inside flips automatically */
   ```

2. **Fallbacks**: `var(--accent, tomato)` — used if `--accent` isn't defined.

3. **Runtime-live**: JavaScript and media queries can change them on the fly — this is how one-line theme switching works (Chapter 11's dark-mode example).

4. **Component APIs**: a component reads `var(--button-bg, var(--brand))`, letting any context re-skin it without new selectors.

Idiomatic uses: design tokens (colors, spacing scale, radii, shadows, font stacks), theming, and reducing repetition in `calc()`-heavy code: `width: calc(var(--cols) * 4rem);`

### `calc()` and math functions

```css
width: calc(100% - 4rem);          /* mix units — the killer feature */
font-size: calc(1rem + 0.5vw);
--gutter: calc(var(--space) * 2);  /* compute with variables */
```

You already met `min()`, `max()`, `clamp()` (Chapter 11); together with `calc()` they replace a whole category of media queries and JS measurements.

### Native CSS nesting

Write rules inside rules; `&` refers to the parent selector:

```css
.card {
  padding: 1rem;
  border: 1px solid #e5e7eb;

  & h3 { margin-top: 0; }            /* .card h3 */

  &:hover { border-color: var(--brand); }   /* .card:hover */

  &.featured { background: #fef9c3; }        /* .card.featured */

  .dark & { border-color: #374151; }         /* .dark .card — parent context! */

  @media (min-width: 48em) {                 /* media queries nest too */
    padding: 2rem;
  }
}
```

Benefits: co-located component styles, hover/focus states next to their base. Danger: deep nesting recreates the specificity/fragility problems of old descendant-selector soup. Keep nesting shallow — states, direct children, and media queries; not four levels of structure.

### Container queries

Media queries ask "how wide is the *viewport*?" — but a card in a sidebar needs to adapt to its *slot*, not the screen. **Container queries** ask "how wide is my *container*?":

```css
/* 1. Declare an element AS a container */
.card-slot {
  container-type: inline-size;      /* observe its width */
  container-name: card-slot;        /* optional name */
}

/* 2. Style descendants based on the container's size */
@container card-slot (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 150px 1fr;   /* horizontal layout when the SLOT is wide */
  }
}
```

The same card component is now correct in a wide main column *and* a narrow sidebar simultaneously — impossible with media queries. This is the biggest layout-thinking shift since grid: **components respond to their container; only page scaffolding responds to the viewport.**

Container units: `cqw`/`cqh`/`cqi`/`cqb` (1% of container width/height/inline/block size) — e.g. `font-size: clamp(1rem, 4cqi, 1.5rem)` for text that scales with its card.

### Modern selectors

```css
/* :is() — grouping without repetition; takes the specificity of its most specific argument */
:is(h1, h2, h3) a { color: inherit; }

/* :where() — same, but ZERO specificity: perfect for defaults meant to be overridden */
:where(ul, ol) { padding-inline-start: 1.25rem; }

/* :has() — "parent selector": style an element by what's INSIDE it */
.card:has(img) { padding-top: 0; }               /* cards containing an image */
form:has(:user-invalid) .submit { opacity: 0.5; } /* form state styling */
label:has(input:checked) { background: #dbeafe; } /* selected option highlight */

/* :nth-child(... of S) — count only matching siblings */
li:nth-child(odd of .visible) { background: #f8fafc; }
```

`:has()` unlocks patterns that previously required JavaScript — style ancestors, previous siblings (`.a:has(+ .b)`), and form-wide states in pure CSS.

### Logical properties

Physical properties (`margin-left`, `top`) assume left-to-right, top-to-bottom text. Logical properties adapt to writing direction — and are increasingly the house style in professional codebases:

| Physical | Logical |
|---|---|
| `margin-left` / `margin-right` | `margin-inline-start` / `-end` |
| `margin-top` / `margin-bottom` | `margin-block-start` / `-end` |
| `padding: 0 1rem` | `padding-inline: 1rem` |
| `width` / `height` | `inline-size` / `block-size` |
| `text-align: left` | `text-align: start` |

`margin-inline: auto` is the modern centering idiom; `padding-block` / `padding-inline` are handy shorthands even if you never ship RTL.

### Small features with outsized value

```css
/* aspect-ratio — no more padding-top hacks */
.video-thumb { aspect-ratio: 16 / 9; object-fit: cover; }
.avatar     { aspect-ratio: 1; border-radius: 50%; }

/* gap in flexbox (not just grid) — you've been using it; it was new once */

/* accent-color — theme native checkboxes/radios/progress in one line */
:root { accent-color: var(--brand); }

/* color-mix() — derive tints/shades from one token */
.button:hover { background: color-mix(in oklab, var(--brand), black 15%); }

/* scroll-behavior + scroll-margin — polished in-page anchors */
html { scroll-behavior: smooth; }
h2[id] { scroll-margin-top: 5rem; }   /* don't hide under the sticky header */

/* text-wrap — typographic niceties */
h1 { text-wrap: balance; }     /* even line lengths in headings */
p  { text-wrap: pretty; }      /* avoid single-word last lines (support growing) */
```

And two HTML-side moderns worth knowing: **`<dialog>`** (native modal with focus trapping — `dialog.showModal()`) and the **popover attribute** (`popover` + `popovertarget` for menus/tooltips with zero JS) — both replace fragile hand-rolled overlay code from Chapter 12.

## Code Examples

### Example 1: A tokened, themeable design system core

```css
:root {
  /* Design tokens */
  --brand-h: 222;
  --brand: hsl(var(--brand-h) 75% 50%);
  --brand-soft: hsl(var(--brand-h) 75% 94%);
  --ink: hsl(var(--brand-h) 20% 12%);
  --paper: hsl(0 0% 100%);
  --surface: hsl(var(--brand-h) 25% 97%);

  --space-1: 0.5rem;  --space-2: 1rem;  --space-3: 1.5rem;  --space-4: 2.5rem;
  --radius: 10px;
  --shadow-1: 0 1px 3px rgb(0 0 0 / 0.12);
  --shadow-2: 0 8px 24px rgb(0 0 0 / 0.14);
}

/* Dark theme: swap tokens, not components */
@media (prefers-color-scheme: dark) {
  :root {
    --ink: hsl(var(--brand-h) 15% 90%);
    --paper: hsl(var(--brand-h) 20% 10%);
    --surface: hsl(var(--brand-h) 20% 14%);
    --brand-soft: hsl(var(--brand-h) 40% 22%);
  }
}

body   { color: var(--ink); background: var(--paper); }
.card  { background: var(--surface); border-radius: var(--radius);
         padding: var(--space-3); box-shadow: var(--shadow-1); }
.badge { background: var(--brand-soft); color: var(--brand);
         padding: 0.2em 0.7em; border-radius: 999px; }
```

### Example 2: A nested component, idiomatically shallow

```css
.button {
  --_bg: var(--button-bg, var(--brand));   /* private var with public override hook */
  display: inline-flex;
  align-items: center;
  gap: 0.5em;
  padding: 0.6em 1.2em;
  background: var(--_bg);
  color: white;
  border: none;
  border-radius: var(--radius);
  cursor: pointer;
  transition: background-color 0.15s ease-out, transform 0.15s ease-out;

  &:hover  { background: color-mix(in oklab, var(--_bg), black 12%); }
  &:active { transform: scale(0.97); }
  &:focus-visible { outline: 3px solid color-mix(in oklab, var(--_bg), white 40%);
                    outline-offset: 2px; }

  &.secondary { --_bg: transparent; color: var(--brand);
                box-shadow: inset 0 0 0 2px var(--brand); }
}

/* Any context can re-skin without touching the component: */
.danger-zone { --button-bg: hsl(0 72% 45%); }
```

### Example 3: Container-query card — one component, every slot

```html
<style>
  .slot { container-type: inline-size; }

  .profile-card {
    display: grid;
    gap: 1rem;
    padding: 1rem;
    border: 1px solid #e5e7eb;
    border-radius: 12px;
  }
  .profile-card img { width: 100%; aspect-ratio: 1; object-fit: cover; border-radius: 8px; }

  /* When the SLOT is at least 380px, go horizontal */
  @container (min-width: 380px) {
    .profile-card { grid-template-columns: 120px 1fr; align-items: center; }
    .profile-card img { width: 120px; }
  }
  @container (min-width: 600px) {
    .profile-card h3 { font-size: clamp(1.2rem, 4cqi, 1.8rem); }
  }

  /* Page: wide main + narrow sidebar, SAME component in both */
  .page { display: grid; grid-template-columns: 2fr 1fr; gap: 1.5rem; }
</style>

<div class="page">
  <div class="slot">
    <article class="profile-card">  <!-- renders horizontal here -->
      <img src="https://picsum.photos/240" alt="" />
      <div><h3>Ada Deng</h3><p>Staff engineer. Writes about layout.</p></div>
    </article>
  </div>
  <div class="slot">
    <article class="profile-card">  <!-- identical markup renders stacked here -->
      <img src="https://picsum.photos/241" alt="" />
      <div><h3>Ada Deng</h3><p>Staff engineer. Writes about layout.</p></div>
    </article>
  </div>
</div>
```

### Example 4: `:has()` doing what used to need JavaScript

```css
/* Form: submit button reflects form validity */
form:has(:user-invalid) button[type="submit"] { opacity: 0.5; cursor: not-allowed; }

/* Card layout adapts to its own content */
.card:has(figure) { padding: 0; }
.card:has(figure) .card-text { padding: 1rem; }

/* Selected-state styling for custom-looking radio cards */
.plan:has(input:checked) {
  border-color: var(--brand);
  background: var(--brand-soft);
}

/* Page-level: no-JS "lightbox open" state */
body:has(#lightbox-toggle:checked) { overflow: hidden; }
```

### Example 5: Native dialog + popover (goodbye, hand-rolled overlays)

```html
<!-- Modal: focus trapping, ESC-to-close, ::backdrop — all built in -->
<button command="show-modal" commandfor="confirm">Delete…</button>
<dialog id="confirm">
  <h2>Delete file?</h2>
  <p>This cannot be undone.</p>
  <form method="dialog">          <!-- buttons inside close the dialog -->
    <button value="cancel">Cancel</button>
    <button value="ok">Delete</button>
  </form>
</dialog>
<style>
  dialog { border: none; border-radius: 12px; padding: 2rem;
           box-shadow: var(--shadow-2); }
  dialog::backdrop { background: rgb(0 0 0 / 0.5); }
</style>
<!-- (If the command attribute isn't supported yet in your target browsers,
      one line of JS does it: document.querySelector('#confirm').showModal()) -->

<!-- Popover: zero-JS dropdown -->
<button popovertarget="menu">Options ▾</button>
<div id="menu" popover>
  <a href="#">Rename</a>
  <a href="#">Duplicate</a>
</div>
```

## Common Pitfalls

1. **Variable name typos fail silently.** `var(--barnd)` just resolves to nothing (or the fallback) — no error anywhere. If a token "isn't applying," check devtools' Computed pane and spelling first; add fallbacks to critical vars.

2. **Defining tokens somewhere that doesn't reach.** A `--brand` declared on `.header` is invisible to `.footer` — inheritance flows *down* only. Global tokens live on `:root`; scoped overrides go on the subtree that wants them.

3. **Nesting like it's 2012 Sass.** Five-level nested structure compiles into brittle, over-specific selectors that are painful to override. Nest states, pseudo-elements, direct-child tweaks, and media/container queries; give real sub-components their own class and top-level block.

4. **Container queries without a container.** `@container` rules silently never match if no ancestor has `container-type`. Also: an element can't query *itself* — the queried sizes belong to an ancestor container, so wrap components in a slot element.

5. **`container-type: size` collapsing heights.** Use `inline-size` (width-only) almost always; `size` requires the container to have an explicit height or it shrinks to nothing.

6. **Forgetting `:where()` when writing resets.** A reset written with plain selectors (specificity 0-0-1+) can beat later component classes in surprising ways; `:where()` makes defaults truly zero-specificity and painless to override.

7. **`:has()` performance abuse.** `body:has(*:hover)`-style selectors force the browser to re-evaluate constantly. Keep `:has()` arguments narrow and anchored (`form:has(:user-invalid)`, not `div:has(div div)`).

8. **Using cutting-edge features without checking support.** `text-wrap: pretty`, the `command` attribute, and friends are newer than the rest. Progressive enhancement: fine to use when the fallback behavior (plain wrapping, a JS shim) is acceptable — but *know* what the fallback is. caniuse.com, always.

9. **Cargo-culting tokens.** Forty variables used once each is noise, not a system. Tokenize what *repeats* or what *themes*; hard-code the genuinely one-off.

## Practice Exercises

1. **Tokenize a past project.** Take your Chapter 8 or Project 3 stylesheet and extract every repeated color, spacing value, radius, and shadow into `:root` custom properties. Then add a complete dark theme by overriding only tokens inside `@media (prefers-color-scheme: dark)` — zero component rules may change.

2. **Nesting conversion.** Rewrite one flat component block (your button or card styles) using native nesting: base, `&:hover`, `&:focus-visible`, one modifier class, and a nested media query. Keep maximum nesting depth ≤ 2 and confirm identical rendering.

3. **Container-query card.** Build a "product card" (image, title, price, button) that: stacks vertically under 350px of *container* width, goes horizontal above it, and enlarges its title with `cqi` units above 550px. Prove it by placing three instances in slots of 300px, 450px, and 700px on one page — same markup, three layouts, no media queries.

4. **`:has()` katas.** Using only CSS: (a) a form whose submit button visually disables while any field is `:user-invalid`; (b) a gallery where a figure containing a `figcaption` gets extra bottom padding and ones without don't; (c) a pricing table where the column containing a checked radio gets highlighted. No JavaScript allowed.

5. **Modernize an overlay.** Replace your Chapter 12 modal mock-up with a real `<dialog>` (styled, with `::backdrop`) and your dropdown with the `popover` attribute. Compare the code size and keyboard behavior against the hand-rolled versions, and note (in comments) what the native versions gave you for free.
