# Prompts for Claude Code in VS Code

Paste **one prompt at a time** into the Claude Code panel. After each one:
1. Read Claude's plan or diff before accepting.
2. Test in the browser (Live Server) using the list Claude gives you.
3. Commit: `git add -A && git commit -m "Step N: ..."` (or ask Claude: "commit these changes with a clear message").

If a step goes wrong: hover over your message in the Claude panel → **Rewind code to here**, then try again with more detail.

`@docs/SPEC.md` style references attach that file to your prompt. Type `@` and pick the file if autocomplete appears.

---

## Prompt 1 — Read everything and plan (use Plan mode)
Switch to **Plan** mode first (click the mode name at the bottom of the prompt box, or type `/plan`).

```text
Read CLAUDE.md, @docs/SPEC.md, @docs/DESIGN.md, @docs/DATA-RAIN.md and @docs/SEO.md.
Don't write any code yet.

Give me:
1. A build plan in the same order as the steps in PROMPTS.md (skeleton → storage → tasks → time blocks → timer → background → SEO → review → build).
2. The exact section layout of the single <script> (banners from CLAUDE.md) and the public functions each section will expose.
3. Anything in the docs that is unclear, risky or contradictory, as a numbered list of questions for me.
Keep it short enough to read in five minutes.
```
Answer Claude's questions, then switch back to **Manual** mode (Claude shows each diff and asks before editing) for the build steps.

---

## Prompt 2 — Skeleton, theme and clock
```text
Create index.html following CLAUDE.md, @docs/DESIGN.md and @docs/SEO.md §4 (use the head template, keep {SITE_NAME}/{SITE_URL} from CLAUDE.md "Project facts").

Build only:
- the full page structure from @docs/SPEC.md §2 with empty, styled panels (Timeline, Focus timer, Tasks), the guide section with placeholder headings, and the footer credit "Built by Dilshani Rathnayake";
- the Tailwind v4 Play CDN, the @custom-variant dark line and the colour tokens from DESIGN.md;
- F5 Theme (System/Light/Dark toggle, no-flash script in <head>, live follow of the OS setting);
- F1 Clock without the midnight carry-over banner (that comes with tasks).
Keep storage minimal for now: save only the theme, inside a small Store stub we'll replace in the next step.

Responsive layout must work at 375 px, 768 px and 1280 px wide with no horizontal scroll.
When done, list what I should test.
```

---

## Prompt 3 — Storage layer and Settings
```text
Implement the full Store from @docs/SPEC.md §4 and F6 Settings.

Requirements:
- single key "tbd:v1", the exact JSON shape in the spec, defaults merged on load, migrate() hook;
- debounced save (~300 ms) plus flush on pagehide and when the tab becomes hidden;
- local day key helper (getFullYear/getMonth/getDate — never toISOString);
- every failure case in the "Failure handling" table, with the toast/banner text given there;
- 'storage' event listener for cross-tab sync;
- Export / Import (validate before overwriting, show a summary and confirm) / Reset (type RESET to confirm);
- prune days older than 90 days on boot;
- Settings <dialog> with all fields from F6 (fields for features not built yet can be saved but do nothing yet).
Move the theme onto the real Store.

Then show me how to test quota and "storage unavailable" behaviour from the DevTools console without breaking my real data.
```

---

## Prompt 4 — Task list
```text
Implement F3 Tasks from @docs/SPEC.md exactly, plus the midnight rollover banner from F1.

- per-day lists stored through Store.day(key);
- add / tick / edit inline / delete with 5-second Undo toast / move up-down / filter / counter / clear completed;
- user text only via textContent;
- full keyboard support and proper labels for screen readers;
- done tasks move below open ones.
Leave the "link to a time block" select in place but disabled until blocks exist.

Run acceptance tests 1, 2, 11, 12 and 13 (task parts) from SPEC §5 in your head against the code and tell me which ones you could not verify.
```

---

## Prompt 5 — Time-block timeline
```text
Implement F2 Time blocks from @docs/SPEC.md (not the drag stretch goal).

- timeline from settings.dayStart to settings.dayEnd in settings.slotMinutes rows;
- create via empty slot click or "+ Block"; edit/delete via the block; native <dialog> form with validation and snapping;
- overlaps allowed but flagged with a label and striped edge;
- "now" line updated every minute; highlight the current block; scroll to now on load;
- block colours from a lookup object of FULL Tailwind class strings (no template-built class names);
- connect the task ↔ block link from Prompt 4 and show the linked-task count on each block;
- "Focus" button on each block that calls a placeholder Timer.focusOn(title) we'll implement next.
Re-render the timeline when day range or slot size change in Settings.
```

---

## Prompt 6 — Focus timer
```text
Implement F4 Focus timer from @docs/SPEC.md.

Must-haves:
- timestamp-based (endsAt / remainingMs), display via requestAnimationFrame, correct after background-tab throttling and after refresh;
- modes, defaults 25/5/15 and long break every 4, all adjustable in Settings (changes apply to the next session);
- Start/Pause, Reset, Skip, mode tabs, SVG progress ring, today's focus dots (store focusCount in the day object);
- chime via Web Audio API, AudioContext created/resumed inside the Start click;
- notifications only after the user clicks "Enable notifications" in Settings; handle "denied" gracefully;
- tab title shows remaining time while running and is restored when stopped;
- aria-live polite announcements only for mode changes and completions;
- Timer.focusOn(title) used by the time-block Focus buttons;
- auto-start next session setting.

Then walk me through acceptance tests 5 and 6 with a 1-minute focus length.
```

