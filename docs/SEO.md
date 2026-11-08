# SEO plan — honest and checked

## 1. Facts first (each one checked against the source)
- **Nobody can promise the #1 spot.** Google's own page on hiring an SEO says no one can guarantee a #1 ranking, and to be wary of anyone who does. What we *can* do is remove every technical obstacle and make the page genuinely the best answer for a few specific searches.
- **`<meta name="keywords">` does nothing on Google.** Google announced on 21 Sept 2009 that its web search ignores the keywords meta tag. We leave it out.
- **Keyword stuffing is spam.** Google's spam policies list filling a page with keywords to manipulate rankings as spam; sites that violate the policies can rank lower or disappear from results. Hidden text is on the same list.
- **What Google uses for the title shown in results:** mainly the `<title>` element and the main visible heading on the page. The meta description is sometimes used for the snippet under the title, but it is not a ranking factor.
- **Page experience:** Core Web Vitals "good" thresholds are LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1, measured at the 75th percentile of real visits.
- **Structured data:** Google's software-app rich result requires `name`, `offers.price`, **and** either `aggregateRating` or `review`. We have no real ratings yet, so we add valid `WebApplication` markup **without** ratings: it describes the app correctly but won't produce the star snippet. Never invent ratings or reviews — that breaks Google's guidelines and can lead to a manual action.

## 2. Who we're for and what they type
Realistic targets for a new site are **specific, longer phrases**. Head terms like "pomodoro timer" are dominated by long-established sites.

| Role | Candidate phrases |
|---|---|
| Primary | time blocking planner · online time blocking app |
| Secondary | daily time blocking template · time block schedule online · pomodoro timer with task list · daily planner that saves automatically · time blocking app no sign up |
| Questions (use as H2s / FAQ) | how to time block your day · time blocking vs pomodoro · how long should a time block be |

These are **candidates, not verified volumes.** Before launch, check them in Google Keyword Planner (free with a Google Ads account). After launch, the Search Console *Performance* report shows the real queries people use — rewrite headings and FAQ around those.

## 3. Where the words go
| Place | What to write | Notes |
|---|---|---|
| `<title>` | `Free Time Blocking Planner with Pomodoro Timer · {Site name}` | Unique, descriptive. ~50–60 characters is a common industry guideline, not a Google rule; long titles get cut off in results. |
| `<meta name="description">` | `Plan your day in time blocks, tick off tasks and focus with an adjustable Pomodoro timer. Free, no sign-up — your schedule saves in your browser.` | Write for humans; it's your ad text in results. |
| `<h1>` (visible) | `Daily time blocking planner` | One H1. It can sit in the header next to the clock. |
| `<h2>`s in the guide | How time blocking works · How to use the focus timer · Where your data is stored · Time blocking vs Pomodoro · FAQ | Real explanations, not padding. |
| Body text (guide) | 400–800 words of genuinely useful help | Use the phrases where they fit naturally. If a sentence sounds odd read aloud, rewrite it. |
| Button and label text | "Add task", "New time block", "Start focus" | Clear labels help users and accessibility first. |
| JSON-LD | name, description, author | See template below. |

**Crawlability:** the guide text and headings are **static HTML in the file**, not injected by JavaScript. The interactive app can be rendered by JS; the content you want found must be in the HTML.

