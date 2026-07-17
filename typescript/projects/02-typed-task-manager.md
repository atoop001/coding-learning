# Project 2: Typed Task Manager (Console)

## Description

Build a console-based task manager — a to-do system with priorities, due dates, and a lifecycle — where the *type design* is the real deliverable. Tasks move through states (`todo → in-progress → done`, or `→ cancelled`), and each state carries different data: only in-progress tasks have a `startedAt`, only done tasks have a `completedAt`, only cancelled tasks have a `reason`. You will model this as a discriminated union so that invalid combinations (a done task without a completion time) *cannot compile*.

Using it should feel like a tiny, bulletproof library: an `addTask`/`startTask`/`completeTask`/`cancelTask`/`listTasks` API where the compiler refuses illegal transitions at the call site, and a formatted console report that renders each state differently.

No persistence and no UI required — in-memory arrays and `console.log` output are the point. The program runs as a scripted demo: create tasks, transition them, print reports.

## Difficulty & Effort

- **Difficulty:** Beginner-plus
- **Estimated effort:** 4–6 hours

## Chapters Used

- `02-basic-types-and-annotations.md`
- `03-interfaces-and-type-aliases.md`
- `04-functions-in-typescript.md`
- `05-unions-intersections-literal-types.md` (core of the project)
- `06-narrowing-and-type-guards.md` (core of the project)

## Requirements Checklist

### Type model
- [ ] `Priority` is a literal union (`"low" | "medium" | "high"`), not `string`
- [ ] `Task` is a discriminated union over a `status` discriminant with members: `todo`, `in-progress`, `done`, `cancelled`
- [ ] Shared fields (`id`, `title`, `priority`, `createdAt`, optional `dueDate`) are factored into a base type composed via intersection or interface extension — written once, not four times
- [ ] State-specific fields exist ONLY on their member: `startedAt` (in-progress and done), `completedAt` (done), `reason: string` (cancelled)
- [ ] A `Result`-style return type (`{ ok: true; ... } | { ok: false; error: ... }`) is used for operations that can fail (e.g., completing a task that isn't in progress); errors are a literal union, not `string`

### Operations
- [ ] `addTask(title, priority, dueDate?)` creates a `todo` task with a generated id
- [ ] `startTask(id)`, `completeTask(id)`, `cancelTask(id, reason)` enforce legal transitions and return the Result type on failure (wrong current state, unknown id)
- [ ] `listTasks(filter?)` supports filtering by status and by priority
- [ ] `overdueTasks(now)` returns tasks past their `dueDate` that are not done/cancelled — handling the optional `dueDate` without `!`

### Narrowing discipline
- [ ] A `formatTask(task: Task): string` function uses a `switch` on the discriminant, rendering different info per state
- [ ] The `switch` includes the `never`-based exhaustiveness check in `default`
- [ ] Prove the safety net: in a comment, describe what error appeared when you temporarily added a fifth status member without updating the switch
- [ ] At least one custom type guard exists (e.g., `isOverdue(task): task is Task & { dueDate: Date }` or a guard for active tasks) and is used with `.filter`
- [ ] Zero `any`, zero `as` (except `as const` if wanted), zero `!`

### Demo script
- [ ] `src/demo.ts` runs a full scenario: create 5+ tasks, perform valid transitions, attempt 2+ invalid transitions (handling their Result errors in output), print a grouped report by status
- [ ] The report renders differently per state (e.g., done tasks show duration from `startedAt` to `completedAt`)
- [ ] A comment block quotes at least two compile errors you provoked deliberately (e.g., reading `completedAt` on an un-narrowed `Task`)

## Hints

- Design the union FIRST, on paper. If you find yourself writing `status?: string` or `completedAt?: Date` on one big interface, stop — that's the anti-pattern this project exists to break.
- Transitions produce *new* task objects with a different member type — spread the shared fields and add the state-specific ones. Trying to mutate `task.status` in place will (correctly) fight the type system.
- For failed lookups, decide once: is "not found" a `Result` error or `undefined` from a helper? Consistency matters more than the choice.
- If `filter` in `listTasks` gets awkward, a small options object with optional properties is cleaner than multiple optional positional parameters.
- `Task & { dueDate: Date }` in a guard's predicate is an intersection doing real work — Chapter 5's composition applied to Chapter 6's guards.

## Stretch Goals

- [ ] Add `blocked` status carrying `blockedBy: string` (a task id) — and enjoy the exhaustiveness check pointing at every switch you must update
- [ ] Add a `TaskEvent` discriminated union (`created`/`started`/`completed`/`cancelled`) and an append-only event log; derive a task's current state by replaying events
- [ ] Sort the report by a `Priority`-to-number mapping typed as a `Record` (previews Chapter 9)
- [ ] Persist tasks to a JSON file and reload them — discovering firsthand that `Date` doesn't survive JSON and revived data needs re-validation (previews Chapter 12)
