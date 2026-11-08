# DataRain background — port from React to vanilla JS

The original component is in `docs/source/data-rain-original.txt`. It was written for a **React + TypeScript + shadcn + Tailwind** project. This app is **one HTML file with vanilla JS**, so we do **not** install React, shadcn or TypeScript. We port the engine.

## What the original prompt asked for vs. what we do
| Original instruction | Our project |
|---|---|
| shadcn structure, put file in `/components/ui` | Not used. No React. The engine becomes a plain function inside `index.html`. |
| TypeScript | Strip the type annotations; logic stays identical. |
| `import "../../index.css"` and `@import "tw-animate-css"` | Not needed. Variables go in our `<style type="text/tailwindcss">` block. `tw-animate-css` is an npm package and nothing in the component uses it. |
| "Fill image assets with Unsplash", "use lucide-react icons" | Not applicable — the component uses no images and no icons. |

## Important: the original CSS is incomplete
The component renders these classes: `dr-root`, `dr-root--contained`, `dr-bloom`, `dr-main`, `dr-vignette`, `dr-grain`.
The CSS that came with it **does not define any of them**. It only declares variables (`--dr-bg-0/1/2`, `--dr-vignette`, `--dr-grain`, `--dr-bloom-blur`, `--dr-bloom-opacity`) and a `dr-fade-in` keyframe that nothing uses. Without the CSS below, the canvases would sit in normal page flow, the bloom canvas would show at quarter size, and the vignette and grain layers would be invisible.

## Target API
Inside the `DATA RAIN` section of the script:

```js
const rain = createDataRain(document.getElementById('rain'), {
  count: undefined,      // undefined = auto (based on screen area), as in the original
  flowSpeed: 1,          // 0.05–5
  obstacleRadius: 64,
  autopilot: true,       // "ghost cursor" wanders when the mouse is idle
  palette: undefined,    // undefined = read --dr-calm/--dr-mid/--dr-hot from CSS
  blend: 'lighter',      // NEW: 'lighter' (dark theme) or 'source-over' (light theme)
});
// returns { refresh(), setOptions(partial), destroy() }
```

Mapping from the React code:
- `propsRef.current` → a plain `opts` object; `setOptions(partial)` merges into it and calls `refresh()` when `count` or `palette` change.
- `apiRef.current.refresh` → the returned `refresh()`.
- The `useEffect` cleanup function → `destroy()` (cancel the animation frame, remove every listener, disconnect the ResizeObserver).
- `className` / `fullscreen` props → just classes on the root element in HTML.

**Copy the engine 1:1.** Keep every constant, formula, and the adaptive-quality logic exactly as written. Only remove types and swap the React wiring. The **one** engine change allowed: replace the hard-coded `ctx.globalCompositeOperation = "lighter"` with `opts.blend`.

## HTML (first child of `<body>`)
```html
<div id="rain" class="dr-root" aria-hidden="true">
  <canvas class="dr-bloom"></canvas>
  <canvas class="dr-main"></canvas>
  <div class="dr-vignette"></div>
  <div class="dr-grain"></div>
</div>
```
All app content sits above it (`position: relative; z-index: 1` on the app wrapper).

## CSS to add
```css
:root {
  --dr-bg-0: #020308; --dr-bg-1: #05070d; --dr-bg-2: #0a1020;
  --dr-calm: #22e6ff; --dr-mid: #8a5cff; --dr-hot: #ff3d9a; --dr-core: #eafcff;
  --dr-vignette: 0.62; --dr-grain: 0.05; --dr-bloom-blur: 10px; --dr-bloom-opacity: 0.7;
}
/* light theme: softer, darker lines on a pale background */
html:not(.dark) {
  --dr-bg-0: #f8fafc; --dr-bg-1: #eef2ff; --dr-bg-2: #e0e7ff;
  --dr-calm: #0891b2; --dr-mid: #7c3aed; --dr-hot: #db2777; --dr-core: #0f172a;
  --dr-vignette: 0.12; --dr-grain: 0.03; --dr-bloom-opacity: 0.25;
}
@keyframes dr-fade-in { from { opacity: 0 } to { opacity: 1 } }

.dr-root {
  position: fixed; inset: 0; z-index: 0;
  pointer-events: none;            /* clicks go through to the app */
  overflow: hidden;
  background: radial-gradient(120% 90% at 50% 0%, var(--dr-bg-2), var(--dr-bg-1) 55%, var(--dr-bg-0));
  animation: dr-fade-in 1.2s ease-out both;
}
.dr-root--contained { position: absolute; }
.dr-root canvas { position: absolute; inset: 0; width: 100%; height: 100%; display: block; }
.dr-bloom { filter: blur(var(--dr-bloom-blur)); opacity: var(--dr-bloom-opacity); }
.dr-vignette {
  position: absolute; inset: 0;
  background: radial-gradient(ellipse at center, transparent 40%, rgb(0 0 0 / var(--dr-vignette)) 100%);
}
.dr-grain {
  position: absolute; inset: 0; opacity: var(--dr-grain); mix-blend-mode: overlay;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.8' numOctaves='2' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");
}
@media (prefers-reduced-motion: reduce) { .dr-root { animation: none; } }
```
The engine reads colours with `parseHex`, so the `--dr-*` colour variables **must be hex** (`#rrggbb` or `#rgb`).

## Theme handling
- Dark theme → `blend: 'lighter'` (additive glow, as designed).
- Light theme → `blend: 'source-over'`. Additive blending on a pale background washes the lines out.
- After toggling `.dark`, call `rain.setOptions({ blend })` then `rain.refresh()` — `refresh()` re-reads the CSS variables.

## On / off and performance
- Settings → "Background animation" off → `rain.destroy()` and hide `#rain`; on → create it again.
- Start the rain **after** the app has rendered (e.g. `requestIdleCallback`, with a `setTimeout` fallback) so it never delays the first paint of the dashboard.
- Already built into the engine (keep it): device-pixel-ratio capped at 2, pixel budget 4.5 million, particle count drops to 40 % on slow frames, animation stops while the tab is hidden, single still frame when the visitor prefers reduced motion.
- Check: with the rain running, typing in the task box must feel instant. In Chrome DevTools → Performance, interactions should stay under 200 ms (the "good" INP threshold).

## Done when
- Rain fills the window behind the app, reacts to the mouse, wanders by itself when idle.
- Every button and input is still clickable everywhere on the page.
- Light theme shows a soft, readable version; dark theme shows the glowing version.
- Reduced-motion users see a still frame.
- Turning it off in Settings stops all CPU use from it (Performance panel shows no frames from it).
