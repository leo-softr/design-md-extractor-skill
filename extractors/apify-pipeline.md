# Path A — Apify + Exa Tiered Extraction Pipeline

This is the pipeline for extracting design tokens from a live website URL. It runs multiple Apify actors plus Exa in three tiers, with each tier filling gaps the previous tier left. Validated end-to-end on real B2B agency marketing sites. See [examples/northwind-studio-DESIGN.md](../examples/northwind-studio-DESIGN.md) for a fictional reference example showing the resulting DESIGN.md format.

## Prerequisites

- Apify MCP connected — see [apify-mcp-install.md](apify-mcp-install.md)
- Exa MCP connected — see [exa-mcp.md](exa-mcp.md)
- One-time approval of `apify/web-scraper` in the Apify Console (only needed if Tier 2b runs) — see [apify-actor-approval.md](apify-actor-approval.md)

If any prerequisite is missing, stop the pipeline, surface the install instruction, and wait for the user to confirm setup before retrying.

## The pipeline

Run tiers in sequence. Tier 1 always runs. Tier 2 runs conditionally based on what Tier 1 returned. Tier 3 is the always-on fallback for tokens still missing.

### Tier 1 — always run, parallel-safe

Call all four actors with the same target URL. They are independent and can run in parallel.

| Step | Tool / Actor | Returns | Notes |
|---|---|---|---|
| 1.1 | `mcp__Apify__call-actor("automation-lab/css-color-extractor")` | All hex/RGB/HSL/named colors with usage counts and CSS-property bindings (background, border, color, fill, outline) | Free, fast, ~$0.04/run on free tier. Always works. |
| 1.2 | `mcp__Apify__call-actor("automation-lab/css-stats-analyzer")` | CSS custom properties, `@font-face` declarations, media queries, `@import` chains | Often returns empty on Tailwind sites. Still cheap to run as a guard for non-Tailwind sites. |
| 1.3 | `mcp__Apify__call-actor("misterkhan/website-tech-stack-scanner")` | Framework, CMS, frontend libs, analytics, hosting (7,000+ technology fingerprints) | Tells you whether the site is Astro / Next.js / Webflow / WordPress / etc. — informs which Tier 2 path to take. |
| 1.4 | `mcp__exa__web_fetch_exa(url)` | Clean markdown content of the page (66KB typical) | Used in Tier 1.5 below for voice / mood / copy register synthesis. Does NOT extract any CSS or tokens. |

**Tier 1 inputs (example shape):**

```json
{
  "actor": "automation-lab/css-color-extractor",
  "input": {"urls": ["https://example.com"]}
}
```

```json
{
  "actor": "misterkhan/website-tech-stack-scanner",
  "input": {"startUrls": [{"url": "https://example.com"}]}
}
```

Note the schemas differ — color extractor takes a flat string array, tech-stack scanner takes an array of `{url}` objects. Always fetch the actor's input schema before calling unless you've confirmed the shape.

### Tier 1.5 — synthesize voice / mood from Exa output

Read the markdown returned by `web_fetch_exa`. Extract:

- **Hero copy** (the first 1–2 visible sentences of brand-direct messaging)
- **CTA pattern** (the action verbs the brand uses on buttons)
- **Pain-point framing** (how the brand describes customer problems — direct, empathetic, technical, playful, etc.)
- **Trust signals** (testimonials, ratings, logos)

Synthesize a 2–3 sentence "Voice & Copy Register" paragraph for the DESIGN.md. Mark `voice_and_mood: extracted` in `extraction_status`.

### Tier 2 — conditional, runs only if Tier 1 left specific gaps

Tier 2 has two sub-passes that fire on different conditions.

#### Tier 2a — HTML body for class-count analysis

**Run when:** Tier 1.2 returned empty fonts/variables AND Tier 1.3 confirmed Tailwind is used.

**Tool:** `mcp__Apify__call-actor("apify/website-content-crawler")` with the following input:

```json
{
  "startUrls": [{"url": "<target>"}],
  "crawlerType": "cheerio",
  "maxCrawlPages": 1,
  "maxCrawlDepth": 0,
  "htmlTransformer": "none",
  "removeElementsCssSelector": "dummy_keep_everything",
  "saveHtml": true,
  "saveMarkdown": false,
  "proxyConfiguration": {"useApifyProxy": true}
}
```

