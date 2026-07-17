# Project 2: Recipe Collection Page

## Description

Build a page (or small set of pages) presenting 2–3 favorite recipes the way a good cooking site would: each recipe with a photo, an at-a-glance info panel, an ingredients list, numbered steps, and a nutrition table. A visitor should be able to jump straight to any recipe from a table of contents, skim the ingredients while shopping, and follow the steps while cooking.

Still HTML-first (a tiny bit of table styling is allowed if the default look bothers you), this project exercises every content-structuring element you've learned — especially lists and tables, used *correctly*.

## Difficulty & Effort

- **Difficulty:** Beginner
- **Estimated effort:** 3–5 hours

## Chapters Used

- `02-html-document-structure-and-semantics.md`
- `03-text-links-images-media.md`
- `04-lists-and-tables.md`

## Requirements Checklist

### Page structure
- [ ] Valid skeleton with landmarks (`header`, `main`, `footer`); one `<h1>` for the collection
- [ ] Each recipe is an `<article>` with its own heading and an `id` for deep-linking
- [ ] A linked table of contents at the top jumps to each recipe (`href="#recipe-id"`)
- [ ] A "Back to top" link after each recipe

### Per recipe (each of the 2–3 recipes)
- [ ] A photo in a `<figure>` with a `<figcaption>` and meaningful `alt`
- [ ] An at-a-glance panel as a `<dl>`: prep time, cook time, servings, difficulty
- [ ] Ingredients as a `<ul>` — with at least one recipe using a *nested* list (e.g., "For the sauce:" sub-list, nested inside the parent `<li>`)
- [ ] Steps as an `<ol>` in correct order (shuffling them must break the recipe)
- [ ] At least one step contains `<strong>` for a critical warning and one uses `<em>` for emphasis

### The nutrition table (at least one recipe)
- [ ] A real `<table>` with `<caption>`, `<thead>`, `<tbody>`
- [ ] Header cells are `<th>` with correct `scope="col"` / `scope="row"`
- [ ] At least one `colspan` or `rowspan` used meaningfully (e.g., a footer note spanning all columns)
- [ ] No layout-abuse: the table contains only genuinely tabular data

### Content quality
- [ ] Fractions/degrees written properly (½ or 1/2 consistently; °C/°F)
- [ ] Zero validator errors

## Hints

- Nested-list syntax is the #1 stumble here: the sub-`<ul>` goes *inside* the parent `<li>`, before its closing tag — reread Chapter 4's pitfall 3 if the indentation fights you.
- For the `<dl>` panel, pairs read naturally as `<dt>Prep time</dt><dd>10 minutes</dd>`.
- Sketch the nutrition table as a grid on paper before writing `<tr>`s; count cells per row including spans.
- Fragment links only work if the `id` matches the `href` exactly — case-sensitive, no `#` inside the `id` attribute itself.
- If a recipe has "variations" or "tips," that's a natural `<section>` or `<aside>` inside the article.

## Stretch Goals

- [ ] Split each recipe onto its own page in a `recipes/` subfolder, with a collection index page — get the `../` relative paths right in both directions
- [ ] Add a comparison table on the index: rows = recipes, columns = time / difficulty / servings, first column as row headers
- [ ] Add minimal CSS for the tables only: `border-collapse`, cell padding, striped rows with `:nth-child(odd)`
- [ ] Add a `<video>` technique clip (or link one) for the trickiest step
