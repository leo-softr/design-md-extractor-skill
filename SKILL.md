---
name: building-design-md
description: >
  Brand-foundation step of the Softr brand-to-blocks workflow. Drives the dembrandt
  design-token extractor (real-browser crawl of a website URL) — or a catalog file,
  an uploaded DESIGN.md, or guided Q&A — to produce a `./DESIGN.md`, then layers on
  what raw extraction can't provide: voice & copy register, resolved font names,
  logo assets, Softr app-pattern scaffolds (modals, empty states, status pills,
  page layouts), and a `custom-code-header.html` for app-level brand inheritance.
  Output is a DESIGN.md tailored for downstream consumption by the
  `softr-vibe-coding` skill. Use when starting a new Softr client project, or when
  asked to "extract brand from website", "generate a DESIGN.md", "build a design
  system", "set up brand colors and fonts", "create a brand kit from this URL", or
  the user pastes a website URL with intent to capture design tokens for a Softr
  app. (For a quick raw DESIGN.md mid-build with none of the layers, softr-vibe-coding's
  own Step 1 drives dembrandt directly — this skill is the thorough session.)
---

# Building Design MD Skill

This skill is the **brand-foundation step of the Softr workflow**:

```
New client → building-design-md (dembrandt → DESIGN.md + layers) → softr-vibe-coding → shipped Softr blocks
```

