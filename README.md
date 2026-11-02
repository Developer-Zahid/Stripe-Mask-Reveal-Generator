# Stripe Mask Reveal Generator

A browser tool for designing a staggered "stripe" image reveal: the image appears through a set of columns or rows that open one after another. You tune the animation in a live preview, then copy production code in one of three implementations:

| Output | Stack | Best for |
|---|---|---|
| **GSAP + mask** | GSAP, ScrollTrigger, CSS `mask-image` | Most projects. Works in every current browser, any strip count, real per-strip easing. |
| **Pure CSS `clamp()`** | CSS only (plus a tiny IntersectionObserver for "play on enter") | No-library builds. Scrubbing needs CSS scroll-driven animations. |
| **GSAP + Canvas** | GSAP, ScrollTrigger, Canvas 2D | Identical rendering in every browser and per-pixel control. |

Every setting in the panel updates the preview and all three code outputs at the same time.

---

## Features

- **Live preview** that scrubs with the page scroll (single mode) or inside a scrollable box with GSAP markers (multiple mode).
- **One mask, no extra DOM.** The reveal is a single `linear-gradient` mask on the image. There are no cloned slices or wrapper elements per strip.
- **Three code generators** (GSAP + mask, pure CSS, GSAP + canvas) with readable variable names and comments that explain each choice.
- **Copy buttons** for the generated code and for the current live `mask-image` value.
- **Single or multiple instances.** Generate code for one pinned hero image or for every matching image on a page, each with its own trigger.
- **GSAP `clamp()` on start and end** so images already in view on page load start fully hidden.
- **Horizontal or vertical stripes** (columns or rows).
- **Scrub or play on enter**, with a per-strip duration for play mode.
- **2 to 500 strips.**
- **Feathered leading edge** for a soft wipe.
- **Accessible defaults.** Generated code respects `prefers-reduced-motion` by showing the image without animation.
- **Light and dark themes**, and a layout that works on phones.

---

## Settings reference

| Setting | Values | What it does |
|---|---|---|
| **Instances** | Single, pinned / Multiple | Single: one image inside a tall, sticky scroll track. Multiple: every `.reveal` (or `.reveal-canvas`) on the page gets its own ScrollTrigger. |
| **Stripe direction** | Horizontal ⇢ columns / Vertical ⇣ rows | Sets the gradient angle: `90deg` for columns, `180deg` for rows. Nothing else changes, because gradient stop positions are measured along the gradient line. |
| **Trigger** | Scrub with scroll / Play on enter | Scrub ties progress to scroll position (`scrub: 0.6`). Play runs once on enter and reverses when scrolled back above (`toggleActions: "play none none reverse"`). |
| **Start / End** *(multiple only)* | Element edge (top, center, bottom) + viewport % (0–100) | ScrollTrigger positions such as `"top 85%"`: the element's top edge meets a line 85% down the viewport. End is hidden in play mode because playing only needs a start. |
| **Wrap in `clamp()`** *(multiple only)* | On / Off | Wraps positions as `"clamp(top 85%)"`. Keeps them inside the page's scroll range, so an image already past its start line on load starts at 0% instead of part-way revealed. |
| **Duration per strip** *(play only)* | 0.1–5 s | How long each strip takes. The panel shows when the last strip finishes. |
| **Strips** | 2–500 (whole number) | Number of columns or rows. Invalid input is flagged and snapped back into range when you leave the box. |
| **Stagger each** | 0.00–1.00 | Gap between strip starts. In scrub mode it is relative to one strip's duration; in play mode it is seconds. |
| **Stagger from** | Start / Center / Edges / End / Random | Order the strips start in, using GSAP's stagger `from` rules. |
| **Each strip grows from** | Left, Center, Right (Top, Center, Bottom for rows) | Where the visible part sits inside each strip as it opens. |
| **Ease (per strip)** | none, power2.inOut, power3.inOut, expo.out, circ.inOut, back.out(1.6) | Applied to each strip separately, not to the whole timeline. |
| **Feathered leading edge** | 0–100% of a strip | Softens the moving edge. The softness shrinks to zero as each strip completes, so no seams remain. |

---

## How it works

### The shared idea

Each strip owns a slot: `stripStart` to `stripEnd`, as a percentage of the width (columns) or height (rows). Every frame, each strip has a `progress` from 0 to 1. Its visible size is:

```
revealedSize = stripSize × progress
```

"Grows from" only decides where that visible part sits inside the slot:

```
Left / Top     |■■■■■■······|   pinned to stripStart, the far edge moves forward
Center         |···■■■■■■···|   centred, both edges move outward
Right / Bottom |······■■■■■■|   pinned to stripEnd, the near edge moves back
```

### GSAP + mask