Pass `callOptions: { memory: 1024 }` to fit within the Apify free-tier limit (default request is 8192MB which exceeds the limit).

The actor returns the page HTML body (head is stripped — see warning below). Run a regex pass to count Tailwind utility classes:

- `font-serif` count
- `font-sans` count
- `font-mono` count
- `font-{semibold|medium|normal|bold}` counts (for weight inference)
- Tailwind arbitrary values like `bg-[#xxxxxx]` or `rounded-[Npx]`

**Warning:** Tier 2a tells you the *dominant utility class*, not the *actual font*. The Tailwind `font-serif` class can be aliased in `tailwind.config.js` to a sans-serif typeface, which means a "serif-dominant" class count can be misleading. **Always confirm with Tier 2b before locking typography.** See [../references/tailwind-class-trap.md](../references/tailwind-class-trap.md) for the worked example.

#### Tier 2b — computed-style extraction (gold standard)

**Run when:** specific font names are still missing after Tier 2a, OR the user explicitly requests high-confidence typography.

**Tool:** `mcp__Apify__call-actor("apify/web-scraper")` with a custom `pageFunction`.

**Required input:**

```json
{
  "startUrls": [{"url": "<target>"}],
  "runMode": "PRODUCTION",
  "maxRequestRetries": 1,
  "maxPagesPerCrawl": 1,
  "maxResultsPerCrawl": 1,
  "linkSelector": "",
  "globs": [],
  "pseudoUrls": [],
  "proxyConfiguration": {"useApifyProxy": true},
  "downloadCss": true,
  "downloadMedia": false,
  "useChrome": false,
  "headless": true,
  "waitUntil": ["networkidle2"],
  "injectJQuery": false,
  "pageFunction": "<see below>"
}
```

Pass `callOptions: { memory: 2048, timeout: 180 }`.

**The `pageFunction`** to extract typography:

```js
async function pageFunction(context) {
  const fontStack = (sel) => {
    const el = document.querySelector(sel);
    if (!el) return null;
    return getComputedStyle(el).fontFamily;
  };
  const computed = {
    body: fontStack('body'),
    h1: fontStack('h1'),
    h2: fontStack('h2'),
    h3: fontStack('h3'),
    p: fontStack('p'),
    button: fontStack('button'),
    serif_class: (function(){ const el = document.querySelector('.font-serif'); return el ? getComputedStyle(el).fontFamily : null; })(),
    sans_class: (function(){ const el = document.querySelector('.font-sans'); return el ? getComputedStyle(el).fontFamily : null; })()
  };
  const links = Array.from(document.querySelectorAll('link'))
    .map(l => ({rel: l.rel, href: l.href, as: l.getAttribute('as')}))
    .filter(l => l.href && (l.href.includes('font') || l.href.includes('typekit')));
  const rootStyle = getComputedStyle(document.documentElement);
  const fontVars = {};
  for (let i = 0; i < rootStyle.length; i++) {
    const name = rootStyle[i];
    if (name.startsWith('--font') || name.startsWith('--ff')) {
      fontVars[name] = rootStyle.getPropertyValue(name);
    }
  }
  return { url: context.request.url, computed, fontLinks: links, fontCssVariables: fontVars };
}
```

This page function returns the actual computed `font-family` for body, h1, h2, h3, p, button, plus what `.font-serif` and `.font-sans` Tailwind classes resolve to, plus any `<link>` tags pointing to font services, plus any `--font-*` CSS variables on `:root`.

**The first call to `apify/web-scraper` will fail with `"This Actor requires full access to your account."`** That is normal — the user must approve once in the Apify Console. See [apify-actor-approval.md](apify-actor-approval.md). Surface that instruction to the user, then retry.

**Extending Tier 2b for radius / shadows / spacing.** The same `pageFunction` pattern can return `getComputedStyle().borderRadius`, `boxShadow`, `padding`, `margin` for representative selectors (`.card`, `.btn-primary`, `header`, etc.). When typography came back from a Tier 2b run but other tokens are still missing, run a second Tier 2b pass with a broader page function rather than calling separate actors.

### Tier 2c — asset extraction (logo wordmark)

**Run when:** Path A is in flight. This tier always runs for Path A — the logo is needed by every downstream Softr block that includes a nav lockup or page header.

