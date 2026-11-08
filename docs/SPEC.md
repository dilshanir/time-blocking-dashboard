# Product spec — Time-Blocking Dashboard

Author: Dilshani Rathnayake · Status: v1 build spec

## 1. Goal
A calm, fast, single-page dashboard where a person plans **today** in time blocks, keeps a short task list, and runs focus sessions. No account, no server: everything is stored in the browser with `localStorage`, so the day is still there after a refresh, a browser restart, or reopening the tab.

Out of scope for v1: accounts, cloud sync, multi-day calendar views, recurring blocks, mobile app.

## 2. Page regions
1. **Header** — site name, today's weekday + date, live clock, theme toggle, settings button.
2. **Main grid**
   - **Timeline** — today's time blocks (left / first on desktop).
   - **Focus timer** — Pomodoro-style timer (right column, top).
   - **Tasks** — today's task list (right column, below the timer).
3. **Guide section** (below the app, static HTML) — short, genuinely useful text: how time blocking works, how to use the timer, where data is stored, FAQ. This is also the main SEO content (see `docs/SEO.md`).
4. **Footer** — "Built by Dilshani Rathnayake", year, link to privacy note ("Your data never leaves your browser").
5. **Background** — DataRain canvas, fixed behind everything (see `docs/DATA-RAIN.md`).

## 3. Features

### F1 — Clock
- Shows `HH:MM:SS`, weekday and full date, using `Intl.DateTimeFormat` with the visitor's locale.
- 12h / 24h toggle in Settings. Default: `null` = follow the locale.
- Updates on the second boundary: schedule the next tick with `setTimeout(tick, 1000 - (Date.now() % 1000))`. Re-sync on `visibilitychange` when the tab becomes visible.
- Uses `<time datetime="...">`. **Not** an `aria-live` region (a live clock would flood screen readers).
- **Midnight rollover:** when the local date changes while the page is open, switch the view to the new day. If yesterday has unfinished tasks, show a banner: "You have N unfinished tasks from {date}. Move them to today?" [Move] [Dismiss].

### F2 — Time blocks (timeline)
- Vertical timeline from `dayStart` to `dayEnd` (defaults 06:00–22:00), in slots of `slotMinutes` (15 / 30 / 60, default 30). All three are Settings.
- **Create:** click an empty slot, or the "+ Block" button. A small form (native `<dialog>`) asks for: title (required, max 80 chars), start, end, colour (5 presets: cyan, violet, pink, amber, emerald), optional note (max 200 chars).
- **Edit / delete:** click a block → same dialog with Delete. Delete asks for confirmation.
- **Validation:** end must be after start; both inside the day range; times snap to the slot size. **Overlaps are allowed but flagged** (striped edge + "overlaps" label) — people do double-book.
- **Now line:** a horizontal line at the current time, updated every minute. The block containing "now" is highlighted. On load, scroll the timeline so "now" is visible.
- **Focus from a block:** each block has a "Focus" button that loads the timer in focus mode with the block title as its label.
- **Keyboard:** blocks are focusable buttons; Enter opens the editor; the create button is reachable by Tab.
- Stretch goal (only after everything else passes): drag to move / resize with pointer events, with keyboard alternatives.

### F3 — Tasks
- Add with a text field + Enter (or Add button). Trim; ignore empty; max 200 chars.
- Check off with a real `<input type="checkbox">` + `<label>`. Done tasks get a strike-through and move below open tasks.
- Edit inline (Edit button, or double-click the text; Enter saves, Esc cancels).
- Delete with an **Undo** toast for 5 seconds.
- Reorder with "Move up" / "Move down" buttons (keyboard-friendly). Drag-and-drop is optional.
- Optional link to a time block (select in the edit row). The block shows how many linked tasks it has.
- Filter: All / Open / Done. Counter: "3 of 7 done". "Clear completed" button (with confirm).
- Lists are **per day** (see data model).

### F4 — Focus timer (Pomodoro-style)
- Modes: **Focus**, **Short break**, **Long break**.
- Defaults (classic Pomodoro pattern): focus 25 min, short break 5 min, long break 15 min, long break after every 4 focus sessions.
- All adjustable in Settings: focus 1–120, short 1–30, long 1–60, long-break interval 2–8. Changing durations while running applies to the **next** session.
- Controls: Start / Pause, Reset, Skip. Mode tabs to switch manually.
- **Timestamp-based:** on Start, `endsAt = Date.now() + remainingMs`. Display updates with `requestAnimationFrame` (or a 250 ms interval) and always computes `remaining = endsAt - Date.now()`. On Pause, store `remainingMs`. This keeps the timer correct even when the browser throttles background tabs.
- **Survives refresh:** state is in the Store. On load, if `endsAt` is in the past, mark that session complete, advance to the next mode, and wait for the user (no surprise auto-start).
- **End of session:** short chime made with the Web Audio API (no audio file). Create/resume the `AudioContext` inside the first Start click, because browsers block audio that isn't started by a user gesture. Optional desktop notification — permission is requested **only** from a button in Settings, never on page load.
- While running, the tab title shows e.g. `12:34 · Focus — Time Blocking Dashboard`.
- Progress ring (SVG). Today's completed focus sessions shown as small dots.
- One `aria-live="polite"` region announces only mode changes and completions — not every second.
- Setting "Auto-start next session" (default off).
- Nice-to-have (do last): during Focus, slow the background rain (`flowSpeed` 0.6); restore 1.0 on breaks.

