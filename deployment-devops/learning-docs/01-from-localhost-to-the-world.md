# Chapter 1: From Localhost to the World

## Overview

Every project you've built so far lives at `http://localhost:3000` or gets opened as a file in your browser. It works — for exactly one person, on exactly one machine, while that machine is awake. Deployment is the set of practices that takes your code from "runs on my laptop" to "runs on a computer that's always on, reachable by anyone with the URL." This chapter is the map of the whole territory: what deployment actually means mechanically, why professional teams run multiple copies of their software in different *environments*, the crucial distinction between your source code and the *build* that actually gets served, and the three big families of hosting — static hosting, servers, and serverless. Everything in the next eleven chapters is a zoom-in on something introduced here. If you finish this chapter able to sketch, on paper, what happens between `git push` and a user seeing your page, you're ready for the rest.

## Definitions & Explanations

**Deployment** — the act of putting a specific version of your software onto infrastructure where its intended users can reach it. "Infrastructure" might be a rented Linux machine, a platform like Render, or a global network of file-serving computers. Deployment is a *repeatable process*, not a one-time event: real projects deploy dozens or hundreds of times.

**Localhost** — the hostname that always means "this machine." When your dev server says it's listening on `http://localhost:3000`, it's reachable only from your own computer. Nothing about writing an app makes it public; publicness is a separate, deliberate step. That step is this whole track.

**Server** — an annoyingly overloaded word with two meanings you must keep separate: (1) a *program* that listens for network requests and responds (your Express app is a server), and (2) a *machine* that runs such programs, typically a Linux computer in a data center. Context usually disambiguates, but when it doesn't, ask "program or machine?"

**Environment** — a complete, independent copy of your application plus everything it needs (configuration, database, URL). Professional teams run at least three:

- **Development (dev)** — your laptop. Fast feedback, fake data, debuggers attached, crashes are fine.
- **Staging** — a production-lookalike copy where changes are verified before real users see them. Same code path as production, but pointed at test data.
- **Production (prod)** — the real thing. Real users, real data, real consequences. Changes arrive here last, and only after passing through the earlier environments.

The same code runs in all three; what differs is *configuration* (Chapter 3). A bug that only appears in production and not in dev is almost always a configuration or data difference, not a code difference.

**Build** — the transformation of your human-friendly source code into the optimized form that actually gets served or executed. For a React app, `npm run build` turns JSX and modules into minified static files in a `dist/` or `build/` folder. For Python, there may be no build step at all. Key insight: **you deploy builds (or the means to produce them), not raw source as-is**. The `dist/` folder is disposable output — it's gitignored, and regenerated from source every deploy.

**Artifact** — the concrete output of a build: a folder of static files, a zip, a Docker image. Artifacts are what actually move to the hosting infrastructure. The same artifact should be deployable to staging and production — build once, deploy many times.

**Static hosting** — serving pre-made files (HTML, CSS, JS, images) exactly as they are, to everyone, with no per-request computation. GitHub Pages, Netlify, and Vercel's static tier do this. Enormously cheap and fast because the host just copies bytes. If your project is "files that don't change per user" — a portfolio, docs, a *built* React SPA — static hosting is the answer. Note that a React app can be static even though it *feels* dynamic: the interactivity happens in the user's browser; the server only ever hands out the same files.

**Dynamic hosting / running a server process** — keeping a program of yours running continuously on a machine, computing a fresh response per request. Needed the moment you have per-user data, a database, authentication, or an API. This is where Express and Flask apps live, and it's fundamentally more expensive than static hosting because a computer must be *awake and running your code* at all times.

**Serverless (Functions-as-a-Service)** — a model where you upload individual functions and the platform runs them on demand, billing per invocation, scaling to zero when idle. There absolutely are servers involved — you just don't manage them. Great for sporadic workloads; comes with quirks (cold starts, execution time limits, statelessness) that make it a "learn later" topic. In this track we mention it for map-completeness and mostly set it aside.

**PaaS (Platform as a Service)** — services like Render, Railway, and Fly.io that accept your code (usually straight from GitHub), build it, run it, give it a URL with HTTPS, and restart it if it crashes. Chapter 2 explains why this is the right starting point.

**Hosting provider** — any company renting you the infrastructure above. The spectrum runs from "hands-off, they do everything" (static hosts, PaaS) to "hands-on, you do everything" (raw cloud machines, Chapter 9).

**Uptime** — the fraction of time your deployment is reachable. Your laptop has terrible uptime (it sleeps, reboots, leaves the house). Data centers exist to provide machines with excellent uptime, redundant power, and fat network connections. That, plus a stable public address, is fundamentally what you're paying hosts for.