---

## Prompt 7 — Animated background (DataRain port)
```text
Port the DataRain component to vanilla JS following @docs/DATA-RAIN.md exactly.
The original React/TSX is in @docs/source/data-rain-original.txt.

- createDataRain(root, opts) → { refresh, setOptions, destroy } in the DATA RAIN section;
- copy the engine 1:1 (constants, physics, render, adaptive quality, reduced-motion still frame, visibility pause); only remove TypeScript and React wiring, and make globalCompositeOperation come from opts.blend;
- add the missing CSS for dr-root, dr-bloom, dr-main, dr-vignette, dr-grain from DATA-RAIN.md;
- light theme uses blend 'source-over' and the light variables; call setOptions + refresh on every theme change;
- start it after the app has rendered (requestIdleCallback with setTimeout fallback);
- Settings "Background animation" on/off creates/destroys it;
- every button and input must stay clickable (pointer-events: none on the background).
Nice-to-have from SPEC F4: flowSpeed 0.6 during focus, 1.0 otherwise.

Tell me where you changed anything compared with the original engine and why.
```

---

## Prompt 8 — SEO content and head
First edit `CLAUDE.md` → Project facts: real site name and live URL.

```text
Apply @docs/SEO.md to index.html.

- finalise <head> from the template with the real {SITE_NAME} and {SITE_URL} from CLAUDE.md;
- one visible <h1> "Daily time blocking planner" placed naturally in the header;
- write the guide section as static HTML: How time blocking works · How to use the focus timer · Where your data is stored (be honest: this browser on this device only, use Export to back up) · Time blocking vs Pomodoro · FAQ (4–6 real questions). 400–800 words total, plain helpful language, phrases from SEO.md §2 used only where they read naturally;
- create favicon.svg (simple, original mark — not a copy of any existing logo), robots.txt and sitemap.xml for the live URL;
- do NOT add a keywords meta tag, hidden text or ratings/reviews markup.

Then show me the final title, description and headings in a short list so I can review the wording.
```

---

## Prompt 9 — Review pass (fresh eyes)
Start a **new conversation** in Claude Code for this one, so the review isn't biased by the build chat.

```text
You are reviewing index.html against CLAUDE.md, @docs/SPEC.md, @docs/DESIGN.md and @docs/DATA-RAIN.md. Don't change anything yet.

Report, as a table (issue · where · severity · fix):
1. every SPEC §5 acceptance test you believe would fail, and why;
2. accessibility problems (labels, focus, contrast risks, aria-live misuse, keyboard traps);
3. storage risks (direct localStorage calls outside Store, missing try/catch, UTC date keys, unvalidated import);
4. security (any user text reaching innerHTML);
5. performance risks (work on every frame, layout thrashing, timers that keep running when hidden);
6. Tailwind classes built dynamically (they won't exist after the production build).
Then wait for me to say which ones to fix.
```

---

## Prompt 10 — Production build (single file, no CDN)
Do section E steps 1–2 of `docs/LAUNCH-CHECKLIST.md` first (install Node.js, `npm install tailwindcss @tailwindcss/cli`).

```text
Follow @docs/LAUNCH-CHECKLIST.md section E step 3.
Create src/input.css and build.mjs so that `npm run build` produces dist/index.html: one self-contained file with compiled, minified Tailwind CSS inlined, no Play CDN script and no text/tailwindcss block. Copy favicon.svg, og-image.png (if present), robots.txt and sitemap.xml into dist/.
Check the Tailwind CLI docs if a command fails rather than guessing.
Add a .gitignore for node_modules and dist.
Run the build, then tell me how to compare the built page with the dev page.
```

---

## Prompt 11 — Project documentation (.md files)
```text
Write the project documentation as separate Markdown files:
- README.md — what it is, screenshot placeholder, features, how data is stored (honest limits), how to run locally (Live Server), how to build (npm run build), how to deploy, credits "Built by Dilshani Rathnayake", licence placeholder;
- docs/ARCHITECTURE.md — the script sections, the Store API, the data model, and how the timer stays accurate;
- docs/USER-GUIDE.md — short how-to for visitors (time blocks, tasks, timer, backup/restore, privacy);
- update CHANGELOG.md with a clean summary of v1.0.0.
Keep each file concise and accurate to the actual code — read the code, don't describe features that don't exist.
```

---

## Helper prompts

**Bug report**
```text
Bug: [what you did] → [what happened] → [what you expected].
Console error (if any): [paste]
Find the cause before changing code. Explain it in two sentences, then fix it with the smallest change and tell me how to confirm the fix.
```

**Make it look better (without changing behaviour)**
```text
Improve the visual polish of [panel] following @docs/DESIGN.md. Don't change any behaviour or storage. Show before/after of the classes you change and check contrast in both themes.
```

**Explain the code to me**
```text
Explain how [Store.save / the timer / the timeline render] works in index.html, step by step, for someone learning JavaScript. Point to line ranges.
```
