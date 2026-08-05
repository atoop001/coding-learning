# Chapter 3: Text, Links, Images & Media

## Overview

Content is the point of the web, and this chapter covers the elements that carry it: paragraphs and inline text semantics, hyperlinks (the "H" in HTML), images (including a primer on SVG), and audio/video. You'll learn the difference between absolute and relative URLs, how to write `alt` text that actually helps people, and how to embed media responsibly.

Master this chapter and you can build genuinely useful pages — everything after it is refinement.

## Definitions & Explanations

### Text elements

- `<p>` — a paragraph. The workhorse of body text.
- `<br>` — a line break *within* a paragraph (poem lines, addresses). Not for spacing between blocks — that's CSS margin.
- `<hr>` — a thematic break (scene change, topic shift). Renders as a horizontal rule.
- `<blockquote>` — an extended quotation from another source. Optional `cite` attribute holds the source URL.
- `<pre>` — preformatted text: whitespace is preserved exactly (useful for code, ASCII art).
- `<code>` — inline code snippet: `<code>console.log()</code>`. Combine: `<pre><code>…</code></pre>` for code blocks.

### Inline semantics: meaning, not looks

- `<strong>` — strong importance (typically bold). *"<strong>Do not</strong> unplug the server."*
- `<em>` — stress emphasis, changing sentence meaning (typically italic). *"I <em>never</em> said that."*
- `<b>` / `<i>` — visually bold/italic *without* extra importance: keywords, product names, ship names, foreign phrases. Prefer `<strong>`/`<em>` when meaning is intended.
- `<mark>` — highlighted relevance (like a highlighter pen), e.g. search-term matches.
- `<small>` — side comments and fine print.
- `<abbr title="HyperText Markup Language">HTML</abbr>` — abbreviation with expansion on hover.
- `<time datetime="2026-07-17">July 17</time>` — machine-readable dates.
- `<sub>`/`<sup>` — subscript/superscript: H<sub>2</sub>O, x<sup>2</sup>.
- `<span>` — the inline `<div>`: no meaning, just a hook for CSS.

### Links: `<a>` and `href`

```html
<a href="https://example.com">Visit Example</a>
```

The `href` (hypertext reference) attribute holds the destination. Kinds of destinations:

1. **Absolute URL** — full address including scheme and domain: `https://developer.mozilla.org/`. Use for *other* websites.
2. **Relative URL** — relative to the current file's location. Use for pages *within your own site*:
   - `about.html` — same folder
   - `pages/contact.html` — into a subfolder
   - `../index.html` — up one folder (`..` means "parent directory")
   - `/images/logo.png` — **root-relative**: starts from the site's root (works on a server, but *not* when opening files via `file://`)
3. **Fragment** — jump to an element with a matching `id` on the same page: `href="#pricing"`. Combine with a path to deep-link: `guide.html#setup`.
4. **Special schemes** — `mailto:hi@example.com` opens the mail app; `tel:+15551234567` dials on phones.

Useful link attributes:

- `target="_blank"` — open in a new tab. **Always pair with** `rel="noopener"` (security: prevents the new page from controlling yours).
- `download` — prompt a download instead of navigating (same-origin files).

Link text matters: it should describe the destination. Screen-reader users often browse a list of a page's links out of context, where ten links all reading "click here" are useless.

### Images: `<img>`

```html
<img src="images/sunset.jpg" alt="Orange sun setting over a calm lake" width="800" height="600" />
```

- `src` — path/URL to the image file (absolute or relative, same rules as links).
- `alt` — **required.** Text alternative used when the image can't be seen: by screen readers, when the file fails to load, and by search engines.
  - Describe the *content and function*, not the fact it's an image ("Photo of…" is redundant — screen readers already announce "image").
  - Purely decorative image? Use an **empty** alt (`alt=""`) so screen readers skip it. Never *omit* the attribute.
- `width`/`height` — the image's intrinsic dimensions in pixels. Providing them lets the browser reserve space before the file loads, preventing content from jumping around (**layout shift**). Actual display size is controlled by CSS.
- `loading="lazy"` — defer loading offscreen images until the user scrolls near them.

Image formats, quickly:

| Format | Best for |
|---|---|
| JPEG (`.jpg`) | Photographs |
| PNG | Screenshots, images needing sharp edges or transparency |
| WebP / AVIF | Modern replacements for both — smaller files, wide support |
| SVG | Logos, icons, illustrations — vector, scales infinitely, tiny files |
| GIF | Mostly legacy; use video for animation |

### A quick SVG primer

SVG (Scalable Vector Graphics) describes images as shapes and paths (math), not pixels — so it scales to any size with zero blur, and simple icons/logos are often just a few hundred bytes.

