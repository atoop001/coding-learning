# Chapter 1: How the Web Works & Setting Up Your Tools

## Overview

Before writing a single line of HTML, it pays to understand what actually happens when you type an address into a browser and press Enter. Web development makes far more sense when you know the machinery underneath: what a server is, what a browser does, what files travel across the wire, and where your code fits into that picture.

This chapter also gets your workspace ready: a code editor, a browser, and the browser's built-in developer tools ("devtools"). By the end you will have written and opened your first web page locally, and you'll know how to inspect any page on the internet.

Why it matters for employability: interviewers frequently ask "what happens when you type a URL into the browser?" — it's a classic screening question. And devtools fluency is the single most-used practical skill of working front-end developers.

## Definitions & Explanations

### Clients and servers

- **Client** — the device/program *requesting* something. Your browser (Chrome, Firefox, Edge, Safari) is a client.
- **Server** — a computer somewhere that *responds* to requests. It stores websites' files (or generates them on demand) and sends them back to clients.
- The web is a giant conversation of clients asking and servers answering. This is called the **client–server model**.

### URLs, domains, and DNS

A **URL** (Uniform Resource Locator) is a web address. Take `https://www.example.com/blog/post.html?id=7#comments`:

| Part | Name | Meaning |
|---|---|---|
| `https://` | scheme/protocol | *How* to talk (HTTPS = secure HTTP) |
| `www.example.com` | domain (host) | *Which server* to talk to |
| `/blog/post.html` | path | *Which resource* on that server |
| `?id=7` | query string | Extra parameters |
| `#comments` | fragment | A location *within* the page |

Computers don't find each other by names — they use **IP addresses** (like `93.184.216.34`). **DNS** (Domain Name System) is the internet's phone book: it translates `example.com` into an IP address so your browser knows where to send the request.

### HTTP and HTTPS

**HTTP** (HyperText Transfer Protocol) is the language clients and servers speak. A request says "GET me `/blog/post.html`"; the response contains a **status code** and the content.

Common status codes you'll see constantly:

- `200 OK` — success
- `301 / 302` — redirect (the resource moved)
- `404 Not Found` — the path doesn't exist on the server
- `500 Internal Server Error` — the server broke

**HTTPS** is HTTP wrapped in encryption (TLS). Always use it in production; browsers mark plain HTTP sites as "not secure."

### What the browser does with what it receives

1. Browser requests the HTML document.
2. It **parses** the HTML top to bottom, building the **DOM** (Document Object Model) — an in-memory tree of every element on the page.
3. When it meets `<link>` tags (CSS) or `<img>`/`<script>` tags, it fires off *additional* requests for those files.
4. CSS is parsed into the **CSSOM**; DOM + CSSOM combine into a render tree.
5. The browser performs **layout** (figuring out where every box goes) and **paint** (drawing pixels).

You write three languages that the browser natively understands:

- **HTML** — structure and meaning ("this is a heading, this is a paragraph").
- **CSS** — presentation ("headings are dark blue and centered").
- **JavaScript** — behavior ("when clicked, open the menu"). Not covered in this track, but know where it fits.

### Static files vs. servers — and why you don't need one yet

A plain `.html` file on your disk can be opened directly by a browser (a `file://` URL). For everything in this track, that's all you need. Later, tools like "Live Server" simulate a real web server locally (a `http://localhost` URL) and auto-refresh when you save — a nice upgrade, not a requirement.

## Setting Up

### 1. Install a code editor

**Visual Studio Code** (free, the industry standard): download from `https://code.visualstudio.com`. Recommended extensions:

- **Live Server** — right-click an HTML file → "Open with Live Server" for auto-reloading.
- **Prettier** — auto-formats your code on save so indentation is always tidy.

Useful VS Code habits from day one:

- `Ctrl+S` constantly — save early, save often.
- Emmet is built in: in an empty `.html` file, type `!` then press `Tab` to expand a full HTML skeleton.

### 2. Pick a browser and find devtools

Any modern browser works. Open **devtools** with `F12` or `Ctrl+Shift+I` (Windows) / `Cmd+Option+I` (Mac). The tabs you'll live in:

