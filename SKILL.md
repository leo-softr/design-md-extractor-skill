---
name: building-design-md
description: >
  Step 1 of the Softr brand-to-blocks workflow. Extracts a brand foundation (colors,
  typography, voice, marketing-page patterns) from a website URL, awesome-design-md
  catalog link, uploaded DESIGN.md file, or guided Q&A, then scaffolds Softr-specific
  app patterns (modals, empty states, status pills, page layouts, etc.) using the
  extracted brand tokens. Output is a DESIGN.md tailored for downstream consumption
  by the `softr-vibe-coding` skill — not a generic cross-framework design system.
  Use when starting a new Softr client project, or when asked to "extract brand from
  website", "generate a DESIGN.md", "build a design system", "set up brand colors and
  fonts", "create a brand kit from this URL", or pastes a website URL with intent to
  capture design tokens for a Softr app.
---

# Building Design MD Skill

This skill is **Step 1 of the Softr workflow**:

```
New client → building-design-md (this skill) → softr-vibe-coding → shipped Softr blocks
```

It produces a single canonical artifact — a `DESIGN.md` file in the project folder — that the `softr-vibe-coding` skill consumes downstream to generate brand-aligned Softr Vibe Coding blocks (custom JSX components inside a Softr app). The output is **deliberately Softr-specific** — it bakes in Softr's tech stack (shadcn/ui, Tailwind, lucide-react, sonner, date-fns) and Softr's bundler constraints (no `?.` or `??`, named React imports only, etc.) — and is not intended as a portable design system for other frameworks.

This skill captures and scaffolds. The `softr-vibe-coding` skill applies and refines as real screens get built.

## Important: foundation, not finished system

A `DESIGN.md` produced by this skill is a **starting point**, not a complete app design system. The skill captures what's actually present in the source (a marketing site, a brand guide, a Q&A) — it cannot invent application patterns that don't exist there. Modals, status pills, empty states, page-header lockups, dashboards, and similar app-specific patterns are **scaffolded with brand-token defaults using Softr's component library** (shadcn/ui), but require team refinement during real implementation.

Every generated `DESIGN.md` opens with a preamble banner that states this explicitly so downstream readers don't mistake the file for a complete spec. See Step 4 for the banner template and Step 4b for the app-patterns scaffolding.

## What the skill produces

By the end of a successful run, the project folder contains:

