# Project 1: Personal Profile Page

## Description

Build a single-page "about me" profile — the web equivalent of a business card with personality. A visitor should immediately learn who you are, what you're into, and how to reach you. It should read cleanly top to bottom: a header with your name and tagline, a photo, a short bio, a few interests or skills, and links (email, GitHub, social).

This is a pure-HTML project (styling comes in later projects): the goal is a page whose *structure* is so good it's pleasant to read even unstyled. Recruiters really do view unstyled/reader-mode pages; semantic HTML is the skill on display here.

## Difficulty & Effort

- **Difficulty:** Beginner
- **Estimated effort:** 2–4 hours

## Chapters Used

- `01-how-the-web-works-and-setup.md`
- `02-html-document-structure-and-semantics.md`
- `03-text-links-images-media.md`

## Requirements Checklist

### Setup & validity
- [ ] Project folder contains `index.html` and an `images/` subfolder
- [ ] Valid HTML5 skeleton: doctype, `<html lang>`, `charset`, viewport meta, descriptive `<title>`
- [ ] Page passes the W3C validator with zero errors
- [ ] All filenames lowercase, no spaces

### Structure
- [ ] Semantic landmarks used: `<header>`, `<main>`, `<footer>` (and `<section>` where appropriate)
- [ ] Exactly one `<h1>` (your name); heading levels never skip
- [ ] At least three distinct sections (e.g., About, Interests/Skills, Contact), each introduced by a heading

### Content
- [ ] A profile photo (or stand-in) as an `<img>` with meaningful `alt`, explicit `width`/`height`
- [ ] A bio of at least two paragraphs using at least three inline semantic elements appropriately (`<em>`, `<strong>`, `<abbr>`, `<time>`, etc.)
- [ ] A quote you like marked up with `<blockquote>` (with attribution)
- [ ] At least one decorative image with correctly empty `alt=""`
- [ ] A contact section with a `mailto:` link and at least two external links that open in a new tab with `rel="noopener"`
- [ ] Every link has descriptive text (no "click here")
- [ ] A footer with a copyright line using the `&copy;` entity

## Hints

- Stuck on structure? Sketch the page as boxes on paper first and label each box with the semantic element it should be.
- For the photo before you have one you like: `https://picsum.photos/300/300` works as a placeholder — but still write the alt text as if it were the real photo.
- `<time datetime="…">` is a natural fit for "learning web development since …".
- Test your `mailto:` link actually opens a mail client, and test every external link lands where intended.
- Read your finished page top to bottom with CSS off (devtools → Rendering → or just imagine it): does the order still make sense? That's the semantic-HTML test.

## Stretch Goals

- [ ] Add a second page (`hobbies.html` or `now.html`) and a `<nav>` linking the pages both ways with correct relative paths
- [ ] Add a short embedded audio or video introduction with `controls` and fallback content
- [ ] Add a favicon via `<link rel="icon">`
- [ ] Add a `<meta name="description">` and check how the page title/description present when shared
