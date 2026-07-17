# Chapter 13: Transitions, Transforms & Animations

## Overview

Motion makes interfaces feel alive and communicates state: a button that lifts on hover invites clicking; a panel that slides in shows where it came from; a shake says "invalid." CSS gives you three motion tools:

- **Transforms** — move, scale, rotate, skew elements (without affecting layout).
- **Transitions** — animate smoothly *between* two states (hover, focus, class change).
- **Animations** (`@keyframes`) — multi-step, self-running choreography: spinners, pulses, entrance effects.

You'll also learn what makes motion *performant* (animate transform and opacity, not layout properties) and *respectful* (`prefers-reduced-motion`). Good motion is felt, not noticed: fast, purposeful, subtle.

## Definitions & Explanations

### Transforms

```css
.box {
  transform: translate(20px, -10px);  /* move right 20, up 10 — layout unaffected */
  transform: translateX(2rem);
  transform: translateY(-50%);        /* % is of the ELEMENT's own size — key fact */
  transform: scale(1.1);              /* 10% bigger; scale(2, 0.5) stretches */
  transform: rotate(45deg);
  transform: skew(10deg);
  /* Multiple transforms: ONE property, space-separated; ORDER MATTERS.
     rotate-then-translate ≠ translate-then-rotate. */
  transform: translateY(-4px) scale(1.05);
}
.box { transform-origin: top left; }  /* pivot point; default is center */
```

Crucial properties of transforms:

1. **They don't affect layout.** Neighbors don't move; the element visually shifts but its "slot" stays. Perfect for motion (nothing reflows); wrong tool for actual layout.
2. **Cheap for the browser.** Transforms and `opacity` can be animated on the GPU compositor without recalculating layout or repainting — the secret to 60fps.
3. A transform creates a stacking context and hijacks `fixed` descendants (Chapter 12 flashbacks).

### Transitions

A transition says: "when this property changes, don't jump — glide."

```css
.button {
  background: #2563eb;
  transform: translateY(0);
  transition: background-color 0.2s ease, transform 0.2s ease;
  /* shorthand: property duration timing-function delay */
}
.button:hover {
  background: #1d4ed8;
  transform: translateY(-2px);
}
```

- Put the `transition` on the **base state**, not the `:hover` — otherwise the exit animation snaps.
- `transition-property`: name specific properties. `all` is tempting but animates things you didn't intend and costs performance.
- **Timing functions**: `ease` (default, fine), `ease-out` (fast start, gentle stop — best for entrances/hover), `ease-in` (for exits), `linear` (mechanical; for spinners), `cubic-bezier(…)` for custom feels.
- **Durations**: UI hover/focus 150–250ms; panels/modals 200–350ms; anything over 500ms feels sluggish for interaction feedback.
- Transitions need *animatable* values: numbers, lengths, colors. You can't transition `display: none` ↔ `block` (it snaps) — the classic workaround is transitioning `opacity` + `visibility`, or `transform`.

### Keyframe animations

For motion not driven by a state change — or with more than two steps:

```css
@keyframes pulse {
  0%   { transform: scale(1);   }
  50%  { transform: scale(1.06); }
  100% { transform: scale(1);   }
}

.badge {
  animation: pulse 2s ease-in-out infinite;
  /* shorthand: name duration timing-function delay iteration-count direction fill-mode */
}
```

The knobs:

- `animation-iteration-count`: a number or `infinite`.
- `animation-direction`: `normal`, `reverse`, `alternate` (ping-pong).
- `animation-delay`: stagger starts (negative values start mid-animation).
- `animation-fill-mode`: what happens outside the run time. `forwards` keeps the final keyframe (essential for one-shot entrance effects); `backwards` applies the first keyframe during the delay; `both` does both.
- `animation-play-state: paused | running` — pause on hover, etc.
- `from`/`to` are aliases for `0%`/`100%`.

Transitions vs animations: **transition** = A→B tied to a state change; **animation** = runs on its own, any number of steps, can loop.

### Performance: the compositor rule

Animating `width`, `height`, `margin`, `top/left`, or `font-size` forces the browser to re-layout (and repaint) every frame — jank on modest hardware. Animating **`transform` and `opacity`** skips straight to compositing.

Translation table:

