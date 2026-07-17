# Chapter 6: CSS Fundamentals — Selectors, Specificity, Cascade & Units

## Overview

CSS (Cascading Style Sheets) controls how HTML looks: colors, sizes, spacing, layout, motion. This chapter is the conceptual core of the entire CSS half of the track. The mechanics you learn here — how rules are written, how selectors target elements, how conflicts are resolved (the *cascade* and *specificity*), how values inherit, and which units to use — explain 90% of the "why isn't my CSS working?!" moments you will ever have.

Read this chapter carefully. Everything later (box model, flexbox, grid, animations) is just new properties; the *rules of the game* are all here.

## Definitions & Explanations

### Anatomy of a rule

```css
selector {
  property: value;   /* one "declaration" */
  property: value;   /* semicolon after every declaration */
}
```

```css
/* Example: */
h1 {
  color: navy;
  font-size: 32px;
}
```

- **Selector** — *which* elements this applies to.
- **Declaration block** — the `{ … }`.
- **Declaration** — one `property: value;` pair.
- Comments: `/* like this */` (no `//` comments in CSS!).

### Three ways to attach CSS

1. **External stylesheet** (the right way):
   ```html
   <link rel="stylesheet" href="styles.css" />
   ```
   One file styles many pages; browsers cache it.
2. **Internal** — a `<style>` element in `<head>`. Fine for single-file demos and exercises.
3. **Inline** — a `style` attribute: `<p style="color:red">`. Avoid: unmaintainable, and it wins over almost everything (see specificity), making later overrides painful.

### Selectors: the essential set

| Selector | Example | Matches |
|---|---|---|
| Type (element) | `p` | every `<p>` |
| Class | `.card` | every element with `class="card"` |
| ID | `#site-logo` | the one element with `id="site-logo"` |
| Universal | `*` | everything |
| Attribute | `input[type="email"]` | inputs whose type is email |
| Grouping | `h1, h2, h3` | all three types (comma = "or") |
| Descendant | `article p` | `<p>` anywhere *inside* an `<article>` |
| Child | `ul > li` | `<li>` that are *direct* children of a `<ul>` |
| Adjacent sibling | `h2 + p` | a `<p>` immediately after an `<h2>` |
| General sibling | `h2 ~ p` | any `<p>` after an `<h2>` (same parent) |
| Compound | `p.intro` | `<p class="intro">` (no space = same element) |

Classes are your daily driver. An element can carry several: `class="card featured"` — two independent hooks. IDs must be unique per page; reserve them for fragment links and JavaScript, and prefer classes for styling.

**`p.intro` vs `p .intro`** — the space changes everything. Without space: a `p` *that has* class intro. With space: an element with class intro *inside* a `p`.

### Pseudo-classes and pseudo-elements

**Pseudo-classes** (single colon) select elements in a particular *state*:

```css
a:hover        { text-decoration: underline; }  /* mouse over */
a:visited      { color: purple; }
button:focus   { outline: 2px solid blue; }     /* keyboard focus — never remove without replacement! */
button:active  { transform: scale(0.98); }      /* while pressed */
input:invalid  { border-color: crimson; }
li:first-child { font-weight: bold; }
li:last-child  { border-bottom: none; }
li:nth-child(odd)  { background: #f5f5f5; }     /* zebra striping */
li:nth-child(3)    { color: red; }              /* the third item */
:not(.active)  { opacity: 0.6; }                /* everything except .active */
```

**Pseudo-elements** (double colon) style *parts* of elements or generate content:

```css
p::first-line   { font-variant: small-caps; }
p::first-letter { font-size: 2em; }
.card::before   { content: "★ "; }   /* inserts generated content before the card's content */
::selection     { background: gold; } /* highlighted text */
```

### The cascade: how conflicts are resolved

Multiple rules often target the same element. The browser resolves conflicts in this order:

