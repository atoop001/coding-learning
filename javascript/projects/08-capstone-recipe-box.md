# Project 8: Capstone — "Recipe Box" Single-Page App

## Description

The capstone: a complete single-page application that exercises every chapter in the track. **Recipe Box** lets a user search a real recipe API, browse results, view full recipe details, save favorites that persist locally, add their *own* recipes through a validated form, and move between "screens" (Search / Favorites / My Recipes / Detail) without any page reloads — including working browser back/forward via the URL hash.

Use **TheMealDB** (`https://www.themealdb.com/api/json/v1/1/search.php?s=<query>`) — free, no key required, browser-friendly. (If you prefer cocktails, TheCocktailDB has an identical shape; any free JSON API you can render as "cards + detail" is acceptable — document your choice in the README.)

This is a portfolio piece: aim for something you'd genuinely show an employer. That means the three fetch states handled everywhere, empty states designed ("No favorites yet — go find something tasty"), no crashes on weird input, and a codebase someone else could navigate. Write a short `README.md` in the project folder covering what it does, how to run it, and the architecture.

## Difficulty & Effort

- **Difficulty:** Advanced
- **Estimated effort:** 15–25 hours (spread it over 1–2 weeks)

## Chapters Used

All of them — explicitly:
`01`–`09` (language core — everywhere), `10-dom-manipulation.md` + `11-events.md` (all rendering/interaction), `12-error-handling.md` (API + form validation), `13-closures-and-higher-order-functions.md` (debounced search, render callbacks), `14-classes-and-prototypes.md` (domain model), `15-asynchronous-javascript.md` + `16-fetch-and-apis.md` (data layer), `17-modules.md` (architecture), `18-modern-js-and-tooling.md` (syntax throughout; optional Vite setup).

## Requirements Checklist

**Architecture & code quality**
- [ ] ES modules with separated layers — suggested: `api.js` (all fetch code), `store.js` (app state + persistence), `recipe.js` (class/model + validation), `views/` or `render.js` (DOM building), `router.js` (hash navigation), `main.js` (wiring). HTML loads only `main.js` (`type="module"`)
- [ ] No module except the render layer touches `document`; no module except `api.js` calls `fetch`; no module except the persistence one touches `localStorage`
- [ ] User-generated and API text rendered with `textContent` (or equivalent) — never raw `innerHTML` interpolation
- [ ] A project `README.md`: description, how to run locally, file-by-file architecture overview, and at least one "hardest bug I fixed" note

**Navigation (SPA behavior)**
- [ ] Four screens — Search, Favorites, My Recipes, Detail — with a persistent nav bar highlighting the active screen
- [ ] Navigation driven by the URL hash (`#/search`, `#/favorites`, `#/recipe/52772`, ...) via the `hashchange` event, so refresh and back/forward both work and a detail link is shareable
- [ ] Unknown hashes route to a not-found message, not a blank page

**Search (API) screen**
- [ ] Search input queries the API with `async/await`; requests **debounced** (~400ms after typing stops — a closure over a timer id) or fired on submit; your choice, stated in the README
- [ ] Loading, error, empty-result ("No recipes for 'xyz'"), and success states all visibly handled
- [ ] Results as cards (image, name, category) — clicking navigates to the detail screen
- [ ] API responses mapped into your own recipe shape in `api.js`; note: TheMealDB returns ingredients as 20 numbered fields (`strIngredient1`...) — transform them into a proper array

**Detail screen**
- [ ] Full recipe: image, name, category/area, ingredient list with measures, instructions, and (for API recipes) fetched by id (`lookup.php?i=`) so deep links work on refresh
- [ ] Favorite/unfavorite toggle, reflected instantly and persistently

**Favorites & My Recipes**
- [ ] Favorites persist in `localStorage` and survive reload; unfavoriting updates every screen consistently
- [ ] "Add recipe" form (name, category, ingredients list with add/remove rows, instructions) with field-level validation that throws/collects errors from the model layer and displays them inline
- [ ] User recipes appear in My Recipes, open in the detail screen, can be favorited and deleted, and persist
- [ ] Both empty states designed (no favorites / no own recipes yet)

**Robustness**
- [ ] Kill your network (DevTools → offline) and click around: every screen shows sensible messages, nothing crashes, previously-persisted data still works
- [ ] Rapid navigation during an in-flight fetch doesn't paint stale results over the wrong screen (guard with a request counter or check the current route before rendering)

## Hints

- **Plan before coding.** Write the state shape first (what's in `store.js`?), list your routes, sketch each screen on paper. One evening of planning saves a week of refactoring.
- Build vertically, not horizontally: get *one* full slice working end-to-end (search → cards → detail) before starting favorites. A thin working app beats four broken screens.
- The router can be small: parse `location.hash` into `{ screen, param }`, then a plain object mapping screen names to render functions. Twenty lines is plenty.
- Debounce skeleton: keep `let timer` in a closure; on each keystroke `clearTimeout(timer)` then `timer = setTimeout(run, 400)`. You wrote `once()` in Chapter 13 — this is its sibling.
- For the stale-response guard: increment a module-level `requestId` before each fetch and capture it; when the response arrives, ignore it if `capturedId !== requestId`.
- Ingredients transform: loop `for (let i = 1; i <= 20; i++)` reading `meal[\`strIngredient${i}\`]`, pushing `{ ingredient, measure }` pairs while the field is non-empty — computed property access from Chapter 8.
- Give user recipes ids that can't collide with API ids (prefix like `"user-" + Date.now()`), and make the detail screen resolve an id from *either* source.
- Commit to git as you go with meaningful messages — the history itself is portfolio evidence.

## Stretch Goals

- **Vite migration:** scaffold with Vite, move the code into `src/`, and deploy the production build (Netlify/GitHub Pages) — a live URL transforms a portfolio.
- **Category browsing:** TheMealDB's category and area endpoints as filter chips on the search screen.
- **"Surprise me":** the random-recipe endpoint behind a dice button.
- **Shopping list:** aggregate ingredients from favorited recipes into a checklist (dedupe with a reduce keyed by ingredient).
- **Edit user recipes:** reuse the add-form prefilled — good practice in making forms bidirectional.
- **Import/export:** download all user data as JSON and re-import it, with validation on import.