**CDN (Content Delivery Network)** — a worldwide network of file-caching servers so users load your static assets from a machine physically near them. Netlify, Vercel, and GitHub Pages put your files on a CDN automatically; you get this for free without doing anything.

**Origin** — in CDN vocabulary, the "real" source of truth the CDN caches from: your host's actual storage or your actual server process. When you hear "the CDN shields the origin," it means most requests are answered from cache and never touch the real thing.

**Release** — one specific deployed version of your app, identified by something precise (a version number, a Git commit). Hosts keep a history of releases, which is what makes "roll back to the previous release" possible (Chapter 8). Deploying is the verb; a release is the noun it produces.

**Zero-downtime deployment** — the standard professional expectation: users never see an outage during a deploy. The usual trick is simple to picture: the platform starts the *new* version alongside the old one, waits until the new one answers correctly, then switches traffic over and retires the old. You'll see this in action on every PaaS deploy log — and it explains why a deploy takes a minute even when the build was fast.

**Downtime / maintenance window** — the old-world alternative: take the site down, change it, bring it back. Still legitimate for some database operations, but "we deploy at 2 a.m. Sunday" as a routine is a smell in modern web work. The whole toolchain you're about to learn exists so deploys are safe enough to do at 2 p.m. Tuesday.

## Code Examples

The fastest way to feel the localhost boundary is to poke at it.

```powershell
# Serve the current folder as a static site, locally.
# (npx runs a package without permanently installing it.)
npx serve .

# It prints something like:  http://localhost:3000
# Open that in your browser: works.
# Open it on your phone (not on the same trick, ignore LAN for now): fails.
# That failure is the entire problem deployment solves.
```

A build step, using any React project you have (or `npm create vite@latest` a throwaway one):

```powershell
# From the project root:
npm run build

# Inspect what came out — this folder IS the deployable artifact:
Get-ChildItem dist   # Vite outputs to dist/; Create React App used build/

# Serve the BUILT artifact instead of the dev server:
npx serve dist
```

Compare the two experiences: `npm run dev` runs a Node process that rebuilds on save (a dev-environment luxury); `npx serve dist` just hands out finished files, which is exactly what a static host does. Notice `dist/` contents are minified gibberish — that's fine, it's output, not source. Confirm it's gitignored:

```powershell
# Should print a line containing dist (or build) — if not, add it.
Get-Content .gitignore | Select-String "dist|build"
```

You can also see the environment concept in miniature today. Many Node apps branch on `NODE_ENV`:

```powershell
# PowerShell sets environment variables like this (bash would use: NODE_ENV=production node server.js)
$env:NODE_ENV = "production"
node server.js

# Frameworks read this to disable hot-reload, verbose errors, etc.
# Same code, different behavior — driven purely by configuration.
```

Finally, a taste of what's coming — this is what deploying a static site amounts to on modern hosts (don't run this yet; Project 1 walks through it properly):

```text
1. Push your repo to GitHub.
2. Tell Netlify/Vercel: "watch this repo; build command is `npm run build`; publish the `dist` folder."
3. Every future `git push` triggers: clone → install → build → publish to CDN → live URL.
```

Steps 1–3 are the skeleton of *every* modern deployment pipeline, static or not. The rest of the track fills in the muscle.

You can inspect real deployments from the outside, right now, with tools you already have. Response headers reveal a surprising amount about how a site is hosted:

```powershell
# -I asks for headers only. Note: curl.exe, not curl — in PowerShell, bare `curl`
# is an alias for Invoke-WebRequest, which behaves differently.
curl.exe -I https://react.dev

# Things to look for in the output of various sites:
#   server: cloudflare / Vercel / GitHub.com   <- who's actually serving the bytes
#   x-vercel-cache / cf-cache-status: HIT      <- a CDN answered from cache; the
#                                                 origin never even saw this request
#   age: 8113                                  <- how long this copy has sat in cache
#   content-type: text/html                    <- static file vs generated response
```

Try it against three kinds of site and compare: a documentation site (almost always static + CDN), a web app's API endpoint (dynamic, often `cache-control: no-store`), and a GitHub Pages site (`server: GitHub.com`). You're reading the hosting decisions of real engineering teams off the wire.

The full journey, sketched — this is the diagram exercise 1 asks you to draw, so look away before you do it:

```text
Static site:
  browser -> DNS ("what IP is example.com?") -> CDN edge node near the user
          -> cached files return in milliseconds
          -> (only on cache miss does the request travel to the origin host)

Server-backed app:
  browser -> DNS -> host's front door (reverse proxy, HTTPS)
          -> YOUR running process (Express/Flask) -> maybe a database
          -> a response computed for THIS request travels all the way back

The static path has no "your code runs here" step. That absence is why it's
cheap, fast, and nearly unbreakable — and why you use it whenever you can.
```

