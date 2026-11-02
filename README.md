# Stripe Mask Reveal Generator

One **mask-image**, one linear-gradient. GSAP staggers an array of plain numbers, one per strip, and every frame rewrites the gradient's hard stops from them. Scroll to scrub it, and change the settings on the right while you watch.

## Settings

Trigger: `Scrub with scroll`

Strips: `12`

Stagger each: `0.08`

Stagger from: `Start`

Each strip grows from: `Left`

Ease (per strip): `power3.inOut`

Feathered leading edge: `0%`

## GSAP staggers numbers, CSS paints them

### 1. One object per strip

GSAP can tween any object, not only elements. `strips` is an array of `{ progress: 0 }`. Passing the whole array to `tl.to()` with `stagger` gives every strip its own delayed, eased progress from 0 to 1.

### 2. Rewrite the gradient every frame

In the timeline's `onUpdate`, `paintMask()` turns each strip into hard stops: black from `stripStart` to `stripStart + stripWidth × progress`, then transparent to the slot's end. Black shows the image and transparent hides it.

### 3. ScrollTrigger drives the timeline

With `scrub`, scroll position sets the timeline's progress, so reversing the scroll reverses the reveal. Swap it for `toggleActions` to play once when the section enters.

## Why not animate a CSS variable?

The pure-CSS version needs one `clamp()` per strip and a linear timeline. Doing the stagger in GSAP gives you real per-strip easing, `from: "center"` or `"random"` ordering, and any strip count, with the same single mask.

## Code for your current settings

### GSAP + ScrollTrigger Version

Needs GSAP and ScrollTrigger. Works in every current browser.


```js
// <script src="gsap.min.js"> + ScrollTrigger.min.js
gsap.registerPlugin(ScrollTrigger);

const revealElement = document.querySelector(".reveal");
const STRIP_COUNT = 12;

// One plain object per strip. GSAP tweens each "progress" from 0 to 1.
const strips = Array.from({ length: STRIP_COUNT }, () => ({ progress: 0 }));

function paintMask() {
  const stripWidth = 100 / STRIP_COUNT; // in %

  const colorStops = strips.map((strip, index) => {
    const stripStart = index * stripWidth;
    const stripEnd = stripStart + stripWidth;
    const progress = Math.min(1, Math.max(0, strip.progress)); // clamp eases that overshoot
    const revealedWidth = stripWidth * progress;

    // Each strip grows from: LEFT
    // Visible part is pinned to stripStart and its right edge moves right.
    const revealEnd = stripStart + revealedWidth;
    return `#000 ${stripStart}%, #000 ${revealEnd}%, transparent ${revealEnd}%, transparent ${stripEnd}%`;
  });

  const maskGradient = `linear-gradient(90deg, ${colorStops.join(", ")})`;
  revealElement.style.webkitMaskImage = maskGradient;
  revealElement.style.maskImage = maskGradient;
}

gsap.timeline({
  scrollTrigger: {
    trigger: ".reveal-track",
    start: "top top",
    end: "bottom bottom",
    scrub: 0.6
  },
  onUpdate: paintMask
})
.to(strips, {
  progress: 1,
  duration: 1,
  ease: "power3.inOut",
  stagger: { each: 0.08, from: "start" }
});

paintMask(); // start fully hidden

/* CSS */
/* .reveal { mask-size: 100% 100%; mask-repeat: no-repeat; } */
/* .reveal-track { height: 300vh; }  put .reveal inside a position: sticky child */

```

### Pure CSS clamp() Version

No JavaScript. Scrubbing uses CSS scroll-driven animations (Chrome and Edge 115+, Safari 26+). Firefox gets the unmasked image through the @supports fallback.



```css
/* Stripe mask reveal · pure CSS
   12 strips · stagger 0.08 from "start" · grows from left · ease power3.inOut */

/* The only animated value. Registered so the browser can interpolate it. */
@property --progress {
  syntax: "<number>";
  inherits: false;
  initial-value: 0;
}

/* Tall scroll area. Its scroll position becomes the timeline. */
.reveal-track {
  height: 300vh;
  view-timeline: --reveal-track block;
}

.reveal-sticky {
  position: sticky;
  top: 0;
  height: 100vh;
}

