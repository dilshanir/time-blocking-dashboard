# START HERE — Build the Time-Blocking Dashboard with Claude Code in VS Code

Author and builder: **Dilshani Rathnayake**

This kit gives Claude Code everything it needs to build your dashboard step by step: project rules, specs, an honest SEO plan, the DataRain background port guide, and copy-paste prompts.

## 1. What's in the kit
```
time-blocking-dashboard/
├── START-HERE.md                 ← this guide (for you)
├── CLAUDE.md                     ← project rules; Claude Code reads it automatically
├── PROMPTS.md                    ← the prompts to paste, in order (1 → 11)
└── docs/
    ├── SPEC.md                   ← features, localStorage data model, acceptance tests
    ├── DESIGN.md                 ← look and feel, light/dark, accessibility
    ├── DATA-RAIN.md              ← how to turn the React DataRain into a vanilla-JS background
    ├── SEO.md                    ← what actually helps ranking, head template, what to avoid
    ├── LAUNCH-CHECKLIST.md       ← tests, production build, deploy, Search Console
    └── source/
        └── data-rain-original.txt  ← your original React component prompt (reference)
```
Claude Code will add `index.html`, `CHANGELOG.md`, `README.md` and more as you go.

## 2. What you need (checked against the official docs)
| Item | Notes |
|---|---|
| VS Code 1.94.0 or later | Help → About shows your version. |
| Claude Code extension for VS Code (publisher: Anthropic) | Needs a paid Claude plan (Pro, Max, Team or Enterprise) or a Claude Console account. No API key needed for a plan login. |
| Live Server extension (by Ritwick Dey) | Serves the page at `http://127.0.0.1:5500` and reloads on save. Needed because some browsers restrict `localStorage` on `file://` pages. |
| Git | Recommended — one commit per step makes undo easy. |
| Node.js (current LTS) | Only for the final production build (Prompt 10). |

## 3. Set up (about 10 minutes)
1. Unzip this kit to a folder, e.g. `Documents/time-blocking-dashboard`.
2. VS Code → **File → Open Folder…** → pick that folder.
3. Install Claude Code: press `Ctrl+Shift+X` (Windows/Linux) or `Cmd+Shift+X` (Mac), search **Claude Code**, click **Install**. If it doesn't show up afterwards, run **Developer: Reload Window** from the Command Palette (`Ctrl/Cmd+Shift+P`).
4. Open any file (e.g. `CLAUDE.md`). Click the **Spark icon** at the top-right of the editor, or **✻ Claude Code** in the status bar at the bottom-right. Sign in when asked.
5. Install **Live Server** the same way. Later, click **Go Live** in the status bar to open the page.
6. Optional but recommended — in the VS Code terminal (`` Ctrl+` ``):
   ```bash
   git init
   git add -A
   git commit -m "Start: Claude Code kit"
   ```

## 4. Before the first prompt — edit `CLAUDE.md`
Open `CLAUDE.md` → **Project facts**:
- The author is set to **Dilshani Rathnayake**. To change it later, use **Edit → Replace in Files** (`Ctrl/Cmd+Shift+H`) across the whole folder — it appears in the meta author tag, the structured data and the footer.
- Site name and live URL can stay as placeholders until Prompt 8.

## 5. How to run the prompts
1. Open `PROMPTS.md` next to the Claude panel.
2. **Prompt 1 in Plan mode:** click the permission mode name at the bottom of the prompt box and choose **Plan** (or type `/plan`). Claude reads the docs and returns a plan plus questions. Answer them.
3. **Prompts 2–11 in Manual mode:** Claude shows a side-by-side diff for every edit and asks before applying it. You learn more and catch mistakes early. (Newer versions start in **Auto** mode, where a classifier reviews most actions instead of asking you — fine once you're comfortable.)
4. After each prompt: Go Live → test what Claude listed → commit.
5. If something breaks, hover over your message in the Claude panel → **Rewind code to here**, or use the **Bug report** helper prompt at the bottom of `PROMPTS.md`.

Useful in the prompt box: type `@` to attach a file; select code in the editor and press `Alt+K` (`Option+K` on Mac) to insert a reference to those exact lines.

## 6. What the build will do about your requests
- **Single HTML file + Tailwind + JS:** yes — `index.html` with the Tailwind Play CDN while you build, then compiled and inlined for launch (Tailwind says the CDN is for development only).
- **Dark/light toggle, real-time clock, checkable task list, adjustable Pomodoro timer:** Prompts 2, 4 and 6.
- **localStorage so everything persists:** Prompt 3 — saved automatically, restored on refresh or reopen, with backup export/import. Note: it is saved **in that browser on that device**; it does not sync to your phone.
- **The DataRain component:** used as the animated page background (Prompt 7). Because this is a plain HTML page, it's ported to vanilla JS instead of installing React/shadcn/TypeScript. The CSS that came with it was missing the classes the component uses — `docs/DATA-RAIN.md` supplies them.
- **SEO keywords to rank first:** Prompt 8 puts the right phrases where search engines actually read them (title, H1, headings, helpful guide text, structured data). Two honest facts: Google says no one can guarantee a #1 ranking, and the `keywords` meta tag has been ignored by Google since 2009 — stuffing keywords is treated as spam and can push a page *down*. Ranking comes after launch from Search Console, real links and improving the copy for real queries (`docs/SEO.md` §6).

## 7. Common problems
| Problem | Fix |
|---|---|
| No Spark icon | Open a file first (the icon needs an open file), or use **✻ Claude Code** in the status bar. |
| Extension not visible after install | Command Palette → **Developer: Reload Window**. |
| My tasks disappeared | Did the address change (e.g. `127.0.0.1:5500` → live site, or a different port)? Each address has its own storage. Use Export / Import. |
| Page is blank | Press `F12` → Console, copy the red error into the **Bug report** prompt. |
| Styles missing after `npm run build` | A Tailwind class was probably built dynamically in JS. Run Prompt 9 and fix item 6. |

## Sources
- Claude Code in VS Code — https://code.claude.com/docs/en/vs-code
- CLAUDE.md and project memory — https://code.claude.com/docs/en/memory
- Tailwind Play CDN — https://tailwindcss.com/docs/installation/play-cdn
- Tailwind CLI — https://tailwindcss.com/docs/installation/tailwind-cli
- MDN localStorage — https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
- MDN storage quotas — https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria
- Chrome timer throttling in background tabs — https://developer.chrome.com/blog/timer-throttling-in-chrome-88
- Live Server extension — https://github.com/ritwickdey/vscode-live-server
- Google: Do you need an SEO? — https://developers.google.com/search/docs/fundamentals/do-i-need-seo
- Google: keywords meta tag — https://developers.google.com/search/blog/2009/09/google-does-not-use-keywords-meta-tag
- Google: Spam policies — https://developers.google.com/search/docs/essentials/spam-policies
- Pomodoro Technique (overview) — https://en.wikipedia.org/wiki/Pomodoro_Technique