One more environment-difference demo worth thirty seconds — frameworks change behavior dramatically based on config alone:

```powershell
# An Express app with default error handling, run two ways:
$env:NODE_ENV = "development"; node server.js
# -> error pages include stack traces (helpful on your laptop)

$env:NODE_ENV = "production"; node server.js
# -> same code, terse error pages (stack traces leak file paths and library
#    versions to strangers — production hides them on purpose)
```

Same source, different environment, different behavior — configuration, not code, is what distinguishes environments. Chapter 3 builds your whole config discipline on this observation.

Finally, the decision you'll make for every project from now on, as a flowchart:

```text
Does any of YOUR code need to run per-request on the host?
│
├─ NO  (portfolio, docs, built React/Vite SPA calling someone else's APIs)
│      -> STATIC HOSTING. GitHub Pages / Netlify / Vercel. Free, fast, done.
│
└─ YES (your own API routes, database, auth, server-rendered pages)
       │
       ├─ Runs continuously, holds connections, "normal web app"?
       │      -> SERVER PROCESS on a PaaS (Render/Railway/Fly). Chapters 2-11.
       │
       └─ Sporadic, tiny, event-shaped tasks?
              -> SERVERLESS is designed for this — file it under "later."

When torn: choose the option higher up this chart. Static beats server beats
serverless for simplicity, and you can always migrate downward when a real
need appears. Most beginner projects that "need a server" actually need a
static frontend plus one small API — two deployments, each dead simple.
```

## Common Pitfalls

1. **Thinking "deploy" means "upload my whole project folder."** What gets served is the *build output* (for static sites) or a *running process built from your code* (for servers) — never your raw folder with `node_modules` and `.env` inside. Correction: identify your project's artifact (`dist/`, or "a running `node server.js`") before thinking about hosting it.

2. **Committing the `dist/`/`build/` folder to Git.** It's generated output; committing it causes merge conflicts on gibberish and stale deployments. Correction: gitignore it and let the host build from source on every deploy.

3. **Assuming a React app needs a "server" because it's interactive.** A built SPA is static files; the interactivity runs in the visitor's browser. Correction: ask "does any code of mine run *per request* on the host?" If no — static hosting. If yes (Express routes, database queries) — you need a server process.

4. **Testing only in dev and being shocked production behaves differently.** Dev servers mask problems: they serve unbuilt code, show friendly errors, and often proxy API calls for you. Correction: before any deploy, run the production build locally (`npm run build` then `npx serve dist`) — a huge class of "works in dev, broken in prod" bugs shows up right there.

5. **Pointing every environment at the same data.** If your "staging" experiments hit the production database, staging isn't safe — it's production with extra steps. Correction: an environment is only real if its configuration *and data* are its own. (Chapter 10 makes this a hard rule.)

6. **Treating deployment as a scary one-time ceremony.** Beginners deploy once at the very end of a project, everything breaks at once, and debugging is miserable. Correction: deploy a walking skeleton on day one, then keep deploying small changes. Frequent small deploys are *safer* than rare big ones — this is the core belief behind Chapters 7–8.

7. **Confusing "it's on GitHub" with "it's deployed."** GitHub stores your source; it doesn't run it (GitHub Pages being the narrow static-files exception). Correction: source hosting and app hosting are different services; deployment is the bridge between them.

## Practice Exercises

1. **Draw the map.** On paper, diagram the journey of a request when someone visits a deployed static site: browser → DNS → CDN/host → files back. Then diagram a request to a deployed Express API: browser → DNS → host machine → your running process → database → response. Annotate which parts you control and which the host controls. Keep both drawings — you'll correct them as the track progresses.

2. **Find the artifact.** For three of your existing projects (ideally one HTML/CSS, one React, one Node or Python), write down: does it have a build step? What exactly is the deployable artifact? Does it need a server process at all times, or is it static? Which hosting family fits?

3. **Production build drill.** Take a React (or any built) project, run its production build, and serve the output with `npx serve dist`. List three differences you can observe versus the dev server (file names, error behavior, network tab in devtools, speed).

4. **Environment spotting.** Pick a real product you use (e.g., a game or app that has a "beta" or "PTR" version). Identify its environments from the outside. Write a paragraph on why the company pays to run a whole second copy of everything.

5. **Break localhost on purpose.** Start any dev server, then find its process and kill it from another PowerShell window (`Get-Process node | Stop-Process` — careful, this kills all Node processes). Refresh the browser. This is what "the server is down" means, and why hosts that auto-restart crashed processes (Chapter 2) earn their keep.

6. **Vocabulary check.** Without looking, define in your own words: environment, build, artifact, static hosting, serverless, PaaS. Then reread the definitions and grade yourself honestly. These six terms appear in nearly every job posting's deployment bullet point.