.reveal {
  /* Each strip: time = its own 0 → 1 slice of --progress (starts at its delay, lasts 0.5319)
     eased = time run through the ease, width = how much of the strip is visible */
  --strip-1-time: clamp(0, (var(--progress) - 0) / 0.5319, 1);
  --strip-1-in: clamp(0, 2 * var(--strip-1-time), 1);
  --strip-1-out: clamp(0, 2 * var(--strip-1-time) - 1, 1);
  --strip-1-eased: clamp(0, 0.5 * var(--strip-1-in) * var(--strip-1-in) * var(--strip-1-in) + 0.5 * (1 - (1 - var(--strip-1-out)) * (1 - var(--strip-1-out)) * (1 - var(--strip-1-out))), 1);
  --strip-1-width: calc(8.3333% * var(--strip-1-eased));

  --strip-2-time: clamp(0, (var(--progress) - 0.0426) / 0.5319, 1);
  --strip-2-in: clamp(0, 2 * var(--strip-2-time), 1);
  --strip-2-out: clamp(0, 2 * var(--strip-2-time) - 1, 1);
  --strip-2-eased: clamp(0, 0.5 * var(--strip-2-in) * var(--strip-2-in) * var(--strip-2-in) + 0.5 * (1 - (1 - var(--strip-2-out)) * (1 - var(--strip-2-out)) * (1 - var(--strip-2-out))), 1);
  --strip-2-width: calc(8.3333% * var(--strip-2-eased));

  --strip-3-time: clamp(0, (var(--progress) - 0.0851) / 0.5319, 1);
  --strip-3-in: clamp(0, 2 * var(--strip-3-time), 1);
  --strip-3-out: clamp(0, 2 * var(--strip-3-time) - 1, 1);
  --strip-3-eased: clamp(0, 0.5 * var(--strip-3-in) * var(--strip-3-in) * var(--strip-3-in) + 0.5 * (1 - (1 - var(--strip-3-out)) * (1 - var(--strip-3-out)) * (1 - var(--strip-3-out))), 1);
  --strip-3-width: calc(8.3333% * var(--strip-3-eased));

  --strip-4-time: clamp(0, (var(--progress) - 0.1277) / 0.5319, 1);
  --strip-4-in: clamp(0, 2 * var(--strip-4-time), 1);
  --strip-4-out: clamp(0, 2 * var(--strip-4-time) - 1, 1);
  --strip-4-eased: clamp(0, 0.5 * var(--strip-4-in) * var(--strip-4-in) * var(--strip-4-in) + 0.5 * (1 - (1 - var(--strip-4-out)) * (1 - var(--strip-4-out)) * (1 - var(--strip-4-out))), 1);
  --strip-4-width: calc(8.3333% * var(--strip-4-eased));

  --strip-5-time: clamp(0, (var(--progress) - 0.1702) / 0.5319, 1);
  --strip-5-in: clamp(0, 2 * var(--strip-5-time), 1);
  --strip-5-out: clamp(0, 2 * var(--strip-5-time) - 1, 1);
  --strip-5-eased: clamp(0, 0.5 * var(--strip-5-in) * var(--strip-5-in) * var(--strip-5-in) + 0.5 * (1 - (1 - var(--strip-5-out)) * (1 - var(--strip-5-out)) * (1 - var(--strip-5-out))), 1);
  --strip-5-width: calc(8.3333% * var(--strip-5-eased));

  --strip-6-time: clamp(0, (var(--progress) - 0.2128) / 0.5319, 1);
  --strip-6-in: clamp(0, 2 * var(--strip-6-time), 1);
  --strip-6-out: clamp(0, 2 * var(--strip-6-time) - 1, 1);
  --strip-6-eased: clamp(0, 0.5 * var(--strip-6-in) * var(--strip-6-in) * var(--strip-6-in) + 0.5 * (1 - (1 - var(--strip-6-out)) * (1 - var(--strip-6-out)) * (1 - var(--strip-6-out))), 1);
  --strip-6-width: calc(8.3333% * var(--strip-6-eased));

  --strip-7-time: clamp(0, (var(--progress) - 0.2553) / 0.5319, 1);
  --strip-7-in: clamp(0, 2 * var(--strip-7-time), 1);
  --strip-7-out: clamp(0, 2 * var(--strip-7-time) - 1, 1);
  --strip-7-eased: clamp(0, 0.5 * var(--strip-7-in) * var(--strip-7-in) * var(--strip-7-in) + 0.5 * (1 - (1 - var(--strip-7-out)) * (1 - var(--strip-7-out)) * (1 - var(--strip-7-out))), 1);
  --strip-7-width: calc(8.3333% * var(--strip-7-eased));

  --strip-8-time: clamp(0, (var(--progress) - 0.2979) / 0.5319, 1);
  --strip-8-in: clamp(0, 2 * var(--strip-8-time), 1);
  --strip-8-out: clamp(0, 2 * var(--strip-8-time) - 1, 1);
  --strip-8-eased: clamp(0, 0.5 * var(--strip-8-in) * var(--strip-8-in) * var(--strip-8-in) + 0.5 * (1 - (1 - var(--strip-8-out)) * (1 - var(--strip-8-out)) * (1 - var(--strip-8-out))), 1);
  --strip-8-width: calc(8.3333% * var(--strip-8-eased));

  --strip-9-time: clamp(0, (var(--progress) - 0.3404) / 0.5319, 1);
  --strip-9-in: clamp(0, 2 * var(--strip-9-time), 1);
  --strip-9-out: clamp(0, 2 * var(--strip-9-time) - 1, 1);
  --strip-9-eased: clamp(0, 0.5 * var(--strip-9-in) * var(--strip-9-in) * var(--strip-9-in) + 0.5 * (1 - (1 - var(--strip-9-out)) * (1 - var(--strip-9-out)) * (1 - var(--strip-9-out))), 1);
  --strip-9-width: calc(8.3333% * var(--strip-9-eased));

  --strip-10-time: clamp(0, (var(--progress) - 0.383) / 0.5319, 1);
  --strip-10-in: clamp(0, 2 * var(--strip-10-time), 1);
  --strip-10-out: clamp(0, 2 * var(--strip-10-time) - 1, 1);
  --strip-10-eased: clamp(0, 0.5 * var(--strip-10-in) * var(--strip-10-in) * var(--strip-10-in) + 0.5 * (1 - (1 - var(--strip-10-out)) * (1 - var(--strip-10-out)) * (1 - var(--strip-10-out))), 1);
  --strip-10-width: calc(8.3333% * var(--strip-10-eased));

  --strip-11-time: clamp(0, (var(--progress) - 0.4255) / 0.5319, 1);
  --strip-11-in: clamp(0, 2 * var(--strip-11-time), 1);
  --strip-11-out: clamp(0, 2 * var(--strip-11-time) - 1, 1);
  --strip-11-eased: clamp(0, 0.5 * var(--strip-11-in) * var(--strip-11-in) * var(--strip-11-in) + 0.5 * (1 - (1 - var(--strip-11-out)) * (1 - var(--strip-11-out)) * (1 - var(--strip-11-out))), 1);
  --strip-11-width: calc(8.3333% * var(--strip-11-eased));

  --strip-12-time: clamp(0, (var(--progress) - 0.4681) / 0.5319, 1);
  --strip-12-in: clamp(0, 2 * var(--strip-12-time), 1);
  --strip-12-out: clamp(0, 2 * var(--strip-12-time) - 1, 1);
  --strip-12-eased: clamp(0, 0.5 * var(--strip-12-in) * var(--strip-12-in) * var(--strip-12-in) + 0.5 * (1 - (1 - var(--strip-12-out)) * (1 - var(--strip-12-out)) * (1 - var(--strip-12-out))), 1);
  --strip-12-width: calc(8.3333% * var(--strip-12-eased));

  /* Grows from left: black is pinned to the strip start, its right edge moves right */
  mask-image: linear-gradient(90deg,
    /* strip 1 */ #000 0%, #000 calc(0% + var(--strip-1-width)), transparent calc(0% + var(--strip-1-width)), transparent 8.3333%,
    /* strip 2 */ #000 8.3333%, #000 calc(8.3333% + var(--strip-2-width)), transparent calc(8.3333% + var(--strip-2-width)), transparent 16.6667%,
    /* strip 3 */ #000 16.6667%, #000 calc(16.6667% + var(--strip-3-width)), transparent calc(16.6667% + var(--strip-3-width)), transparent 25%,
    /* strip 4 */ #000 25%, #000 calc(25% + var(--strip-4-width)), transparent calc(25% + var(--strip-4-width)), transparent 33.3333%,
    /* strip 5 */ #000 33.3333%, #000 calc(33.3333% + var(--strip-5-width)), transparent calc(33.3333% + var(--strip-5-width)), transparent 41.6667%,
    /* strip 6 */ #000 41.6667%, #000 calc(41.6667% + var(--strip-6-width)), transparent calc(41.6667% + var(--strip-6-width)), transparent 50%,
    /* strip 7 */ #000 50%, #000 calc(50% + var(--strip-7-width)), transparent calc(50% + var(--strip-7-width)), transparent 58.3333%,
    /* strip 8 */ #000 58.3333%, #000 calc(58.3333% + var(--strip-8-width)), transparent calc(58.3333% + var(--strip-8-width)), transparent 66.6667%,
    /* strip 9 */ #000 66.6667%, #000 calc(66.6667% + var(--strip-9-width)), transparent calc(66.6667% + var(--strip-9-width)), transparent 75%,
    /* strip 10 */ #000 75%, #000 calc(75% + var(--strip-10-width)), transparent calc(75% + var(--strip-10-width)), transparent 83.3333%,
    /* strip 11 */ #000 83.3333%, #000 calc(83.3333% + var(--strip-11-width)), transparent calc(83.3333% + var(--strip-11-width)), transparent 91.6667%,
    /* strip 12 */ #000 91.6667%, #000 calc(91.6667% + var(--strip-12-width)), transparent calc(91.6667% + var(--strip-12-width)), transparent 100%
  );
  mask-size: 100% 100%;
  mask-repeat: no-repeat;

  /* Linear on purpose: the easing already happens per strip above */
  animation: stripe-reveal linear both;
  animation-timeline: --reveal-track;
  animation-range: contain 0% contain 100%; /* track top hits viewport top → track bottom hits viewport bottom */
}

@keyframes stripe-reveal {
  from { --progress: 0; }
  to { --progress: 1; }
}

/* Browsers without scroll-driven animations: show the image unmasked */
@supports not (animation-timeline: view()) {
  .reveal { mask-image: none; }
}

@media (prefers-reduced-motion: reduce) {
  .reveal { animation: none; mask-image: none; }
}
```