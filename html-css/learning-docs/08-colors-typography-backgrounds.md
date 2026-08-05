# Chapter 8: Colors, Typography & Backgrounds

## Overview

This is the chapter where pages start looking designed. You'll learn every way CSS expresses color (and when to use which), the font properties that make text readable and attractive, web fonts, and the background system — colors, images, gradients, and how they layer.

Typography deserves special attention: most of the web is text, and small changes (line height, measure, contrast) transform amateur-looking pages into professional ones.

## Definitions & Explanations

### Color values

```css
color: red;                        /* named color (147 exist; fine for demos) */
color: #ff6600;                    /* hex: RRGGBB, 00–ff per channel */
color: #f60;                       /* 3-digit shorthand: doubles each digit */
color: #ff660080;                  /* 8-digit hex: last pair = alpha (opacity) */
color: rgb(255 102 0);             /* same color, decimal channels 0–255 */
color: rgb(255 102 0 / 0.5);       /* 50% transparent */
color: hsl(24 100% 50%);           /* hue(0–360°) saturation lightness */
color: hsl(24 100% 50% / 0.5);
color: transparent;
color: currentColor;               /* "whatever this element's text color is" — handy for borders/SVG icons */
```

**HSL is the most human-friendly**: hue picks the color (0 red → 120 green → 240 blue → 360 red), saturation is intensity, lightness runs black (0%) → color (50%) → white (100%). Building a palette = fix a hue, vary lightness. Modern CSS also offers `oklch()` with more perceptually uniform lightness — same idea, better math; use it when you're comfortable.

`opacity: 0.5;` on an element fades the **entire element including children**. For a see-through background with solid text, use an alpha *color* instead.

### Contrast (non-negotiable)

Text must contrast with its background: WCAG requires a ratio of at least **4.5:1** for body text (3:1 for large text). Grey-on-grey aesthetics fail real users. Check with devtools (the color picker shows the ratio) or webaim.org/resources/contrastchecker.

### Typography properties

```css
p {
  font-family: Georgia, "Times New Roman", serif;  /* fallback stack, generic last */
  font-size: 1.125rem;
  font-weight: 400;        /* 100–900; 400 normal, 700 bold */
  font-style: italic;
  line-height: 1.6;        /* unitless multiplier — best practice */
  letter-spacing: 0.02em;  /* tracking */
  text-align: left;        /* left | right | center | justify */
  text-transform: uppercase;
  text-decoration: underline;   /* none | underline | line-through */
  text-indent: 2em;
}
```

Key concepts:

- **Font stack** — a comma list; the browser uses the first installed font, ending with a generic family (`serif`, `sans-serif`, `monospace`, `system-ui`). Quote multi-word names.
- **System font stack** — instant, native-feeling text with zero downloads: `font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;`
- **Line height** — 1.4–1.8 for body text. Unitless values scale with each element's own font size; `px` values don't (and cause bugs in nested content).
- **Measure (line length)** — 45–75 characters is comfortable: `max-width: 65ch`.
- **Type scale** — pick a ratio and stick to it, e.g. 1rem body, 1.25rem, 1.563rem, 1.953rem, 2.441rem for ascending headings (a 1.25 ratio). Consistency beats cleverness.

### Web fonts

Load fonts users don't have installed. Easiest route — a hosted service like Google Fonts (fonts.google.com): pick a family/weights, copy its `<link>` tags into `<head>`, use the family in CSS.

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet" />
```

```css
body { font-family: "Inter", system-ui, sans-serif; }
```

Self-hosting alternative with `@font-face`:

```css
@font-face {
  font-family: "MyFont";
  src: url("fonts/myfont.woff2") format("woff2");
  font-weight: 400;
  font-display: swap;  /* show fallback text immediately, swap when loaded */
}
```

Costs: every font file is a download. Two families, 2–3 weights total, is plenty. Always keep fallbacks in the stack.

### Backgrounds

```css
.hero {
  background-color: #1a2b3c;
  background-image: url("images/hero.jpg");
  background-repeat: no-repeat;     /* default is repeat (tiling) */
  background-position: center;      /* keywords, %, or lengths */
  background-size: cover;           /* cover = fill box, may crop
                                       contain = fit inside, may letterbox */
  background-attachment: fixed;     /* parallax-ish; janky on mobile, use sparingly */
}

