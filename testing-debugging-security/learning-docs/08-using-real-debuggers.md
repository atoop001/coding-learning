# Chapter 8: Using Real Debuggers — VS Code & Browser DevTools

## Overview

A debugger lets you freeze your program mid-run and look around: every variable, the full call stack, the exact path execution took. Once you can drive one comfortably, whole categories of "add a print, re-run, add another print, re-run…" sessions collapse into a single inspection. This chapter is hands-on: setting breakpoints in VS Code for both Python and JavaScript (Node), stepping through code, using watch expressions and conditional breakpoints, and debugging front-end code in the browser's DevTools. Everything is Windows-friendly — VS Code and Chrome/Edge are all you need.

## Definitions & Explanations

**Breakpoint** — a marker on a line telling the debugger "pause before executing this line." Set one in VS Code by clicking in the gutter (left of the line number); a red dot appears. When execution reaches it, the program freezes and the editor shows the live state.

**Stepping** — moving through the frozen program one piece at a time:
- **Step Over (F10)** — run the current line; if it calls a function, run that whole function without going inside. Your default gear.
- **Step Into (F11)** — enter the function being called on this line. Use when the bug might be *inside* that call.
- **Step Out (Shift+F11)** — finish the current function and pause at its caller. Use when you've stepped in too deep.
- **Continue (F5)** — resume full speed until the next breakpoint (or the end).

**Call stack panel** — the live version of a stack trace: the chain of function calls that got you here. Clicking a frame jumps to that function *with its local variables* — you can inspect the caller's state without stepping backward.

**Variables panel & hover** — when paused, the Variables panel shows every local; hovering any identifier in the source shows its current value. This replaces a dozen prints.

**Watch expression** — an expression you type once (e.g. `len(rows)`, `cart.items.length`, `user["email"]`) that the debugger re-evaluates and displays at every pause. Perfect for tracking one value across many steps.

**Conditional breakpoint** — a breakpoint with an attached condition; it only pauses when the condition is true. Right-click the red dot → *Edit Breakpoint* → e.g. `order_id == 4177`. Essential inside loops: "pause on iteration 900" beats pressing F5 nine hundred times. A **hit count** variant ("break after 900 hits") and **logpoints** (print a message *without* pausing or editing code) live in the same menu.

**Debug console (REPL)** — while paused, you can *run code* in the program's current context: call functions, inspect `dict(user)`, even mutate values to test "what if it were 5?" — all without editing source.

**launch.json** — VS Code's per-project debug configuration file (`.vscode/launch.json`). For simple scripts you rarely need it (`F5` and pick the language), but it matters for Flask apps, apps needing env vars, or attaching to running processes.

**Browser DevTools debugger** — the Sources panel (F12 in Chrome/Edge) is a full debugger for front-end JS: breakpoints in your scripts, plus browser-specific superpowers — pause on any *DOM change*, on any *event listener* (e.g. every click), or on any *XHR/fetch* request.

## Code Examples

### Debugging Python in VS Code, end to end

Bug hunt target — this has a real bug:

```python
# stats.py — find the median. Contains a bug; we'll catch it live.
def median(values):
    ordered = values.sort()          # BUG lurks on this line
    mid = len(values) // 2
    if len(values) % 2 == 1:
        return ordered[mid]
    return (ordered[mid - 1] + ordered[mid]) / 2

print(median([3, 1, 2]))
```

Session walkthrough:

1. Open `stats.py` in VS Code (with the Python extension installed).
2. Click the gutter next to `mid = len(values) // 2` — red dot appears.
3. Press **F5** → choose **Python File**. The program runs and freezes at your breakpoint.
4. Look at the **Variables** panel: `ordered` is `None`. There's the bug, caught without a single print: Python's `list.sort()` sorts *in place* and returns `None`. (Fix: `ordered = sorted(values)`.)
5. Before fixing, try the **Debug Console**: type `sorted(values)` — it evaluates against live state and shows `[1, 2, 3]`, confirming the fix will work.

### A conditional breakpoint earning its keep

```python
# invoice.py — one of 5,000 orders computes a wrong total. Which one?
def invoice_total(orders):
    total = 0
    for order in orders:
        total += order["price"] * order["qty"]   # <- breakpoint HERE
    return total
```

Right-click the breakpoint → *Edit Breakpoint…* → Expression:

```
order["price"] * order["qty"] < 0
```

Press F5: execution flies through thousands of good orders and freezes exactly on the pathological one. Inspect `order` in the Variables panel — perhaps `qty` is `-3` from a refund record you forgot existed.

### Debugging Node.js in VS Code

```javascript
// budget.js — run under the debugger, no config needed
function remaining(budget, expenses) {
  let left = budget;
  for (const e of expenses) {
    left -= e.amount;               // set a breakpoint here
  }
  return left;
}

console.log(remaining(100, [{ amount: 30 }, { amount: "25" }, { amount: 10 }]));
// prints 35? 3525? NaN? Step through and watch `left` change type.
```

