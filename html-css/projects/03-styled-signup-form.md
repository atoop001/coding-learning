# Project 3: Styled Sign-Up Form

## Description

Build the sign-up page for an imaginary service (a coding club, a hiking-tour company, a game — your pick): a polished, centered form card on a pleasant page, with grouped fields, working browser validation, and real visual design. Filling it out should feel effortless: clear labels, sensible input types (right mobile keyboards!), obvious required fields, helpful validation messages, and a submit button that looks clickable.

This is your first full HTML + CSS project: the form itself exercises Chapter 5; the look exercises the CSS core (selectors, box model, colors, typography).

## Difficulty & Effort

- **Difficulty:** Beginner–Intermediate
- **Estimated effort:** 4–7 hours

## Chapters Used

- `05-forms-and-inputs.md`
- `06-css-fundamentals.md`
- `07-the-box-model.md`
- `08-colors-typography-backgrounds.md`

## Requirements Checklist

### Form structure & behavior
- [ ] A `<form>` with `method="get"` (so you can verify submissions in the URL) containing at least 8 controls
- [ ] At least 6 different input types used appropriately (e.g., text, email, password, date, tel/number, select, radio, checkbox, textarea)
- [ ] Every control has a properly wired `<label>` (click the label text → control focuses/toggles)
- [ ] Every control has a `name`; a test submission shows every expected `name=value` pair in the URL
- [ ] A radio group (single shared `name`) and a checkbox group, each inside a `<fieldset>` with a `<legend>`
- [ ] Validation: at least one `required`, one `minlength`/`maxlength`, one `min`/`max`, and one `pattern` with a helpful `title`
- [ ] Placeholders (if used) show *examples*, never replace labels
- [ ] A terms-agreement checkbox that is `required`
- [ ] Submit button is a `<button type="submit">` with clear text

### Styling
- [ ] External stylesheet; the universal `box-sizing: border-box` reset; no inline styles
- [ ] Form presented as a centered card: `max-width` + `margin: auto`, padding, `border-radius`, subtle `box-shadow`
- [ ] A deliberate color palette (recommend HSL-derived): page background, card background, brand color, text color — all body-text pairs pass 4.5:1 contrast
- [ ] Typography: chosen font stack (system or one web font), unitless line-height, consistent sizes in `rem`
- [ ] Inputs styled consistently: padding, border, `border-radius`, full width within the card
- [ ] Visible `:focus` / `:focus-visible` styles on every control (never remove outlines without replacement)
- [ ] `:hover` and `:active` states on the submit button
- [ ] `:invalid` or `:user-invalid` styling that doesn't rely on color alone (e.g., border + message styling)
- [ ] Consistent vertical rhythm between fields (margins — check gaps in devtools)
- [ ] Fieldsets/legends restyled from the ugly defaults (border, padding, legend typography)

## Hints

- Style the *skeleton first*: get all fields present and wired before touching CSS. Two half-done layers is misery; one done layer at a time is smooth.
- Default `fieldset` borders look dated — you can restyle or remove the border while *keeping* the fieldset/legend for accessibility.
- The "label above input" pattern is easiest: make labels `display: block` with a small bottom margin.
- If your inputs overflow the card when set to `width: 100%`, revisit `box-sizing` — that's the classic Chapter 7 pitfall.
- Test the tab order by keyboard: tab through the entire form and watch your focus styles. `:focus-visible` (Chapter 6, pseudo-classes) is the better choice here — it skips the ring on mouse clicks and shows it for keyboard users.
- Try submitting every invalid state on purpose; adjust `title` attributes until every browser message would make sense to your grandmother. Use `:user-invalid` (Chapter 6, pseudo-classes) instead of `:invalid` if you don't want required fields turning red before the user has touched them.

## Stretch Goals

- [ ] A visual "required" convention (e.g., asterisk with a legend explaining it) implemented with `::after` generated content
- [ ] Custom-styled checkboxes/radios using `accent-color` (one line!) — or go further with the label-based technique
- [ ] A gradient or subtle patterned page background (pure CSS, no image file)
- [ ] A confirmation page (`thanks.html`) the form's `action` points to
- [ ] Dark-mode variant via `prefers-color-scheme` swapping only your color custom properties
