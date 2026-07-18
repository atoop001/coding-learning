# Project 1: Interactive Profile Card Page

## Description

Build a personal "developer profile" page as your first React app: a hero card with your info, a skills list, and a set of project cards — all rendered from typed data structures, composed from small reusable components. No interactivity beyond hover styling yet; the goal is mastering project setup, JSX, typed props, composition, lists, and conditional rendering before state enters the picture.

## Difficulty

**Beginner** — estimated effort: 3–5 hours.

## Chapters Used

- Chapter 1: Why React & Project Setup
- Chapter 2: JSX & Components
- Chapter 3: Rendering Lists & Conditional Rendering

## Requirements Checklist

### Setup
- [ ] Create a fresh Vite + React + **TypeScript** project named `profile-card-page`
- [ ] Remove the demo content from the scaffold (`App.tsx`, unused CSS/assets) so you start clean
- [ ] The app runs with `npm run dev` and builds without errors with `npm run build` (type-checks pass)

### Data modeling
- [ ] Define TypeScript interfaces for your data in a separate file (e.g. `src/data.ts`): a `Profile` (name, title, location, bio, avatar URL or emoji, `openToWork: boolean`), a `Skill` (name + level as a union type `'learning' | 'comfortable' | 'strong'`), and a `Project` (id, name, description, tags array, optional repo URL)
- [ ] Create typed arrays/objects of real (or realistic) data — at least 6 skills and 4 projects

### Components & composition
- [ ] At least five distinct components in separate files (e.g. `ProfileHero`, `SkillBadge`, `SkillList`, `ProjectCard`, `ProjectGrid`) — each with an explicit typed props interface
- [ ] `App` reads like an outline: it composes components and passes data down; no markup soup
- [ ] At least one component uses `children` (e.g. a generic `Section` with a title that wraps arbitrary content)
- [ ] At least one prop is a union type that changes rendering (e.g. skill level changes badge color/class)

### Lists & conditional rendering
- [ ] Skills and projects rendered with `.map()` using **stable keys from the data** (justify your key choice in a code comment)
- [ ] "Open to work" badge renders only when `openToWork` is true
- [ ] Project cards show the repo link **only when the URL exists**
- [ ] Each project's tags render as a nested list with correct keys
- [ ] An empty-state message renders if the projects array is empty (test it by emptying the array)

### Polish
- [ ] Basic CSS (plain CSS or CSS Modules) — layout doesn't need to be beautiful, but cards should look like cards
- [ ] No `any` types anywhere; no TypeScript or console errors/warnings (including missing-key warnings)

## Hints

- Start from the data, not the markup: write the interfaces first, then design components to *fit the data*.
- If a component's props interface has more than ~6 fields, you probably want to pass the whole typed object (`project: Project`) — decide deliberately either way.
- For the skill-level styling, a `Record<SkillLevel, string>` mapping level → className is cleaner than chained ternaries.
- `{project.repoUrl && <a ...>}` — remember why `&&` is safe here (it's a string-or-undefined, not a number).
- Preview the "empty projects" state by passing `[]` from `App` — designing empty states now builds the right habit for data fetching later.

## Stretch Goals

- [ ] Add a `variant` prop to `ProjectCard` (`'featured' | 'normal'`) and render the first project as featured (larger, different layout)
- [ ] Add a `<StatsRow>` computing derived numbers from your data (project count, tag count across all projects, strongest-skill count) — computed in render, no state
- [ ] Make the page responsive with CSS grid (cards reflow at narrow widths)
- [ ] Add a second "persona" data object and a hard-coded toggle constant at the top of `App` that switches which profile renders — everything should just work, proving your components are data-driven