| Instead of animating… | Animate… |
|---|---|
| `left`/`top` (position) | `transform: translate()` |
| `width`/`height` (grow) | `transform: scale()` |
| `margin` (nudge) | `transform: translate()` |
| visibility fades | `opacity` (+ `visibility`) |

`will-change: transform;` hints the browser to optimize ahead of time — use sparingly, only on elements that actually animate.

### Respecting `prefers-reduced-motion`

Vestibular disorders make large motion physically unpleasant. Users signal this via an OS setting; honoring it is required for accessible sites:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

(A rare legitimate `!important`.) Better still: design reduced alternatives — a fade instead of a slide — rather than removing all feedback.

## Code Examples

### Example 1: The modern button

```css
.button {
  display: inline-block;
  padding: 0.6rem 1.4rem;
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.2);
  transition: transform 0.15s ease-out,
              box-shadow 0.15s ease-out,
              background-color 0.15s ease-out;
}
.button:hover {
  background: #1d4ed8;
  transform: translateY(-2px);                  /* lift… */
  box-shadow: 0 6px 16px rgb(37 99 235 / 0.35); /* …shadow grows with height */
}
.button:active {
  transform: translateY(0) scale(0.98);         /* press down */
  box-shadow: 0 1px 2px rgb(0 0 0 / 0.2);
  transition-duration: 0.05s;                   /* presses feel instant */
}
.button:focus-visible {
  outline: 3px solid #93c5fd;                   /* keyboard users get feedback too */
  outline-offset: 2px;
}
```

### Example 2: Card hover with image zoom

```html
<style>
  .card {
    width: 280px;
    border-radius: 12px;
    overflow: hidden;                 /* clips the zooming image */
    box-shadow: 0 2px 8px rgb(0 0 0 / 0.12);
    transition: transform 0.25s ease-out, box-shadow 0.25s ease-out;
  }
  .card:hover {
    transform: translateY(-6px);
    box-shadow: 0 14px 30px rgb(0 0 0 / 0.18);
  }
  .card img {
    width: 100%; height: 170px; object-fit: cover; display: block;
    transition: transform 0.4s ease-out;
  }
  .card:hover img { transform: scale(1.07); }   /* parent hover drives child motion */
  .card .body { padding: 1rem; }
</style>

<article class="card">
  <img src="https://picsum.photos/560/340" alt="Lakeside cabin at dusk" />
  <div class="body"><h3>Lakeside Cabin</h3><p>$120 / night</p></div>
</article>
```

### Example 3: Loading spinner and skeleton shimmer

```css
/* Spinner: a circle with one colored edge, rotating forever */
@keyframes spin { to { transform: rotate(360deg); } }
.spinner {
  width: 36px; height: 36px;
  border: 4px solid #e5e7eb;
  border-top-color: #2563eb;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;   /* linear: mechanical motion */
}

/* Skeleton: placeholder bars that shimmer while content loads */
@keyframes shimmer {
  from { background-position: -200% 0; }
  to   { background-position:  200% 0; }
}
.skeleton {
  height: 1rem;
  border-radius: 4px;
  background: linear-gradient(90deg, #eee 25%, #ddd 37%, #eee 63%);
  background-size: 200% 100%;
  animation: shimmer 1.4s ease infinite;
}
```

### Example 4: Staggered entrance animation

```html
<style>
  @keyframes rise-in {
    from { opacity: 0; transform: translateY(16px); }
    to   { opacity: 1; transform: translateY(0);    }
  }
  .feature {
    /* 'backwards' applies the from-state during each item's delay,
       so late items don't flash visible before animating */
    animation: rise-in 0.5s ease-out backwards;
  }
  .feature:nth-child(2) { animation-delay: 0.1s; }
  .feature:nth-child(3) { animation-delay: 0.2s; }
  .feature:nth-child(4) { animation-delay: 0.3s; }

  @media (prefers-reduced-motion: reduce) {
    .feature { animation: none; }
  }
</style>

<section class="features">
  <div class="feature">One</div>
  <div class="feature">Two</div>
  <div class="feature">Three</div>
  <div class="feature">Four</div>
</section>
```

### Example 5: Fading a menu in and out (the display problem, solved)