1. **`./DESIGN.md`** — the canonical brand artifact (YAML frontmatter + prose sections), pre-tagged with Softr's tech stack so `softr-vibe-coding` can pick it up without re-asking.
2. **`./custom-code-header.html`** — a snippet the user pastes into Softr's `Settings → Custom Code → Code inside header` to load Google Fonts and brand CSS variables once for the whole app (with `!important` on the `html, body` and `h1–h6` rules so brand fonts win against Softr's theme CSS).
3. **`./assets/`** (Path A only) — brand assets downloaded from the source. Currently the logo wordmark (SVG preferred, PNG/JPG/WebP fallback). Both the **public CDN URL** (canonical reference for production blocks — Softr cannot resolve project-relative paths) and the **local backup file** (for manual upload to Softr's media library) are recorded in DESIGN.md's `assets` block.

Both files (and the assets folder) are overwritten if they already exist. The skill warns the user before overwriting.

## When this skill should run

Run when the user has signalled intent to set up or capture a brand for a **new Softr project**. Typical triggers:

- "Extract the brand from `https://example.com`."
- "Build me a DESIGN.md from this site."
- "Set up the design tokens for this Softr project."
- "I want a brand kit for `<url>`, then we'll build the Softr blocks."
- "What does Linear's design system look like? Save it."

Do **not** run this skill when the user is asking to generate or modify a Softr block, a React component, or any other piece of UI. That's `softr-vibe-coding`'s job — and it expects the DESIGN.md to already exist. This skill ends when `DESIGN.md` is written; the user then runs `softr-vibe-coding` to build the blocks.

## Workflow — checklist Claude should track

Copy this checklist at the start of each invocation and check items off:

- [ ] Step 0 — Check for existing `DESIGN.md` in project folder
- [ ] Step 1 — Ask the gate question (custom design? yes/no)
- [ ] Step 2 — If yes, ask the path question (URL / catalog / file / Q&A)
- [ ] Step 3 — Run the chosen path (load the relevant extractor reference)
- [ ] Step 3b — (Path A only) Extract brand assets — download the logo (SVG preferred, PNG/JPG/WebP fallback) into `./assets/`, capture both the public CDN URL and the local backup path
- [ ] Step 4 — Synthesize `DESIGN.md` from the extracted/provided tokens, with the preamble banner
- [ ] Step 4b — Append the Application Patterns scaffold using brand-token defaults
- [ ] Step 5 — Mark every uncertain token with a confidence flag
- [ ] Step 6 — Run the verification prompt for any flagged tokens
- [ ] Step 7 — Generate `custom-code-header.html`
- [ ] Step 8 — Tell the user what was written, what to do next, and how to evolve the file

### Step 0 — Discover existing context

Look for `./DESIGN.md` in the project folder. If present:

> "I found an existing `DESIGN.md` in this project. Do you want to:
> 1. Use it as is (skip extraction)
> 2. Replace it (start fresh)
> 3. Update specific tokens (treat existing as a base, fill in flagged values)"

If the user picks (1), exit cleanly. If (2), proceed to Step 1. If (3), load the existing file and skip to Step 6 (verification of flagged tokens only).

### Step 1 — The gate question

Ask exactly:

> "Do you want to apply a custom design to this project? (yes / no)"

- **No** → Write a minimal `DESIGN.md` with `style: premium-default` and the line "User opted out of custom design. Downstream tools should apply their own default style." Skip to Step 7.
- **Yes** → Proceed to Step 2.

Treat "no" as a first-class choice. Many users want the downstream tool's default aesthetic and should not be pushed into customization.

### Step 2 — The path question

Ask exactly:

> "How would you like to provide the design?
> A. Extract from a website URL (recommended for real brands)
> B. Use a pre-built file from the awesome-design-md catalog (fastest if your brand is already there)
> C. Upload an existing `DESIGN.md` file
> D. Guided Q&A (no extraction; I'll ask 8 questions)"

Wait for the user to pick A / B / C / D.

### Step 3 — Run the chosen path

Load the relevant reference and follow it:

| Path | Reference to load |
|---|---|
| A — URL extraction | [extractors/apify-pipeline.md](extractors/apify-pipeline.md) |
| B — Catalog | [extractors/awesome-design-md-catalog.md](extractors/awesome-design-md-catalog.md) |
| C — Upload | [design-md-format.md](design-md-format.md) (validate the uploaded file against the schema) |
| D — Q&A | [extractors/guided-qa.md](extractors/guided-qa.md) |

Each reference returns a structured set of tokens (colors, typography, etc.) plus per-token confidence flags. Hand those to Step 4.

### Step 3b — (Path A only) Extract brand assets

After token extraction completes for Path A, run an asset-extraction pass. The goal is to get the brand's logo into `./assets/` and its public CDN URL into `DESIGN.md` so downstream Softr blocks have both: a public URL that Softr can hot-link, and a local backup the user can manually upload to Softr's media library if they prefer to host on Softr's own CDN.

**Why both URL and file matter:** Softr's runtime cannot resolve project-relative paths (`./assets/foo.svg`) inside Vibe Coding blocks — image `src` attributes must be absolute URLs. The public CDN URL is therefore the canonical reference for production code. The local file exists as a backup for the case where the source CDN URL becomes unstable (re-published, expired token, CMS migration) or the user wants to self-host.

**Procedure** (Path A only — paths B/C/D handle assets through their own routes):

1. Run a targeted Apify `apify/web-scraper` page function that returns all `<img>` URLs along with `alt` text, dimensions, and class names. Reuse the Tier 2b infrastructure with this `pageFunction`:

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

2. Identify the **logo wordmark** from the returned image list using these heuristics, in order:

   - `alt` text contains the brand name (case-insensitive substring match)
   - `alt` text contains "logo"
   - `src` contains "logo" (filename or path)
   - First non-payment-method, non-icon image on the page (filter out `payment-method`, `icon`, `favicon`, `social` class names and small dimensions ≤ 64×64)

   If multiple candidates remain, prefer SVG over raster, then highest resolution.

3. Also check the inline SVG list — some brands ship the wordmark as inline `<svg>` rather than `<img>`. If a logo SVG is found inline, save its `outerHTML` to `./assets/<brand-slug>-wordmark.svg` and treat the live page URL itself as the "public URL" placeholder (record this with a note that hot-linking inline SVG isn't possible — the user should self-host).

4. **Format preference** — when multiple sources for the same logo exist, prefer in this order:

   1. **SVG** (vector, scales perfectly, smallest file)
   2. **PNG with transparency** (raster, lossless, supports alpha)
   3. **WebP** (raster, smaller than PNG but less universal browser support inside email clients)
   4. **JPG** (raster, no transparency — last resort)

5. **Download** the chosen logo to `./assets/<brand-slug>-wordmark.<ext>` using the file extension from the source URL or the response `Content-Type` header.

   - If the source URL has query parameters (e.g., `?width=2400&optimize=medium` from Weebly's CDN), strip them when constructing the local filename but **keep them in the recorded `public_url`** — they may matter for the CDN to serve the right variant.
   - Use `curl -sSL -o <path> <url>` for the download.
   - Create `./assets/` if it doesn't exist (`mkdir -p`).

6. **Record both values** in DESIGN.md's `assets` block (see [design-md-format.md](design-md-format.md) for the schema):

   ```yaml
   assets:
     status: extracted
     logo_wordmark:
       public_url: "<canonical CDN URL — what blocks should reference>"
       public_url_note: "Canonical reference for production use. Softr cannot resolve project-relative paths."
       local_backup: "./assets/<brand-slug>-wordmark.<ext>"
       local_backup_note: "Downloaded copy for manual upload to Softr's media library if/when you'd rather host on Softr's own CDN."
       format: "<SVG | PNG | WebP | JPG> — <one-line note about why this format>"
       color: "<dominant logo color, ideally one of the extracted color tokens>"
       usage: "<short usage hint, e.g., 'Use as <img src=\"<public_url>\" alt=\"<brand>\"/> in nav lockups.'>"
   ```

7. **If asset extraction fails** (no plausible logo image found, page blocks scraping, etc.), set `assets.status: needs-verification` and record an empty `logo_wordmark` block with a note in `Known Gaps in This Extraction`. Surface this in the Step 6 verification prompt — ask the user to paste the logo URL manually.

8. **Other assets** (hero photos, decorative imagery, brand pattern fills) are **not extracted by default**. Only the logo wordmark is universally needed for downstream blocks. If the source page has a clearly-branded hero image and the user has signalled they want it captured, follow the same pattern (download + record `public_url` + `local_backup`).

This step does not run for Paths B/C/D:

- **Path B (catalog)** — the catalog entry already includes its own asset references; reuse them as-is.
- **Path C (upload)** — the uploaded `DESIGN.md` either contains an `assets` block or doesn't; if missing, prompt the user during Step 6 verification.
- **Path D (Q&A)** — the Q&A explicitly asks for the logo URL as one of its 8 questions; record what the user provides without download (the user can drop the file into `./assets/` themselves).

### Step 4 — Synthesize `DESIGN.md` (with preamble)

Use [design-md-format.md](design-md-format.md) as the schema. Populate:

- **Preamble banner** at the very top of the file, above the YAML frontmatter (see template below). This is required on every generated DESIGN.md, no exceptions.
- **YAML frontmatter:** name, description, extraction_status, tech_stack, colors, typography, rounded, spacing, elevation, components, assets (Path A only — see Step 3b).
- **Prose sections:** Overview, Voice & Copy Register, Colors, Typography, Layout, Elevation, Components, Do's and Don'ts, Known Gaps in This Extraction.

Write to `./DESIGN.md`. Do not paraphrase — use the exact field names and section headings from the template.

**Preamble banner template** (paste at the very top, before any frontmatter):

```markdown
> ⚠️ **Softr-specific brand foundation, not a complete app design system.**
>
> Extracted from `<source>` on `<YYYY-MM-DD>` for use in a Softr app. The
> colors, typography, voice, and marketing-page components below come directly
> from the source and are ready to use. **Application patterns** (modals,
> status pills, empty/loading/error states, page-header lockups, form-field
> cards, etc.) are not present in the source and have been **scaffolded with
> brand-token defaults using Softr's stack** (shadcn/ui, Tailwind, lucide-react,
> sonner, date-fns) in § Application Patterns at the bottom. Treat those
> scaffolds as starting points — refine them when you build the Softr blocks
> that use them.
>
> **Next step**: paste `custom-code-header.html` into Softr → Settings → Custom
> Code → Code inside header, then run the `softr-vibe-coding` skill to generate
> brand-aligned Vibe Coding blocks from this file.
>
> When you've built 2–3 real blocks, capture the patterns that emerged and
> promote them out of "scaffolded" status. (See § Evolving this file.)

---
```

Substitute `<source>` with the actual input (URL for Path A, catalog entry for Path B, "uploaded `<filename>`" for Path C, "guided Q&A" for Path D) and `<YYYY-MM-DD>` with today's date.

### Step 4b — Append Application Patterns scaffold

Load [references/app-patterns-stubs.md](references/app-patterns-stubs.md) and append the entire Application Patterns section after the Components section in the prose body. Substitute every `{colors.x}` / `{typography.x}` / `{rounded.x}` reference with the actual extracted token names so the stubs are immediately usable.

The scaffold covers the categories that are nearly universal in app UI but cannot be extracted from a marketing source:

- Page Header lockup
- Dialog / Modal anatomy (header, body, footer)
- Alert Dialog (destructive / positive confirmation)
- Lightbox / media viewer
- Pill family (status, summary count, email-avatar)
- Section card with header strip
- Form field card
- Search input + filter row + reset button
- Eyebrow labels (section + in-card)
- Meta rows (label-on-top, value-below)
- Empty / loading / error states (with skeleton shape table)
- Document icons (PDF / image / file color convention)
- Interaction states (hover / focus / disabled / loading-button)
- Toast / notification convention

Each stub is marked `status: "scaffolded"` (a fourth confidence flag — see Step 5). Each section ends with a one-line "Refine when you build:" hint pointing to the screen type that will need it.

Do not skip this step. It is the highest-leverage difference between a `DESIGN.md` that gets used and one that gets reverse-engineered from the JSX six weeks later.

### Step 5 — Mark every uncertain token

For each token in the YAML frontmatter, set `status` to one of four values per [references/confidence-flags.md](references/confidence-flags.md):

- `extracted` — Pulled directly from the source. High confidence.
- `inferred` — Derived from indirect signals (e.g. class counts, framework detection). Medium confidence; may be wrong.
- `needs-verification` — Could not be captured. The skill must prompt the user before locking.
- `scaffolded` — App pattern stub generated in Step 4b using brand-token defaults. Not extracted from the source, not yet refined by the team. Indicates "starting point — revisit when building."

Always set `status` explicitly. Never leave it blank.

`scaffolded` tokens do **not** trigger the verification prompt in Step 6 (they're intentional placeholders the team will refine later). Only `inferred` and `needs-verification` tokens trigger that prompt.

### Step 6 — Verification prompt

If any tokens are flagged `needs-verification` or `inferred`, ask the user one consolidated prompt:

> "I extracted X confidently but couldn't fully confirm:
> - `<token name>` (currently: `<inferred value>`, status: `<flag>`)
> - …
> Paste the correct value, confirm the inferred default, or say 'skip' to leave the flag in place."

Update `./DESIGN.md` in place with whatever the user provides. Do not re-run extractors — just edit the file.

### Step 7 — Generate `custom-code-header.html`

Use [references/custom-code-header.md](references/custom-code-header.md) to build the snippet. It should contain:

- Google Fonts `<link>` tags for any fonts in the DESIGN.md that are available on Google Fonts.
- A `<style>` block defining `:root` CSS custom properties for brand tokens (`--brand-primary`, `--brand-text`, `--brand-radius-md`, etc.).
- A single `font-family` rule on `html, body` if the brand specifies a body font.

Save to `./custom-code-header.html`.

### Step 8 — Final report to the user

Tell the user:

1. What was written and where (full paths, computer:// links if relevant).
2. Which tokens are confidence-flagged and may need their attention. Call out separately: how many tokens are `extracted`, `inferred`, `needs-verification`, and `scaffolded` so they understand the file's mix.
3. **The Softr workflow handoff** — the exact next step:
   > "Step 1 (this skill) is done. To continue:
   > 1. Paste `custom-code-header.html` into your Softr app: **Settings → Custom Code → Code inside header**, then save and publish.
   > 2. Run the **`softr-vibe-coding`** skill to generate brand-aligned Vibe Coding blocks from this DESIGN.md. Trigger it with something like 'build me a Softr block for X' or 'create a claims dashboard.' The skill auto-reads `./DESIGN.md` and applies the tokens.
   >    - *Not installed yet?* Run `npx softr-vibe-coding@latest init` in your terminal. It installs the skill into `~/.claude/skills/` and keeps it auto-updated on each Claude Code session.
   >
   > That's the whole pipeline: brand → blocks. No other skills needed."
4. The evolution path:
   > "After you've built 2–3 real Softr blocks, the patterns you actually shipped will diverge from the scaffolds. At that point, refine the affected `scaffolded` tokens in DESIGN.md to match what you built and promote them to `extracted`. A future companion skill (`building-design-md-refine`) will automate this by reading your block JSX and proposing refinements."

End the skill there. Do not generate any UI components — that's `softr-vibe-coding`'s job, and running it is the user's next explicit action.

## Constraints (do these / never these)

**Do:**

- Always ask the gate question (Step 1) before running any extractor. The user must opt in.
- Always set `status` flags explicitly on every token.
- Always save to the project folder, never to the temporary outputs directory.
- Always run the verification prompt (Step 6) when flags fire — do not silently leave `needs-verification` tokens in the file.
- Always treat the awesome-design-md catalog as the fastest path when applicable — check for catalog availability before running Apify.

**Never:**

- Never invent a hex color, font name, or CSS value without flagging it. If extraction failed, mark it `needs-verification` and prompt.
- Never run extraction tools (Apify, Exa) before the user picks Path A. Paths B/C/D do not need them.
- Never produce Softr Vibe Coding blocks, JSX, or any UI code. That is a different skill's job.
- Never overwrite an existing `DESIGN.md` without asking the user first (Step 0).
- Never paste API keys (Apify, Exa) into `DESIGN.md` or `custom-code-header.html`. Keys live in MCP config only.
- Never claim the brand is one thing (e.g. "serif-dominant") based on Tailwind class counts alone. Tailwind utility names lie. See [references/tailwind-class-trap.md](references/tailwind-class-trap.md).

## When to load references

Reference files are loaded on demand. Load them when the indicated condition is true:

| File | When to load |
|---|---|
| [design-md-format.md](design-md-format.md) | Always, before writing `DESIGN.md` (Step 4). |
| [references/app-patterns-stubs.md](references/app-patterns-stubs.md) | Always, before appending the Application Patterns section (Step 4b). |
| [references/intake-flow.md](references/intake-flow.md) | If the user's intent is unclear and you need to disambiguate. |
| [extractors/apify-pipeline.md](extractors/apify-pipeline.md) | When user picks Path A (Step 3A). |
| [extractors/apify-mcp-install.md](extractors/apify-mcp-install.md) | When the Apify MCP is not connected and Path A is requested. |
| [extractors/apify-actor-approval.md](extractors/apify-actor-approval.md) | When `apify/web-scraper` returns "requires full access". |
| [extractors/exa-mcp.md](extractors/exa-mcp.md) | When voice / mood / copy-register extraction is requested. |
| [extractors/awesome-design-md-catalog.md](extractors/awesome-design-md-catalog.md) | When user picks Path B. |
| [extractors/guided-qa.md](extractors/guided-qa.md) | When user picks Path D. |
| [references/confidence-flags.md](references/confidence-flags.md) | When setting `status` on any token (Step 5). |
| [references/tailwind-class-trap.md](references/tailwind-class-trap.md) | When the target site is Tailwind-based (detected via tech-stack scanner). |
| [references/google-fonts-substitutes.md](references/google-fonts-substitutes.md) | When the brand specifies a paid / licensed font. |
| [references/custom-code-header.md](references/custom-code-header.md) | Always, in Step 7. |

## Output examples

Concrete examples of what the skill produces — not abstract rules.

**Path A success (URL extraction worked end to end):**

```yaml
extraction_status:
  colors: complete (via Apify automation-lab/css-color-extractor)
  framework: complete (via Apify misterkhan/website-tech-stack-scanner)
  voice_and_mood: complete (via Exa web_fetch_exa)
  typography_specific_fonts: complete (via Apify apify/web-scraper Tier 2b)
  spacing_radius_shadows: needs-verification
```

User report: "DESIGN.md written. Color, framework, voice, and typography all extracted directly from the live site. Spacing/radius/shadow tokens are inferred from typical Astro+Tailwind patterns and need your spot-check. Run `/verify-brand` if you want to confirm them now."

**Path C success (user uploaded their own DESIGN.md):**

```yaml
extraction_status:
  colors: extracted (from uploaded file)
  typography: extracted (from uploaded file)
  voice_and_mood: needs-verification (not present in uploaded file)
```

User report: "Uploaded `airbnb-DESIGN.md` parsed against the schema. Colors and typography are good. Voice section was missing — fill it in manually or paste a hero paragraph from the brand and I'll synthesize it."

**Path D fallback (user has nothing):**

```yaml
extraction_status:
  colors: extracted (from Q&A)
  typography: extracted (from Q&A — single brand color, soft radius, system sans, flat shadow vibe)
  voice_and_mood: extracted (from Q&A — mood: friendly-consumer)
  spacing_radius_shadows: inferred (from radius vibe)
```

User report: "Built `DESIGN.md` from your 8 answers. This is a minimum-viable brand kit; for higher fidelity, run me again with a website URL or paste your full brand guide."

## Skill ends

The skill always ends after Step 8. It does not produce blocks, code, or screenshots. The user runs a separate skill for those.