Two ways to use one:

- `<img src="icon.svg" alt="…" />` — treat it like any other image. Simple, cacheable, but you can't style its internals with your page's CSS.
- Inline `<svg>…</svg>` directly in the HTML — heavier markup, but now it's part of the DOM: you can style it with CSS (`fill: currentColor` makes an icon inherit the surrounding text color) and animate it.

Every SVG needs a `viewBox="minX minY width height"` — it defines the coordinate system the shapes are drawn in and is *why* the image scales cleanly at any size.

```html
<svg viewBox="0 0 24 24" width="24" height="24" aria-hidden="true">
  <circle cx="12" cy="12" r="10" fill="currentColor" />
</svg>
```

For a handful of icons, hand-writing paths like this is fine. For a whole icon set, reach for an icon library/font (e.g. an SVG sprite sheet or a package like Lucide) instead of collecting one-off paths — it's less to maintain.

**Pitfall:** omit `viewBox` and the SVG stops scaling proportionally when you resize it with CSS/attributes — it may clip or distort instead. Always set it, even if `width`/`height` match it exactly.

### Figures with captions

```html
<figure>
  <img src="chart.png" alt="Bar chart: sales doubled from 2024 to 2026" />
  <figcaption>Fig. 1 — Sales growth, 2024–2026.</figcaption>
</figure>
```

`<figure>` marks self-contained illustrative content; `<figcaption>` is its visible caption. The caption does not replace `alt` — they serve different audiences.

### Audio and video

```html
<video controls width="640" poster="preview.jpg">
  <source src="demo.webm" type="video/webm" />
  <source src="demo.mp4" type="video/mp4" />
  Sorry, your browser doesn't support embedded video.
  <a href="demo.mp4">Download the video</a> instead.
</video>

<audio controls src="podcast-episode-1.mp3"></audio>
```

- `controls` — show play/pause/volume UI. Without it there's no way to play (unless you script one).
- Multiple `<source>` children let the browser pick the first format it supports; fallback text renders only in ancient browsers.
- `poster` — image shown before playback starts.
- `autoplay` exists but browsers block autoplaying audio; `autoplay muted loop` is the pattern for silent background video. Use sparingly — autoplay is hostile to users on metered data.
- Provide captions for video with `<track kind="captions" src="captions.vtt" srclang="en" label="English" />` — required for accessibility.

### Embedding other pages: `<iframe>`

```html
<iframe src="https://www.openstreetmap.org/export/embed.html?bbox=..." width="600" height="450" title="Map of our office location"></iframe>
```

An `<iframe>` embeds another document (maps, videos hosted elsewhere). Always give it a `title` describing its content. Note many sites forbid being iframed.

## Code Examples

### Example 1: An article with rich text semantics

```html
<article>
  <h2>Why I Switched to Mechanical Keyboards</h2>
  <p>
    Published <time datetime="2026-03-02">March 2, 2026</time> ·
    <small>5 minute read</small>
  </p>

  <p>
    I was skeptical at first — <em>really</em> skeptical. But after a month,
    I can say this: <strong>my wrists have never felt better.</strong>
  </p>

  <blockquote cite="https://example.com/ergonomics-study">
    <p>Typing comfort correlates strongly with key travel and actuation force.</p>
  </blockquote>

  <p>
    The board I chose uses <abbr title="Polybutylene Terephthalate">PBT</abbr>
    keycaps, and remapping keys took one line of config:
    <code>capslock = ctrl</code>.
  </p>

  <hr />
  <p><small>Disclosure: no keyboards sponsored this post, sadly.</small></p>
</article>
```

### Example 2: A navigation system across a multi-page site

Folder layout:

```
site/
├── index.html
├── about.html
└── blog/
    └── first-post.html
```

In `blog/first-post.html`:

```html
<nav>
  <ul>
    <!-- ../ climbs out of blog/ to reach the site root -->
    <li><a href="../index.html">Home</a></li>
    <li><a href="../about.html">About</a></li>
    <!-- same-folder link needs no prefix -->
    <li><a href="first-post.html" aria-current="page">First Post</a></li>
    <!-- external link: absolute URL, new tab, secured -->
    <li><a href="https://developer.mozilla.org" target="_blank" rel="noopener">MDN Docs</a></li>
    <!-- jump to a section further down this page -->
    <li><a href="#comments">Skip to comments</a></li>
  </ul>
</nav>

<!-- ...much later in the page... -->
<section id="comments">
  <h2>Comments</h2>
</section>
```

### Example 3: Images done right