1. **Origin & importance** — author styles (yours) beat browser defaults; `!important` declarations jump the queue (avoid it; it's a maintainability trap).
2. **Specificity** — more specific selectors win (below).
3. **Source order** — among equal specificity, *the last rule in the file wins*.

### Specificity: the scoring system

Count a selector's parts as three numbers **(ID, CLASS, TYPE)**:

- Each **ID** (`#x`) → 1-0-0
- Each **class, attribute, or pseudo-class** (`.x`, `[type=…]`, `:hover`) → 0-1-0
- Each **type or pseudo-element** (`p`, `::before`) → 0-0-1
- `*` and combinators (`>`, `+`, `~`, space) → 0-0-0

Compare left to right; higher number in an earlier column wins outright (ten classes never beat one ID).

```css
p                     /* 0-0-1 */
.intro                /* 0-1-0  beats p */
p.intro               /* 0-1-1  beats .intro */
article p.intro       /* 0-1-2 */
#main p               /* 1-0-1  beats all of the above */
```

Inline `style=""` outranks any selector; `!important` outranks that. If you're reaching for `!important` to win a fight, the real fix is usually a more sensible selector.

Practical advice that prevents most specificity wars: **style almost everything with single classes** (0-1-0). Flat specificity means source order — which you control — settles ties.

### Inheritance

Some properties flow from parent to child automatically: text-related ones like `color`, `font-family`, `font-size`, `line-height`, `text-align`. Box-related ones (`margin`, `padding`, `border`, `width`, `background`) do **not** inherit.

```css
body {
  font-family: Georgia, serif;  /* every element inherits this… */
  color: #333;                  /* …and this */
}
```

Special values usable on any property: `inherit` (take parent's value), `initial` (spec default), `unset` (inherit if inheritable, else initial).

### Units

**Absolute:**
- `px` — CSS pixels. Fine for borders, small fixed details.

**Relative (prefer these for sizes and spacing):**
- `em` — relative to the element's *own font size* (for `font-size`, relative to the parent's). Compounds when nested — powerful but surprising.
- `rem` — relative to the **root** (`<html>`) font size, default 16px. `1.5rem` = 24px. Doesn't compound; the workhorse for font sizes, spacing, widths. Crucially, rems scale when users raise their browser's default font size — an accessibility win over px.
- `%` — relative to the parent element's corresponding dimension.
- `vw` / `vh` — 1% of viewport width/height. `100vh` = full screen height.
- `ch` — width of the "0" character; great for readable line lengths (`max-width: 65ch`).

Unitless: `line-height: 1.5` (multiplier — preferred over units for line-height), and `0` never needs a unit.

Rule of thumb: `rem` for font sizes and most spacing, `%`/`fr`(later)/`vw` for fluid layout widths, `px` for hairline borders and shadows, `ch` for text measure.

## Code Examples

### Example 1: A styled page exercising core selectors

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Selector practice</title>
  <style>
    /* Type selectors + inheritance from body */
    body {
      font-family: system-ui, sans-serif;
      color: #222;
      line-height: 1.6;
    }

    /* Class selector — the everyday tool */
    .card {
      border: 1px solid #ccc;
      padding: 1rem;
    }

    /* Compound: only paragraphs that carry .warning */
    p.warning { color: darkred; }

    /* Descendant: links anywhere inside nav */
    nav a { text-decoration: none; }

    /* Child: only top-level list items, not nested ones */
    ul.menu > li { font-weight: bold; }

    /* Adjacent sibling: the paragraph right after any h2 */
    h2 + p { font-style: italic; }

    /* Attribute */
    input[type="email"] { border: 2px solid steelblue; }

    /* Pseudo-classes */
    a:hover { color: crimson; }
    li:nth-child(odd) { background: #f7f7f7; }
  </style>
</head>
<body>
  <nav><a href="#">Home</a> <a href="#">About</a></nav>
  <h2>Section title</h2>
  <p>This paragraph is italic because it directly follows the h2.</p>
  <p>This one is not.</p>
  <div class="card">
    <p class="warning">This is a warning inside a card.</p>
  </div>
  <ul class="menu">
    <li>Bold top-level item
      <ul><li>Normal nested item (child selector skips me)</li></ul>
    </li>
    <li>Another bold item</li>
  </ul>
  <input type="email" placeholder="steelblue border" />
</body>
</html>
```

### Example 2: The cascade in action — predict, then verify

```css
/* All three rules target the same paragraph: <p class="intro" id="lead"> */

p       { color: gray;   }   /* 0-0-1 */
.intro  { color: green;  }   /* 0-1-0 — beats p */
#lead   { color: purple; }   /* 1-0-0 — beats .intro → paragraph is PURPLE */

/* Equal specificity → source order decides: */
.intro  { color: green; }
.intro  { color: teal;  }    /* later rule wins → teal (if #lead weren't there) */
```

Open devtools → Elements → select the element → Styles pane: overridden declarations show ~~struck through~~, with the winning rule on top. This panel is your specificity debugger for life.

### Example 3: rem-based sizing scale

```css
html { font-size: 16px; }         /* 1rem = 16px (browser default anyway) */

h1   { font-size: 2.25rem; }      /* 36px */
h2   { font-size: 1.5rem;  }      /* 24px */
body { font-size: 1rem;    }      /* 16px */
small{ font-size: 0.875rem;}      /* 14px */

.section  { padding: 2rem; margin-bottom: 3rem; }
.narrow   { max-width: 65ch; }    /* comfortable reading measure */
.full-hero{ min-height: 100vh; }  /* fill the viewport */
```

### Example 4: em compounding vs rem stability

```css
/* Demonstrates why nested ems surprise people */
.parent { font-size: 20px; }
.child-em  { font-size: 1.5em; }   /* 30px (1.5 × parent's 20px) */
.child-rem { font-size: 1.5rem; }  /* 24px (1.5 × root 16px) — regardless of nesting */

/* em IS useful when you want scaling relative to local text: */
.button {
  font-size: 1rem;
  padding: 0.5em 1em;   /* padding scales automatically if button font-size changes */
}
```

### Example 5: Generated content with pseudo-elements

```css
/* External-link marker */
a[href^="http"]::after {   /* ^= means "attribute starts with" */
  content: " ↗";
  font-size: 0.8em;
}

/* Decorative divider without extra HTML */
h2::before {
  content: "";
  display: block;
  width: 3rem;
  height: 4px;
  background: coral;
  margin-bottom: 0.5rem;
}
```

## Common Pitfalls

1. **Missing semicolons / braces.** One missing `;` silently kills the *next* declaration too. A missing `}` can kill the rest of the file. When "everything below line X stopped working," look for an unclosed block at line X.

2. **Selector typos that fail silently.** `.calss`, `#hero` when the HTML says `id="Hero"` (case matters for classes/IDs!), or `h1.title` when title is on a `span`. CSS never errors — it just doesn't match. Devtools → Elements → confirm the class actually appears on the element.

3. **Specificity wars escalating to `!important`.**
   ```css
   /* ❌ arms race */
   #sidebar div.widget p a { color: red !important; }

   /* ✅ flat, calm, controllable */
   .widget-link { color: red; }
   ```

4. **Confusing `p.intro` with `p .intro`.** The space is a descendant combinator. If your rule matches nothing (or too much), check for accidental/missing spaces.

5. **Expecting non-inherited properties to inherit.** Setting `border` on `body` doesn't border every element — only text-ish properties cascade down. Conversely `width: 50%` on a child is 50% *of its parent*, not of the page.

6. **Sizing everything in px.** Users who bump their browser's base font size get nothing from a px-locked page. Use rem for type and spacing; save px for borders.

7. **Overriding rule written *above* the rule it should override.** Equal specificity → later wins. Your override must come after (in file order, or in a later-linked stylesheet).

8. **Styling with IDs.** Works, until you need to override it or reuse the style. Classes everywhere keeps specificity flat and code reusable.

## Practice Exercises

1. **Specificity scoring.** Without a browser, compute the (ID-CLASS-TYPE) score for each selector, then rank them: `nav ul li a`, `.menu .item`, `#header .logo`, `a:hover`, `ul > li.active`, `#main #content p`, `*`. Verify one tricky pair in devtools by writing conflicting `color` rules.

2. **Selector target practice.** Given one HTML page you write containing a nav, two articles with paragraphs, a nested list, and three links (one external), write single rules that: (a) style only paragraphs inside articles; (b) bold only *top-level* list items; (c) color only the external link (attribute selector, no extra class); (d) zebra-stripe the list; (e) style the first paragraph after each `h2`.

3. **Cascade detective.** Create a page where one `<p>` is targeted by five different rules setting `color`. Predict the winning color on paper before opening the page. Then reorder/adjust selectors to make each of the other four colors win, one at a time, *without using* `!important`.

4. **Unit conversion drill.** With the default 16px root: express 20px, 28px, and 12px in rem; give a hero section a height of half the viewport; give an article a max-width of 60 characters; set line-height 1.5 unitlessly. Build a page proving each works.

5. **Refactor a specificity mess.** Write (or borrow from an old project) a stylesheet using at least two ID selectors, one inline style, and one `!important`. Refactor it to classes-only with no `!important`, preserving the identical rendered result. Use devtools' struck-through declarations to confirm nothing unintended changed.
