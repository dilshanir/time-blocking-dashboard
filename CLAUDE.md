# Time-Blocking Dashboard — project rules for Claude Code

Claude Code reads this file at the start of every session. Keep it short.
Detailed specs live in `docs/` — open the one the current task needs.

## Project facts (edit these first)
- Author and builder: **Dilshani Rathnayake** — use exactly this name in `<meta name="author">`, the JSON-LD `author`, and the footer credit.
- Site name: **Time Blocking Dashboard** ← replace when the brand is chosen
- Live URL: **https://example.com/** ← replace before the SEO step. Never invent a URL.
- Page language: `en`

## What we are building
One single-page web app in **one file: `index.html`** (HTML + Tailwind CSS + vanilla JavaScript).
A minimalist daily time-blocking dashboard with:
- real-time clock and date
- time-block timeline for the day
- task list with check-off
- adjustable Pomodoro-style focus timer
- dark / light / system theme toggle
- animated "data rain" canvas background (ported from a React component)
- everything saved in `localStorage`, restored on refresh or reopen

## Where the details are
- `docs/SPEC.md` — features, data model, storage rules, acceptance tests
- `docs/DESIGN.md` — layout, colours, type, dark/light, accessibility
- `docs/DATA-RAIN.md` — how to port the React DataRain component to vanilla JS
- `docs/source/data-rain-original.txt` — the original React/TSX + CSS (reference only; never ship it as-is)
- `docs/SEO.md` — head tags, structured data, on-page copy, what NOT to do
- `docs/LAUNCH-CHECKLIST.md` — final checks, production build, deploy

## Hard rules
- No frameworks, no React, no TypeScript, no bundler during development.
- Tailwind v4 Play CDN while developing: `<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>`.
  Custom CSS goes inside `<style type="text/tailwindcss">`.
- Dark mode is class-based: `@custom-variant dark (&:where(.dark, .dark *));` with `.dark` on `<html>`.
- Write every Tailwind class as a complete literal string. Never build class names like `` `bg-${c}-500` `` — Tailwind only generates classes it can find written out in full. Use a lookup object of full class strings instead.
- No external images, web fonts or icon libraries. Icons are small inline SVGs with `aria-hidden="true"`.
- All persistence goes through the single `Store` object (see `docs/SPEC.md`). No direct `localStorage` calls elsewhere. Every storage read/write is inside try/catch.
- Never put user-typed text into `innerHTML`. Use `textContent` / `createElement`.
- The day key is the **local** date built from `getFullYear()`, `getMonth()`, `getDate()`. Never `toISOString().slice(0, 10)` — that is UTC and gives the wrong day before 05:30 in Sri Lanka (UTC+05:30).
- Timers are timestamp-based (`endsAt`), never "subtract 1 every second". Browsers throttle timers in background tabs.
- Respect `prefers-reduced-motion`.
- SEO: follow `docs/SEO.md`. No keyword stuffing, no hidden text, no `<meta name="keywords">`, no invented ratings or reviews in JSON-LD.
- One `<script>` block at the end of `<body>`, organised with comment banners in this order:
  `UTIL · STORE · THEME · CLOCK · TASKS · BLOCKS · POMODORO · SETTINGS · DATA RAIN · BOOT`

## How to work with me
- Small steps. For anything bigger than a few lines, show a short plan first.
- After each step, give me a numbered list of things to test in the browser (I use the VS Code Live Server extension, usually at http://127.0.0.1:5500) and say clearly what you could not verify yourself.
- Don't add features that aren't in the spec. Write ideas in `docs/IDEAS.md` instead.
- Add one line per finished step to `CHANGELOG.md` (create it if missing).
- If a doc and my latest message disagree, ask me which one wins.
