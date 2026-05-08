# DESIGN.md Format — Schema Reference

The `DESIGN.md` file follows the format originally specified by Google Stitch and now hosted at <https://github.com/VoltAgent/awesome-design-md>, with **Softr-specific extensions** that downstream tooling (`softr-vibe-coding`) relies on.

## Softr-specific assumptions

Every `DESIGN.md` produced by this skill targets a **Softr app**. That assumption is baked into the schema in three places:

1. **`tech_stack` frontmatter** is auto-populated with Softr's stack — see [Required frontmatter fields](#required-frontmatter-fields). Downstream skills read this to confirm the file is theirs to consume.
2. **`components` and `Application Patterns` sections** scaffold tokens against shadcn/ui, Tailwind, lucide-react, sonner, and date-fns by name — not "your component library here." See [references/app-patterns-stubs.md](references/app-patterns-stubs.md).
3. **`custom-code-header.html`** is generated alongside the DESIGN.md and applies brand fonts/colors at the Softr app level (with `!important` so brand wins against Softr's theme CSS — see [references/custom-code-header.md](references/custom-code-header.md)).

If you're producing a DESIGN.md for a non-Softr target (Webflow, custom React, etc.), this skill is the wrong tool. Either swap the tech stack manually after generation, or use a generic design-system extractor.

## File anatomy

Every `DESIGN.md` has three parts:

1. **Preamble banner** above the YAML frontmatter — sets expectations that this is a brand foundation, not a complete app design system.
2. **YAML frontmatter** between `---` markers — structured tokens for machine consumption.
3. **Markdown body** — prose explanation, do's and don'ts, voice notes, and the Application Patterns scaffold for human + LLM consumption.

```
> ⚠️ **This is a brand foundation, not a complete app design system.** [...]
> See § Evolving this file at the bottom.

---

---
version: alpha
name: <Brand Name>
description: <2–4 sentence brand mood paragraph>
extraction_status: { ... }
tech_stack: { ... }
colors: { ... }
typography: { ... }
rounded: { ... }
spacing: { ... }
elevation: { ... }
components: { ... }
assets: { ... }      # Path A only — logo wordmark with public_url + local_backup
---

## Overview
[prose paragraph]

## Voice & Copy Register
[prose with quoted hero copy]

## Colors
[breakdown by role]

## Typography
[hierarchy + decisions]

## Layout
[grid + whitespace philosophy]

## Elevation
[shadow rules]

## Components
[per-component recipes — extracted from source]

## Application Patterns (scaffolded)
[scaffolded stubs for app patterns the source can't provide — see references/app-patterns-stubs.md]

## Do's and Don'ts
[guardrails]

## Known Gaps in This Extraction
[what was not captured]

## Evolving this file
[how the team should refine scaffolded patterns over time]
```

## Required frontmatter fields

| Field | Type | Purpose |
|---|---|---|
| `version` | string | Always `alpha` for now. |
| `name` | string | Brand name (e.g. "Northwind Studio"). |
| `description` | string | 2–4 sentence brand atmosphere. The single highest-leverage field for downstream LLMs. |
| `target_platform` | string | Always `softr` for files produced by this skill. Downstream tooling (`softr-vibe-coding`) reads this to confirm the file is intended for a Softr app. |
| `tech_stack` | object | Pre-populated with Softr's stack (see below). Documents the stack the `Application Patterns` scaffold targets — `softr-vibe-coding` uses these names verbatim when generating imports. |
| `extraction_status` | object | Per-section confidence flags. See [references/confidence-flags.md](references/confidence-flags.md). |

### `target_platform` and `tech_stack` (Softr defaults)

Always include these blocks at the top of the frontmatter:

```yaml
target_platform: softr
downstream_skill: softr-vibe-coding

tech_stack:
  framework: "Softr Vibe Coding (custom JSX blocks inside a Softr app)"
  component_library: "shadcn/ui"
  component_import_path: "@/components/ui/<name>"
  styling: "Tailwind CSS"
  icons: "lucide-react"
  toasts: "sonner"
  date_formatting: "date-fns"
  data_layer: "@/lib/datasource (useRecords, useRecordCreate, useRecordUpdate, q.select, etc.)"
  user_context: "@/lib/user (useCurrentUser)"
  bundler_constraints:
    - "No optional chaining (?.) — Softr's bundler rejects it. Use `(obj && obj.key) || fallback` instead."
    - "No nullish coalescing (??)."
    - "No `import React from 'react'` — use named imports for hooks: `import { useState, useEffect } from 'react'`."
    - "No CSS modules, no styled-components — Tailwind only."
  field_access: "record.fields.aliasName (NOT record.aliasName)"
  outer_wrapper: "<div className='container py-X'><div className='content'>...</div></div>"
```

These values are **not extracted** from the source — they're Softr-specific defaults that the skill auto-populates so the downstream `softr-vibe-coding` skill knows exactly what stack to write against. Mark `tech_stack.status` as `softr-default`.

## Token sections (frontmatter)

### `colors`

```yaml
colors:
  primary: "#hex"            # Single dominant brand color
  primary-pressed: "#hex"    # Hover/active state (optional)
  primary-tint-*: "#hex"     # Tint ramp shades (optional)
  ink: "#hex"                # Dominant text color (warm near-black or true black)
  body: "#hex"               # Secondary running text
  muted: "#hex"              # Sub-titles, inactive labels
  canvas: "#hex"             # Page floor (white for light systems, near-black for dark)
  surface-soft: "#hex"       # Lightest fill above canvas
  surface-strong: "#hex"     # Heavier fill (sidebars, cards)
  hairline: "#hex"           # 1px border default
  destructive: "#hex"        # Errors, delete actions
  on-primary: "#hex"         # Text color on primary background (usually white)
  # Plus brand-specific accents (amber, orange, etc.) as flat keys
```

Always include at minimum: `primary`, `ink`, `canvas`, `destructive`, `on-primary`.

### `typography`

```yaml
typography:
  font-display-name: "PPNeueMontreal"
  font-display-fallback-stack: "PPNeueMontreal, Arial, sans-serif"
  font-display-open-source-substitute: "Inter"
  font-display-licensing: "Paid font from <foundry>"

  font-body-name: "KitSans"
  font-body-fallback-stack: "KitSans, Arial, sans-serif"
  font-body-open-source-substitute: "Inter"

  font-system-fallback: "Arial, sans-serif"

  display-xl:
    fontFamily: "..."
    fontSize: 48px
    fontWeight: 700
    lineHeight: 1.1
    letterSpacing: -1px
  # display-lg, display-md, title-md, body-md, body-sm, caption, button-md
```

Include the named font fields even if the font is on Google Fonts — name + open-source substitute are both useful for downstream tools.

### `rounded`

```yaml
rounded:
  status: "extracted"  # or inferred / needs-verification
  none: 0px
  xs: 4px
  sm: 8px
  md: 14px
  lg: 20px
  xl: 32px
  full: 9999px
```

Include the full scale even if some values are inferred.

### `spacing`

```yaml
spacing:
  status: "extracted"
  xxs: 2px
  xs: 4px
  sm: 8px
  md: 12px
  base: 16px
  lg: 24px
  xl: 32px
  xxl: 48px
  section: 64px
```

### `elevation`

```yaml
elevation:
  status: "extracted"
  inferred-rule: "one-tier — cards lift on hover only"
  shadow-card: "0 1px 3px rgba(0,0,0,0.04), 0 4px 12px rgba(0,0,0,0.06)"
  shadow-card-hover: "0 4px 8px rgba(0,0,0,0.06), 0 8px 24px rgba(0,0,0,0.08)"
```

### `assets`

Path A only (URL extraction). Records the brand's logo wordmark with **both** a public CDN URL and a local backup path. Both are needed because:

- **Public URL** is canonical for production: Softr's runtime cannot resolve project-relative paths (`./assets/*`) inside Vibe Coding blocks — image `src` attributes must be absolute URLs. Downstream blocks reference `assets.logo_wordmark.public_url`.
- **Local backup** exists for the case where the source CDN URL becomes unstable (re-published, expired token, CMS migration) or the user prefers self-hosting on Softr's media library.

```yaml
assets:
  status: extracted    # or needs-verification if extraction failed
  logo_wordmark:
    public_url: "https://cdn.example.com/uploads/.../brand-logo.svg"
    public_url_note: "Canonical reference for production use. Softr cannot resolve project-relative paths."
    local_backup: "./assets/brand-wordmark.svg"
    local_backup_note: "Downloaded copy for manual upload to Softr's media library if/when you'd rather host on Softr's own CDN."
    format: "SVG"      # SVG | PNG | WebP | JPG, with one-line note about why
    color: "{colors.primary}"  # dominant logo color, ideally tied to a color token
    usage: "Use as <img src='<public_url>' alt='<Brand>' /> in nav lockups. Do not reproduce the wordmark in CSS."
```

**Format preference order** when multiple sources for the same logo exist: SVG → PNG (transparent) → WebP → JPG. SVG is preferred wherever available because it scales perfectly at any size and has the smallest file size for vector wordmarks.

If extraction fails (no plausible logo image found, page blocks scraping, etc.), set `assets.status: needs-verification` and `logo_wordmark: null`, then add a row in `Known Gaps in This Extraction` so the user can paste the URL during the verification prompt.

Other assets (hero photos, decorative imagery) are not extracted by default — only the logo wordmark is universally needed for downstream Softr blocks.

### `components`

```yaml
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.button-md}"
    rounded: "{rounded.md}"
    padding: "12px 24px"
    height: "48px"
  # Plus button-secondary, card, modal, input, etc.
```

The `{colors.x}` and `{typography.x}` references are interpreted by downstream tools — keep them as literal placeholders.

## Required prose sections

### Overview

3–4 paragraphs capturing the brand atmosphere in plain language. Reference frontmatter tokens with the `{colors.primary}` syntax. This is the single most important prose section — it gives downstream LLMs the "feel" the tokens alone can't convey.

Example opening: *"Northwind Studio is a B2B workflow-automation consulting agency. The brand identity is anchored on a single vivid red voltage (`{colors.primary}` — `#e82d42`) carried across all primary CTAs..."*

### Voice & Copy Register

Quote 3–6 lines of hero/CTA copy from the source site (or from the Q&A answers if Path D). Describe the register in 1–2 sentences ("confident but friendly, slightly playful, never corporate-stiff"). Tell downstream tools what microcopy register to use.

### Colors

One subsection per color role: Brand Voltage, Surface, Text, Accent System, Semantic. Each color named with its hex and one-sentence usage note.

### Typography

Confirm specific font names. Explicitly note any traps (e.g. "Tailwind's `font-serif` class maps to a sans typeface — the brand is fully sans, despite the misleading utility name"). Include open-source fallback recommendations.

### Layout

Grid, max content width, gutters, section padding. Mark `status: needs-verification` if not directly extracted.

### Elevation

Flat / one-tier / layered. Quote actual `box-shadow` values when extracted.

### Components

Per-component prose recipes (button-primary, card, etc.) — explain the *why*, not just the values. The frontmatter has values; this section explains them.

### Do's and Don'ts

5–8 actionable guardrails per side. Examples: "Never use pure black for body text — the brand's near-black ink is `#120b0c`." Concrete. Specific. Not generic.

### Known Gaps in This Extraction

A transparent record of what was not captured. List each missing field, why extraction failed, and what would resolve it. This section is required even if the extraction is "complete" — note `None` in that case.

## Confidence flags

Every section under `extraction_status` and every token's `status` field uses one of:

- `extracted` — pulled directly from the source. High confidence.
- `inferred` — derived from indirect signals. Medium confidence; user should verify.
- `needs-verification` — could not be captured. The skill must prompt the user.
- `scaffolded` — generated as a starting-point stub in the Application Patterns section using the brand's extracted tokens. Not present in the source; awaits team refinement when the relevant screen is built.
- `complete` — used at the section level when all tokens in that section are `extracted`.

Use the section-level value `complete` only when every token within that section is `extracted`. If even one token is `inferred`, `needs-verification`, or `scaffolded`, mark the section accordingly and itemize in `Known Gaps` (for inferred/needs-verification) or `Application Patterns` (for scaffolded).

See [references/confidence-flags.md](references/confidence-flags.md) for the full rubric.

## Validating an uploaded `DESIGN.md` (Path C)

When the user uploads their own `DESIGN.md` (Path C), validate against this schema:

1. **Required frontmatter:** `version`, `name`, `description`, `extraction_status`, `colors`, `typography`. Reject if any are missing.
2. **Required prose sections:** Overview, Colors, Typography, Do's and Don'ts. Reject if any are missing.
3. **Format checks:** `colors.*` values are valid hex/rgb/named colors; `rounded.*` are valid CSS lengths.

If validation fails, return a list of the specific missing or malformed fields and ask the user to fix or supply them. Do not silently fill gaps.

## Reference: real example

See [examples/northwind-studio-DESIGN.md](examples/northwind-studio-DESIGN.md) for a complete `DESIGN.md` produced by Path A on a real website (example.com). It demonstrates the format with real extracted tokens, confidence flags, and a known-gaps section.
