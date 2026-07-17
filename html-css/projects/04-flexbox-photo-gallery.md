# Project 4: Flexbox Photo Gallery Site

## Description

Build a one-page photography (or artwork/food/travel) gallery site with a proper site frame: sticky-feeling top navigation bar, a hero/intro strip, a filter-style button row, the gallery itself as a wrapping flexbox card grid, an "about the photographer" media-object section, and a footer that stays at the bottom even when content is short. Every layout structure on the page is flexbox — this project is deliberate, concentrated flexbox practice inside a real page.

It should feel like a real portfolio: images uniform and tidy, captions readable, cards equal height, everything calmly aligned.

## Difficulty & Effort

- **Difficulty:** Intermediate
- **Estimated effort:** 5–8 hours

## Chapters Used

- `07-the-box-model.md`
- `08-colors-typography-backgrounds.md`
- `09-flexbox.md`
- (structure/content from `02`–`03` throughout)

## Requirements Checklist

### Site frame
- [ ] Navbar: flexbox with logo/site-name left and nav links right (`justify-content: space-between`, `align-items: center`), links themselves a nested flex list with `gap`
- [ ] Sticky footer: `body` as a flex column with `min-height: 100vh` and `main` taking `flex: 1` — verify with a nearly-empty page
- [ ] A hero section with a heading and tagline centered both ways using flexbox
- [ ] A centered `.container` (max-width + auto margins) aligning all section content

### Gallery
- [ ] At least 12 images (own photos or picsum placeholders), each inside a card (`<figure>` with `<figcaption>` encouraged)
- [ ] Cards laid out with `display: flex; flex-wrap: wrap; gap: …` on the container — **no floats, no inline-block**
- [ ] Cards use `flex: 1 1 <basis>` so the count per row adapts as the window resizes — verify at narrow, medium, wide
- [ ] All images uniform within cards: `width: 100%`, fixed height or `aspect-ratio`, `object-fit: cover` (no squished/stretched photos)
- [ ] Cards are equal height per row (flex stretch — don't fight it, use it), with captions pinned to the card bottom when text lengths differ (hint: card = flex column)
- [ ] Every image has appropriate `alt`; decorative flourishes use `alt=""`

### Details
- [ ] A filter-style button row (All / Landscape / Portrait / …) as a wrapping flex row, `justify-content: center` — visual only, no JS needed
- [ ] An about section using the media-object pattern: fixed-size round avatar (`flex-shrink: 0`) beside flexible text (`flex: 1`)
- [ ] Hover treatment on cards (e.g., shadow or caption emphasis) — transitions optional until Chapter 13, snap changes fine
- [ ] `box-sizing` reset, `rem`-based type scale, palette with passing contrast
- [ ] No horizontal page scrollbar at any window width down to ~320px

## Hints

- Whitespace between cards misbehaving? You're probably mixing margins with wrap — delete the margins and let `gap` do all spacing.
- Cards refusing to shrink on narrow screens usually means a fixed `width` snuck in — inside flex layouts, prefer `flex-basis` and drop `width`.
- For caption-pinned-to-bottom: make the card `display: flex; flex-direction: column;` and give the caption `margin-top: auto` (auto margins absorb free space — a flexbox superpower this project wants you to discover).
- `object-fit: cover` needs the image to have a constrained height (or aspect-ratio) to do anything.
- Devtools tip: hover the flex badge in the Elements panel to toggle the flexbox overlay and *see* your axes and gaps.
- Resize continuously from 320px to full-screen — smooth dragging exposes awkward in-between states that fixed test sizes hide.

## Stretch Goals

- [ ] Make one "featured" image card span wider than the others (`flex-grow` weighting) without breaking wrap
- [ ] A lightbox mock: clicking isn't required, but build the overlay itself (fixed, `inset: 0`, centered image) as a hidden-by-default element you can reveal by temporarily adding a class in devtools — previews Chapter 12
- [ ] Zebra alternation: even-numbered sections get a tinted background (`:nth-of-type`)
- [ ] Swap the gallery to `repeat(auto-fit, minmax(…))` grid on a branch/copy and write a short comment comparing the two approaches — previews Chapter 10