1. Breakpoint on the `left -= e.amount;` line.
2. **F5** → **Node.js**. Add a **watch expression**: `typeof left`.
3. Press **F5** (continue) repeatedly: watch `typeof left` flip from `"number"` to `"string"`→ no — it becomes `"number"` → `NaN`? Actually observe it: `100 - 30 = 70`, then `70 - "25"` — JS coerces to `45` here, but `+` would have concatenated. The debugger shows you what coercion *actually* did rather than what you guessed.

Also useful: the **JavaScript Debug Terminal** (Terminal panel → `+` dropdown). Any node/npm command run in it — including `npm test` — automatically attaches the debugger, so you can put breakpoints *inside failing Vitest tests* and inspect the moment an assertion is about to fail.

### Debugging in the browser (DevTools)

```html
<!-- counter.html — open in Chrome/Edge, press F12 -> Sources -->
<button id="inc">+1</button> <span id="count">0</span>
<script>
  let count = 0;
  document.getElementById("inc").addEventListener("click", () => {
    count += 1;
    document.getElementById("count").textContent = count;
  });
</script>
```

Techniques to practice, in order:

1. **Sources panel breakpoint**: find your script, click the line number on `count += 1;`, click the button on the page — DevTools freezes with the page mid-click. Hover `count` for its value; use the same step buttons as VS Code.
2. **`debugger;` statement**: add the keyword `debugger;` on any line of your JS; DevTools (when open) treats it as a breakpoint. Great for code that's hard to find in the Sources tree. Remove before committing.
3. **Event Listener Breakpoints** (right sidebar): tick *Mouse → click* — the debugger now pauses on *every* click handler, yours or a library's. Perfect for "what code runs when I click this?"
4. **Fetch/XHR breakpoints**: pause whenever the page makes a request matching a URL fragment — ideal for "who keeps calling this API?"
5. **Watch + Scope panes** mirror VS Code's panels; the **Console** while paused evaluates in the paused scope, exactly like the Debug Console.

### Debugging a Flask app (launch.json)

```json
// .vscode/launch.json — lets breakpoints work inside route handlers
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Flask app",
      "type": "debugpy",
      "request": "launch",
      "module": "flask",
      "args": ["--app", "app", "run", "--no-reload"],
      "env": { "FLASK_DEBUG": "0" },
      "jinja": true
    }
  ]
}
```

Note `--no-reload`: Flask's auto-reloader runs your app in a child process, which silently detaches breakpoints — the classic "my breakpoints don't hit" cause. Start with F5, set a breakpoint inside a route, hit the URL in your browser, and VS Code freezes at the moment of the request with `request` fully inspectable.

## Common Pitfalls

- **Stepping line-by-line from the program's start.** You'll fall asleep before the bug. Correction: put the breakpoint *near the suspected crime scene* (use the stack trace or a bisect probe to pick it), then step.
- **Using Step Into everywhere** and ending up ten frames deep inside library code. Correction: default to Step Over; Step Into only for your own suspicious calls; Step Out when lost. "Just My Code" settings hide library internals.
- **Breakpoints that never hit.** Usual causes: debugging a different file than the one running, a reloader/child process (Flask above), or the code path genuinely not executing. Correction: put an unmissable breakpoint on line 1 to verify attachment, then bisect forward.
- **Forgetting `debugger;` statements or logpoints in committed code.** They halt production JS with DevTools open. Correction: grep for `debugger` before committing (many linters flag it — enable that rule).
- **Watching complex expressions with side effects** (e.g. a watch on `queue.pop()`) — the debugger re-evaluates watches constantly and will *mutate your program's state while you watch it*. Correction: watch pure expressions only.
- **Treating the debugger as a substitute for tests.** You've found the bug interactively — great — but nothing prevents its return. Correction: every debugger discovery ends with a regression test (Chapter 9).

## Practice Exercises

1. Reproduce the `median` session above exactly: breakpoint, F5, identify `None` in the Variables panel, verify the fix in the Debug Console *before* editing the file. Then fix it and add two pytest tests (odd- and even-length lists).
2. Generate a list of 1,000 fake orders in Python where exactly one has a negative quantity at a random position. Using only ONE conditional breakpoint (no prints), find it and report its index.
3. In the JavaScript Debug Terminal, run a Vitest suite with a deliberately failing test. Set a breakpoint on the failing assertion's line, inspect the actual value at the pause, and write down how it differs from the expected value.
4. Build a small HTML page with three buttons whose handlers live in one shared function. Using *only* Event Listener Breakpoints (no reading the code first), determine which lines run for each button.
5. Set up `launch.json` for one of your Flask apps, put a breakpoint inside a POST route, submit a real form from the browser, and — while paused — use the Debug Console to inspect `request.form` and `request.headers`. Write three facts you learned about the request that you didn't know before.
