# Project 1: Ship a Static Site

## Description

Take something you've already built — an HTML/CSS project from the html-css track, or a built React project — and put it on the public internet *twice*: once on GitHub Pages, and once on Netlify or Vercel (your choice). Add a custom 404 page to both. Then write a short comparison of the two experiences, because the differences between them teach you more about static hosting than either one alone. By the end you'll have two live URLs anyone on Earth can visit, and — more importantly — you'll have crossed the psychological line between "person who builds things locally" and "person who ships." Everything else in this track builds on the muscle you form here.

## Difficulty & Estimated Effort

Beginner — 2–4 hours (more if this is your first time and DNS curiosity strikes; that's allowed).

## Chapters Used

- Chapter 1: From Localhost to the World (core)
- Chapter 12: Domains, DNS & HTTPS (partially — only if you do the custom-domain stretch goal; the platforms' HTTPS is automatic)

## Requirements

- [ ] Choose a project to ship: a plain HTML/CSS site, or a React project with a working `npm run build`. It must be in its own Git repository pushed to GitHub.
- [ ] Before deploying anywhere, identify in writing: does this project have a build step? What exactly is the deployable artifact (which folder)? Confirm build output is gitignored if a build step exists.
- [ ] Run the production build locally (if applicable) and serve the artifact folder with a local static server to prove it works *as built files*, not just under the dev server.
- [ ] Deploy to **GitHub Pages**: enable it for the repo (via the GitHub Actions source or branch source — investigate both, pick one, note why), and get the site live at your `github.io` URL.
- [ ] Deploy the **same project** to **Netlify or Vercel**: connect the GitHub repo, configure the build command and publish directory, and get it live at the platform's generated URL.
- [ ] Verify auto-deploy on the Netlify/Vercel side: push a visible change to the default branch and confirm the live site updates without you touching the dashboard.
- [ ] Determine what happens on the GitHub Pages side for the same push, and document the difference (if any) in how the two platforms picked up the change.
- [ ] Create a custom **404 page** and get it working on *both* hosts. (Each platform has its own convention for how a 404 page is recognized — finding those conventions is part of the exercise.) Verify by visiting a nonsense path on each live site.
- [ ] Confirm HTTPS works on both URLs and that you did nothing to set it up. Note who obtained the certificate.
- [ ] If your project is a React SPA with client-side routing: test a deep link (a route other than `/`) on both hosts, document what happens, and fix it where broken (each platform has a mechanism for SPA fallbacks — find it).
- [ ] Write `COMPARISON.md` in the repo: setup friction, build configuration, deploy speed, 404 handling, and which you'd reach for by default — 200–400 words, honest.
- [ ] Add both live URLs to the top of the repo's README.

## Hints

- Chapter 1's "find the artifact" exercise is literally the first requirement. If you skipped it, do it now — every deployment mistake in this project comes from being fuzzy about what the artifact is.
- GitHub Pages serves projects at `https://<user>.github.io/<repo>/` — note the trailing path segment. If your site's CSS or JS loads on Netlify but 404s on Pages, that subpath is almost certainly why. Vite's `base` option and plain HTML's relative-vs-absolute paths are the threads to pull.
- The platforms' docs answer the 404 question in under a minute each — searching "netlify custom 404" beats guessing. The two conventions are different in an instructive way.
- For the SPA deep-link requirement: think about what a static file server does when asked for `/about` and no file named `about` exists. Chapter 1's static-vs-server distinction predicts the failure exactly.
- If a deploy fails, the platform shows you the build log. Read it bottom-up (the CI habit from Chapter 7 applies early). The error is almost always a wrong build command or wrong publish directory.
- Don't gold-plate the 404 page. A heading, a joke, and a link home is complete.

## Stretch Goals

- Buy a domain (Chapter 12) and attach it to whichever deployment you preferred — apex and www, canonical redirect, verified with `Resolve-DnsName` and curl.
- Add a deploy status badge from Netlify/Vercel to the README next to the URLs.
- Use Lighthouse (in Chrome DevTools) on both live URLs; record the performance scores and investigate any difference between hosts.
- Deploy a *second* project to the same platform in under ten minutes, timed, to prove the skill is now cheap to reuse.
- Explore preview deploys: open a pull request against the repo and find the temporary preview URL Netlify/Vercel created for it — then write two sentences on why teams love this feature.
