# HTML & CSS Learning Track

A self-paced, beginner-to-advanced path to job-ready HTML and CSS. Fifteen study chapters in `learning-docs/` build skills in order; seven guided projects in `projects/` cement them. Every chapter has runnable examples, real-world pitfalls, and exercises; every project is a spec with a requirements checklist (no solution code — building it is the point).

**How to use this track:** read a chapter, do its exercises, and build the projects at the checkpoints below. Don't skip projects — they're where learning actually sticks. Expect roughly 8–12 weeks at 5–8 hours/week.

## Chapters (`learning-docs/`)

| # | File | Topic |
|---|------|-------|
| 01 | `01-how-the-web-works-and-setup.md` | Client/server, URLs, HTTP, browser rendering; editor + devtools setup |
| 02 | `02-html-document-structure-and-semantics.md` | The HTML skeleton, elements/attributes, semantic layout elements, headings |
| 03 | `03-text-links-images-media.md` | Text semantics, links & paths, images & alt text, audio/video |
| 04 | `04-lists-and-tables.md` | ul/ol/dl, nesting, accessible data tables, colspan/rowspan |
| 05 | `05-forms-and-inputs.md` | Inputs, labels, fieldsets, radio/checkbox groups, built-in validation |
| 06 | `06-css-fundamentals.md` | Rules, selectors, pseudo-classes, cascade, specificity, inheritance, units |
| 07 | `07-the-box-model.md` | Content/padding/border/margin, box-sizing, display, margin collapse, overflow |
| 08 | `08-colors-typography-backgrounds.md` | Color systems & contrast, font properties, web fonts, backgrounds & gradients |
| 09 | `09-flexbox.md` | One-dimensional layout: axes, alignment, wrap, grow/shrink, gap |
| 10 | `10-css-grid.md` | Two-dimensional layout: tracks, fr, placement, template areas, auto-fit/minmax |
| 11 | `11-responsive-design-and-media-queries.md` | Mobile-first, media queries, fluid sizing with clamp(), responsive images |
| 12 | `12-positioning-and-z-index.md` | relative/absolute/fixed/sticky, stacking contexts, overlay patterns |
| 13 | `13-transitions-transforms-animations.md` | Transforms, transitions, @keyframes, performance, reduced motion |
| 14 | `14-accessibility-and-best-practices.md` | WCAG, keyboard/screen-reader support, ARIA, testing, code organization |
| 15 | `15-modern-css-features.md` | Custom properties, nesting, container queries, :has(), dialog/popover |

## Projects (`projects/`), easiest → hardest

| # | File | Build | After chapters |
|---|------|-------|----------------|
| 1 | `01-personal-profile-page.md` | Semantic "about me" page | 01–03 |
| 2 | `02-recipe-collection-page.md` | Recipe pages with lists, tables, deep links | 02–04 |
| 3 | `03-styled-signup-form.md` | Polished, validated sign-up form card | 05–08 |
| 4 | `04-flexbox-photo-gallery.md` | Full gallery site laid out entirely with flexbox | 07–09 |
| 5 | `05-responsive-landing-page.md` | Mobile-first product landing page with grid | 08–11 |
| 6 | `06-animated-portfolio-site.md` | Your portfolio, with positioning + tasteful motion | 10–13 |
| 7 | `07-capstone-accessible-multipage-site.md` | Complete accessible 4+ page site (everything) | 01–15 |

## Suggested Cadence

1. **Chapters 1–3** → **Project 1** (profile page)
2. **Chapter 4** → **Project 2** (recipes)
3. **Chapters 5–8** → **Project 3** (sign-up form) — your first real HTML+CSS build
4. **Chapter 9** → **Project 4** (flexbox gallery)
5. **Chapters 10–11** → **Project 5** (responsive landing page) — the classic portfolio piece
6. **Chapters 12–13** → **Project 6** (animated portfolio — showcase Projects 1–5 in it)
7. **Chapters 14–15** → **Project 7** (capstone) — then deploy Projects 6 and 7 online

## Working Tips

- Type every code example yourself and run it — reading CSS is not learning CSS.
- Keep devtools open constantly; the Elements/Styles panels are your debugger.
- Validate your HTML (validator.w3.org) after every project.
- When stuck, reread the relevant chapter's **Common pitfalls** section first — it exists because everyone hits the same walls.
- Reference docs for everything beyond this track: MDN (developer.mozilla.org). Support tables: caniuse.com.