```css
/* You cannot transition display:none → block. Standard workaround: */
.menu {
  opacity: 0;
  visibility: hidden;                /* unlike display:none, it's animatable-adjacent:
                                        hidden elements can't be clicked/focused */
  transform: translateY(-8px);
  transition: opacity 0.2s ease-out,
              transform 0.2s ease-out,
              visibility 0s linear 0.2s;   /* flip visibility AFTER the fade-out */
}
.menu.open {
  opacity: 1;
  visibility: visible;
  transform: translateY(0);
  transition: opacity 0.2s ease-out, transform 0.2s ease-out, visibility 0s;
}
/* (Adding/removing .open needs one line of JavaScript, or :hover/:focus-within
   on a parent for pure-CSS dropdowns.) */
```

### Example 6: Shake for invalid input

```css
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%      { transform: translateX(-6px); }
  40%      { transform: translateX(6px); }
  60%      { transform: translateX(-4px); }
  80%      { transform: translateX(4px); }
}
/* :user-invalid fires only after the user has interacted — kinder than :invalid */
input:user-invalid {
  border-color: crimson;
  animation: shake 0.3s ease-in-out;
}
```

## Common Pitfalls

1. **Transition declared on the hover state.**
   ```css
   /* ❌ smooth in, instant snap out */
   .card:hover { transform: scale(1.05); transition: transform 0.3s; }

   /* ✅ smooth both directions */
   .card       { transition: transform 0.3s; }
   .card:hover { transform: scale(1.05); }
   ```

2. **`transition: all`.** Animates properties you never intended (including layout-triggering ones added later), causes surprise jank, and makes debugging motion a guessing game. List properties explicitly.

3. **Animating layout properties.** `transition: width 0.3s` or keyframing `margin-left` re-layouts every frame. Use `transform: scale()/translate()`; if you truly must animate a box size (accordions), know it's a cost you're choosing (or look up the modern `interpolate-size`/grid-rows techniques).

4. **Second `transform` declaration overwriting the first.**
   ```css
   /* ❌ the translate is LOST on hover — transform is one property */
   .box       { transform: translateY(10px); }
   .box:hover { transform: scale(1.1); }

   /* ✅ restate everything */
   .box:hover { transform: translateY(10px) scale(1.1); }
   ```

5. **One-shot animations snapping back.** The element animates beautifully, then jumps to its pre-animation state — missing `animation-fill-mode: forwards` (or `backwards` for delayed starts; `both` covers both).

6. **Hover-growth shoving neighbors.** Enlarging with `width`/`padding`/`font-size` on hover reflows the whole row (and can cause hover flicker loops at the boundary). `transform: scale()` grows *visually* without moving anyone.

7. **Motion excess.** Everything sliding, bouncing, and pulsing at 800ms reads as a toy. Interaction feedback ≤ 250ms; one accent animation per view; loops reserved for loading states.

8. **Ignoring `prefers-reduced-motion`.** Not optional for professional work — parallax and large slides can cause real nausea. Ship the reduced-motion block from day one.

9. **Content that exists only on `:hover`** — unusable on touch screens and for keyboard users. Pair hover with `:focus-visible`/`:focus-within`, and ensure the content is reachable some other way on touch.

## Practice Exercises

1. **Button feel lab.** Build one button and give it five variants (classes) differing only in transition timing: 100ms ease-out, 250ms ease-out, 500ms ease-in-out, 1s linear, and a custom `cubic-bezier` with overshoot. Hover each and write a one-line comment on how each *feels* and which you'd ship.

2. **Card grid, motion edition.** Take your Chapter 9/10 card grid and add: lift+shadow hover (transform/opacity only), image zoom inside `overflow: hidden`, and visible `:focus-visible` treatment. Verify with devtools' Performance panel (or just your eyes at 6x CPU throttle) that hovering doesn't reflow neighbors.

3. **Pure-CSS loading screen.** Compose three `@keyframes` animations on one page: a spinner (linear, infinite), three dots pulsing with staggered `animation-delay`, and a progress-bar stripe sliding via `background-position` or transform. All must freeze appropriately under `prefers-reduced-motion` (test with devtools emulation).

4. **Entrance choreography.** Build a hero section where, on load: the heading rises+fades in, then the subheading (0.15s later), then two buttons (0.3s later) — using one shared keyframe, per-element delays, and correct `fill-mode` so nothing flashes early or snaps back.

5. **Dropdown done right.** Using Example 5's visibility pattern with `:focus-within` (no JavaScript), build a nav dropdown that fades/slides open when its trigger is hovered *or* keyboard-focused, closes cleanly both ways, and shows instantly (no motion, still functional) under reduced-motion.
