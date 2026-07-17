# Chapter 2: HTML Document Structure & Semantic Elements

## Overview

HTML (HyperText Markup Language) is how you tell the browser — and search engines, and screen readers — what your content *is*. Not what it looks like (that's CSS), but its meaning and structure: "this is the main heading," "this is navigation," "this is an article."

This chapter covers the skeleton every HTML document shares, the anatomy of elements and attributes, and the **semantic elements** (`header`, `nav`, `main`, `article`, etc.) that give a page its structure. Writing semantic HTML is a hallmark of professional front-end work: it improves accessibility, SEO, and maintainability for free.

## Definitions & Explanations

### Elements, tags, and attributes

An **element** is a piece of content with meaning attached:

```html
<p class="intro">Welcome to my site.</p>
```

- `<p ...>` is the **opening tag**, `</p>` the **closing tag**.
- `class="intro"` is an **attribute** — a name/value pair giving extra information. Attributes always go in the opening tag.
- Everything between the tags is the element's **content**.

Some elements are **void elements** (also called self-closing or empty): they have no content and no closing tag. Examples: `<img>`, `<br>`, `<hr>`, `<meta>`, `<link>`, `<input>`. Writing them as `<img />` (with a trailing slash) is optional in HTML — both forms are valid.

### Nesting and the document tree

Elements nest inside each other, forming a tree (the DOM from Chapter 1). Nesting must be *properly contained*:

```html
<!-- ✅ properly nested -->
<p>This is <strong>very</strong> important.</p>

<!-- ❌ overlapping tags — invalid -->
<p>This is <strong>very</p></strong>
```

Indent nested elements consistently (2 spaces is common) so the tree structure is visible in your code.

### The required skeleton

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Page Title</title>
  </head>
  <body>
    <!-- visible content -->
  </body>
</html>
```

Line by line:

- `<!DOCTYPE html>` — not an element, a declaration. It switches the browser into **standards mode**. Without it, browsers emulate 1990s bugs ("quirks mode") and your CSS behaves strangely.
- `<html lang="en">` — the root element. `lang` tells screen readers how to pronounce the text and helps translation tools. Always set it.
- `<head>` — metadata container. Nothing here renders on the page.
- `<meta charset="UTF-8">` — declares the character encoding. UTF-8 handles every language and emoji; omitting it can garble non-ASCII characters (é → Ã©).
- `<meta name="viewport" ...>` — tells mobile browsers not to zoom out to a fake desktop width. Required for responsive design (Chapter 11); include it from day one.
- `<title>` — the browser-tab text, bookmark name, and search-result headline.
- `<body>` — everything visible.

Other common `<head>` residents:

```html
<link rel="stylesheet" href="styles.css" />          <!-- attach CSS -->
<link rel="icon" href="favicon.ico" />                <!-- tab icon -->
<meta name="description" content="A summary shown in search results." />
```

### Block vs. inline (a first look)

By default, elements render in one of two ways:

- **Block-level** elements start on a new line and stretch the full available width: `<h1>`–`<h6>`, `<p>`, `<div>`, `<section>`, `<ul>`, etc.
- **Inline** elements flow within a line of text: `<a>`, `<strong>`, `<em>`, `<span>`, `<img>`.

Rule of thumb: don't put block elements inside inline ones, and don't put block elements inside `<p>` (the browser will silently close your paragraph early — a classic source of "why is my CSS not applying" confusion).

### Semantic vs. generic elements

**Semantic** elements describe their content's role. **Generic** elements (`<div>` for block, `<span>` for inline) mean nothing — they exist purely as styling/grouping hooks.

The main semantic layout elements:

| Element | Role |
|---|---|
| `<header>` | Introductory content — site logo, title, top navigation. (A page can have multiple headers, e.g. inside an article.) |
| `<nav>` | A major block of navigation links. |
| `<main>` | The unique, primary content of *this* page. **Exactly one per page.** |
| `<section>` | A thematic grouping of content, usually with its own heading. |
| `<article>` | A self-contained composition that would make sense on its own: blog post, news story, product card, comment. |
| `<aside>` | Tangential content: sidebar, pull-quote, related links. |
| `<footer>` | Closing content — copyright, contact links. |
| `<figure>` + `<figcaption>` | An illustration/diagram/photo with its caption. |

Why bother, when `<div>` "works"?

1. **Accessibility** — screen readers announce landmarks ("navigation," "main content") and let users jump between them. A page of anonymous divs is a wall of noise.
2. **SEO** — search engines weigh content by structural role.
3. **Readability** — `<nav>` tells the next developer (or future you) what the block is at a glance.

Use `<div>` when *no* semantic element fits — pure layout wrappers. That's still often! Semantic-when-possible, div-when-necessary.

### Headings and document outline

`<h1>` through `<h6>` are ranked headings. Rules that matter:

- One `<h1>` per page — the page's main topic.
- **Never skip levels going down** (`h1` → `h3` with no `h2`). Screen-reader users navigate by heading level; skips are disorienting.
- **Never choose a heading for its size.** If `h4` "looks right," use the structurally correct level and resize it with CSS.

### Comments, whitespace, and special characters

```html
<!-- This is an HTML comment. The browser ignores it. -->
```

HTML **collapses whitespace**: any run of spaces, tabs, and newlines renders as a single space. Layout comes from elements and CSS, never from typing extra spaces.

Some characters have meaning in HTML and need **entities** to display literally:

- `&lt;` → `<` &nbsp;&nbsp; `&gt;` → `>` &nbsp;&nbsp; `&amp;` → `&`
- `&quot;` → `"` &nbsp;&nbsp; `&copy;` → © &nbsp;&nbsp; `&nbsp;` → non-breaking space

## Code Examples

### Example 1: Minimal valid page

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>About Me</title>
  </head>
  <body>
    <h1>About Me</h1>
    <p>I'm learning HTML, and this page proves it.</p>
  </body>
</html>
```

### Example 2: Headings forming a proper outline

```html
<body>
  <h1>Coffee Brewing Guide</h1>          <!-- the one and only h1 -->

  <h2>Equipment</h2>                     <!-- major section -->
    <h3>Grinders</h3>                    <!-- subsection of Equipment -->
    <h3>Kettles</h3>

  <h2>Methods</h2>
    <h3>Pour Over</h3>
    <h3>French Press</h3>
      <h4>Steeping Time</h4>             <!-- detail under French Press -->
</body>
```

(Indentation here is just to visualize hierarchy — headings are siblings in the markup; their *rank* creates the outline.)

### Example 3: A full semantic page layout

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>The Daily Byte — Tech News</title>
  </head>
  <body>
    <!-- Site-wide header: appears on every page -->
    <header>
      <h1>The Daily Byte</h1>
      <nav>
        <ul>
          <li><a href="index.html">Home</a></li>
          <li><a href="reviews.html">Reviews</a></li>
          <li><a href="about.html">About</a></li>
        </ul>
      </nav>
    </header>

    <!-- The unique content of THIS page -->
    <main>
      <article>
        <header>
          <!-- Articles can have their own headers -->
          <h2>New Browser Released</h2>
          <p>Published <time datetime="2026-07-15">July 15, 2026</time></p>
        </header>

        <p>The latest version ships with impressive speed gains…</p>

        <section>
          <h3>Performance</h3>
          <p>Benchmarks show a 20% improvement…</p>
        </section>

        <section>
          <h3>New Features</h3>
          <p>The headline addition is…</p>
        </section>

        <footer>
          <p>Filed under: Browsers</p>
        </footer>
      </article>

      <!-- Related-but-secondary content -->
      <aside>
        <h2>Popular This Week</h2>
        <ul>
          <li><a href="#">Keyboard shortcuts you're missing</a></li>
          <li><a href="#">Is dark mode actually better?</a></li>
        </ul>
      </aside>
    </main>

    <!-- Site-wide footer -->
    <footer>
      <p>&copy; 2026 The Daily Byte. All rights reserved.</p>
    </footer>
  </body>
</html>
```

### Example 4: Semantic vs. div soup — same page, two ways

```html
<!-- ❌ "Div soup": works visually, but meaningless to machines and readers -->
<div class="top">
  <div class="links">…</div>
</div>
<div class="content">
  <div class="post">…</div>
</div>
<div class="bottom">…</div>

<!-- ✅ Semantic: identical layout potential, self-documenting -->
<header>
  <nav>…</nav>
</header>
<main>
  <article>…</article>
</main>
<footer>…</footer>
```

## Common Pitfalls

1. **Choosing headings by appearance.**
   ```html
   <!-- ❌ h4 chosen because it "looks the right size" -->
   <h1>My Blog</h1>
   <h4>First Post</h4>

   <!-- ✅ correct rank; adjust size later with CSS -->
   <h1>My Blog</h1>
   <h2>First Post</h2>
   ```

2. **Multiple `<main>` elements or none.** `<main>` marks the page's unique content — exactly one, and it shouldn't live inside `<header>`, `<footer>`, `<article>`, or `<aside>`.

3. **Wrapping everything in `<section>` "for structure."** A `<section>` without a heading is usually a `<div>` in disguise. If you can't name the section with a heading, it's probably not one.

4. **Forgetting the closing tag.** Unclosed `<div>`s cause the rest of the page to nest inside them, breaking layout mysteriously. Match every opener with a closer; let your editor's auto-close and indentation help. The W3C validator (`validator.w3.org`) catches these instantly.

5. **Block elements inside `<p>`.**
   ```html
   <!-- ❌ browser closes the <p> before the <ul>, silently -->
   <p>My hobbies: <ul><li>Chess</li></ul></p>

   <!-- ✅ -->
   <p>My hobbies:</p>
   <ul><li>Chess</li></ul>
   ```

6. **Unquoted or mis-quoted attributes.** Always quote attribute values: `class="intro"`, not `class=intro`. Use straight quotes `"` — pasting from Word can bring curly quotes `”` that break parsing.

7. **Missing `lang` and viewport meta.** Both are invisible until they bite you (screen readers mispronouncing, mobile pages microscopic). Bake them into your muscle-memory skeleton.

## Practice Exercises

1. **Skeleton from memory.** In a new file, type the full valid HTML5 skeleton (doctype through closing `</html>`, including charset, viewport, and title) without looking at this chapter. Verify against Example 1.

2. **Outline an article.** Pick a Wikipedia article you like. In HTML, reproduce only its heading structure (`h1`–`h4`, no body text) matching the article's table of contents. Check that no level is skipped.

3. **Semantic rebuild.** Here is a div-soup fragment. Rewrite it using appropriate semantic elements, and add anything the skeleton is missing:
   ```html
   <div id="top"><div class="menu"><a href="/">Home</a> <a href="/blog">Blog</a></div></div>
   <div class="stuff">
     <div class="story"><div class="big">Local Cat Elected Mayor</div><p>In a stunning…</p></div>
     <div class="side">Ads and related links here</div>
   </div>
   <div id="bottom">© 2026</div>
   ```

4. **Landmark hunt.** Open three real websites and, using devtools Elements, find whether they use `<header>`, `<nav>`, `<main>`, and `<footer>` (search the DOM tree with `Ctrl+F` inside the Elements panel). Note one site doing it well and one doing it badly.

5. **Validate.** Take any page you've written so far, paste its source into `https://validator.w3.org/#validate_by_input`, and fix every error and warning it reports.
