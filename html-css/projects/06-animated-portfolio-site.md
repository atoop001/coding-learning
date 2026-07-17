# Project 6: Animated Portfolio Site

## Description

Build *your* portfolio — the site you could genuinely link on a resume — as a responsive multi-section page (or 2–3 pages) showcasing the projects from this track, with tasteful, performant motion throughout. A sticky navbar that stays out of the way, a hero that animates in on load, project cards that lift and reveal on hover, a skills section, and a contact section. Overlay elements (badges, image-caption scrims, a modal mock or dropdown) put positioning to work.

The bar for motion: felt, not noticed. Everything under 300ms for interaction feedback, transform/opacity only, and fully respectful of `prefers-reduced-motion`. This project is where "knows CSS" starts looking like "has taste."

## Difficulty & Effort

- **Difficulty:** Intermediate–Advanced
- **Estimated effort:** 10–15 hours

## Chapters Used

- `10-css-grid.md`
- `11-responsive-design-and-media-queries.md`
- `12-positioning-and-z-index.md`
- `13-transitions-transforms-animations.md`
- (everything earlier, in service of the above)

## Requirements Checklist

### Structure & layout
- [ ] Sections: hero, projects, skills, about, contact — semantic landmarks, correct heading outline, one `<h1>`
- [ ] Projects displayed in a responsive grid (`auto-fit`/`minmax` or breakpointed), each card containing image, title, description, tech tags, and a link
- [ ] Mobile-first responsive from 320px up, fluid type via `clamp()`, no horizontal scroll anywhere
- [ ] At least 4 real project cards — use Projects 1–5 from this track (screenshot each; a screenshot of your own work is the correct image here)

### Positioning
- [ ] Sticky navbar (`position: sticky; top: 0`) with a `z-index` that keeps it above all page content
- [ ] In-page nav links that jump to sections, with `scroll-margin-top` (or padding compensation) so headings don't hide under the sticky bar
- [ ] Each project card has an absolutely positioned element over a `relative` anchor (e.g., a "Featured" or year badge in a corner)
- [ ] An image-overlay treatment: gradient scrim caption pinned to an image's bottom edge, or a hover-revealed full-cover overlay (`inset: 0`) with a centered link
- [ ] A deliberate z-index scale documented in a CSS comment (e.g., raised=1, nav=100, overlay=1000) — no arbitrary 99999s

### Motion
- [ ] Hero entrance: heading, subheading, and CTA animate in staggered with `@keyframes`, per-element `animation-delay`, and correct `fill-mode` (no flash, no snap-back)
- [ ] Card hover: lift via `transform: translateY` + shadow growth, image zoom inside `overflow: hidden` — smooth in *and* out (transition on base state)
- [ ] All transitions/animations animate **only `transform` and `opacity`** (shadows/colors allowed for accents); nothing animates width/height/margin/top
- [ ] Explicit `transition-property` lists — no `transition: all`
- [ ] Interaction feedback durations ≤ 300ms; entrance animations ≤ 700ms
- [ ] Every hover effect has a keyboard equivalent (`:focus-visible` / `:focus-within`) and nothing is *only* reachable by hover
- [ ] A `prefers-reduced-motion: reduce` block that disables/reduces all motion — test via devtools emulation

### Craft
- [ ] Design tokens (colors, spacing, radii, shadows) as custom properties on `:root`
- [ ] Visible, styled focus states on all interactive elements
- [ ] All text passes contrast; images have proper `alt`; Lighthouse Accessibility ≥ 95

## Hints

- Motion plan before motion code: list every animated element, its trigger, duration, and properties in a comment block at the top of the stylesheet. If the list is long, cut it — restraint reads as professionalism.
- The stagger pattern is Chapter 13 Example 4 almost verbatim; the card hover is Example 2. Adapting worked examples is real practice, not cheating.
- If your fixed/sticky/overlay layers misbehave, check for accidental stacking contexts — a `transform` on a hero wrapper is the usual culprit (Chapter 12, Example 5).
- Sticky nav overlapping anchored sections: `html { scroll-padding-top: <nav-height> }` fixes every in-page jump at once.
- Screenshot tip: open each old project, size the window to a consistent ratio, and capture — uniform thumbnails make the grid look designed.
- Test the whole page tab-only: every card link reachable, every focus visible, dropdown (if any) operable.

## Stretch Goals

- [ ] Scroll-triggered reveals using pure CSS `animation-timeline: view()` (check caniuse; fine to feature-gate) or the `:has()`-free fallback of simply animating on load
- [ ] Dark/light theme with a polished toggle look (`prefers-color-scheme` base; a checkbox-hack or JS toggle if you're feeling brave)
- [ ] A real `<dialog>`-based project detail modal, styled with `::backdrop` — previews Chapter 15
- [ ] Animated skill meters or tag clouds — keeping the transform/opacity rule
- [ ] Deploy it: GitHub Pages or Netlify free tier — a portfolio that isn't online isn't a portfolio; this is worth doing properly
