# Project 1: Anatomy of a Page Load

## Description

Pick one real website and document the complete story of loading its homepage — every phase, with evidence. Using nothing but the browser DevTools Network tab (plus a couple of supporting commands), you'll produce a written "load report": what was requested, in what order, from where, why, and how the page went from empty tab to rendered pixels. This is the observational foundation for the whole track: you're not building anything yet — you're learning to *see* what has been happening in front of you all along.

Choose a medium-complexity site: a news site, a university homepage, or a documentation site. (Avoid the extremes: `example.com` is too empty; a social feed is overwhelming.)

## Difficulty

**Beginner.** Estimated effort: 2–4 hours.

## Chapters Used

- Chapter 1 (the big picture, URL anatomy, the journey)
- Chapter 2 (IPs, ports, latency)
- Chapter 4 (HTTP requests, responses, status codes, headers)
- Chapter 6 (resource discovery, waterfall, caching — first pass)

## Requirements Checklist

Produce a single markdown report (`page-load-report.md`) in a folder for this project, containing:

- [ ] The site chosen, the exact URL loaded, date, and browser used
- [ ] The URL dissected into its parts (scheme, host, port — including the implicit one, path)
- [ ] The site's IP address(es), found with a command-line tool, and the round-trip time to it
- [ ] Headline numbers from a cold load (cache disabled, hard refresh): total requests, total transferred size, total load/finish time
- [ ] A breakdown of requests by type (document / CSS / JS / images / fonts / other) with counts — DevTools' type filters do the counting for you
- [ ] A close reading of the **first request** (the HTML document): method, status code, and at least five request headers and five response headers, each with a one-line plain-English explanation
- [ ] The **waterfall narrated**: identify when the document finished, when the first burst of sub-resources began, and explain why the sub-resources could not start earlier
- [ ] At least one request each with a non-200 story: a redirect (3xx) or a cached/304 response — with an explanation of what happened (load a second time with cache enabled if needed)
- [ ] The three largest resources and the three slowest resources, and whether they overlap
- [ ] A comparison table: cold load (cache disabled) vs warm load (cache enabled, revisit) — requests, bytes, time — with two sentences interpreting the difference
- [ ] A closing "journey diagram" (ASCII is fine): the steps from Enter-press to rendered page *for this specific site*, annotated with real numbers you measured

## Hints

- The Network tab's status bar (bottom edge) shows total requests, transferred bytes, and finish time — no manual adding required.
- "Disable cache" only works while DevTools is open. For the warm-load comparison, uncheck it and *navigate* to the site again rather than hard-refreshing.
- Hovering a waterfall bar reveals the phase breakdown (DNS, connect, TLS, waiting, download) — gold for your narration.
- If a header baffles you, MDN's HTTP headers reference explains every one; quoting it in your own words is fine and encouraged.
- Right-click the column headers to add columns like Domain and Protocol — they make several checklist items easier.
- `ping` gives you RTT; remember from Chapter 2 what floor that number sets for every single request.

## Stretch Goals

- Repeat the headline-number capture with network throttling set to "Slow 4G" and add a third column to your comparison table.
- Add the Domain column and count how many distinct hosts the page pulls from; classify each as first-party, CDN, or third-party.
- Use an incognito window vs your normal window and diff the request *cookies* on the document request — what does your everyday browser reveal that incognito doesn't?
- Export the load as a HAR file (right-click in the request list → "Save all as HAR") and open it in a text editor — find your document request inside the JSON and confirm the headers match what DevTools showed.
