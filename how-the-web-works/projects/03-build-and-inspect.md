# Project 3: Build and Inspect — Your Site Under the Microscope

## Description

Time to sit on both sides of the wire at once. You'll build a small multi-page static site (using your HTML/CSS/JS track skills), serve it locally with Python, and then subject it to the *same* forensic inspection you gave real sites in Projects 1–2 — except this time you control the server, so every request in the browser has a matching line in your server's log, and every mystery has an answer you can verify. You'll deliberately cause failures (404s, wrong paths, port conflicts), watch caching happen to your own files, give your site a fake domain via the hosts file, and finish with a written request-by-request reconciliation of client view vs server view.

## Difficulty

**Intermediate.** Estimated effort: 4–6 hours (excluding time spent making the site itself pretty — that's the other tracks' job).

## Chapters Used

- Chapter 1 (client/server roles, being your own server)
- Chapter 2 (ports, localhost, port conflicts, netstat)
- Chapter 3 (hosts file override)
- Chapter 4 (HTTP anatomy, status codes, curl against your own server)
- Chapter 6 (caching, 304s, conditional requests, DOM vs source)
- Chapter 9 (static serving, access logs, MIME types)

> **Look-ahead note:** this project cites Chapter 9 before the suggested cadence gets there. You don't need the whole chapter — just skim the static-serving/MIME-type sections when the checklist below asks for them.

## Requirements Checklist

Build (in a project folder, e.g. `my-inspected-site/`):

- [ ] A static site with at least: `index.html`, one more page (linked both ways), one CSS file, one JavaScript file that visibly does something, and at least two images
- [ ] The site served locally with `python -m http.server` on a port of your choice

Then produce an inspection report (`inspection-report.md`) documenting:

- [ ] **The reconciliation**: load your homepage cold (cache disabled) and list every request the browser made side by side with the corresponding server log line — counts must match, and any browser-initiated surprise (favicon!) must be explained
- [ ] Each resource's **status code and Content-Type** as served, verified two ways: DevTools and `curl.exe -i` — including how the server decided each MIME type
- [ ] **Failure gallery**: at least three provoked errors, each with browser evidence and server-log evidence — a 404 (bad link you temporarily add), a request to a directory without index behavior you can explain, and a connection error (server stopped — what *exactly* does the browser/curl say, and why does no server log line exist?)
- [ ] **Port experiments**: proof (netstat output + the error message) of what happens when a second server claims your port, and your site running simultaneously on two different ports
- [ ] **Caching study**: with cache *enabled*, document the second-load behavior of your files (200 vs 304 vs memory/disk cache), including one hand-made conditional request with curl (`If-Modified-Since` — Python's simple server supports it) that yields a 304
- [ ] **Hosts-file domain**: your site reachable at a made-up domain (e.g. `mysite.test`) with the hosts-file line quoted, plus the observed difference between `ping mysite.test` and `nslookup mysite.test`, explained — and confirmation you cleaned the entry up afterwards
- [ ] **DOM vs source**: one screenshot-or-description pair showing your JS visibly changing the DOM such that Elements differs from view-source, with two sentences on why the served file is untouched
- [ ] A final ASCII diagram of your whole setup: browser, curl, ports, server process, folder on disk — arrows showing who talks to whom

## Hints

- Keep the PowerShell window with the server visible next to the browser: the log lines appearing in real time as you click *is* the project's central experience.
- `python -m http.server` logs every request with method, path, and status — exactly the columns your reconciliation table needs.
- For the conditional-request task: first `curl.exe -i` a file and note its `Last-Modified` header, then replay with `-H "If-Modified-Since: <that exact value>"`.
- The browser requests `/favicon.ico` uninvited; watching a 404 for it in your log is a rite of passage, and shipping a favicon is the cure.
- If `mysite.test` works in ping but not the browser, remember both the OS *and* the browser cache DNS — `ipconfig /flushdns` plus a browser restart.
- Editing the hosts file needs an Administrator editor; and always remove test entries when done (leave a checklist item ticked only when the cleanup is real).

## Stretch Goals

- Replace `python -m http.server` with a ten-line custom server (Chapter 9's `dyn.py` pattern) that serves your static files *and* adds one dynamic route `/api/time` returning JSON — then show your page fetching it with JS.
- Find your machine's LAN IP (`ipconfig`) and load your site from a phone on the same Wi-Fi — document what URL worked, and why `localhost` on the phone could not.
- Add a deliberately huge image, measure its effect on load time throttled to Slow 4G, then optimize it and measure again.
- Write a tiny PowerShell script that curls every page of your site and prints just `URL -> status`, a one-line uptime checker for your one-machine empire.