- `strips` is an array of plain objects, `{ progress: 0 }`. GSAP can tween any object, so `.to(strips, { progress: 1, stagger: { each, from } })` gives every strip its own delayed, eased progress.
- On each timeline update, `paintMask()` converts the strips into gradient hard stops and writes one `mask-image`. Black areas show the image; transparent areas hide it.
- Two stops at the same position with different colours (`#000 40%, transparent 40%`) create a crisp edge with no blend.

```js
function createMaskReveal(revealElement, scrollTrigger) { /* strips, paintMask, timeline */ }

// Multiple instances
document.querySelectorAll(".reveal").forEach((revealElement) => {
  createMaskReveal(revealElement, {
    trigger: revealElement,
    start: "clamp(top 85%)",
    end: "clamp(center 50%)",
    scrub: 0.6
  });
});
```

### Pure CSS `clamp()`

- One registered property, `@property --progress`, animates from 0 to 1 on a **linear** timeline.
- Each strip derives its own values from it:
  - `--strip-N-time`: `clamp(0, (progress − start) / span, 1)`, the strip's own 0 → 1 slice. The start is computed from the same stagger rules GSAP uses.
  - `--strip-N-eased`: GSAP's ease formula written as CSS math. The `inOut` eases are split into two clamped halves because CSS has no `if`.
  - `--strip-N-size`: the visible size.
- **Single scrub** uses a named `view-timeline` on `.reveal-track` with `animation-range: contain 0% contain 100%`, the same range as ScrollTrigger's `"top top"` to `"bottom bottom"`.
- **Multiple scrub** uses `animation-timeline: view()`. GSAP-style positions are translated into `entry-crossing` offsets; for example, `"center 50%"` becomes `entry-crossing calc(50% + 50vh)`.
- **Play on enter** adds an `.is-revealed` class from a short IntersectionObserver whose `rootMargin` and `threshold` match the start position.

The animation keyframes stay linear on purpose: easing the keyframes would distort the stagger timing itself.

### GSAP + Canvas

- GSAP staggers the same `strips` array. On every update, `draw()`:
  1. fills one black rectangle per strip (`fillBand`), plus a gradient rectangle for the feather (`fillFade`);
  2. switches to `globalCompositeOperation = "source-in"` and draws the image, which then only appears inside the black bands. This is the canvas equivalent of a CSS mask.
- Rectangles are rounded to whole canvas pixels, so neighbouring strips meet without anti-aliased seams.
- The canvas renders at device-pixel resolution (capped at 2×), redraws on resize through a `ResizeObserver`, and scales the image like `object-fit: cover`.

---

## Using the generated code

### Single, pinned (scrub)

```html
<div class="reveal-track">          <!-- tall: e.g. height: 300vh -->
  <div class="reveal-sticky">       <!-- position: sticky; top: 0; height: 100vh -->
    <img class="reveal" src="photo.jpg" alt="…">
  </div>
</div>
```

### Multiple instances

```html
<img class="reveal" src="photo-1.jpg" alt="…">
<img class="reveal" src="photo-2.jpg" alt="…">
```

For the canvas version, replace each image with:

```html
<canvas class="reveal-canvas" data-src="photo.jpg" role="img" aria-label="Describe the photo"></canvas>
```

### In Webflow

1. Add the GSAP and ScrollTrigger scripts in **Page settings → Before `</body>` tag** (not needed for the pure-CSS scrub version).
2. Give the image the `reveal` class (or use a canvas Embed for the canvas version).
3. For single mode, give the wrapper the `reveal-track` class and a tall height, and make the inner wrapper sticky.
4. Paste the generated JS into the same custom code area, after the GSAP scripts. Paste generated CSS into an Embed `<style>` block.

---

## Browser support

| Version | Support |
|---|---|
| GSAP + mask | All current browsers (unprefixed `mask-image`: Chrome 120+, Safari 15.4+, Firefox 53+). |
| GSAP + Canvas | All current browsers. |
| Pure CSS, play on enter | All current browsers. `expo.out` and `circ.inOut` need CSS `pow()` / `sqrt()` (2023+ browsers). |
| Pure CSS, scrub | CSS scroll-driven animations: Chrome and Edge 115+, Safari 26+. Firefox gets the unmasked image through an `@supports` fallback. |

---

## Known limits

- **Pure CSS at high strip counts** produces one block of custom properties per strip, so 500 strips is a very long stylesheet. The GSAP versions stay the same size at any count.
- **GSAP `clamp()` has no CSS equivalent.** In the pure-CSS multiple-instance version, an element already past its start line on load begins part-way revealed.
- **Random order** is fixed when the CSS is generated. The GSAP versions reshuffle on every page load.
- **Overshooting eases** (`back.out`) are clamped at 100%, so a strip never grows past its slot.
- **Canvas trade-off:** the image becomes pixels, so you lose native `<img>` features such as `srcset`, lazy loading and text alternatives (use `role="img"` and `aria-label` instead).
- **Preview images** are generated in the browser, because the tool can't load outside images. In your project, use your own photos.