/* Shorthand: color image repeat position / size */
.hero { background: #1a2b3c url("images/hero.jpg") no-repeat center / cover; }
```

**Content image or background image?** If the image *means* something (product photo, portrait), it's content → `<img>` with alt text. If it's decoration behind content → CSS background. Backgrounds have no alt and are invisible to assistive tech.

### Gradients (backgrounds you generate)

```css
.a { background: linear-gradient(to right, #ff8a00, #e52e71); }
.b { background: linear-gradient(135deg, navy 0%, teal 60%, gold 100%); } /* angle + color stops */
.c { background: radial-gradient(circle at top left, white, steelblue); }
.d { background: conic-gradient(red, yellow, lime, cyan, blue, magenta, red); } /* color wheel */
```

Gradients are images, so they use `background-image`/`background`, and can layer.

### Layering multiple backgrounds

Comma-separate; **first listed is on top**:

```css
.hero {
  background:
    linear-gradient(rgb(0 0 0 / 0.55), rgb(0 0 0 / 0.55)),  /* darkening overlay */
    url("images/hero.jpg") center / cover no-repeat;
  color: white;   /* readable thanks to the overlay */
}
```

This overlay trick is *the* standard way to keep text legible over photos.

### Shadows

```css
.card   { box-shadow: 0 2px 8px rgb(0 0 0 / 0.15); }   /* x y blur color */
.card:hover { box-shadow: 0 8px 24px rgb(0 0 0 / 0.2); }
h1.hero { text-shadow: 0 2px 4px rgb(0 0 0 / 0.5); }
```

Subtlety wins: low-opacity black, more blur than offset.

## Code Examples

### Example 1: A typographic base every project can start from

```css
*, *::before, *::after { box-sizing: border-box; }

html { font-size: 100%; }             /* respect user's browser setting */

body {
  margin: 0;
  font-family: system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
  font-size: 1rem;
  line-height: 1.6;
  color: #1f2937;                     /* near-black: softer than #000 */
  background-color: #ffffff;
}

h1, h2, h3 {
  line-height: 1.2;                   /* headings need less leading */
  margin: 2rem 0 0.75rem;
}

h1 { font-size: 2.441rem; }
h2 { font-size: 1.953rem; }
h3 { font-size: 1.563rem; }

p, ul, ol { margin: 0 0 1rem; }

article { max-width: 65ch; margin: 0 auto; padding: 0 1rem; }

a       { color: #1d4ed8; }
a:hover { color: #1e40af; }
```

### Example 2: An HSL-derived palette

```css
/* One hue (220 = blue), varied lightness = instant coherent palette */
:root {
  /* (custom properties — fully explained in Chapter 15; pattern shown early
      because you'll see it everywhere) */
  --brand:       hsl(220 70% 50%);
  --brand-dark:  hsl(220 70% 35%);
  --brand-light: hsl(220 70% 92%);
  --ink:         hsl(220 15% 15%);
  --paper:       hsl(220 20% 98%);
}

body     { color: var(--ink); background: var(--paper); }
.button  { background: var(--brand); color: white; }
.button:hover { background: var(--brand-dark); }
.callout { background: var(--brand-light); border-left: 4px solid var(--brand); }
```

### Example 3: Hero section with overlaid gradient

```html
<style>
  .hero {
    min-height: 60vh;
    display: grid;              /* quick centering; details in Chapter 10 */
    place-items: center;
    text-align: center;
    color: white;
    background:
      linear-gradient(rgb(10 20 40 / 0.6), rgb(10 20 40 / 0.6)),
      url("https://picsum.photos/1600/900") center / cover no-repeat;
  }
  .hero h1 {
    font-size: 3rem;
    margin: 0 0 0.5rem;
    text-shadow: 0 2px 6px rgb(0 0 0 / 0.5);
  }
  .hero p { font-size: 1.25rem; opacity: 0.9; }
</style>

<section class="hero">
  <div>
    <h1>Explore the Coast</h1>
    <p>Guided kayak tours, every weekend.</p>
  </div>
</section>
```

### Example 4: Web font pairing

```html
<head>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700&family=Inter:wght@400;600&display=swap" rel="stylesheet" />
  <style>
    body     { font-family: "Inter", system-ui, sans-serif; }
    h1, h2   { font-family: "Playfair Display", Georgia, serif; } /* display serif for headings */
    strong   { font-weight: 600; }  /* the weight we actually loaded */
  </style>
</head>
```

### Example 5: Patterned and textured backgrounds without images

```css
/* Subtle stripes from a repeating gradient */
.stripes {
  background: repeating-linear-gradient(
    45deg,
    #f8fafc 0 12px,
    #eef2f7 12px 24px
  );
}

/* Dot grid */
.dots {
  background-image: radial-gradient(#cbd5e1 1px, transparent 1px);
  background-size: 16px 16px;
}
```

## Common Pitfalls

1. **Insufficient contrast.** `#999` text on white (2.8:1) fails accessibility and squints everyone. Check ratios; aim ≥ 4.5:1 for body text. Devtools' color picker shows the ratio right in the Styles pane.

2. **`opacity` when you meant an alpha color.**
   ```css
   /* ❌ fades the text too */
   .banner { background: black; opacity: 0.5; }

   /* ✅ only the background is translucent */
   .banner { background: rgb(0 0 0 / 0.5); }
   ```

3. **Meaningful images as CSS backgrounds.** The team photo set via `background-image` has no alt text and disappears for assistive tech and image search. Content → `<img>`; decoration → background.

4. **Fixed-height boxes with `background-size: cover` text heroes** where cropping cuts the focal point. Set `background-position` deliberately (`center 30%`, etc.) and test at several widths.

5. **Font soup.** Five families and nine weights make pages slow and messy. Two families (one for headings, one for body — or just one family) with 2–3 weights is the professional norm.

6. **Forgetting fallback fonts.** `font-family: "Inter";` alone means invisible/ugly text if the font fails to load. Always end with a generic family.

7. **px line-height inheritance bug.** `line-height: 24px` on `body` looks fine until a large heading inherits it and its lines overlap. Unitless `line-height: 1.5` sidesteps the whole class of bug.

8. **Justified text on the web.** `text-align: justify` without hyphenation creates "rivers" of whitespace, especially on narrow screens. Prefer left-aligned ragged-right.

9. **Missing shorthand resets.** `background: red;` *clears* any previously set background-image (shorthand resets all sub-properties). When layering rules, prefer the specific property (`background-color: red`).

## Practice Exercises

1. **Color conversion drill.** Take the color `#e52e71`. Express it as `rgb()`, approximate it in `hsl()` (get the hue close by eye/devtools), and produce a 40%-opacity version in two different syntaxes. Then create lighter and darker variants by only changing HSL lightness, and display all five as swatch divs.

2. **Typography makeover.** Take any plain HTML article (e.g. your Chapter 3 exercise) and, without touching the HTML, give it: a Google-Fonts heading face with system-stack body, a modular type scale, unitless line-height 1.6, `max-width: 65ch` centered, and styled links with visible hover states. Compare before/after screenshots.

3. **Contrast audit.** Build a page showing five text/background color pairs of your choosing. Using devtools' contrast checker, record each ratio in a caption and fix any pair under 4.5:1 by adjusting HSL lightness only.

4. **Hero trio.** Build three hero sections stacked on one page: (a) photo background + dark gradient overlay + white text; (b) pure CSS gradient background (no image) at an angle with 3 color stops; (c) patterned background from repeating gradients. Each needs a readable heading and subheading.

5. **Palette from one hue.** Choose a hue. Derive a 6-color palette in HSL (brand, brand-dark, brand-light, ink, paper, accent at a complementary hue). Apply it to a card component: background, border, heading, body text, a button with hover state, and a subtle box-shadow.
