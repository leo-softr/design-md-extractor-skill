> ⚠️ **Softr-specific brand foundation, not a complete app design system.**
>
> **This is a synthesized example — Northwind Studio does not exist.** No dembrandt run produced this file: every token below was written by hand to demonstrate the v2 format, and `extraction_status` says `example-only` throughout. In a real DESIGN.md this banner names the source and date instead ("Extracted from `<url>` on `<YYYY-MM-DD>` for use in a Softr app").
>
> The colors, typography, spacing, and components below follow the shape dembrandt emits and are ready to use. **Application patterns** (modals, status pills, empty/loading/error states, page-header lockups, form-field cards, etc.) are not present in a marketing source and have been **scaffolded with brand-token defaults using Softr's stack** (shadcn/ui, Tailwind, lucide-react, sonner, date-fns) in § Application Patterns at the bottom. Treat those scaffolds as starting points — refine them when you build the Softr blocks that use them.
>
> **Next step**: paste `custom-code-header.html` into Softr → Settings → Custom Code → Code inside header, then run the `softr-vibe-coding` skill to generate brand-aligned Vibe Coding blocks from this file.
>
> When you've built 2–3 real blocks, capture the patterns that emerged and promote them out of "scaffolded" status. (See § Evolving this file.)

---
# ── BASE layer (dembrandt-shaped — in real runs this layer is emitted verbatim) ──
name: Northwind Studio
description: "Design tokens extracted from northwindstudio.example. The brand is anchored on a single deep teal voltage carried across primary CTAs and brand emphasis, over a white floor with warm cream and cool teal-tinted section surfaces. One sans-serif — Inter — covers display through body and UI; hierarchy is done with weight, not a second face. The voice is measured, results-oriented, and lightly assertive — built for B2B operations leaders, not consumer brands."

colors:
  primary: "#0E6B7A"      # Deep teal — the voltage: primary CTAs, links, brand emphasis
  secondary: "#2D8A9A"    # Mid teal — hover tints, secondary emphasis, gradient ends
  tertiary: "#F59E0B"     # Amber — attention markers and warning accents, never surfaces
  surface: "#FFFFFF"      # Page floor and card backgrounds
  on-surface: "#0F172A"   # Near-black charcoal — dominant text color, never pure #000
  error: "#DC2626"        # Errors and destructive actions — distinct from the teal family

typography:
  headline-display:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 48px
    fontWeight: 700
    lineHeight: 1.1
    letterSpacing: "-1px"
  headline-lg:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 36px
    fontWeight: 700
    lineHeight: 1.15
    letterSpacing: "-0.5px"
  headline-md:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 28px
    fontWeight: 600
    lineHeight: 1.25
  headline-sm:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 20px
    fontWeight: 600
    lineHeight: 1.3
  label-lg:               # buttons
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 15px
    fontWeight: 500
    lineHeight: 1.25
  label-md:               # links, captions, eyebrows
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 13px
    fontWeight: 500
    lineHeight: 1.4
  body-md:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.5

spacing:
  base: 16px
  xs: 4px
  sm: 8px
  md: 12px
  lg: 24px
  xl: 32px
  xxl: 48px
  xxxl: 64px
  xxxxl: 80px             # section rhythm

rounded:
  none: 0px
  sm: 6px
  md: 8px
  lg: 12px
  xl: 16px
  full: 9999px

components:
  button-observed:
    backgroundColor: "{colors.primary}"
    textColor: "#FFFFFF"
    rounded: "{rounded.md}"
    padding: "10px 20px"
    height: "44px"
  input-observed:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.on-surface}"
    rounded: "{rounded.md}"
    padding: "10px 14px"

# ── AUGMENTATION layer (appended by building-design-md) ──
version: 2
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
  display-name: "Inter"
  display-fallback-stack: "'Inter', system-ui, sans-serif"
  display-licensing: "Google Fonts"
  body-name: "Inter"
  body-fallback-stack: "'Inter', system-ui, sans-serif"
  body-licensing: "Google Fonts"

assets:
  status: example-only           # a real Path A run records `extracted` or `needs-verification`
  logo_wordmark:
    public_url: "https://cdn.northwindstudio.example/brand/northwind-wordmark.svg"
    public_url_note: "Canonical reference for production use — Softr cannot resolve project-relative paths inside Vibe Coding blocks, so image src attributes must be absolute URLs. (Fictional URL — .example is a reserved TLD.)"
    local_backup: "./assets/northwind-studio-wordmark.svg"
    local_backup_note: "Downloaded copy for manual upload to Softr's media library."
    format: "SVG"
    color: "{colors.primary}"
    usage: "Use as <img src='<public_url>' alt='Northwind Studio' /> in nav lockups. Do not rebuild the wordmark in CSS."

