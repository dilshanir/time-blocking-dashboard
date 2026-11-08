# Launch checklist

Work top to bottom. Don't deploy until sections A–D pass.

## A. Functional
- [ ] All 14 acceptance tests in `docs/SPEC.md` §5 pass in Chrome.
- [ ] Spot-check tests 1, 5, 6, 7 in Firefox and Safari (or Safari on an iPhone) too.
- [ ] No errors or warnings in the DevTools Console on load, during use, or after refresh.

## B. Accessibility
- [ ] Whole app usable with keyboard only (Tab / Shift+Tab / Enter / Space / Esc).
- [ ] Focus ring visible on every control, in both themes.
- [ ] Text contrast ≥ 4.5:1 over the panels in both themes (DevTools colour picker shows the ratio).
- [ ] Screen reader quick test (NVDA on Windows or VoiceOver on Mac): headings make sense, timer announces only mode changes.
- [ ] OS "reduce motion" on → rain is still, no transitions.
- [ ] Lighthouse → Accessibility: fix every item it lists.

## C. Performance
- [ ] Lighthouse (mobile) on the **built** page. Fix what it flags.
- [ ] With the rain on, typing in the task box stays instant (DevTools → Performance: interactions under 200 ms).
- [ ] Rain turned off in Settings → no animation frames in the Performance panel.
- [ ] No layout jump when the page loads (CLS close to 0).

## D. SEO
- [ ] `{SITE_NAME}` and `{SITE_URL}` replaced everywhere; no `example.com` left.
- [ ] One `<h1>`; guide section is static HTML; FAQ answers are visible on the page.
- [ ] JSON-LD is valid in the Schema Markup Validator (https://validator.schema.org). Google's Rich Results Test will say the page isn't eligible for the software-app rich result because there are no ratings — that is expected; don't add fake ones.
- [ ] `og-image.png` is a real screenshot of the app; `favicon.svg` exists.
- [ ] `robots.txt` and `sitemap.xml` exist and use the live URL.

## E. Production build (replace the Play CDN)
Tailwind's documentation says the Play CDN is for development only. For launch we compile the CSS and inline it, so the page stays a single HTML file.

1. Install a current LTS version of Node.js (https://nodejs.org).
2. In the project folder terminal:
   ```bash
   npm init -y
   npm install tailwindcss @tailwindcss/cli
   ```
3. Ask Claude Code to run **Prompt 10** in `PROMPTS.md`. It will:
   - create `src/input.css` with `@import "tailwindcss";`, a `@source` line pointing at `index.html`, the `@custom-variant dark` line, the `@theme` tokens and the DataRain CSS (moved out of the `text/tailwindcss` block);
   - add `build.mjs` that runs `npx @tailwindcss/cli -i src/input.css -o dist/app.css --minify`, copies `index.html` to `dist/index.html`, removes the Play CDN `<script>` and the `text/tailwindcss` block, and inlines `dist/app.css` in a normal `<style>` tag;
   - add `"build": "node build.mjs"` to `package.json`.
4. Run `npm run build`, open `dist/index.html` with Live Server, and repeat sections A and C on the built file.
5. If the Tailwind CLI command has changed, follow https://tailwindcss.com/docs/installation/tailwind-cli.

## F. Deploy
- [ ] Put `dist/index.html`, `favicon.svg`, `og-image.png`, `robots.txt`, `sitemap.xml` on a static host with HTTPS (GitHub Pages, Netlify and Cloudflare Pages are common choices with free plans — check their current terms).
- [ ] Open the live URL in a private window: everything works, the page loads over HTTPS.
- [ ] Remember: data saved while testing on `127.0.0.1:5500` does **not** appear on the live domain (different origin). Use Export → Import if you want to keep it.

## G. After launch
- [ ] Google Search Console: verify, submit sitemap, URL Inspection → Request indexing.
- [ ] Bing Webmaster Tools: verify, submit sitemap.
- [ ] Share it in a few real places (see `docs/SEO.md` §6).
- [ ] In 3–4 weeks: read Search Console → Performance → Queries and improve the guide copy for the real queries.