**Tool:** `mcp__Apify__call-actor("apify/web-scraper")` with a logo-targeted `pageFunction`. Reuse the same actor invocation as Tier 2b (same `runMode`, `proxyConfiguration`, etc.) but swap in this page function:

```js
async function pageFunction(context) {
  var images = Array.from(document.querySelectorAll('img'))
    .map(function(i) {
      return {
        src: i.src,
        alt: i.alt,
        width: i.naturalWidth,
        height: i.naturalHeight,
        classList: (i.className && typeof i.className === 'string') ? i.className.slice(0, 80) : null
      };
    })
    .filter(function(i) { return i.src && !i.src.startsWith('data:'); });
  var svgInline = Array.from(document.querySelectorAll('svg'))
    .filter(function(s) {
      var aria = s.getAttribute('aria-label') || '';
      var role = s.getAttribute('role') || '';
      return aria.toLowerCase().indexOf('logo') !== -1 || role === 'img';
    })
    .map(function(s) { return { outerHTML: s.outerHTML.slice(0, 4000) }; });
  return { url: context.request.url, images: images, svgInline: svgInline };
}
```

This page function can be **combined with the Tier 2b typography page function** in a single `web-scraper` call to save one Actor run — return `{ computed, fontLinks, fontCssVariables, images, svgInline }` from a merged function.

#### Logo identification heuristics (apply in order)

Given the returned `images` list:

1. `alt` contains the brand name (case-insensitive substring match, brand name from page title or domain).
2. `alt` contains "logo" (case-insensitive).
3. `src` filename or path contains "logo".
4. First image that is **NOT** in the payment-method / icon / favicon / social-media-icon family. Filter out class names matching `payment-method`, `icon`, `favicon`, `social`, and dimensions ≤ 64×64px.

If multiple candidates remain after filtering, prefer in this order:

1. **SVG** by file extension or `image/svg+xml` content-type (vector, scales perfectly)
2. **PNG with transparency** (raster, lossless)
3. **WebP** (raster, smaller)
4. **JPG** (raster, no transparency — last resort)

If `svgInline` is non-empty, the logo may be inline-SVG (no `<img>` tag exists). In that case, save the `outerHTML` to `./assets/<brand-slug>-wordmark.svg` and set `public_url` to the page URL with a note that the user must self-host (inline SVG cannot be hot-linked).

#### Download

```bash
mkdir -p ./assets && curl -sSL -o "./assets/<brand-slug>-wordmark.<ext>" "<chosen-image-url>"
```

- Strip query parameters from the URL when constructing the local filename, but **keep them in the recorded `public_url`** — CDNs often use query params to serve sized variants.
- If `curl` returns non-200 or zero bytes, try with a User-Agent header: `curl -sSL -H "User-Agent: Mozilla/5.0" ...`.

#### Failure modes

| Failure | Cause | Recovery |
|---|---|---|
| No `<img>` and no inline `<svg>` matches | Logo is set as CSS `background-image` on a `<div>` | Run a follow-up page function that scans `getComputedStyle(el).backgroundImage` on `header`, `.logo`, `.brand`, and the first 5 children of `<header>` / `<nav>`. Extract URLs from `url("…")` patterns. |
| `curl` returns 403 | CDN blocks scrapers | Add `-H "User-Agent: Mozilla/5.0" -H "Referer: <site-url>"`. If still blocked, surface the URL to the user and ask them to download manually. |
| Multiple plausible logos | Brand has wordmark + icon mark + favicon | Save the wordmark only (largest image with brand-name alt). Do not auto-save the icon mark — that's a separate concern the user can request. |
| Logo image is a sprite or composite | Source uses a sprite sheet with the logo as one frame | Surface to the user — auto-cropping is out of scope. Record the sprite URL in `public_url` with `status: needs-verification`. |

#### Output → DESIGN.md

Hand the result to Step 4 synthesis as an `assets` block:

```yaml
assets:
  status: extracted
  logo_wordmark:
    public_url: "<canonical CDN URL>"
    public_url_note: "Canonical reference for production use. Softr cannot resolve project-relative paths."
    local_backup: "./assets/<brand-slug>-wordmark.<ext>"
    local_backup_note: "Downloaded copy for manual upload to Softr's media library if/when you'd rather host on Softr's own CDN."
    format: "<SVG | PNG | WebP | JPG>"
    color: "<dominant logo color>"
    usage: "<short usage hint>"
```

