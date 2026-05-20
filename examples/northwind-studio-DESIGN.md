---
version: example
name: Northwind Studio
description: A fictional B2B workflow-automation studio used as the canonical reference example for this skill. Northwind's brand is anchored on a deep teal voltage (#0E6B7A) carried across primary CTAs and brand surfaces, supported by warm white and cool teal-tinted section backgrounds. A single sans-serif typeface (Inter) is used for everything from display headings to body text and UI — a deliberate restraint. The studio's voice is measured, results-oriented, and lightly assertive — built for B2B operations leaders, not playful consumer brands. Tech stack is Next.js + Contentful + Vercel + Plausible. Type sits in a near-black charcoal (#0F172A) with a warm gray (#475569) for secondary copy. This DESIGN.md illustrates the schema and serves as a downstream reference for the `softr-vibe-coding` skill.

extraction_status:
  note: Fictional example for documentation purposes. Not produced by the Apify+Exa pipeline against a real site. All tokens are synthesized to demonstrate the schema, confidence flags, and Application Patterns scaffold.
  colors: example-only
  framework: example-only
  voice_and_mood: example-only
  typography: example-only
  spacing_radius_shadows: example-only

tech_stack:
  framework: Next.js
  cms: Contentful
  styling: Tailwind CSS
  frontend_libs: [Framer Motion]
  cdn: Vercel
  analytics: [Plausible]
  features: [Open Graph, PWA, HTTP/3]

colors:
  # Brand primary — the voltage
  primary: "#0E6B7A"           # Deep teal — primary CTA, brand surface
  primary-pressed: "#0B5462"   # One step darker
  primary-active: "#093E48"    # Two steps darker
  primary-deep: "#072E36"      # Borders / depth surfaces
  primary-deepest: "#051F24"   # Darkest shade

  # Brand teal tints
  primary-tint-md: "#2D8A9A"   # Mid teal
  primary-tint-sm: "#5BABBC"   # Light teal
  primary-tint-xs: "#8FCCD9"   # Soft teal — featured backgrounds
  primary-tint-xxs: "#D9EEF3"  # Very pale teal — section backgrounds

  # Text
  ink: "#0F172A"               # Near-black charcoal — dominant text color
  body: "#475569"              # Warm gray — secondary running text
  muted: "#64748B"             # Tertiary text
  muted-soft: "#94A3B8"        # Captions / disabled

  # Surface
  canvas: "#FFFFFF"            # Page floor
  surface-warm: "#FAF7F2"      # Cream tint — warm content sections
  surface-cool: "#F0F4F6"      # Cool teal-tinted — cool content sections
  surface-card: "#FFFFFF"      # Card backgrounds

  # Accent system — used only for status indicators, never primary surfaces
  accent-amber: "#F59E0B"       # Warning states, attention markers
  accent-emerald: "#10B981"     # Success states, positive metrics

  # Semantic
  destructive: "#DC2626"        # Tailwind red-600 — errors, destructive actions
  on-primary: "#FFFFFF"
  on-dark: "#FFFFFF"

  # Borders / hairlines
  hairline: "#E2E8F0"           # 1px subtle border

typography:
  font-display-name: "Inter"
  font-display-fallback-stack: "Inter, system-ui, -apple-system, sans-serif"
  font-display-licensing: "Google Fonts — SIL Open Font License. Free for commercial use, no @font-face hosting required."

  font-body-name: "Inter"
  font-body-fallback-stack: "Inter, system-ui, -apple-system, sans-serif"
  font-body-licensing: "Google Fonts — SIL Open Font License. Free for commercial use."

  font-system-fallback: "system-ui, -apple-system, sans-serif"

  # Single-typeface system: Inter's variable axes (weight, optical size) cover
  # display-to-body needs. Hierarchy is done with weight, not a second font.

  display-xl:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 48px
    fontWeight: 700
    lineHeight: 1.1
    letterSpacing: -1px
  display-lg:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 36px
    fontWeight: 700
    lineHeight: 1.15
    letterSpacing: -0.5px
  display-md:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 28px
    fontWeight: 600
    lineHeight: 1.25
  title-md:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 20px
    fontWeight: 600
    lineHeight: 1.3
  body-md:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.5
  body-sm:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 14px
    fontWeight: 400
    lineHeight: 1.5
  caption:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 13px
    fontWeight: 500
    lineHeight: 1.4
  button-md:
    fontFamily: "Inter, system-ui, sans-serif"
    fontSize: 15px
    fontWeight: 500
    lineHeight: 1.25

rounded:
  none: 0px
  xs: 4px
  sm: 6px
  md: 8px
  lg: 12px
  xl: 16px
  full: 9999px

spacing:
  xxs: 2px
  xs: 4px
  sm: 8px
  md: 12px
  base: 16px
  lg: 24px
  xl: 32px
  xxl: 48px
  section: 80px

elevation:
  inferred-rule: "two-tier — subtle resting shadow on cards, lift on hover, deeper float for popovers/modals"
  shadow-card: "0 1px 3px rgba(15,23,42,0.04), 0 4px 12px rgba(15,23,42,0.06)"
  shadow-card-hover: "0 4px 8px rgba(15,23,42,0.06), 0 8px 24px rgba(15,23,42,0.08)"
  shadow-elevated: "0 12px 32px rgba(15,23,42,0.10)"

components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.button-md}"
    rounded: "{rounded.md}"
    padding: "10px 20px"
    height: "44px"
  button-primary-hover:
    backgroundColor: "{colors.primary-pressed}"
    textColor: "{colors.on-primary}"
    rounded: "{rounded.md}"
  button-secondary:
    backgroundColor: "{colors.canvas}"
    textColor: "{colors.ink}"
    border: "1px solid {colors.hairline}"
    typography: "{typography.button-md}"
    rounded: "{rounded.md}"
    padding: "9px 19px"
    height: "44px"
  card:
    backgroundColor: "{colors.surface-card}"
    textColor: "{colors.ink}"
    typography: "{typography.body-md}"
    rounded: "{rounded.lg}"
    padding: "24px"
    elevation: "{elevation.shadow-card}"
    border: "1px solid {colors.hairline}"
  feature-section-warm:
    backgroundColor: "{colors.surface-warm}"
  feature-section-cool:
    backgroundColor: "{colors.surface-cool}"
