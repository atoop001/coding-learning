# Project 7 (Capstone): Fully Accessible Multi-Page Site

## Description

Build a complete, production-quality website for a plausible small organization — a local café, an animal shelter, a community sports club, a small conference — as if they hired you. Minimum four pages: Home, About/Services, a data-rich page (menu, adoptable animals, schedule — something with real tables and lists), and a Contact page with a full form. Shared navigation and footer on every page, one coherent design system, responsive everywhere, and — the defining requirement — **verifiably accessible to WCAG 2.1 AA intent**, using modern CSS architecture (design tokens, container queries where sensible, `:has()` where it earns its place).

This is the track's exam. Everything from all 15 chapters appears; nothing new is introduced. Done well, it's the strongest piece in your portfolio — a multi-page, accessible, responsive site is precisely what junior front-end interviews ask you to walk through.

## Difficulty & Effort

- **Difficulty:** Advanced
- **Estimated effort:** 15–25 hours (plan multiple sessions; treat it like a real client project)

## Chapters Used

All of them:
- `01`–`05` (HTML: structure, content, lists/tables, forms)
- `06`–`08` (CSS core: selectors/cascade, box model, color/type/backgrounds)
- `09`–`11` (layout: flexbox, grid, responsive)
- `12`–`13` (positioning, motion)
- `14` (accessibility & practices — the spine of this project)
- `15` (modern CSS: tokens, nesting, container queries, `:has()`, `<dialog>`/popover)

## Requirements Checklist

### Site architecture
- [ ] ≥ 4 pages with consistent shared header/nav/footer and working relative links throughout (test from every page, including any subfolder)
- [ ] One shared external stylesheet, organized per Chapter 14's structure (tokens → base → layout → components → utilities → preferences) with section comments
- [ ] Design tokens: all colors, spacing scale, radii, shadows, and font stacks as custom properties on `:root` — zero repeated magic values in components
- [ ] Every page: valid HTML (validator-clean), unique descriptive `<title>`, `<meta name="description">`, `lang`, viewport

### Accessibility (the core)
- [ ] Skip link on every page, visible on focus, targeting `<main>`
- [ ] Landmarks on every page: `header`, `nav` (with `aria-label` if more than one), `main` (exactly one), `footer`
- [ ] Logical heading outline per page — no skips, one `<h1>` describing that page
- [ ] Current page indicated in nav both visually and with `aria-current="page"`
- [ ] All interactive elements are native (`a`, `button`, form controls) — zero clickable divs
- [ ] Visible `:focus-visible` styles everywhere; full keyboard walk of every page passes (reach, operate, see focus, escape)
- [ ] All text contrast ≥ 4.5:1 (≥ 3:1 large text/UI); no color-only meaning anywhere
- [ ] All images: meaningful `alt` or empty `alt=""` per role; icon-only buttons have accessible names
- [ ] Contact form: labels, fieldsets/legends, `autocomplete` attributes, error/hint text linked via `aria-describedby`, errors marked beyond color, keyboard-completable end to end
- [ ] Motion respectful: `prefers-reduced-motion` block; no infinite decorative loops outside loading states
- [ ] Zoom test: 200% text zoom and 320px width both fully usable
- [ ] Lighthouse Accessibility = 100 on every page **plus** a written note (comment or short `NOTES.md`) of three issues you found manually that the tool couldn't

### Content & layout requirements
- [ ] Data page: at least one real accessible table (`caption`, `scope`d `<th>`s) that handles narrow screens via a scroll wrapper — plus meaningful use of `ul`/`ol`/`dl`
- [ ] Home: hero + at least two distinct grid-based sections; `grid-template-areas` used at least once
- [ ] A reusable card component that appears on ≥ 2 pages, adapting via **container query** to at least one narrow slot (e.g., sidebar vs main)
- [ ] At least two justified uses of positioning (sticky nav, badge, overlay, etc.) with a documented z-index scale
- [ ] Motion: subtle transitions on interactive elements; at most one entrance animation; all transform/opacity-only
- [ ] At least one earned use of `:has()` (e.g., form validity styling, content-aware cards) and native nesting used cleanly (≤ 2 levels)
- [ ] Fully responsive mobile-first; every media query commented with its reason

### Professional finish
- [ ] Real (invented but realistic) content — coherent business name, consistent voice, no lorem ipsum
- [ ] A favicon and consistent visual identity across pages
- [ ] Cross-browser check in at least two engines (e.g., Chrome/Edge + Firefox); note any differences found

## Hints

- **Plan like a professional:** before any code, write a one-page brief (who is this org, who visits the site, what must each page achieve), sketch each page's boxes on paper, and list your components. An hour of planning saves five of restructuring.
- Build the *system* first: tokens, base typography, container, button, card — on a scratch page. Then assemble pages from the system. This is how real front-end teams work.
- Copy your own best code. Projects 3–6 contain a working form, gallery grid, landing sections, and motion patterns. Reuse and improve; don't re-derive.
- Do the accessibility checklist *continuously*, not as a final gate — retrofitting focus styles and label wiring at the end is 3x the work.
- The container-query card requirement wants an actual narrow slot to exist: a sidebar ("opening hours" beside content, "related items") is the natural excuse.
- Schedule a mid-project keyboard-and-Lighthouse audit when two pages exist — findings there are cheap to fix on the remaining pages.
- When you think you're done: full test pass — validator on every page, keyboard walk, 320px drag, 200% zoom, reduced-motion emulation, dark scheme if you built it, both browsers. Fix, re-test, then stop polishing and ship.

## Stretch Goals

- [ ] Dark mode across the whole site via token swap, with `color-scheme` set so native controls match
- [ ] A native `<dialog>` (e.g., "reserve a table" mini-form) and a `popover` menu, both keyboard-tested
- [ ] Print stylesheet for the data page (menu/schedule prints cleanly)
- [ ] A 404 page matching the design system
- [ ] Deploy to GitHub Pages/Netlify with a custom-ish URL, and add the live link plus per-project descriptions to your Project 6 portfolio
- [ ] Write a short case study (challenges, decisions, before/after Lighthouse numbers) — interview gold