extraction_status:
  note: "Synthesized reference example — NOT produced by a real dembrandt run. Every value in this file was written by hand to demonstrate the v2 format; real files carry extracted / inferred / needs-verification / scaffolded flags here."
  tokens: example-only (no dembrandt crawl was run)
  fonts: example-only (a real run resolves these from the body's Font URLs)
  voice_and_mood: example-only (a real run quotes the live site)
  assets: example-only (fictional CDN URL on the reserved .example TLD)
  app_patterns: example-only (in a real file this reads scaffolded)
---

# Design System

## Overview

Northwind Studio is a B2B workflow-automation consultancy. The identity is anchored on **a single deep teal voltage** (`{colors.primary}` — `#0E6B7A`) carried across primary CTAs and brand emphasis. Depth treatments darken it (observed pressed value `#0B5462`); tints of the same hue lighten it toward `#D9EEF3` for featured backgrounds — range without a second color family.

Surrounding the teal is **a system of pale section backgrounds** — one warm cream (`#FAF7F2`) and one cool teal-tint (`#F0F4F6`) — that distinguish categorical content blocks (services, case studies, testimonials) without competing for attention. The page floor is pure white (`{colors.surface}`).

Type sits in a **near-black charcoal** (`{colors.on-surface}` — `#0F172A`) rather than pure black, with a warm gray (`#475569`) for secondary running text. A single sans-serif typeface — Inter — is used for everything from display headings to body text and UI. A deliberate restraint: no second typeface, no serif.

## Colors

| Role | Value | Observed use |
|---|---|---|
| `primary` | `#0E6B7A` | Primary CTAs, links, brand emphasis — the single voltage |
| `secondary` | `#2D8A9A` | Mid-teal hover tints, secondary emphasis, gradient ends |
| `tertiary` | `#F59E0B` | Amber attention markers and warnings — status only, never surfaces |
| `surface` | `#FFFFFF` | Page floor and card backgrounds |
| `on-surface` | `#0F172A` | Dominant text — charcoal, never pure black |
| `error` | `#DC2626` | Errors and destructive actions |

Supporting values observed but not promoted to roles: pressed teal `#0B5462`, pale teal tint `#D9EEF3`, warm cream section surface `#FAF7F2`, cool teal-tint section surface `#F0F4F6`, secondary text gray `#475569`, hairline border `#E2E8F0`, success emerald `#10B981`. Usage guidance for all of these lives in § Do's and Don'ts.

## Typography

One typeface for every role: **Inter**, weights 400–700. Hierarchy is done with weight and size, not a second family — display headings at 700, sub-headings at 600, body and UI at 400–500. Negative letter-spacing (−0.5px to −1px) tightens the two largest headline sizes only.

**Font URLs:**

- `https://fonts.gstatic.com/s/inter/v18/UcCO3FwrK3iLTeHuS_nVMrMxCp50SjIw2boKoduKmMEVuLyfAZ9hiA.woff2` — Inter variable (latin), served via Google Fonts (fictional-but-well-formed entry)

The computed `fontFamily` values in the frontmatter already name Inter directly — no generic fallback masking here. (On many real sites the computed value is a generic stack like `ui-sans-serif` while the true brand font hides in these Font URLs; that is why the `fonts` frontmatter block, not `typography.*.fontFamily`, is the source of truth for font loading.)

**For Softr:** Inter is the simplest possible choice — it's a Softr font default, needs no custom `@font-face`, and renders consistently across browsers. The generated `custom-code-header.html` just loads Inter from Google Fonts.

## Layout

Spacing runs on a 16px base (`{spacing.base}`) with a compact lower half (4 / 8 / 12px) for in-component gaps and a generous upper half (24 / 32 / 48 / 64px) for card and section composition. Section rhythm between page-level content bands is 80px (`{spacing.xxxxl}`). Content sits in a centered max-width column; section backgrounds alternate warm cream and cool teal-tint for categorical variety, never solid brand-color bands.

## Elevation & Depth

Two-tier shadows tinted toward the charcoal ink — the brand reads light, ordered, structured. Avoid heavy or layered elevation.

- Resting card: `0 1px 3px rgba(15,23,42,0.04), 0 4px 12px rgba(15,23,42,0.06)`
- Card hover: `0 4px 8px rgba(15,23,42,0.06), 0 8px 24px rgba(15,23,42,0.08)`
- Floating panels (popovers, modals): `0 12px 32px rgba(15,23,42,0.10)`

## Shapes

Soft-but-not-round: 8–12px (`{rounded.md}` / `{rounded.lg}`) on most surfaces, 6px (`{rounded.sm}`) on compact controls, 16px (`{rounded.xl}`) on modals and feature cards, full pill (`{rounded.full}`) on chips and badges. No sharp corners anywhere.

## Components

- **`button-observed`** — solid `{colors.primary}` fill, white text, `{typography.label-lg}`, `{rounded.md}` corners, 44px tall. Hover darkens to the observed pressed teal `#0B5462`.
- **`input-observed`** — white field, `{colors.on-surface}` text, 1px `#E2E8F0` hairline border, `{rounded.md}` corners, 44px tall. Focus ring in `{colors.primary}` at reduced opacity.
- Secondary buttons observed as white fills with charcoal text and a hairline border, at the same dimensions as the primary.

## Voice & Copy Register

The voice is **measured, results-oriented, lightly assertive — built for B2B operations leaders, not consumer brands.** Hero copy patterns:

- "Operations teams ship faster when their tools stop fighting them."
- "We design workflows your team actually wants to use."
- "Automation that scales with you, not against you."

Body copy is direct and evidence-led, with concrete outcome claims — case studies cite specific metrics, not vague "growth". CTAs are action-oriented ("Book a discovery call", "See how it works", "Get the playbook"). Trust signals lean on named-client case studies with quantified business outcomes.

Microcopy in Softr Vibe Coding blocks built for a brand like Northwind should match: clear, confident, no jargon, no exclamation marks, no emoji, never playful for the sake of it.

## Do's and Don'ts

**Do:**

- Use `{colors.primary}` (`#0E6B7A`) as the single voltage on primary CTAs — the only color allowed to demand attention.
- Use **Inter** for everything — display, body, UI. Weight does the hierarchy work.
- Use the warm cream (`#FAF7F2`) and cool teal-tint (`#F0F4F6`) surfaces for *section* backgrounds, not individual component fills.
- Use `{colors.on-surface}` (`#0F172A`) for body text — never pure `#000`.
- Use `{colors.tertiary}` amber (and the observed emerald `#10B981`) only for status indicators — warning and success — never as surfaces.

**Don't:**

- Don't introduce a second brand color family — darkening and tinting the one teal gives sufficient range.
- Don't introduce a serif or a second sans-serif. Single typeface, single voltage, single voice.
- Don't use heavy shadows or layered elevation — stay close to flat with the two-tier shadows in § Elevation & Depth.
- Don't use `{colors.error}` for anything but errors and destructive actions — it must never compete with the primary teal on the same screen.
- Don't use emoji or playful microcopy — the voice is measured and confident, not chatty.

## Application Patterns (scaffolded)

The patterns below are common across most app UIs but do not appear on a typical marketing source. Each is **scaffolded** using the brand tokens captured above, so it ships with brand-aligned defaults out of the box — starting points, not finished specs.

> This example shows four representative stubs. Real runs append the **full** scaffold list (alert dialogs, lightbox, pill family, section cards, form field cards, search/filter rows, eyebrows, meta rows, document icons, interaction states, toasts) from `references/app-patterns-stubs.md`.

### Block scaffolding (Softr — every block needs this)

`status: "scaffolded"`

Every Vibe Coding block must wrap its content in Softr's standard container/content pattern, otherwise the block won't size correctly inside the Softr page. `container` and `content` are Softr-recognized class names that constrain the block to the app's max-width and apply page-level padding. For admin pages (lists, dashboards), wrap the inner content in a soft brand-tinted container:

```jsx
<div className="container py-8">
  <div className="content">
    <div className="rounded-[30px] p-8" style={{ backgroundColor: "#F0F4F6" }}>
      {/* admin content — fallback: colors block has no surface-soft role;
          using the observed cool section surface (§ Colors) */}
    </div>
  </div>
</div>
```

**Refine when you build:** the first block.

### Page Header

`status: "scaffolded"`

Top-of-page lockup used at the top of every admin / detail page.

- Icon square: 44×44, `{rounded.md}` corners, `{colors.primary}` background, white lucide icon at 20px (fallback: no primary-deep role extracted — using `{colors.primary}`).
- Title: h2, `{typography.headline-md}`, color `{colors.on-surface}`.
- Description: 14px muted (`text-muted-foreground`), `mt-1 ml-14` — aligned past the icon.

**Refine when you build:** the first screen that has a header.

### Dialogs / Modals

`status: "scaffolded"`

Four-part anatomy. The dialog content uses `p-0 overflow-hidden` so each part renders flush against the rounded corners.

- **dialog-content**: `{rounded.xl}` corners, `sm:max-w-lg` for standard, 90vw × 90vh for media lightboxes.
- **dialog-header**: white strip, 20px / 24px padding, 1px `#E2E8F0` hairline border-bottom. 40×40 `{rounded.md}` icon square in `{colors.primary}` with a white lucide icon. DialogTitle (`text-lg font-semibold`, `{colors.on-surface}`) and DialogDescription (`text-xs text-muted-foreground mt-0.5`) — use `DialogDescription`, not a plain `<p>`.
- **dialog-body**: `#F0F4F6` wash (fallback: no surface-soft role — nearest observed brand surface, § Colors), 16px / 24px padding. **Always** apply `max-h-[60vh] overflow-y-auto` so short viewports scroll the body instead of pushing the footer offscreen.
- **dialog-footer**: subtle gray-tint strip, 16px / 24px padding, 1px hairline border-top. `flex justify-end gap-2`.

**Refine when you build:** the first modal — most likely a confirmation dialog or an edit form.

### Empty / loading / error states

`status: "scaffolded"`

Every data-driven block must handle four states with consistent treatment:

- **Loading** — Skeleton shapes matching the final card layout, at `{rounded.xl}` (the actual card radius) so the skeleton previews real geometry. Page-header icon `h-10 w-10 {rounded.md}`, title `h-8 w-48`, list card `h-28 w-full`.
- **Error** — Centered icon + heading + retry button. Icon: 64×64 rounded-full, `{colors.error}` at 15% opacity background + `{colors.error}` AlertCircle. Heading uses `{colors.on-surface}`. Retry is `{components.button-observed}`.
- **Empty (no data)** — Centered icon + heading + description. Icon: 80×80 `{rounded.lg}` with `linear-gradient(135deg, {colors.primary} → {colors.secondary})` background + white icon. The gradient is reserved for empty-state icons; don't use it elsewhere.
- **Empty after filtering** — Same shape, heading reads "No matching X", description suggests adjusting filters. Always include a Reset / Clear Filters action.

**Refine when you build:** the first data-driven block.

## Known Gaps in This Extraction

None (synthesized example). Every value in this file was chosen deliberately to demonstrate the format. In a real run this section itemizes what the crawl could not capture — each missing field, why, and what would resolve it — plus anything severe from dembrandt's `get_findings` lint (contrast failures, token collisions).

## Evolving this file

Every stub in § Application Patterns is marked `status: "scaffolded"`. The first time you build a block that uses one, refine the stub to match what you actually shipped, promote it to `extracted`, and update the prose. After 2–3 real blocks the file shifts from "brand foundation + scaffolds" to a working Softr design system — and dembrandt's `compute_drift` can later score the published app against a fresh extraction of the brand site.

## Notes on This Example

**Northwind Studio does not exist.** This file lives in the skill's `examples/` directory to show what a complete, well-formed v2 DESIGN.md looks like — the dembrandt-shaped base layer, the appended augmentation keys, the Application Patterns scaffold. It was **not** produced by a real dembrandt run; every `extraction_status` value is `example-only` instead of the real flags.

What a real extraction has that this example fakes:

- The `description` opens with the actual source URL dembrandt crawled, and the preamble banner names the source and extraction date.
- The Typography section's Font URLs list the webfont files actually observed on the site — the evidence used to resolve the `fonts` block when computed `fontFamily` values name a generic fallback.
- The `assets` block carries a real, hot-linkable CDN URL (Softr cannot resolve project-relative paths inside Vibe Coding blocks, so production `src` attributes must use the public URL) plus a real downloaded backup in `./assets/`.
- `extraction_status` mixes `extracted` / `inferred` / `needs-verification` / `scaffolded`, and § Known Gaps lists genuine misses and lint findings.

See [extractors/dembrandt-pipeline.md](../extractors/dembrandt-pipeline.md) for how the real pipeline produces all of the above.