---

> **This is a fictional example.** Northwind Studio does not exist. This DESIGN.md lives in the skill's `examples/` directory to show what a complete, well-formed DESIGN.md looks like — the schema, the brand-token references, the Application Patterns scaffold. Treat it as a reference *format*, not a real extraction. Real extractions produced by this skill's Apify + Exa pipeline include `extracted` / `inferred` / `needs-verification` / `scaffolded` confidence flags on individual tokens reflecting actual extraction outcomes.

## Overview

Northwind Studio is a fictional B2B workflow-automation consultancy used as the reference brand for this skill. The identity is anchored on **a single deep teal voltage** (`{colors.primary}` — `#0E6B7A`) carried across primary CTAs and brand surfaces. That teal is explored as a 9-shade ramp from `#051F24` (deepest) through `#0E6B7A` (primary) to `#D9EEF3` (palest tint), giving range without introducing a second color family.

Surrounding the brand teal is **a system of pale section backgrounds** — one warm cream (`{colors.surface-warm}`) and one cool teal-tinted (`{colors.surface-cool}`) — that distinguish categorical content blocks (services, case studies, testimonials) without competing for visual attention. The page floor is pure white (`{colors.canvas}` — `#FFFFFF`).

Type sits in a **near-black charcoal** (`{colors.ink}` — `#0F172A`) rather than pure black, with a **warm gray** (`{colors.body}` — `#475569`) for secondary copy. A single sans-serif typeface — Inter — is used for everything from display headings to body text and UI. A deliberate restraint: no second typeface, no serif.

## Tech Stack

- **Framework / build tool:** Next.js (App Router)
- **CMS:** Contentful (headless content platform)
- **Styling:** Tailwind CSS
- **Frontend interactivity:** Framer Motion (page transitions, on-scroll animations)
- **Hosting / CDN:** Vercel
- **Analytics:** Plausible (privacy-first, cookieless, first-party — no consent banner required)
- **Performance features:** PWA, HTTP/3, Open Graph

This stack is typical of a modern small B2B agency prioritizing performance scores and avoiding the consent-banner overhead of GA/Clarity-based analytics stacks.

## Voice & Copy Register

The voice is **measured, results-oriented, lightly assertive — built for B2B operations leaders, not consumer brands.** Hero copy patterns:

- "Operations teams ship faster when their tools stop fighting them."
- "We design workflows your team actually wants to use."
- "Automation that scales with you, not against you."

Body copy is direct, evidence-led, with measured language and concrete outcome claims (case studies cite specific metrics, not vague "growth"). The CTA pattern is action-oriented ("Book a discovery call", "See how it works", "Get the playbook"). Trust signals lean on named-client case studies with quantified business outcomes.

Microcopy in Softr Vibe Coding blocks built for a brand like Northwind should match: clear, confident, no jargon, no exclamation marks, no emoji, never playful for the sake of it.

## Colors

### Brand Voltage

