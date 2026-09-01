# DESIGN.md Format — Schema Reference (v2, dembrandt-based)

Since v2.0.0, every `DESIGN.md` produced by this skill is built in two layers:

1. **Base document** — the output of [dembrandt](https://github.com/dembrandt/dembrandt)'s `generate_design_md`, which follows Google's DESIGN.md draft format (spec 0.4; see <https://stitch.withgoogle.com/docs/design-md> and the catalog at <https://github.com/VoltAgent/awesome-design-md>). The base is written **verbatim and never restructured** — its frontmatter keys and body sections are exactly what dembrandt emitted, so any tool that understands dembrandt output can read the file.
2. **Softr augmentation layers** — the preamble banner, appended frontmatter keys, and appended body sections this skill adds on top. Additions only: nothing in the base is renamed, reordered, or reshaped. Mechanically the file has ONE frontmatter block: banner, then the base's opening `---`, base keys, appended keys, one closing `---`, base body sections, appended body sections — never two stacked frontmatter blocks.

For paths that don't run dembrandt (catalog, upload, guided Q&A), the skill emits the same shape: dembrandt-style frontmatter keys populated from the path's own source, plus the augmentation layers.

## Softr-specific assumptions

Every `DESIGN.md` produced by this skill targets a **Softr app**. That assumption lives in three places:

1. **`tech_stack` frontmatter** is auto-populated with Softr's stack — see [Appended frontmatter](#appended-frontmatter-keys-skill-owned). Downstream skills read this to confirm the file is theirs to consume.
2. **The `Application Patterns` section** scaffolds tokens against shadcn/ui, Tailwind, lucide-react, sonner, and date-fns by name — see [references/app-patterns-stubs.md](references/app-patterns-stubs.md).
3. **`custom-code-header.html`** is generated alongside the DESIGN.md and applies brand fonts/colors at the Softr app level — see [references/custom-code-header.md](references/custom-code-header.md).

If you're producing a DESIGN.md for a non-Softr target, either run dembrandt directly (`--design-md` gives you the unaugmented base) or swap the tech stack manually after generation.

## File anatomy

```
> ⚠️ Preamble banner (above the frontmatter — brand foundation, not a finished system)

---
# ── BASE (dembrandt, verbatim — never restructure) ──
name: <Brand Name>
description: "Design tokens extracted from <url>. <2–3 appended brand-mood sentences.>"
colors: { primary, secondary, tertiary, surface, on-surface, error… }
typography: { headline-display, headline-lg…, label-lg, body-md, text-N… }
spacing: { base, xs…xxxxl }
rounded: { sm…xl, none, full }
components: { button-observed, input-observed }
# ── AUGMENTATION (this skill) ──
version: 2
target_platform: softr
downstream_skill: softr-vibe-coding
tech_stack: { … }
fonts: { … }
assets: { … }              # Path A only — logo with public_url + local_backup
extraction_status: { … }
---

# Design System           ← base body (dembrandt, verbatim)
## Overview
## Colors
## Typography             ← incl. Font URLs (real webfont files)
## Layout
## Elevation & Depth
## Shapes
## Components
                          ← appended body sections (this skill)
## Voice & Copy Register
## Do's and Don'ts
## Application Patterns (scaffolded)
## Known Gaps in This Extraction
## Evolving this file
```

**The one permitted edit inside the base:** the `description` value. dembrandt emits `"Design tokens extracted from <url>"` — keep that as the **first sentence** (downstream tools read the source URL out of this field) and append 2–3 sentences of brand mood after it. The mood sentences are the single highest-leverage content for downstream LLMs.

## Base frontmatter keys (dembrandt-owned — never restructure)

| Key | Shape | Notes |
|---|---|---|
| `name` | string | Brand/site name as detected. |
| `description` | string | Source URL sentence + appended mood sentences (see above). |
| `colors` | role → hex | Semantic roles: `primary` / `secondary` / `tertiary` / `surface` / `on-surface` (+ `error` when detected). Uppercase hex. |
| `typography` | token → object | Context-named: `headline-display`, `headline-lg`/`-md`/`-sm`, `label-lg` (buttons), `label-md` (links), `body-md`, with `text-N` fallback for unclassified styles. Each has `fontFamily`/`fontSize`/`fontWeight`/`lineHeight` (sometimes `letterSpacing`/`fontFeature`). **`fontFamily` is a computed value and may name a generic fallback (`ui-sans-serif`)** — the real brand font is resolved into the `fonts` block (below) via the body's Font URLs. |
| `spacing` | step → px | `base` plus named steps. |
| `rounded` | step → px | `sm`…`xl`, plus `none`/`full` when observed. |
| `components` | name → object | `button-observed` / `input-observed`; values may reference other tokens (`"{rounded.lg}"`). |

Keys with no extracted evidence are absent, not invented — leave them absent.

## Appended frontmatter keys (skill-owned)

```yaml
version: 2                       # format version of the augmentation layer
target_platform: softr
downstream_skill: softr-vibe-coding

tech_stack:
  status: softr-default          # not extracted — Softr platform defaults
  framework: "Softr Vibe Coding (custom TSX/JSX blocks inside a Softr app)"
  component_library: "shadcn/ui"
  component_import_path: "@/components/ui/<name>"
  styling: "Tailwind CSS (JIT build — arbitrary values compile)"
  icons: "lucide-react"
  toasts: "sonner"
  date_formatting: "date-fns"
  data_layer: "@/lib/datasource (useRecords, useRecordCreate, useRecordUpdate, q.select, …)"
  user_context: "@/lib/user (useCurrentUser)"
  language_constraints:
    - "Modern TypeScript compiles — ?., ??, arrow functions, const, generics (verified live 2026-08-25). Prefer .tsx."
    - "Named React imports only — `import { useState } from 'react'`, never `import React from 'react'`."
    - "Tailwind + inline styles only — no CSS modules, no styled-components."
  field_access: "record.fields.aliasName (NOT record.aliasName)"
  outer_wrapper: "<div className='container py-0'><div className='content …'>…</div></div> — or a deliberate full-bleed layout recorded in the block's placement comment"

fonts:
  display-name: "Nohemi"                       # real font, resolved from the body's Font URLs — NOT from computed fontFamily
  display-fallback-stack: "'Nohemi', system-ui, sans-serif"
  display-open-source-substitute: "Space Grotesk"   # only when the font is paid/licensed
  display-licensing: "Free (foundry download)"      # or "Paid — <foundry>" / "Google Fonts"
  body-name: "Inter"
  body-fallback-stack: "'Inter', system-ui, sans-serif"
  body-licensing: "Google Fonts"

assets:                          # Path A only — see below
  status: extracted
  logo_wordmark: { … }

extraction_status:               # section-level confidence — see references/confidence-flags.md
  tokens: extracted (dembrandt, <N>-page crawl of <url>)
  fonts: extracted (resolved from Font URLs)
  voice_and_mood: extracted (hero copy read from the live site)
  assets: extracted
  app_patterns: scaffolded
```

Notes:

- **`fonts` exists because dembrandt's computed `fontFamily` can lie.** The body's Typography section lists the real webfont files (Font URLs). Resolve the actual family names from those URLs (and the live site when ambiguous), then record them here. `custom-code-header.html` generation reads this block, not `typography.*.fontFamily`. Substitutes come from [references/google-fonts-substitutes.md](references/google-fonts-substitutes.md) when the font is paid.
- **No per-token `status` fields.** dembrandt tokens are all observed values by definition — adding `status` objects would break the base shape downstream tools parse. Confidence lives at section level in `extraction_status`, with specifics in `Known Gaps`. (This differs from v1, which flagged every token.)
- **`tech_stack.language_constraints` reflects the current platform.** v1 files claimed "no `?.` / no `??`" — that was true of the old bundler and is retired; do not copy it forward.

### `assets` (Path A only)

Records the brand's logo wordmark with **both** a public URL and a local backup:

- **Public URL** is canonical for production: Softr's runtime cannot resolve project-relative paths (`./assets/*`) inside Vibe Coding blocks — image `src` attributes must be absolute URLs.
- **Local backup** covers CDN instability (re-published assets, expired tokens, CMS migrations) and users who prefer self-hosting via Softr's media library.

```yaml
assets:
  status: extracted              # or needs-verification if no plausible logo was found
  logo_wordmark:
    public_url: "https://cdn.example.com/uploads/brand-logo.svg"
    public_url_note: "Canonical reference for production use. Softr cannot resolve project-relative paths."
    local_backup: "./assets/brand-wordmark.svg"
    local_backup_note: "Downloaded copy for manual upload to Softr's media library."
    format: "SVG"                # SVG | PNG | WebP | JPG — prefer in that order
    color: "{colors.primary}"    # dominant logo color, ideally tied to a color token
    usage: "Use as <img src='<public_url>' alt='<Brand>' /> in nav lockups. Do not rebuild the wordmark in CSS."
```

The logo comes from dembrandt's extraction JSON (`logo`, `logoInstances`, `favicons` — source URL, dimensions, type, safe zone), not from a separate scrape. If no plausible logo was captured, set `status: needs-verification`, record `logo_wordmark: null`, and add a row to `Known Gaps` so the verification prompt asks the user for the URL.

## Appended body sections

### Voice & Copy Register

Quote 3–6 lines of hero/CTA copy from the source site (read the live pages directly — the extractions this pipeline runs carry no voice data; see [extractors/dembrandt-pipeline.md](extractors/dembrandt-pipeline.md#voice--copy-register-not-part-of-dembrandt)) and describe the register in 1–2 sentences ("confident but friendly, never corporate-stiff"). This tells downstream tools what microcopy register to use.

### Do's and Don'ts

5–8 actionable guardrails per side, referencing real tokens: "Never use pure black for body text — the brand's `on-surface` is `#242424`." Concrete and brand-specific, not generic. This is also where color-role usage guidance lives (the base's Colors section only lists observed roles).

### Application Patterns (scaffolded)

The full scaffold from [references/app-patterns-stubs.md](references/app-patterns-stubs.md), with every `{colors.x}`/`{typography.x}`/`{rounded.x}` placeholder substituted with the file's actual token names (dembrandt vocabulary: `{colors.primary}`, `{colors.on-surface}`, `{typography.headline-md}`, `{rounded.full}`, …). Every stub carries `status: "scaffolded"`.

### Known Gaps in This Extraction

A transparent record of what was not captured: each missing field, why, and what would resolve it. Required even when extraction is complete — write `None` in that case.

### Evolving this file

How the team should refine scaffolded patterns after building 2–3 real blocks, promoting them out of `scaffolded` status.

## Confidence flags

Section-level flags in `extraction_status` (full rubric in [references/confidence-flags.md](references/confidence-flags.md)):

- `extracted` — observed directly (dembrandt tokens; hero copy quoted from the live site; logo from the extraction JSON).
- `inferred` — derived from indirect signals. User should verify.
- `needs-verification` — could not be captured; the skill must prompt before locking.
- `scaffolded` — Application Patterns stubs. Intentional placeholders, refined during real builds.
- `partial` — section-level mix: some values observed, others inferred or missing; itemize in `Known Gaps`.

Run dembrandt's `get_findings` on the extraction before writing the file — its contrast/consistency lint feeds `Known Gaps` directly.

## Validating an uploaded `DESIGN.md` (Path C)

Uploaded files may be dembrandt-shaped (v2), legacy v1 (this skill pre-2.0: `brand:`/`source:`/`extracted:`-era frontmatter with `Application Patterns` and `tech_stack`), or third-party (awesome-design-md catalog format). Validate pragmatically:

1. **Required frontmatter:** `name`, `description`, `colors`, `typography` — every supported shape has these. Reject only if one is missing or `colors` values aren't valid CSS colors.
2. **Body:** at least one prose section. List (don't silently fill) anything the v2 anatomy has that the upload lacks.
3. **Augment, don't rewrite:** append the missing skill-owned layers (`tech_stack`, `extraction_status`, banner) and flag the rest in `Known Gaps`. Never reshape the uploaded token structure.

## Legacy note (v1 files)

DESIGN.md files produced by this skill before v2.0.0 use a different schema (per-token `status` fields, `ink`/`canvas`/`destructive` color roles, `font-display-name` typography fields, retired bundler constraints in `tech_stack`). They remain fully consumable downstream — `softr-vibe-coding` honours whatever token sections are present — and do not need migration. When one is regenerated through dembrandt, warn the user that v1-specific refinements (edited scaffolds, verified tokens) are not carried over automatically.

## Reference: example

See [examples/northwind-studio-DESIGN.md](examples/northwind-studio-DESIGN.md) for a complete v2 `DESIGN.md` (fictional reference brand — synthesized to demonstrate the format, marked `example-only` throughout).