- **Elements** — the live DOM tree and the CSS applied to each element. You can edit both *live* (changes vanish on refresh — it's a sandbox).
- **Console** — errors and warnings show up here. Check it whenever something "doesn't work."
- **Network** — every request the page makes: files, sizes, timings, status codes.

### 3. Create a project folder

Make a folder for your practice work, e.g. `my-website/`. Keep related files together:

```
my-website/
├── index.html      ← homepage (servers look for this name by default)
├── styles.css
└── images/
    └── photo.jpg
```

`index.html` is a convention: when a URL points at a folder, servers serve `index.html` automatically.

## Code Examples

### Example 1: Your very first page

Create `index.html` in your project folder, paste this, save, then double-click the file (or right-click → Open With → your browser):

```html
<!DOCTYPE html>
<!-- The line above tells the browser: "this is modern HTML" -->
<html lang="en">
  <head>
    <!-- head = information ABOUT the page (not visible content) -->
    <meta charset="UTF-8" />
    <title>My First Page</title> <!-- shows in the browser tab -->
  </head>
  <body>
    <!-- body = everything the visitor actually sees -->
    <h1>Hello, web!</h1>
    <p>This page lives on my own computer.</p>
  </body>
</html>
```

### Example 2: Proving the request/response cycle to yourself

1. Open any website, open devtools → **Network** tab, then refresh.
2. The first row is the HTML document. Click it: you can see the **status code** (200), **response headers**, and the raw HTML in the Response sub-tab.
3. Watch the rest of the list fill with CSS, images, fonts — each one a separate HTTP request, exactly as described above.

### Example 3: Live-editing a real site

1. Go to any news site. Open devtools → **Elements**.
2. Click the "select element" arrow (top-left of devtools, or `Ctrl+Shift+C`) and click a headline.
3. Double-click the headline text in the Elements panel and change it. The page updates instantly.
4. Refresh — your edit disappears. Lesson: devtools edits the browser's *copy*, never the server.

### Example 4: A page that links a CSS file (preview of things to come)

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Styled Page</title>
    <!-- rel="stylesheet" tells the browser this file is CSS -->
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <h1>Now with style</h1>
  </body>
</html>
```

`styles.css` (same folder):

```css
/* CSS comments look like this */
h1 {
  color: darkblue; /* change the heading color */
}
```

Save both, open `index.html`, and confirm the heading is dark blue. If it isn't, check the Network tab — a red/404 `styles.css` row means the `href` path is wrong.

## Common Pitfalls

1. **Editing in Word/Notepad with rich text.** HTML must be *plain text*. Word inserts invisible formatting that breaks everything. Use VS Code.

2. **Wrong file extension.** `index.html.txt` (Windows hides extensions by default!) will open as text, not a web page. In File Explorer enable *View → File name extensions* so you can see the truth.

3. **Forgetting to save before refreshing.** You refresh, nothing changed, you assume your code is wrong — but the file was never saved. The dot on the VS Code tab means unsaved changes.

4. **Broken relative paths.**
   ```html
   <!-- ❌ Wrong: styles.css is in a css/ subfolder, but the path says otherwise -->
   <link rel="stylesheet" href="styles.css" />

   <!-- ✅ Correct -->
   <link rel="stylesheet" href="css/styles.css" />
   ```
   The Network tab's 404s tell you exactly which paths are wrong.

5. **Expecting devtools edits to persist.** Devtools changes are a scratchpad. Real changes go in your files, in your editor.

6. **Case-sensitivity surprises.** Windows treats `Photo.JPG` and `photo.jpg` as the same file; most web servers do not. Lowercase everything (`photo.jpg`, `about.html`) and use hyphens instead of spaces (`my-page.html`, never `my page.html`).

## Practice Exercises

1. **Anatomy of a URL.** Write out the scheme, domain, path, query string, and fragment for: `https://shop.example.co.uk/products/shoes?color=red&size=9#reviews`. Then invent your own URL containing all five parts.

2. **First page from scratch.** Without copying from this chapter, create a folder `exercise-01/`, add an `index.html` with a proper doctype, `head`, `title`, one heading, and two paragraphs about why you're learning web development. Open it in your browser.

3. **Network detective.** Open a website you use daily, then devtools → Network → refresh. Answer: How many total requests were made? What's the largest file? Find one request with a status other than 200.

4. **Live-edit safari.** Using Elements in devtools, change three things on a real website: some text, one color (find the `color` property in the Styles pane and click its value), and hide one element (uncheck a `display` declaration or press `H` with the element selected). Screenshot your masterpiece, then refresh to undo it.

5. **Break it on purpose.** In your Exercise 2 page, link a stylesheet with a deliberately wrong filename. Open the page, find the failing request in the Network tab, note the status code, then fix the path and confirm the fix.