- **Primary** (`{colors.primary}` — `#0E6B7A`): The single brand teal. Used for primary CTAs, accent surfaces, and brand emphasis.
- **Primary Pressed / Active / Deep / Deepest** (`#0B5462`, `#093E48`, `#072E36`, `#051F24`): Hover/pressed states and depth treatments.

### Brand Teal Tints

- `#2D8A9A`, `#5BABBC`, `#8FCCD9`, `#D9EEF3` — Highlights, feature backgrounds, subtle hover states.

### Surface

- **Canvas** (`{colors.canvas}` — `#FFFFFF`): The page floor.
- **Surface Warm** (`{colors.surface-warm}` — `#FAF7F2`): Cream tint for warm content sections.
- **Surface Cool** (`{colors.surface-cool}` — `#F0F4F6`): Cool teal-tinted background for cool content sections.

### Text

- **Ink** (`{colors.ink}` — `#0F172A`): Dominant text color. Charcoal, near-black, never pure black.
- **Body** (`{colors.body}` — `#475569`): Secondary running text — warm gray.
- **Muted** (`{colors.muted}` — `#64748B`): Tertiary text and captions.

### Accent System

A minimal secondary palette — used only for status indicators, not primary surfaces:

- **Amber** (`#F59E0B`): Warning states, attention markers.
- **Emerald** (`#10B981`): Success states, positive metrics in dashboards.

### Semantic

- **Destructive** (`{colors.destructive}` — `#DC2626`): Tailwind red-600 for errors and destructive actions. Distinct from brand teal so error states never compete with primary CTAs.

## Typography

**The brand uses a single sans-serif typeface — Inter — for all roles.** This is a deliberate restraint, not a gap.

- **Display, body, UI:** Inter (Google Fonts, SIL Open Font License — free for commercial use).
- **Fallback stack:** `Inter, system-ui, -apple-system, sans-serif` — graceful degradation to the OS system font when Inter fails to load.

Inter's variable axes (weight, optical size) cover display-to-body needs without a second typeface. Font weight does the hierarchy work: `font-weight: 700` for display headings, `600` for sub-headings, `400-500` for body and UI.

**For Softr Vibe Coding blocks:** Inter is the simplest possible choice — it's already a Softr font default, requires no custom `@font-face` declaration, and renders consistently across browsers. The `custom-code-header.html` generated by this skill for Northwind would just load Inter from Google Fonts (no licensed-font fallbacks needed).

## Layout, Elevation, Components

**Rounded:** 8–12px on most surfaces, 6px on small buttons/inputs, fully pill (`9999px`) on chips and badges. No sharp corners anywhere.

**Elevation:** Two tiers — a subtle resting shadow on cards (`shadow-card`) and a lift-on-hover state (`shadow-card-hover`). A third `shadow-elevated` for floating panels (popovers, modals). Avoid heavy shadows — the brand reads as light, ordered, and structured.

**Components:**
- Primary button: solid teal background, white text, 44px height, 8px radius.
- Secondary button: white background, charcoal text, 1px hairline border, same dimensions.
- Card: white surface, hairline border, 12px radius, 24px padding, subtle resting shadow.
- Section backgrounds: alternate `surface-warm` and `surface-cool` for categorical variety, never solid color blocks.

## Do's and Don'ts

**Do:**
- Use the brand teal (`#0E6B7A`) as the single voltage on primary CTAs.
- Use **Inter** for everything — display, body, UI. Don't introduce a second typeface.
- Use warm cream (`surface-warm`) and cool teal (`surface-cool`) for *section* backgrounds, not individual component fills.
- Use charcoal (`#0F172A`) for body text — never pure `#000`.
- Use accent amber/emerald only for status indicators (warning/success), never for primary surfaces.

**Don't:**
- Don't introduce a second brand color in the teal family — the existing 9-shade ramp gives sufficient range.
- Don't introduce a serif typeface or a second sans-serif. Single typeface, single voltage, single voice.
- Don't use heavy shadows or layered elevation — modern B2B style stays close to flat with subtle resting elevation.
- Don't use emoji or playful microcopy — the voice is measured and confident, not chatty.

## Notes on This Example

Because this is a synthesized reference rather than a real extraction:

- All tokens are example-only — there are no `extracted` / `inferred` / `needs-verification` confidence flags on individual tokens.
- The `extraction_status` block at top is marked as `example-only` rather than reflecting a real pipeline outcome.
- There's no "Known Gaps" section listing what couldn't be captured — every value is intentional.

In a *real* extraction produced by the Apify + Exa pipeline, you'd see those elements throughout the document. See [extractors/apify-pipeline.md](../extractors/apify-pipeline.md) for the pipeline details and [references/tailwind-class-trap.md](../references/tailwind-class-trap.md) for common extraction gotchas (e.g., misleading class-count signals on Tailwind sites).