If extraction fails, set `assets.status: needs-verification` with `logo_wordmark: null` and add a row to `Known Gaps in This Extraction` so the user can paste the URL during Step 6.

### Tier 3 — fallback for any token still missing

After Tier 1 + Tier 2 complete, any token still without a value gets:

1. A sensible default — see the open-source substitute table at [../references/google-fonts-substitutes.md](../references/google-fonts-substitutes.md), and standard defaults: 8px radius, white canvas, near-black text (`#1a1a1a`), `Inter` font, one-tier shadow.
2. A `status: needs-verification` flag in the YAML.
3. A line in the "Known Gaps in This Extraction" prose section explaining why and suggesting the resolution.

The skill then proceeds to Step 6 in `SKILL.md` (verification prompt) where the user can override.

## Combined output → DESIGN.md

After all tiers complete, you have:

- 31+ colors with usage counts (Tier 1.1)
- Tech stack including framework + CMS (Tier 1.3)
- Voice / mood paragraph synthesized from page content (Tier 1.5)
- Specific font names with computed-style confirmation (Tier 2b, when run)
- Class-count typography signals as a sanity check (Tier 2a)
- Confidence flags marking what was extracted vs. inferred vs. needs-verification

Hand this combined data to Step 4 in `SKILL.md` for synthesis into `DESIGN.md`. Use [../design-md-format.md](../design-md-format.md) as the schema.

## Cost and timing notes

| Tier | Typical cost (free tier) | Typical wall time |
|---|---|---|
| 1.1 css-color-extractor | $0.04 | ~10s |
| 1.2 css-stats-analyzer | $0.04 | ~10s |
| 1.3 tech-stack-scanner | $0 (free actor) | ~15s |
| 1.4 Exa web_fetch | $0 (free tier 1k/mo) | ~5s |
| 2a website-content-crawler | $0 (free actor) | ~30s |
| 2b web-scraper | $0 (free actor) | ~45s |

Total wall time for a complete extraction (Tier 1 + 1.5 + 2a + 2b): about 90–120 seconds. Total cost: roughly $0.08 on free tier, often less.

## Failure modes and recovery

| Failure | Cause | Recovery |
|---|---|---|
| `"requires full access to your account"` from `apify/web-scraper` | One-time approval not yet granted | Direct user to Apify Console, retry. See [apify-actor-approval.md](apify-actor-approval.md). |
| Color extractor returns empty array | Site is JS-rendered (Framer, dynamic Webflow) — static CSS doesn't carry tokens | Skip to Tier 2b directly, use computed-style extraction for colors too |
| Tech-stack scanner returns empty | Site blocks scrapers or is CDN-cached aggressively | Continue without framework data; mark `tech_stack: needs-verification` |
| website-content-crawler memory error | Default request is 8192MB, exceeds free tier | Pass `callOptions: { memory: 1024 }` |
| All tiers return empty | Site is heavily anti-bot or behind auth | Tell the user, suggest Path D (Q&A) instead |

If two tiers fail in a row, stop the pipeline and tell the user honestly. Do not synthesize a `DESIGN.md` from imagination.

## Worked example — recent run

Recent end-to-end run against a B2B agency marketing site (Astro + Sanity + Tailwind stack):

1. Tier 1.1 → 31 colors extracted with full hex+usage map, including a single dominant brand voltage.
2. Tier 1.2 → empty (site uses Tailwind utility classes, not `@font-face`).
3. Tier 1.3 → Astro + Sanity CMS + Tailwind + Cloudflare + Alpine.js + Preact.
4. Tier 1.4 → ~66KB of clean markdown including hero copy and CTA patterns.
5. Tier 1.5 → Synthesized voice paragraph capturing the site's register (confident, customer-success-driven).
6. Tier 2a → Class counts: `font-serif` 209x, `font-sans` 3x. Misleading signal — would have produced wrong "serif-dominant" reading.
7. Tier 2b → Computed `font-family` showed both heading and body were sans-serif. Tailwind `font-serif` aliased to a sans typeface. Trap caught.
8. DESIGN.md synthesized with all extracted tokens, no `needs-verification` flags.

This run took about 100 seconds and cost about $0.08. See [../examples/northwind-studio-DESIGN.md](../examples/northwind-studio-DESIGN.md) for a fictional reference DESIGN.md showing the resulting schema.