## 4. `<head>` template
Replace `{SITE_NAME}` and `{SITE_URL}` with the values from `CLAUDE.md`. Don't publish while `example.com` is still there.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Free Time Blocking Planner with Pomodoro Timer · {SITE_NAME}</title>
  <meta name="description" content="Plan your day in time blocks, tick off tasks and focus with an adjustable Pomodoro timer. Free, no sign-up — your schedule saves in your browser.">
  <link rel="canonical" href="{SITE_URL}">
  <meta name="author" content="Dilshani Rathnayake">
  <meta name="robots" content="index, follow">
  <meta name="color-scheme" content="dark light">
  <meta name="theme-color" content="#05070d" media="(prefers-color-scheme: dark)">
  <meta name="theme-color" content="#f8fafc" media="(prefers-color-scheme: light)">

  <!-- Social previews -->
  <meta property="og:type" content="website">
  <meta property="og:title" content="Free Time Blocking Planner with Pomodoro Timer">
  <meta property="og:description" content="Plan your day in time blocks, tick off tasks and focus. Free, no sign-up.">
  <meta property="og:url" content="{SITE_URL}">
  <meta property="og:image" content="{SITE_URL}og-image.png">
  <meta property="og:image:alt" content="Time blocking dashboard with clock, timeline, tasks and focus timer">
  <meta name="twitter:card" content="summary_large_image">

  <link rel="icon" href="favicon.svg" type="image/svg+xml">

  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "WebApplication",
    "name": "{SITE_NAME}",
    "url": "{SITE_URL}",
    "description": "A free daily time blocking planner with a task list and adjustable Pomodoro-style focus timer. Data is saved in your browser.",
    "applicationCategory": "BusinessApplication",
    "operatingSystem": "Any",
    "browserRequirements": "Requires JavaScript and a modern browser",
    "inLanguage": "en",
    "isAccessibleForFree": true,
    "offers": { "@type": "Offer", "price": "0", "priceCurrency": "USD" },
    "author": { "@type": "Person", "name": "Dilshani Rathnayake" }
  }
  </script>

  <!-- theme before first paint (no flash) -->
  <script>/* read tbd:v1 → settings.theme; add .dark to <html> if needed; wrap in try/catch */</script>

  <!-- development only — replaced by compiled CSS for launch (docs/LAUNCH-CHECKLIST.md) -->
  <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
  <style type="text/tailwindcss">/* @custom-variant dark, @theme tokens, DataRain CSS */</style>
</head>
```
Check `applicationCategory` against the list on Google's SoftwareApplication documentation page and pick the closest fit. The OG image (a real screenshot, 1200 × 630 is the commonly recommended size) and `favicon.svg` are created in the launch step.

## 5. Technical basics for launch
- Host on **HTTPS** with a stable URL (custom domain if possible — easier for people to remember and link to).
- `robots.txt` that allows crawling and points to the sitemap; `sitemap.xml` with the one URL.
- Compile Tailwind instead of using the Play CDN in production (Tailwind's docs say the Play CDN is for development only). This also removes a render-blocking script.
- Start the background animation after the app renders so it never delays first paint.
- Run Lighthouse (Chrome DevTools) on the live URL: aim for green on Performance, Accessibility, Best Practices, SEO — then fix what it flags rather than chasing the number.

## 6. After launch (this is where ranking actually comes from)
1. **Google Search Console:** verify the site, submit `sitemap.xml`, then use *URL Inspection → Request indexing* for the home page.
2. **Bing Webmaster Tools:** verify the site and submit the same sitemap.
3. **Earn real mentions:** share it where people discuss productivity tools, write one genuinely helpful article (e.g. "How I time-block a work day with a Pomodoro timer") on a blog or LinkedIn that links to it, put the link in a public GitHub repo README. Links from real people are a strong signal; bought or swapped links violate Google's spam policies.
4. **Read the data monthly:** Search Console → Performance → Queries. Improve the headings and FAQ for queries where you appear on page 2–3.

## 7. Do NOT
- Add a `keywords` meta tag or a block of comma-separated phrases anywhere.
- Hide text (white-on-white, `display:none` keyword blocks, text behind the canvas).
- Add `aggregateRating` / `review` markup without real, visible reviews.
- Create many near-identical pages for each keyword ("doorway pages" are also on Google's spam list).
- Claim affiliation with the Pomodoro Technique®: the name is a registered trademark of Francesco Cirillo. Describing a "Pomodoro-style timer" descriptively is common, but don't present the site as official. (Not legal advice.)

## Sources
- Google: Do you need an SEO? — https://developers.google.com/search/docs/fundamentals/do-i-need-seo
- Google: keywords meta tag not used (2009) — https://developers.google.com/search/blog/2009/09/google-does-not-use-keywords-meta-tag
- Google: Spam policies — https://developers.google.com/search/docs/essentials/spam-policies
- Google: Title links — https://developers.google.com/search/docs/appearance/title-link
- Google: SoftwareApplication structured data — https://developers.google.com/search/docs/appearance/structured-data/software-app
- web.dev: Core Web Vitals thresholds — https://web.dev/articles/defining-core-web-vitals-thresholds
- Tailwind: Play CDN — https://tailwindcss.com/docs/installation/play-cdn
