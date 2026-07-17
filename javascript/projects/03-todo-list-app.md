# Project 3: To-Do List App (Arrays + Objects + DOM)

## Description

Build the classic to-do list — the rite of passage for every web developer, and for good reason: it forces you to keep *data* (an array of task objects) and the *screen* (the DOM) in sync, which is the core problem of all UI programming.

The user can add tasks, mark them complete (with a visual strikethrough), delete them, and filter the view (All / Active / Completed). A counter shows how many tasks remain. The app should feel snappy and predictable: the list always reflects the data exactly, and adding an empty task is politely refused.

The golden rule for this project: **the array of task objects is the single source of truth.** Every user action modifies the array, then a single `render()` function redraws the list from it. No reading state back out of the DOM.

## Difficulty & Effort

- **Difficulty:** Intermediate−
- **Estimated effort:** 5–8 hours

## Chapters Used

- `05-loops-and-iteration.md` — rendering loops
- `06-functions.md` — organizing actions as functions
- `07-arrays-and-array-methods.md` — `map`, `filter`, `find` are the whole app
- `08-objects.md` — task objects, immutable updates
- `09-strings-and-template-literals.md` — text handling, trimming input
- `10-dom-manipulation.md` — `createElement`, `classList`, data attributes
- `11-events.md` — submit handling and **event delegation**

## Requirements Checklist

- [ ] Tasks are stored as objects with at least `{ id, text, done }` in a single array — the source of truth
- [ ] Each task gets a unique `id` (a counter or `Date.now()` — your choice, but collisions must be impossible within a session)
- [ ] An input + Add button (inside a `<form>`, handled via the `submit` event with `preventDefault`) adds a task; input is trimmed and empty submissions are rejected with visible feedback
- [ ] A single `render()` function clears and redraws the list from the array; *every* action ends by calling it
- [ ] Clicking a task (or its checkbox) toggles its `done` state — completed tasks display with a strikethrough via a CSS class
- [ ] Each task has a delete button that removes it from the array
- [ ] Toggle and delete are implemented with **one delegated listener** on the list container, using `data-id` attributes and `closest()` — no per-item listeners
- [ ] Toggle and delete update the array **immutably** (`map` / `filter` producing new arrays), not with `splice`
- [ ] Filter buttons (All / Active / Completed) change which tasks are rendered without destroying data; the active filter is highlighted
- [ ] A live counter shows "N items left" (only non-completed count), correctly pluralized
- [ ] Task text is inserted with `textContent` (never `innerHTML` for user text) — try adding a task named `<b>hi</b>` to prove it renders literally

## Hints

- State first: get `addTask`, `toggleTask(id)`, `deleteTask(id)` working with `console.log(tasks)` *before* writing any DOM code. If the array is right, the UI is just a drawing of it.
- The chapter 8 pattern `tasks.map(t => t.id === id ? { ...t, done: !t.done } : t)` is exactly your toggle.
- For the filter, keep a `currentFilter` variable (`"all" | "active" | "completed"`); `render()` derives the visible list with `filter()` at draw time. Don't maintain three arrays.
- `data-id` on the `<li>`, then in the delegated handler: `const li = e.target.closest("li"); const id = Number(li.dataset.id);` — remember `dataset` values are strings!
- Distinguish "clicked delete" from "clicked task text" inside one handler by checking `e.target.classList.contains("delete-btn")` (or `closest(".delete-btn")`).
- If clicks stop working after re-rendering, you attached listeners to list items instead of the container — reread Chapter 11's delegation section.

## Stretch Goals

- **Edit in place:** double-click a task to turn it into an input; Enter saves, Escape cancels.
- **Clear completed:** one button removing all done tasks (one `filter` call).
- **Drag-free reordering:** "move up / move down" buttons per task.
- **Persistence preview:** save the array with `localStorage.setItem("tasks", JSON.stringify(tasks))` and load on startup — you'll formalize this in Project 7.
- **Created-at timestamps:** store `createdAt` on each task and offer sorting newest/oldest.