### F5 — Theme
- Options: Light / Dark / System. Default: System. Stored in settings.
- Apply before first paint with a tiny inline script in `<head>` (reads the stored theme, sets `.dark` on `<html>`) so there is no white flash.
- In System mode, follow `matchMedia('(prefers-color-scheme: dark)')` changes live.
- On every theme change, call `rain.refresh()` so the canvas picks up the new palette.

### F6 — Settings (native `<dialog>`)
Clock 12/24h · day start/end · slot size · timer durations + long-break interval · auto-start · sound on/off · "Enable notifications" button · background animation on/off · **Export backup** · **Import backup** · **Reset all data** (type-to-confirm).

### F7 — Background
DataRain canvas behind the app. Full rules in `docs/DATA-RAIN.md`.

## 4. Data model and storage

One key, one JSON document:

```json
{
  "version": 1,
  "settings": {
    "theme": "system",
    "clock24h": null,
    "dayStart": "06:00",
    "dayEnd": "22:00",
    "slotMinutes": 30,
    "sound": true,
    "notifications": false,
    "rain": true,
    "pomodoro": { "focus": 25, "short": 5, "long": 15, "longEvery": 4, "autoStart": false }
  },
  "days": {
    "2026-11-08": {
      "blocks": [
        { "id": "b_k3x9", "title": "Deep work", "start": "09:00", "end": "10:30", "color": "cyan", "note": "" }
      ],
      "tasks": [
        { "id": "t_p2m1", "text": "Draft the spec", "done": false, "blockId": "b_k3x9", "createdAt": 1794096000000, "doneAt": null }
      ],
      "focusCount": 2
    }
  },
  "pomodoro": { "mode": "focus", "status": "idle", "endsAt": null, "remainingMs": 1500000, "cycle": 0, "label": "" }
}
```

- **Key:** `tbd:v1`
- **IDs:** `crypto.randomUUID()` when available, otherwise `Date.now().toString(36) + Math.random().toString(36).slice(2)`.
- **Day key:** local `YYYY-MM-DD` from `getFullYear/getMonth/getDate` (see `CLAUDE.md`).

### The `Store` object (the only code that touches localStorage)
- `Store.load()` — read + `JSON.parse` inside try/catch; fill missing fields from defaults; run `migrate()` when `version` is older.
- `Store.save()` — **debounced ~300 ms**; also flush immediately on `pagehide` and when `visibilitychange` goes to hidden.
- `Store.day(dateKey)` — returns that day's object, creating an empty one if needed.
- `Store.export()` — downloads `time-blocks-backup-YYYY-MM-DD.json` via a Blob link.
- `Store.import(file)` — parse, validate shape and types, show a summary, confirm before overwriting.
- `Store.reset()` — remove the key after confirmation.
- Prune `days` older than 90 days on boot (data is small; this just keeps it tidy).

### Failure handling (must be visible to the user, never silent)
| Situation | What the app does |
|---|---|
| `QuotaExceededError` on save | Toast: "Browser storage is full — export a backup." Keep running in memory. |
| `localStorage` unavailable / throws `SecurityError` (some private modes, blocked site data, `file://` pages) | Run in memory. Persistent banner: "Changes won't be saved in this browser mode." |
| Stored JSON is corrupt | Copy the raw string to `tbd:corrupt:<timestamp>`, start fresh, toast explaining what happened. |
| Another tab changes the data | Listen to the `storage` event and reload state, so two open tabs don't overwrite each other. |

### Facts to keep in mind
- `localStorage` stores **strings only** and has **no expiry date**; data stays until the user or the browser clears it.
- It is **per origin** (protocol + host + port). `http://127.0.0.1:5500` and your live domain are different origins — data does not move between them. Use Export / Import.
- Browsers allow about **5 MiB** of localStorage per origin; going over throws `QuotaExceededError`.
- It is **per browser and per device**. Nothing syncs to phone or another computer. Say this honestly in the UI and the guide text.

## 5. Acceptance tests (manual — run after each step and before launch)
1. Add 3 tasks, tick one, refresh → same 3 tasks, same one ticked.
2. Close the tab, reopen the page → everything is still there.
3. Create a block 09:00–10:30, edit it to 09:30–11:00, refresh → edited times shown.
4. Create an overlapping block → saved and visibly flagged.
5. Start a 1-minute focus, switch to another tab for 2 minutes, come back → session shows complete, chime played (if sound on), next mode is waiting.
6. Start a focus, refresh mid-way → timer continues from the right remaining time.
7. Toggle theme → whole UI and the rain change; refresh → theme kept; no white flash on load.
8. Set the OS to "reduce motion" → rain is a still image, no animations.
9. Open in a private window → app works; banner appears only if storage is actually unavailable.
10. Export, Reset all data, Import the file → everything is back.
11. Two tabs open: tick a task in tab A → tab B updates after the storage event.
12. Type a task like `<img src=x onerror=alert(1)>` → shown as plain text, nothing runs.
13. Keyboard only: you can add, tick, edit and delete a task; create and edit a block; run the timer.
14. Change the computer clock to 00:01 (or wait for midnight) → new day view + carry-over banner.