```html
<!-- Informative image: descriptive alt -->
<img
  src="images/team-photo.jpg"
  alt="The five members of the design team standing in front of the office mural"
  width="1200"
  height="800"
  loading="lazy"
/>

<!-- Decorative flourish: empty alt so assistive tech skips it -->
<img src="images/divider-swirl.svg" alt="" width="400" height="20" />

<!-- Linked logo: alt describes the DESTINATION, since the image acts as a link -->
<a href="index.html">
  <img src="images/logo.svg" alt="Acme Co. — home" width="120" height="40" />
</a>

<!-- Figure with caption -->
<figure>
  <img src="images/latte-art.jpg" alt="A rosetta pattern poured in latte foam" width="600" height="600" />
  <figcaption>My first successful rosetta, attempt #47.</figcaption>
</figure>
```

### Example 4: Media page

```html
<h2>Watch the workshop</h2>
<video controls width="720" poster="images/workshop-poster.jpg">
  <source src="media/workshop.webm" type="video/webm" />
  <source src="media/workshop.mp4" type="video/mp4" />
  <track kind="captions" src="media/workshop-en.vtt" srclang="en" label="English" default />
  Your browser can't play this video. <a href="media/workshop.mp4">Download it</a>.
</video>

<h2>Or listen to the audio version</h2>
<audio controls>
  <source src="media/workshop.ogg" type="audio/ogg" />
  <source src="media/workshop.mp3" type="audio/mpeg" />
</audio>
```

## Common Pitfalls

1. **"Click here" links.**
   ```html
   <!-- ❌ meaningless out of context -->
   <p>To read the report, <a href="report.pdf">click here</a>.</p>

   <!-- ✅ the link text describes the destination -->
   <p>Read the <a href="report.pdf">2026 annual report (PDF)</a>.</p>
   ```

2. **Missing or lazy alt text.**
   ```html
   <!-- ❌ omitted attribute: screen readers read the filename aloud -->
   <img src="IMG_20260704_183512.jpg" />
   <!-- ❌ useless -->
   <img src="dog.jpg" alt="image" />
   <!-- ✅ -->
   <img src="dog.jpg" alt="A golden retriever catching a frisbee mid-air" />
   ```

3. **Absolute local paths.** `src="C:\Users\me\Pictures\photo.jpg"` works only on *your* machine. Copy assets into your project folder and use relative paths (`images/photo.jpg`).

4. **Spaces and capitals in filenames.** `My Photo (1).JPG` becomes an encoding headache in URLs. Rename to `my-photo-1.jpg` before referencing.

5. **Using `<br>` for spacing.**
   ```html
   <!-- ❌ presentational hack -->
   <p>First paragraph</p><br /><br />
   <p>Second paragraph</p>

   <!-- ✅ separate paragraphs; adjust gaps with CSS margin later -->
   <p>First paragraph</p>
   <p>Second paragraph</p>
   ```

6. **`target="_blank"` without `rel="noopener"`.** The opened page gets a handle to yours (`window.opener`) and can redirect it — a phishing vector. Modern browsers mitigate this, but the habit costs nothing.

7. **Giant unoptimized images.** A 6MB phone photo squeezed to 300px wide by CSS still downloads all 6MB. Resize/compress images to roughly their display size before adding them (tools: Squoosh.app, any image editor).

8. **Forgetting `title` on iframes** and captions on videos — both are accessibility requirements, not niceties.

## Practice Exercises

1. **Semantic text sampler.** Write a short "how-to" article (real or invented) that correctly uses all of: `<em>`, `<strong>`, `<blockquote>`, `<code>`, `<abbr>`, `<time>`, `<mark>`, and `<small>`. Every usage must be semantically justified, not decorative.

2. **Three-page mini-site.** Create `index.html`, `hobbies.html`, and `favorites/music.html` (note the subfolder). Give every page a shared `<nav>` with working relative links between all three. Test every link in the browser, including from the subfolder page back to the root pages.

3. **Image gallery page.** Build a page with four `<figure>`s, each containing an image (use your own photos or `https://picsum.photos/600/400` placeholders) with meaningful `alt`, explicit `width`/`height`, `loading="lazy"`, and a `<figcaption>`.

4. **Table-of-contents jumps.** Write a single long page with five `<h2>` sections (each with an `id`) and a linked table of contents at the top. Add a "Back to top" link at the end of each section (hint: give the `<h1>` or `<body>` an id).

5. **Media embed.** Find a Creative-Commons audio clip and a video clip (or record short ones yourself). Embed both with `controls`, give the video a `poster`, and include fallback download links. Bonus: write a tiny `.vtt` captions file for the first ten seconds of the video and attach it with `<track>`.
