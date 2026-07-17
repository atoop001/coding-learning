# Project 5: Responsive Product Landing Page

## Description

Build the marketing landing page for an invented product or service (an app, a coffee subscription, a course — your pick), engineered mobile-first and genuinely responsive from 320px phones to wide desktops. The classic landing-page anatomy: nav, big hero with headline + call-to-action, a features grid, a "how it works" section, testimonials, a pricing table, and a footer with link columns.

Grid drives the page-level and section-level layout; flexbox handles the components inside; media queries and fluid techniques (`clamp()`, `auto-fit`/`minmax`) make it adapt. This is the portfolio piece format employers most expect to see — make it one you'd actually show.

## Difficulty & Effort

- **Difficulty:** Intermediate
- **Estimated effort:** 8–12 hours

## Chapters Used

- `08-colors-typography-backgrounds.md`
- `09-flexbox.md`
- `10-css-grid.md`
- `11-responsive-design-and-media-queries.md`

## Requirements Checklist

### Foundations
- [ ] Viewport meta present; `box-sizing` reset; external stylesheet organized in labeled sections
- [ ] **Mobile-first**: base styles are the single-column phone layout; all media queries are `min-width`, placed after their base rules
- [ ] Breakpoints chosen from where *your* content breaks (comment each query with why), not device names
- [ ] `img { max-width: 100%; height: auto; }` global rule; no fixed pixel widths on layout containers (use `max-width`/`min()`)
- [ ] No horizontal page scroll at any width from 320px up — test by dragging, not just presets

### Sections & layout
- [ ] Hero: full-impact section with `clamp()`-based fluid headline size, background image or gradient with a contrast-preserving overlay, and a prominent CTA button
- [ ] Features: grid — 1 column base → 2 columns → 3–4 columns at breakpoints (explicit `grid-template-columns` in queries, *or* justify why `auto-fit`/`minmax` needed no queries)
- [ ] "How it works": three numbered steps that stack on mobile and sit in a row on desktop
- [ ] Testimonials: at least two quote cards (`<blockquote>` + attribution) using the media-object pattern for avatar + text
- [ ] Pricing: three tiers that stack on mobile, sit side-by-side on desktop, with the middle tier visually "featured"; equal heights; buy buttons aligned to card bottoms
- [ ] Footer: link columns via grid or flex-wrap — 1–2 columns mobile, 3–4 desktop
- [ ] At least one use of `grid-template-areas` somewhere (hero split, page shell, or a feature section) — the layout must be readable from the area strings

### Responsive craft
- [ ] Fluid type: headline and section-title sizes via `clamp()` with rem bounds
- [ ] Fluid section spacing: `clamp()` or `min()` padding so sections breathe on desktop without wasting phone space
- [ ] Images: `object-fit` where cropping is needed; `loading="lazy"` below the fold; at least one image with `srcset`/`sizes`
- [ ] Nav adapts: full link row on desktop; on mobile either a clean wrapped row or a visual hamburger placeholder (JS not required)
- [ ] Page remains usable with text zoomed to 200%

## Hints

- Build order that works: content in semantic HTML first (whole page, no CSS) → mobile styles top to bottom → *then* widen the window and add breakpoints as things get roomy.
- The hero overlay recipe (readability over any image) is Chapter 8's layered-background pattern: `linear-gradient(rgb(0 0 0 / .5), rgb(0 0 0 / .5)), url(…) center / cover`.
- "Featured" pricing tier: on desktop, a slight `transform: scale(1.05)` or a stronger border + shadow reads as featured without breaking alignment.
- Buttons pinned to card bottoms is the same auto-margin trick as Project 4 — patterns repeat; that's the point.
- If a section's grid feels fiddly, write the `grid-template-areas` strings *first* as ASCII art in a comment, then make the CSS match.
- Real content beats lorem ipsum: invent a product with actual feature names and testimonial sentences — placeholder text hides layout problems (all boxes the same size) that real ragged content exposes.

## Stretch Goals

- [ ] Zero-media-query variant of the features grid via `repeat(auto-fit, minmax(…))` — compare against your breakpoint version
- [ ] Dark mode via `prefers-color-scheme` and color custom properties
- [ ] A sticky navbar (`position: sticky; top: 0`) with a border/shadow that distinguishes it from the hero — previews Chapter 12
- [ ] An FAQ section using `<details>`/`<summary>` (native accordion, zero JS)
- [ ] Print stylesheet: hide nav/hero media, linearize the pricing table
- [ ] Run Lighthouse; get Performance and Accessibility both above 90 and note what you fixed