It produces a single canonical artifact — a `DESIGN.md` file in the project folder — that the `softr-vibe-coding` skill consumes downstream to generate brand-aligned Softr Vibe Coding blocks (custom TSX/JSX components inside a Softr app). Since v2.0.0, the token extraction engine is **[dembrandt](https://github.com/dembrandt/dembrandt)** (real-browser computed-style extraction), and the file's base layer is dembrandt's own DESIGN.md output — this skill's value is everything layered on top: voice, resolved fonts, assets, app-pattern scaffolds, verification, and the Softr custom-code snippet. The output is **deliberately Softr-specific** — it bakes in Softr's tech stack (shadcn/ui, Tailwind, lucide-react, sonner, date-fns; modern TypeScript compiles on the current platform) — and is not intended as a portable design system for other frameworks.

This skill captures and scaffolds. The `softr-vibe-coding` skill applies and refines as real screens get built. (softr-vibe-coding's Step 1 can also generate a quick *raw* dembrandt DESIGN.md mid-build; run this skill when the project deserves the full foundation.)

## Important: foundation, not finished system

A `DESIGN.md` produced by this skill is a **starting point**, not a complete app design system. The skill captures what's actually present in the source (a marketing site, a brand guide, a Q&A) — it cannot invent application patterns that don't exist there. Modals, status pills, empty states, page-header lockups, dashboards, and similar app-specific patterns are **scaffolded with brand-token defaults using Softr's component library** (shadcn/ui), but require team refinement during real implementation.

Every generated `DESIGN.md` opens with a preamble banner that states this explicitly so downstream readers don't mistake the file for a complete spec. See Step 4 for the banner template and Step 4b for the app-patterns scaffolding.

## What the skill produces

By the end of a successful run, the project folder contains:

1. **`./DESIGN.md`** — the canonical brand artifact: dembrandt's emitted document as the base (kept verbatim), plus this skill's augmentation layers (banner, `tech_stack`, `fonts`, `assets`, `extraction_status`, Voice & Copy Register, Do's and Don'ts, Application Patterns, Known Gaps). Full anatomy: [design-md-format.md](design-md-format.md).
2. **`./custom-code-header.html`** — a snippet the user pastes into Softr's `Settings → Custom Code → Code inside header` to load brand fonts and CSS variables once for the whole app (with `!important` so brand rules win against Softr's theme CSS).
3. **`./assets/`** (Path A only) — the brand logo downloaded from the source. Both the **public CDN URL** (canonical reference for production blocks — Softr cannot resolve project-relative paths) and the **local backup file** (for manual upload to Softr's media library) are recorded in DESIGN.md's `assets` block.

All of these are overwritten if they already exist. The skill warns the user before overwriting.

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
- [ ] Step 3b — (Path A only) Extract brand assets — pull the logo from dembrandt's extraction JSON, download it (SVG preferred, PNG/WebP/JPG fallback) into `./assets/`, capture both the public CDN URL and the local backup path
- [ ] Step 4 — Assemble `DESIGN.md`: dembrandt base + augmentation layers, with the preamble banner
- [ ] Step 4b — Append the Application Patterns scaffold using brand-token defaults
- [ ] Step 5 — Set section-level confidence flags in `extraction_status`
- [ ] Step 6 — Run the verification prompt for any flagged sections
- [ ] Step 7 — Generate `custom-code-header.html`
- [ ] Step 8 — Tell the user what was written, what to do next, and how to evolve the file

### Step 0 — Discover existing context

Look for `./DESIGN.md` in the project folder. If present:

> "I found an existing `DESIGN.md` in this project. Do you want to:
> 1. Use it as is (skip extraction)
> 2. Replace it (start fresh)
> 3. Update specific tokens (treat existing as a base, fill in flagged values)"

If the user picks (1), exit cleanly. If (2), proceed to Step 1 — and if the existing file is a pre-v2 one (legacy schema — see [design-md-format.md](design-md-format.md#legacy-note-v1-files)), warn that hand-refined scaffolds and verified tokens in it are not carried over automatically. If (3), load the existing file and skip to Step 6 (verification of flagged sections only).

### Step 1 — The gate question

Ask exactly:

> "Do you want to apply a custom design to this project? (yes / no)"

- **No** → Write a minimal `DESIGN.md` with `style: premium-default` and the line "User opted out of custom design. Downstream tools should apply their own default style." Skip to Step 7.
- **Yes** → Proceed to Step 2.

Treat "no" as a first-class choice. Many users want the downstream tool's default aesthetic and should not be pushed into customization.

### Step 2 — The path question

Ask exactly:

> "How would you like to provide the design?
> A. Extract from a website URL with dembrandt (recommended for real brands)
> B. Use a pre-built file from the awesome-design-md catalog (fastest if your brand is already there)
> C. Upload an existing `DESIGN.md` file
> D. Guided Q&A (no extraction; I'll ask 8 questions)"

Wait for the user to pick A / B / C / D.

### Step 3 — Run the chosen path

Load the relevant reference and follow it:

| Path | Reference to load |
|---|---|
| A — URL extraction (dembrandt) | [extractors/dembrandt-pipeline.md](extractors/dembrandt-pipeline.md) |
| B — Catalog | [extractors/awesome-design-md-catalog.md](extractors/awesome-design-md-catalog.md) |
| C — Upload | [design-md-format.md](design-md-format.md) (validate the uploaded file against the schema) |
| D — Q&A | [extractors/guided-qa.md](extractors/guided-qa.md) |

Path A returns the dembrandt base document, the full extraction JSON, lint findings, resolved font names, and a voice paragraph. Paths B/C/D return token sets in their own ways; map them into the same dembrandt-shaped frontmatter keys (see [design-md-format.md](design-md-format.md)) so every output has one format. Hand everything to Step 4.

### Step 3b — (Path A only) Extract brand assets

Get the brand's logo into `./assets/` and its public CDN URL into `DESIGN.md` so downstream Softr blocks have both: a public URL that Softr can hot-link, and a local backup the user can upload to Softr's media library if they prefer.

**Why both URL and file matter:** Softr's runtime cannot resolve project-relative paths (`./assets/foo.svg`) inside Vibe Coding blocks — image `src` attributes must be absolute URLs. The public CDN URL is therefore the canonical reference for production code. The local file is the backup for when the source URL becomes unstable (re-published, expired token, CMS migration) or the user wants to self-host.

**Procedure** (Path A only — paths B/C/D handle assets through their own routes):

1. Read the **`logo`, `logoInstances`, and `favicons` keys from dembrandt's extraction JSON** (already in hand from Step 3 — no separate scrape). They carry source URLs, dimensions, `type` (`wordmark` / `logomark` / `combination`), alt text, and safe-zone data.

2. Pick the **logo wordmark**: prefer the entry dembrandt typed as `wordmark`; among alternatives, prefer alt/filename containing the brand name, then "logo". Never auto-save the icon mark (`logomark`) as the primary asset.

3. **Format preference** — when multiple sources for the same logo exist, prefer in this order:

   1. **SVG** (vector, scales perfectly, smallest file)
   2. **PNG with transparency** (raster, lossless, supports alpha)
   3. **WebP** (raster, smaller than PNG but less universal support)
   4. **JPG** (raster, no transparency — last resort)

4. **Download** the chosen logo to `./assets/<brand-slug>-wordmark.<ext>` (`mkdir -p ./assets` first; `curl -sSL -o <path> <url>`). If the source URL has query parameters (e.g. `?width=2400&optimize=medium`), strip them from the local filename but **keep them in the recorded `public_url`** — the CDN may need them to serve the right variant.

5. **Record both values** in DESIGN.md's `assets` block per the schema in [design-md-format.md](design-md-format.md#assets-path-a-only).

6. **If it fails** (no plausible logo in the extraction, download blocked): apply the failure-modes table in [extractors/dembrandt-pipeline.md](extractors/dembrandt-pipeline.md#logo--assets-from-the-extraction-json), and if still stuck set `assets.status: needs-verification`, record an empty `logo_wordmark`, note it in `Known Gaps`, and ask for the logo URL in the Step 6 prompt.

7. **Other assets** (hero photos, decorative imagery, brand pattern fills) are **not extracted by default**. Only the logo wordmark is universally needed. If the source has a clearly-branded hero image and the user asked for it, follow the same download + record pattern.

### Step 4 — Assemble `DESIGN.md` (base + layers, with preamble)

Use [design-md-format.md](design-md-format.md) as the spec. Assembly order:

1. **Preamble banner** at the very top, above the YAML frontmatter (template below). Required on every generated DESIGN.md, no exceptions.
2. **Base document**: dembrandt's `generate_design_md` output, verbatim — frontmatter keys and body sections untouched. One permitted edit: keep `description`'s "Design tokens extracted from `<url>`" as the first sentence and **append 2–3 brand-mood sentences** after it.
3. **Appended frontmatter keys**: `version: 2`, `target_platform`, `downstream_skill`, `tech_stack`, `fonts` (real font names resolved from the body's Font URLs — never trust computed `fontFamily` blindly), `assets` (Path A), `extraction_status`.
4. **Appended body sections**: Voice & Copy Register, Do's and Don'ts, Application Patterns (Step 4b), Known Gaps in This Extraction, Evolving this file. If extraction caught a misleading signal (a lying utility class, a fallback-masked font name), document the catch in the Typography prose so downstream consumers stay honest.

Write to `./DESIGN.md`. Do not paraphrase the base — it ships as emitted. Mechanically, the merge is: banner first, then ONE frontmatter block — the base's opening `---`, its keys, the appended keys, one closing `---` (never two stacked frontmatter blocks) — then the base's body sections, then the appended body sections after the base's last section.

**Preamble banner template** (paste at the very top, before any frontmatter):

```markdown
> ⚠️ **Softr-specific brand foundation, not a complete app design system.**
>
> Extracted from `<source>` on `<YYYY-MM-DD>` for use in a Softr app. The
> colors, typography, spacing, and components below were observed by dembrandt
> on the source and are ready to use. **Application patterns** (modals,
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

Load [references/app-patterns-stubs.md](references/app-patterns-stubs.md) and append the entire Application Patterns section after the appended `## Do's and Don'ts` section, before `## Known Gaps in This Extraction` (per the anatomy in [design-md-format.md](design-md-format.md)). Substitute every `{colors.x}` / `{typography.x}` / `{rounded.x}` reference with the file's actual token names — dembrandt vocabulary: `{colors.primary}`, `{colors.on-surface}`, `{colors.surface}`, `{typography.headline-md}`, `{typography.label-lg}`, `{rounded.lg}`, `{rounded.full}` — so the stubs are immediately usable.

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

Each stub is marked `status: "scaffolded"`. Each section ends with a one-line "Refine when you build:" hint pointing to the screen type that will need it.

Do not skip this step. It is the highest-leverage difference between a `DESIGN.md` that gets used and one that gets reverse-engineered from the JSX six weeks later.

### Step 5 — Set section-level confidence flags

Fill `extraction_status` with one flag per section, per [references/confidence-flags.md](references/confidence-flags.md):

- `extracted` — observed directly (dembrandt tokens; hero copy quoted from the live site; logo from the extraction JSON). High confidence.
- `inferred` — derived from indirect signals. Medium confidence; may be wrong.
- `needs-verification` — could not be captured. The skill must prompt the user before locking.
- `scaffolded` — app pattern stubs generated in Step 4b. Intentional placeholders — the team refines them later.
- `partial` — section-level only: some values observed, others inferred or missing. Itemize the gaps in `Known Gaps`; fires the optional (inferred-style) Step 6 prompt.

Dembrandt frontmatter tokens carry **no per-token status** (they are observed values by definition; restructuring them would break the base shape) — confidence lives at section level, with specifics itemized in `Known Gaps`. Always set every `extraction_status` entry explicitly; never leave one blank.

`scaffolded` sections do **not** trigger the verification prompt in Step 6. Only `inferred`, `partial`, and `needs-verification` do.

### Step 6 — Verification prompt

If any section is flagged `needs-verification`, `inferred`, or `partial`, ask the user one consolidated prompt:

> "I extracted X confidently but couldn't fully confirm:
> - `<section / token>` (currently: `<value or guess>`, status: `<flag>`)
> - …
> Paste the correct value, confirm the inferred default, or say 'skip' to leave the flag in place."

Include anything severe from dembrandt's `get_findings` lint here too (contrast failures, token collisions). Update `./DESIGN.md` in place with whatever the user provides. Do not re-run the extraction — just edit the file.

### Step 7 — Generate `custom-code-header.html`

Use [references/custom-code-header.md](references/custom-code-header.md) to build the snippet. It should contain:

- Font loading for the fonts in the `fonts` block: Google Fonts `<link>` tags when the font is on Google Fonts, or the open-source substitute when it's paid.
- A `<style>` block defining `:root` CSS custom properties for brand tokens (`--brand-primary`, `--brand-on-surface`, `--brand-radius-md`, etc.) from the DESIGN.md `colors`/`rounded` blocks.
- `html, body` and `h1–h6` rules applying the brand fonts/colors with `!important` (Softr's theme CSS loads after this snippet).

Save to `./custom-code-header.html`.

### Step 8 — Final report to the user

Tell the user:

1. What was written and where (full paths).
2. Which sections are confidence-flagged and may need attention — how many are `extracted`, `inferred`, `needs-verification`, and `scaffolded`, so they understand the file's mix. Include dembrandt's findings scores when Path A ran.
3. **The Softr workflow handoff** — the exact next step:
   > "The brand foundation is done. To continue:
   > 1. Paste `custom-code-header.html` into your Softr app: **Settings → Custom Code → Code inside header**, then save and publish.
   > 2. Run the **`softr-vibe-coding`** skill to generate brand-aligned Vibe Coding blocks from this DESIGN.md. Trigger it with something like 'build me a Softr block for X' or 'create a claims dashboard.' The skill auto-reads `./DESIGN.md` and applies the tokens.
   >    - *Not installed yet?* Run `npx softr-vibe-coding@latest init` in your terminal. It installs the skill into `~/.claude/skills/` and keeps it auto-updated on each Claude Code session.
   >
   > That's the whole pipeline: brand → blocks."
4. The evolution path:
   > "After you've built 2–3 real Softr blocks, the patterns you actually shipped will diverge from the scaffolds. At that point, refine the affected `scaffolded` stubs in DESIGN.md to match what you built and promote them to `extracted`. To re-check the shipped app against the brand later, dembrandt's `compute_drift` can score the published Softr app against a fresh extraction of the brand site."

End the skill there. Do not generate any UI components — that's `softr-vibe-coding`'s job, and running it is the user's next explicit action.

## Constraints (do these / never these)

**Do:**

- Always ask the gate question (Step 1) before running any extraction. The user must opt in.
- Always set every `extraction_status` entry explicitly.
- Always save to the project folder, never to a temporary outputs directory.
- Always run the verification prompt (Step 6) when flags fire — do not silently leave `needs-verification` sections in the file.
- Always check the awesome-design-md catalog first when the brand might be there — a hit skips extraction entirely.
- Always crawl multiple pages on Path A (`pages: 3`–`5`) — single-page extractions are noisy.

**Never:**

- Never invent a hex color, font name, or CSS value without flagging it. If extraction failed, mark it `needs-verification` and prompt.
- Never run dembrandt extraction before the user picks Path A. Paths B/C/D never require it (guided Q&A may optionally use `get_color_palette` when the user offers a URL and dembrandt is already set up) — and never extract a site the user doesn't own or have permission to analyze.
- Never restructure the dembrandt base — no renaming its frontmatter keys, no per-token status objects inside them, no reordering its body sections. Additions only (plus the `description` mood-append).
- Never produce Softr Vibe Coding blocks, JSX, or any UI code. That is a different skill's job.
- Never trust computed `fontFamily` values blindly — a generic stack (`ui-sans-serif`) with real webfonts in Font URLs means the brand font is hiding. Resolve names via the Font URLs (see [extractors/dembrandt-pipeline.md](extractors/dembrandt-pipeline.md#resolving-real-font-names-the-computed-fontfamily-trap)).
- Never claim the brand is one thing (e.g. "serif-dominant") from class names or weak signals alone — only computed styles tell the truth; hedge and verify.
- Never copy v1's retired bundler constraints ("no `?.`", "no `??`") into `tech_stack` — the current platform compiles modern TypeScript (verified 2026-08-25).

## When to load references

Reference files are loaded on demand. Load them when the indicated condition is true:

| File | When to load |
|---|---|
| [design-md-format.md](design-md-format.md) | Always, before assembling `DESIGN.md` (Step 4). |
| [references/app-patterns-stubs.md](references/app-patterns-stubs.md) | Always, before appending the Application Patterns section (Step 4b). |
| [references/intake-flow.md](references/intake-flow.md) | If the user's intent is unclear and you need to disambiguate. |
| [extractors/dembrandt-pipeline.md](extractors/dembrandt-pipeline.md) | When user picks Path A (Step 3) — setup, extraction, fonts, voice, logo, failure handling. |
| [extractors/awesome-design-md-catalog.md](extractors/awesome-design-md-catalog.md) | When user picks Path B. |
| [extractors/guided-qa.md](extractors/guided-qa.md) | When user picks Path D. |
| [references/confidence-flags.md](references/confidence-flags.md) | When setting `extraction_status` (Step 5). |
| [references/google-fonts-substitutes.md](references/google-fonts-substitutes.md) | When the brand specifies a paid / licensed font. |
| [references/custom-code-header.md](references/custom-code-header.md) | Always, in Step 7. |

## Output examples

Concrete examples of what the skill produces — not abstract rules.

**Path A success (dembrandt extraction worked end to end):**

```yaml
extraction_status:
  tokens: extracted (dembrandt v0.30.0, 5-page crawl of example.com)
  fonts: extracted (resolved from Font URLs — computed fontFamily was a generic fallback)
  voice_and_mood: extracted (hero + pricing copy read from the live site)
  assets: extracted (wordmark SVG from the extraction's logo key)
  app_patterns: scaffolded
```

User report: "DESIGN.md written. Tokens observed across 5 merged pages; dembrandt's lint scored consistency 91 / contrast 88, with one AA contrast warning on the primary — flagged in Known Gaps. Fonts resolved from the site's webfont files. Say the word if you want to spot-check the flagged contrast pair now."

**Path C success (user uploaded their own DESIGN.md):**

```yaml
extraction_status:
  tokens: extracted (from uploaded file)
  voice_and_mood: needs-verification (not present in uploaded file)
  app_patterns: scaffolded (appended — upload had none)
```

User report: "Uploaded `airbnb-DESIGN.md` parsed and augmented. Colors and typography are good. Voice section was missing — fill it in manually or paste a hero paragraph from the brand and I'll synthesize it."

**Path D fallback (user has nothing):**

```yaml
extraction_status:
  tokens: extracted-by-user (Q&A — single brand color, soft radius, system sans)
  voice_and_mood: extracted (from Q&A — mood: friendly-consumer)
  fonts: inferred (from Q&A vibe answers)
  app_patterns: scaffolded
```

User report: "Built `DESIGN.md` from your 8 answers. This is a minimum-viable brand kit; for higher fidelity, run me again with a website URL and I'll extract the real tokens with dembrandt."

## Skill ends

The skill always ends after Step 8. It does not produce blocks, code, or screenshots. The user runs a separate skill for those.
