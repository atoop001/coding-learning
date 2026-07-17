# Chapter 5: Forms & Inputs

## Overview

Forms are how the web collects information: logins, searches, checkouts, sign-ups, surveys. Even without any backend server, HTML gives you a rich toolkit — a dozen input types, built-in validation, and accessibility wiring via labels.

Forms are also where sloppy HTML hurts most: an unlabeled input is unusable to a screen-reader user and annoying to everyone (no click-target on the label, no autofill). This chapter teaches forms the professional way from the start.

## Definitions & Explanations

### The `<form>` element

```html
<form action="/subscribe" method="post">
  <!-- controls go here -->
</form>
```

- `action` — the URL that receives the submitted data. (Without a backend, submission just reloads the page — that's fine for practice.)
- `method` — `get` (data appended to the URL; for searches/filters, shareable) or `post` (data in the request body; for anything that changes state or is sensitive).

### `<input>` — one element, many types

`<input>` is a void element whose behavior is set by its `type` attribute:

| `type` | Purpose / behavior |
|---|---|
| `text` | Single-line free text (the default) |
| `email` | Text with email-format validation; mobile shows @ keyboard |
| `password` | Masked characters |
| `number` | Numeric with spinner; `min`, `max`, `step` |
| `tel` | Phone; mobile shows dial pad (no format validation — formats vary worldwide) |
| `url` | Requires a URL format |
| `search` | Like text, styled as search on some platforms |
| `date`, `time`, `datetime-local`, `month`, `week` | Native pickers |
| `checkbox` | On/off toggle; multiple can be checked |
| `radio` | Pick exactly one from a group (grouped by shared `name`) |
| `range` | Slider |
| `color` | Color picker |
| `file` | File chooser; `accept="image/*"`, `multiple` |
| `hidden` | Sent with the form, not shown |
| `submit` / `reset` / `button` | Button variants (prefer the `<button>` element) |

Core attributes shared by most inputs:

- `name` — **the key under which the value is submitted.** No `name` = the field is silently excluded from submission.
- `id` — unique identifier, used to connect the `<label>`.
- `value` — initial/current value (for radios/checkboxes, the value submitted when checked).
- `placeholder` — hint text shown when empty. **Not a substitute for a label** (it disappears on typing and fails contrast guidelines).
- `required`, `minlength`/`maxlength`, `min`/`max`/`step`, `pattern` — validation (below).
- `disabled` — uneditable *and not submitted*; `readonly` — uneditable but still submitted.
- `autocomplete` — helps browsers autofill: `autocomplete="email"`, `"name"`, `"postal-code"`, etc.

### Labels: the most important part

Every control needs a `<label>`. Two wiring styles:

```html
<!-- Explicit (preferred): label's for = input's id -->
<label for="email">Email address</label>
<input type="email" id="email" name="email" />

<!-- Implicit: input nested inside the label -->
<label>
  Email address
  <input type="email" name="email" />
</label>
```

What labels buy you:

1. Screen readers announce the label when the field is focused.
2. Clicking the label focuses/toggles the control (huge for small checkboxes).
3. Browsers autofill more reliably.

### Multi-line text, dropdowns, and buttons

```html
<!-- Multi-line text. NOT void — value goes between the tags. -->
<label for="msg">Message</label>
<textarea id="msg" name="message" rows="5" cols="40"></textarea>

<!-- Dropdown -->
<label for="country">Country</label>
<select id="country" name="country">
  <option value="">-- Choose --</option>
  <option value="ca">Canada</option>
  <option value="us" selected>United States</option>
  <optgroup label="Europe">
    <option value="fr">France</option>
    <option value="de">Germany</option>
  </optgroup>
</select>

<!-- Buttons: type matters! Inside a form, the default type is "submit". -->
<button type="submit">Send</button>
<button type="button">Do something with JavaScript</button>
<button type="reset">Clear the form</button>
```

### Grouping: `<fieldset>` and `<legend>`

Related controls — especially radio groups — belong in a `<fieldset>` with a `<legend>` caption:

```html
<fieldset>
  <legend>Preferred contact method</legend>
  <label><input type="radio" name="contact" value="email" checked /> Email</label>
  <label><input type="radio" name="contact" value="phone" /> Phone</label>
  <label><input type="radio" name="contact" value="none" /> Don't contact me</label>
</fieldset>
```

Screen readers announce the legend with each option ("Preferred contact method, Email, radio button"), which is the only way the options make sense out of visual context.

### Radios vs. checkboxes — the `name` rule

- **Radios sharing the same `name` form one group**: selecting one deselects the others. Different `name`s = independent radios that can't be unchecked — a classic bug.
- **Checkboxes** are independent; give a set of related checkboxes the same `name` (each with a distinct `value`) to submit multiple values.

### Built-in validation

The browser validates on submit and blocks submission with an error bubble:

```html
<input type="email" name="email" required />
<input type="text" name="username" required minlength="3" maxlength="20" />
<input type="number" name="age" min="13" max="120" />
<input type="text" name="zip" pattern="[0-9]{5}" title="Five digits, e.g. 90210" />
```

- `required` — must not be empty.
- `pattern` — a regular expression the whole value must match; `title` supplies the error hint.
- CSS can style validity states: `input:invalid { border-color: red; }` (Chapter 6 covers pseudo-classes).

Built-in validation is a first line of defense and a UX courtesy. Real applications must *also* validate on the server — never trust the client — but that's beyond this track.

## Code Examples

### Example 1: Complete contact form

```html
<form action="/contact" method="post">
  <p>
    <label for="name">Your name</label><br />
    <input type="text" id="name" name="name" autocomplete="name" required />
  </p>

  <p>
    <label for="email">Email</label><br />
    <input type="email" id="email" name="email" autocomplete="email" required
           placeholder="you@example.com" />
  </p>

  <p>
    <label for="topic">Topic</label><br />
    <select id="topic" name="topic">
      <option value="">-- Please choose --</option>
      <option value="support">Support</option>
      <option value="billing">Billing</option>
      <option value="other">Other</option>
    </select>
  </p>

  <p>
    <label for="message">Message</label><br />
    <textarea id="message" name="message" rows="6" required
              minlength="10" maxlength="1000"></textarea>
  </p>

  <p>
    <label>
      <input type="checkbox" name="copy" value="yes" />
      Email me a copy of this message
    </label>
  </p>

  <button type="submit">Send message</button>
</form>
```

### Example 2: Sign-up form with fieldsets and validation

```html
<form action="/signup" method="post">
  <fieldset>
    <legend>Account details</legend>

    <label for="username">Username (3–20 characters, letters/numbers/underscore)</label>
    <input type="text" id="username" name="username" required
           minlength="3" maxlength="20" pattern="[A-Za-z0-9_]+"
           title="Letters, numbers, and underscores only" />

    <label for="pw">Password (at least 8 characters)</label>
    <input type="password" id="pw" name="password" required minlength="8"
           autocomplete="new-password" />

    <label for="birth">Date of birth</label>
    <input type="date" id="birth" name="birthdate" required
           min="1900-01-01" max="2013-12-31" />
  </fieldset>

  <fieldset>
    <legend>Experience level</legend>
    <!-- Same name = one radio group -->
    <label><input type="radio" name="level" value="beginner" checked /> Beginner</label>
    <label><input type="radio" name="level" value="intermediate" /> Intermediate</label>
    <label><input type="radio" name="level" value="advanced" /> Advanced</label>
  </fieldset>

  <fieldset>
    <legend>Interests (choose any)</legend>
    <label><input type="checkbox" name="interests" value="html" /> HTML</label>
    <label><input type="checkbox" name="interests" value="css" /> CSS</label>
    <label><input type="checkbox" name="interests" value="js" /> JavaScript</label>
  </fieldset>

  <p>
    <label>
      <input type="checkbox" name="terms" value="accepted" required />
      I agree to the <a href="terms.html">terms of service</a>
    </label>
  </p>

  <button type="submit">Create account</button>
</form>
```

### Example 3: Watching what a form submits

Use `method="get"` temporarily and the data appears in the URL — a great learning tool:

```html
<form action="" method="get">
  <label for="q">Search</label>
  <input type="search" id="q" name="q" />
  <label><input type="checkbox" name="exact" value="1" /> Exact match</label>
  <button type="submit">Go</button>
</form>
<!-- Submitting "css grid" with the box checked reloads the page as:
     ?q=css+grid&exact=1
     Every name=value pair is visible. Remove an input's name attribute
     and watch it vanish from the URL. -->
```

### Example 4: Nicer specialized inputs

```html
<form>
  <label for="qty">Quantity</label>
  <input type="number" id="qty" name="qty" min="1" max="10" step="1" value="1" />

  <label for="brightness">Brightness: </label>
  <input type="range" id="brightness" name="brightness" min="0" max="100" value="60" />

  <label for="theme">Accent color</label>
  <input type="color" id="theme" name="accent" value="#3366ff" />

  <label for="avatar">Profile photo</label>
  <input type="file" id="avatar" name="avatar" accept="image/png, image/jpeg" />

  <label for="appt">Appointment</label>
  <input type="datetime-local" id="appt" name="appointment" />
</form>
```

## Common Pitfalls

1. **Placeholder instead of label.**
   ```html
   <!-- ❌ label vanishes the moment you type; screen readers may get nothing -->
   <input type="text" name="city" placeholder="City" />

   <!-- ✅ label + optional placeholder as an EXAMPLE, not a name -->
   <label for="city">City</label>
   <input type="text" id="city" name="city" placeholder="e.g. Toronto" />
   ```

2. **Forgetting `name`.** The field looks fine, validates fine — and submits nothing. If a value is missing from your `get`-method URL test, check `name` first.

3. **Radio groups with mismatched `name`s.**
   ```html
   <!-- ❌ three independent radios; user can select all three, deselect none -->
   <input type="radio" name="size-s" value="s" />
   <input type="radio" name="size-m" value="m" />

   <!-- ✅ one group -->
   <input type="radio" name="size" value="s" />
   <input type="radio" name="size" value="m" />
   ```

4. **`for` not matching `id` (or duplicate `id`s).** The label silently fails to connect. Click the label text: if the input doesn't focus, the wiring is broken.

5. **`<button>` inside a form unintentionally submitting.** A button's default type is `submit`. Any decorative/JS button inside a form needs `type="button"` explicitly, or every click submits the form.

6. **Using `disabled` when you meant `readonly`.** Disabled fields aren't submitted at all; readonly fields are. Pre-filled values the user shouldn't edit but you still need sent → `readonly`.

7. **Number inputs for things that aren't numbers.** Phone numbers, ZIP codes, and credit cards have leading zeros and aren't math — use `type="tel"` or `type="text"` with a `pattern`, not `type="number"` (which strips leading zeros and adds a useless spinner).

8. **No `<fieldset>`/`<legend>` around radio groups.** Sighted users see the question above the options; screen-reader users hear only "Email, radio button" with no question. The legend is the fix.

## Practice Exercises

1. **Label audit.** Build a form with five different input types, each properly labeled with the explicit `for`/`id` technique. Verify every label by clicking its text and confirming focus moves to (or toggles) the control.

2. **Pizza order form.** Create a form with: a radio group for size (S/M/L, one preselected), checkboxes for toppings (same `name`, different `value`s), a `number` input for quantity (1–10), a `textarea` for delivery notes, and a select for pickup time. Group logically with fieldsets/legends. Use `method="get"` and confirm the submitted URL contains exactly what you expect.

3. **Validation gauntlet.** Build a registration form where: username is required, 4–12 characters, letters/digits only (`pattern`); email is a required `email` type; password requires `minlength="10"`; age must be 18–99. Try to submit each invalid case and observe every browser error message.

4. **Survey page.** Design a 6-question survey mixing at least: one `range` with visible min/max labels, one `date`, one radio group, one checkbox group, one `select` with an `<optgroup>`, and one long-answer `textarea`. Every question labeled, groups in fieldsets.

5. **Break-and-fix.** Take your pizza form and deliberately introduce three bugs: remove one `name`, mismatch one `for`/`id`, and change one radio's `name`. Confirm each symptom in the browser (missing URL param, dead label, un-deselectable radio), then fix all three.
