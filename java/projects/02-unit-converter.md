# Project 2: Unit Converter

## Description

A menu-driven console tool that converts between units in three categories: temperature (Celsius/Fahrenheit/Kelvin), length (meters/feet/miles/kilometers), and weight (kilograms/pounds/ounces). The user picks a category, picks a conversion, enters a value, and gets a formatted result. The emphasis is on **clean method decomposition**: every conversion is its own small, testable method — no math buried inside `main`.

## Difficulty

**Beginner** — estimated effort: 3–5 hours.

## Chapters used

- 02 (Variables & Types) — doubles, parsing input, casting
- 03 (Operators & Control Flow) — switch for menus
- 04 (Loops) — menu loop, input validation
- 05 (Methods & static) — one method per conversion, method overloading, helper methods
- 06 (Strings & Text) — `printf` formatting, building the output messages

## Requirements checklist

- [ ] Main menu offers: 1) Temperature 2) Length 3) Weight 4) Quit — repeated until quit
- [ ] Each category has its own submenu listing its conversions (e.g., "C → F", "F → C", "C → K"...)
- [ ] Every conversion is a dedicated `static` method taking a `double` and returning a `double` (e.g., `celsiusToFahrenheit(double c)`)
- [ ] Results print with sensible precision via `printf` (e.g., `%.2f`) and include both units in the output line ("100.00 °C = 212.00 °F")
- [ ] Invalid menu choices print a message and re-show the menu (no crash, no silent exit)
- [ ] Non-numeric value input is handled gracefully (re-prompt)
- [ ] Physically impossible inputs are rejected where they exist (temperature below absolute zero; negative length/weight)
- [ ] At least one method is reused by another (e.g., C → K built from C → F's structure, or a shared `round2(double)` helper)
- [ ] Quitting prints a summary: how many conversions were performed this session

## Hints

- A modern switch expression (Chapter 3) maps menu numbers to actions cleanly; `default` handles junk.
- Design the method *names* first, as a list on paper — `metersToFeet`, `poundsToKilograms`, ... — then implement them one at a time and test each with a known value (1 m = 3.28084 ft, 0 °C = 32 °F, 1 kg ≈ 2.20462 lb) before wiring the menu.
- Chains save work: if you have A→B and B→C, then A→C is one line calling both.
- Keep `main` short: it should read like a table of contents (show menu → read choice → dispatch to a `runTemperatureMenu()`-style method).
- Watch integer division: `9 / 5` is 2. Write `9.0 / 5.0`.

## Stretch goals

- Free-form input mode: parse a line like `12.5 kg to lb` using `split` and dispatch from the strings.
- A "convert everything" option that prints a value in *all* units of its category as a formatted table.
- Currency category with hardcoded rates, displaying an honest "rates as of <date>" disclaimer.
- History: store each conversion performed as a formatted string in an array (fixed size, e.g., last 10) and add a "show history" menu item — a preview of why growable collections (Chapter 12) will feel so good.
