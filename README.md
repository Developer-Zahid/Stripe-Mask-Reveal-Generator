CSS mask · GSAP ScrollTrigger · stagger

# Stripe mask reveal

One **mask-image**, one linear-gradient. GSAP staggers an array of plain numbers, one per strip, and every frame rewrites the gradient's hard stops from them. Scroll to scrub it, and change the settings on the right while you watch.

Scroll to reveal

scroll **0%** **12** strips from **start**

masked area

Generated dusk landscape revealed through vertical strips

## Settings

Trigger

Strips 12

Stagger each 0.08

Stagger from

Each strip grows from

Ease (per strip)

Feathered leading edge 0%

Live mask-image

```

```

How it works

## GSAP staggers numbers, CSS paints them

### 1. One object per strip

GSAP can tween any object, not only elements. `strips` is an array of `{ v: 0 }`. Passing the whole array to `tl.to()` with `stagger` gives every strip its own delayed, eased progress from 0 to 1.

### 2. Rewrite the gradient every frame

In the timeline's `onUpdate`, each strip becomes two hard stops: black from the slot's start to `start + width × v`, then transparent to the slot's end. Black shows the image and transparent hides it.

### 3. ScrollTrigger drives the timeline

With `scrub`, scroll position sets the timeline's progress, so reversing the scroll reverses the reveal. Swap it for `toggleActions` to play once when the section enters.

### Why not animate a CSS variable?

The pure-CSS version needs one `clamp()` per strip and a linear timeline. Doing the stagger in GSAP gives you real per-strip easing, `from: "center"` or `"random"` ordering, and any strip count, with the same single mask.

```
// <script src="gsap.min.js"> + ScrollTrigger.min.js
gsap.registerPlugin(ScrollTrigger);

const el = document.querySelector(".reveal");
const N = 12;
const strips = Array.from({ length: N }, () => ({ v: 0 }));

function paint() {
  const w = 100 / N;
  const stops = strips.map((s, i) => {
    const a = i * w, b = a + w;
    const r = a + w * Math.min(1, Math.max(0, s.v));
    return `#000 ${a}%, #000 ${r}%, transparent ${r}%, transparent ${b}%`;
  });
  const g = `linear-gradient(90deg, ${stops.join(", ")})`;
  el.style.webkitMaskImage = el.style.maskImage = g;
}

gsap.timeline({
  scrollTrigger: {
    trigger: ".reveal-track",
    start: "top top",
    end: "bottom bottom",
    scrub: 0.6
  },
  onUpdate: paint
})
.to(strips, {
  v: 1,
  duration: 1,
  ease: "power3.inOut",
  stagger: { each: 0.08, from: "start" }
});

paint(); // start fully hidden

/* CSS */
/* .reveal { mask-size: 100% 100%; mask-repeat: no-repeat; } */
```