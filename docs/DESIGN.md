# Design — Time-Blocking Dashboard

## Direction: "night-shift instrument panel"
Quiet, precise, minimal. The animated data rain is the only loud thing on the page; everything on top of it is calm, readable glass panels with crisp numbers. Think of a cockpit display at night, not a toy.

- **One focal point per panel.** The clock, the timer ring, and the "now" line are the stars.
- **Numbers are tabular.** Clock and timer use `font-variant-numeric: tabular-nums` so digits don't jitter.
- **Few colours, used with meaning.** Accent colours come from the rain palette and mean the same thing everywhere.
- **No decoration that moves** except the background and the timer ring.

## Colour tokens
Define these as CSS variables inside `<style type="text/tailwindcss">` with `@theme`, so Tailwind utilities like `bg-surface` exist. Light values on `:root`, dark values under `.dark`.

| Token | Dark | Light | Used for |
|---|---|---|---|
| `--color-ink` | `#e6edf7` | `#0b1020` | main text |
| `--color-ink-soft` | `#9aa7bd` | `#4a5568` | secondary text |
| `--color-surface` | `rgb(8 12 24 / 0.72)` | `rgb(255 255 255 / 0.80)` | panels (glass) |
| `--color-line` | `rgb(148 163 184 / 0.18)` | `rgb(15 23 42 / 0.12)` | borders, grid lines |
| `--color-calm` | `#22e6ff` | `#0891b2` | focus mode, "now" line, primary buttons |
| `--color-mid` | `#8a5cff` | `#7c3aed` | short break |
| `--color-hot` | `#ff3d9a` | `#db2777` | long break, warnings, overlap flag |

Block colour presets (keep as a lookup object of **full** class strings — see `CLAUDE.md`): cyan, violet, pink, amber, emerald.

**Readability over the rain:** panels need `backdrop-blur` *and* enough opacity that text never sits directly on moving lines. Check every text/background pair for **at least 4.5:1 contrast** (WCAG AA for normal text) in both themes with the DevTools colour picker.

## Type
- System stacks only (no web fonts → nothing to download, no layout shift):
  - UI: `ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, sans-serif`
  - Numbers: `ui-monospace, "SF Mono", "Cascadia Mono", Menlo, monospace`
- Scale: clock 2.5–3.5 rem (responsive), timer 3–4 rem, panel titles 0.8 rem uppercase with letter-spacing, body 1 rem.

## Layout
- **≥ 1024 px:** header row; main grid 12 columns → timeline 7, right column 5 (timer on top, tasks below). Max width ~1200 px, centred.
- **640–1023 px:** two columns; timeline full width under timer + tasks.
- **< 640 px:** single column in this order: clock → timer → tasks → timeline. 16 px side padding, no horizontal scrolling.
- Guide section below the app: max ~70 characters per line for comfortable reading.
- Touch targets at least 44 × 44 px.

## Components
- **Panel:** rounded-2xl, 1 px `--color-line` border, `--color-surface` fill, backdrop-blur, padding 20–24 px, small uppercase title.
- **Buttons:** primary = calm accent fill; secondary = outline; icon buttons always have an `aria-label`.
- **Timer:** SVG ring (stroke-dasharray), mode tabs above, Start/Pause big, Reset/Skip small. Ring colour = mode colour.
- **Timeline:** hour labels on the left, faint slot lines, blocks as rounded cards with a left colour bar, title, time range, linked-task count, Focus button. "Now" line in calm accent with a small dot.
- **Task row:** checkbox, text, linked-block chip, Edit / Move / Delete icon buttons shown on hover **and** always reachable by keyboard.
- **Toasts:** bottom centre, one at a time, auto-hide 5 s, with action button (Undo).
- **Dialogs:** native `<dialog>` with `showModal()` — focus is trapped and Esc closes it for free.

## Light vs dark
- Dark is the "hero" look (the rain was designed for a dark background).
- Light mode must still feel intentional: pale background, deeper accent colours, the rain rendered softer (see `docs/DATA-RAIN.md`).
- The theme toggle cycles System → Light → Dark and shows the current choice with an icon + visually-hidden label.

## Accessibility checklist
- Landmarks: `header`, `main`, `footer`; one `h1`; panel headings are `h2`.
- Visible focus: `focus-visible:ring-2` on every interactive element.
- Never rely on colour alone: overlap flag has a label, done tasks have strike-through + checkbox state.
- `prefers-reduced-motion: reduce` → no CSS transitions/animations, rain shows a single still frame.
- Background container is `aria-hidden="true"` and `pointer-events: none`.
