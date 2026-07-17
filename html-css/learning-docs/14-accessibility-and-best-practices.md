# Chapter 14: Accessibility & Best Practices

## Overview

Accessibility (**a11y** — "a", 11 letters, "y") means building pages that work for everyone: people using screen readers, keyboard-only navigation, magnification, voice control, or simply aging eyes on a bright day. Roughly one in six people has a disability; in many jurisdictions accessible websites are a legal requirement; and every accessibility technique also improves SEO, mobile usability, and code quality. Employers increasingly screen for it.

The good news: if you've followed this track — semantic elements, labeled forms, alt text, contrast, focus styles, reduced motion — you've been doing accessibility all along. This chapter consolidates it into a checklist-driven practice, introduces ARIA (and when *not* to use it), and finishes with professional habits: code organization, naming, and validation.

## Definitions & Explanations

### How people actually use your page

- **Screen reader users** (NVDA and JAWS on Windows, VoiceOver on Mac/iOS, TalkBack on Android) hear the page linearly and navigate by *landmarks*, *headings*, *links*, and *form controls* — which is why semantics matter: they're the navigation data.
- **Keyboard users** (motor disabilities, power users, broken trackpads) move with `Tab`/`Shift+Tab` between interactive elements, `Enter`/`Space` to activate. If it can't be reached and operated by keyboard, it's broken.
- **Low-vision users** zoom to 200–400% and/or raise default font size (rem-based sizing pays off here), and depend on contrast.
- **Vestibular/motion-sensitive users** rely on `prefers-reduced-motion` (Chapter 13).
- **Cognitive load** affects everyone: clear headings, plain language, consistent navigation.

### The POUR principles (WCAG's framework)

The Web Content Accessibility Guidelines organize requirements as **P**erceivable, **O**perable, **U**nderstandable, **R**obust. Aim for WCAG 2.1 AA — the standard cited by most laws and job postings. Practical highlights:

- Text contrast ≥ 4.5:1 (3:1 for large text and UI components).
- All functionality keyboard-operable, no keyboard traps.
- Page has a `lang`, a descriptive `<title>`, one logical heading outline.
- Form inputs have labels; errors are described in text (not color alone).
- Nothing conveys meaning by *color only* (add icons/text: "● Error" not just red).
- Touch targets comfortably sized (≥ 44×44px is the common guideline).
- Motion respectful; no content flashing more than 3 times/second.

### The keyboard contract

- **Focus must be visible.** Never `outline: none` without a replacement (`:focus-visible` styles only keyboard focus, so mouse users don't see rings).
- **Focus order = source order.** Another reason to avoid `order`/`dense`/absolute-position layouts that scramble visual vs DOM order.
- **Use real interactive elements.** A `<button>` is focusable, keyboard-activatable, and announced as a button — free. A `<div onclick>` is none of those and needs a pile of attributes and JS to fake it:

```html
<!-- ❌ inaccessible fake button -->
<div class="btn" onclick="save()">Save</div>

<!-- ✅ everything for free -->
<button type="button" class="btn">Save</button>
```

Links navigate (`href` somewhere); buttons act. Choose by behavior, style however you like.

- **Skip link** — the first focusable thing on the page lets keyboard users jump past repeated navigation:

```html
<a class="skip-link" href="#main">Skip to main content</a>
…
<main id="main">…</main>
```

```css
.skip-link {
  position: absolute;
  top: -100%;                 /* parked offscreen… */
}
.skip-link:focus {
  top: 0; left: 0;            /* …appears when tabbed to */
  background: #111; color: white; padding: 0.75rem 1.25rem; z-index: 1000;
}
```

### ARIA: powerful, dangerous, mostly unnecessary

ARIA attributes (`role`, `aria-*`) add semantics for assistive tech when HTML alone can't express them. **The first rule of ARIA: don't use ARIA if a native element does the job.** `<button>` beats `role="button"`; `<nav>` beats `role="navigation"`. Bad ARIA is worse than none — it makes confident false promises to screen readers.

The genuinely common, safe uses:

```html
<!-- Label something with no visible text -->
<button aria-label="Close dialog">×</button>

<!-- Distinguish two navs -->
<nav aria-label="Primary">…</nav>
<nav aria-label="Breadcrumb">…</nav>

<!-- Mark the current page in a nav -->
<a href="/pricing" aria-current="page">Pricing</a>

<!-- Hide decorative elements from screen readers -->
<span aria-hidden="true">🎉</span>

<!-- State on interactive widgets (JS toggles the value) -->
<button aria-expanded="false" aria-controls="menu">Menu</button>

<!-- Announce dynamic updates (form errors, "saved!" toasts) -->
<p role="status" aria-live="polite">Changes saved.</p>

<!-- Tie an error message to its field -->
<input id="em" aria-describedby="em-err" aria-invalid="true" />
<p id="em-err">Enter a valid email address.</p>
```

Related CSS pattern — **visually hidden but readable text** (for icon buttons, extra screen-reader context):

```css
.visually-hidden {
  position: absolute;
  width: 1px; height: 1px;
  padding: 0; margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}
```

### Testing accessibility (15 minutes per page)

1. **Keyboard walk**: unplug the mouse. Tab through everything. Can you see where you are? Reach and operate everything? Escape everything?
2. **Automated scan**: Lighthouse (devtools → Lighthouse → Accessibility) or the axe DevTools extension. Catches ~30–40% of issues — the mechanical ones.
3. **Headings & landmarks**: devtools or a screen reader's rubric — is the outline sensible out of context?
4. **Zoom to 200%** and 320px width: everything still readable and operable?
5. **Screen reader spot-check**: NVDA (free) or VoiceOver — listen to your page's header, nav, one form. Humbling and irreplaceable.
6. **Contrast pass**: devtools color picker on every text/background pair.

### Professional practices (the rest of "best practices")

**HTML hygiene**
- Validate (validator.w3.org). Semantic elements first, divs second. One `<h1>`, no skipped levels, alt on every image, labels on every control.

**CSS organization** — a scalable single-file order:

```css
/* 1. Custom properties & resets */
/* 2. Base element styles (body, headings, links) */
/* 3. Layout primitives (.container, .grid, .stack) */
/* 4. Components (.card, .button, .navbar) — one block each */
/* 5. Utilities (.visually-hidden, .text-center) */
/* 6. Media query refinements (or nest them per component) */
```

**Naming** — name by *role*, not appearance: `.button-primary`, not `.big-blue-button` (it will not stay blue). Conventions like BEM (`.card__title`, `.card--featured`) keep large stylesheets unambiguous; even without full BEM, be consistent and lowercase-hyphenated.

**Maintainability**
- Custom properties for every repeated color/size (full treatment in Chapter 15).
- Comment the *why*, not the what: `/* negative margin counteracts the gap on wrap */`.
- Delete dead code; commented-out graveyards rot.
- Consistent formatting (Prettier).
- Test in more than one browser; check caniuse.com before leaning on a shiny feature.

## Code Examples

### Example 1: An accessible page skeleton (assembling the whole track)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Order Confirmation — Trailhead Co.</title>
</head>
<body>
  <a class="skip-link" href="#main">Skip to main content</a>

  <header>
    <nav aria-label="Primary">
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/tours" aria-current="page">Tours</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>
  </header>

  <main id="main">
    <h1>Your booking is confirmed</h1>
    <p role="status">Confirmation #48291 has been emailed to you.</p>

    <section aria-labelledby="details-h">
      <h2 id="details-h">Trip details</h2>
      <!-- content -->
    </section>
  </main>

  <footer>
    <h2 class="visually-hidden">Site footer</h2>
    <nav aria-label="Footer">…</nav>
    <p>&copy; 2026 Trailhead Co.</p>
  </footer>
</body>
</html>
```

### Example 2: Icon buttons and links, done right

```html
<!-- Icon-only button: name it -->
<button type="button" aria-label="Search">
  <svg aria-hidden="true" width="20" height="20" viewBox="0 0 20 20">
    <path d="M8 3a5 5 0 1 0 3.9 8.1l4 4 1.4-1.4-4-4A5 5 0 0 0 8 3z"/>
  </svg>
</button>

<!-- "Read more" links: give screen-reader context without visual clutter -->
<a href="/post/css-grid">
  Read more<span class="visually-hidden"> about CSS Grid layouts</span>
</a>

<!-- Never color-only status -->
<p class="error">
  <strong>Error:</strong> Your card was declined.  <!-- word + color, not color alone -->
</p>
```

### Example 3: Accessible form with error messaging

```html
<form novalidate>
  <div class="field">
    <label for="email">Email address</label>
    <input type="email" id="email" name="email" required
           aria-describedby="email-hint email-error" aria-invalid="true"
           autocomplete="email" />
    <p id="email-hint" class="hint">We'll only use this for your receipt.</p>
    <p id="email-error" class="error" role="alert">
      <strong>Error:</strong> "sam@" is missing a domain — e.g. sam@example.com
    </p>
  </div>
  <button type="submit">Subscribe</button>
</form>

<style>
  .field { margin-bottom: 1.25rem; }
  label  { display: block; font-weight: 600; margin-bottom: 0.25rem; }
  input  { padding: 0.6rem; width: 100%; max-width: 24rem;
           border: 2px solid #9ca3af; border-radius: 6px; }
  input[aria-invalid="true"] { border-color: #b91c1c; }
  .hint  { color: #4b5563; font-size: 0.9rem; }
  .error { color: #b91c1c; }                     /* 6.1:1 on white — passes AA */
  input:focus-visible, button:focus-visible {
    outline: 3px solid #1d4ed8; outline-offset: 2px;
  }
</style>
```

### Example 4: Focus styles that look designed

```css
/* Remove the double-ring problem, keep keyboard visibility */
:focus { outline: none; }                 /* only safe BECAUSE of the next rule */
:focus-visible {
  outline: 3px solid var(--brand, #1d4ed8);
  outline-offset: 2px;
  border-radius: 4px;                     /* outline follows radius in modern browsers */
}

/* Interactive elements get a minimum touch target */
button, a.button, input[type="checkbox"], input[type="radio"] {
  min-width: 44px;
  min-height: 44px;
}
```

### Example 5: A stylesheet skeleton showing professional organization

```css
/* ============ 1. Tokens & reset ============ */
:root {
  --brand: hsl(220 70% 45%);
  --ink: hsl(220 15% 15%);
  --paper: hsl(0 0% 100%);
  --space-1: 0.5rem; --space-2: 1rem; --space-3: 2rem;
  --radius: 8px;
}
*, *::before, *::after { box-sizing: border-box; }
body { margin: 0; font-family: system-ui, sans-serif; line-height: 1.6;
       color: var(--ink); background: var(--paper); }

/* ============ 2. Base elements ============ */
img { max-width: 100%; height: auto; display: block; }
a { color: var(--brand); }

/* ============ 3. Layout primitives ============ */
.container { width: min(100% - 2rem, 70rem); margin-inline: auto; }
.stack > * + * { margin-block-start: var(--space-2); }  /* uniform vertical rhythm */

/* ============ 4. Components ============ */
.card { padding: var(--space-2); border-radius: var(--radius);
        border: 1px solid #e5e7eb; }
/* .button, .navbar, … each in its own labeled block */

/* ============ 5. Utilities ============ */
.visually-hidden { position: absolute; width: 1px; height: 1px; padding: 0;
                   margin: -1px; overflow: hidden; clip-path: inset(50%);
                   white-space: nowrap; border: 0; }

/* ============ 6. Preferences ============ */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: 0.01ms !important;
                           transition-duration: 0.01ms !important; }
}
```

## Common Pitfalls

1. **Divs and spans doing interactive work.** `<div class="button">` fails keyboard, focus, and screen-reader announcement simultaneously. Real `<button>`, `<a>`, `<input>`, `<select>` — then style them.

2. **`outline: none` with no replacement.** The most common accessibility bug on the web. Use the `:focus-visible` pattern (Example 4).

3. **ARIA as a fix-all.** `role="button"` slapped on a div announces "button" but adds zero behavior — worse than honest silence. Native element first; ARIA for states (`aria-expanded`) and names (`aria-label`), not for impersonation.

4. **Redundant/wrong alt text.** `alt="image of logo"` (screen reader says "image, image of logo"); decorative flourishes with verbose alts; meaningful charts with `alt=""`. Revisit Chapter 3's rules — they're exam favorites for a reason.

5. **Color as the only signal.** Red border = error, green dot = online: invisible to colorblind users (~8% of men). Pair color with text or icons, always.

6. **Placeholder-as-label** (again — it's that common in the wild). Labels persist; placeholders vanish and fail contrast.

7. **Heading levels chosen for looks** breaking the outline — Chapter 2's rule, now with its real justification: heading navigation is the screen-reader user's primary map.

8. **Trusting the automated scan.** Lighthouse 100 ≠ accessible; it can't tell if your alt text is *useful*, your focus order sensible, or your error messages clear. Automated + keyboard walk + screen-reader spot-check.

9. **Appearance-based class names and magic numbers.** `.blue-text`, `margin-top: 37px` with no comment. Name by role; extract repeated values to custom properties; comment the weird ones.

## Practice Exercises

1. **Keyboard audit.** Take your most complex previous exercise (or any small live site). Mouse away: navigate entirely by keyboard and log every failure (unreachable control, invisible focus, illogical order, hover-only content). Fix each and re-walk.

2. **Retrofit.** Take your Chapter 5 sign-up form and upgrade it to Example 3's standard: hints and errors tied via `aria-describedby`, `aria-invalid`, error text with icon/word (not color alone), `:focus-visible` styling, 44px targets, and a skip link on the page. Run Lighthouse before and after; record both scores.

3. **Icon bar.** Build a toolbar of five icon-only buttons (SVG or emoji): each needs an accessible name, `aria-hidden` on the glyph, visible focus, adequate size, and a tooltip-style visible label on hover *and* focus.

4. **Screen-reader hour.** Install NVDA (Windows, free) or enable VoiceOver. Learn five commands (read next, headings list, landmarks list, links list, forms mode). Browse one page you made and one professional site; write down three things your page announces badly and fix them.

5. **Stylesheet refactor.** Take your messiest stylesheet from earlier chapters and reorganize it per Example 5: tokens extracted to custom properties, sections labeled and ordered, appearance-based names renamed, dead rules deleted, weird values commented, reduced-motion block added. Confirm the page renders identically (screenshot diff) — refactors change structure, not output.
