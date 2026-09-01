# Path D — Guided Q&A

When the user has no website, no catalog match, and no existing `DESIGN.md` to upload — only a brand "in their head" — Path D builds a minimal `DESIGN.md` from 8 targeted questions.

This produces a minimal brand kit: core color / typography / radius tokens plus a voice paragraph, mapped into the same dembrandt-shaped frontmatter keys as every other path (see [../design-md-format.md](../design-md-format.md)). Remaining sections get sensible defaults marked `inferred` in `extraction_status`. The user can iterate later by re-running Path A on a future reference site or uploading a more complete file.

## When to use Path D

Suggest Path D when:

- User says "I don't have a site yet" or "we're pre-launch."
- User describes a brand verbally without referencing a URL.
- Path A and Path B both failed (site is bot-blocked AND not in catalog).

Path D is also a good "warm-up" path — for users who want to draft a brand kit and refine it later.

## The 8 questions

Ask in this order. Use multi-choice formatting where possible. Always allow free-text for "Other".

### Q1. Brand primary color (hex)

> "What's your brand's primary color? Paste the hex value (e.g. `#386AF5`).
> If you don't know, paste a URL with the color and I'll eyedropper it."

If the user provides a URL instead of a hex, run a quick dembrandt `get_color_palette` call on it (or CLI: `npx -y dembrandt@latest <url> --json-only`) and pull the primary role's hex. The MCP call is async by default — pass `sync: true` for this single-page case (blocks ~15–40s), or make one `get_job_status` poll. If dembrandt isn't set up, don't install it for this — ask the user to paste the hex (or describe the color) instead. Confirm the hex with the user before continuing.

### Q2. Accent / secondary color (hex, optional)

> "Do you have a second brand color? Paste the hex, or say 'no' if you only have one brand color.
> Single-color brands are common — I won't invent a second if you don't have one."

Accept "no" / "none" / "skip" / single color as valid. Do not invent a secondary if the user says they don't have one.

### Q3. Visual mood (pick one or describe)

> "Which best describes your brand's mood?
> 1. Friendly consumer (warm, approachable, photo-led)
> 2. Editorial / minimal (lots of whitespace, sentence-case, restrained)
> 3. Dark cinematic (dark canvas, neon accents, dramatic)
> 4. Corporate clean (light canvas, structured, conservative)
> 5. Playful / illustrated (bright, hand-drawn, whimsical)
> 6. Fintech / precise (technical, blue-grey, dense)
> 7. Other — describe in 1-2 sentences"

Map the chosen mood to default values for typography, radius, and shadow tier:

| Mood | Default radius | Default shadow | Default font vibe |
|---|---|---|---|
| Friendly consumer | 14px (soft) | One-tier on hover | Sans, modest weights (Inter / Manrope) |
| Editorial minimal | 4px (sharp) | Flat, no shadow | Sans, light weights (Inter / Public Sans) |
| Dark cinematic | 8px | Layered, high contrast | Sans, heavy display weights (Inter / Geist) |
| Corporate clean | 6px | One-tier subtle | Sans, structured (Inter / IBM Plex) |
| Playful illustrated | 20px (very rounded) | Layered with color tint | Sans, friendly (Manrope / Plus Jakarta) |
| Fintech precise | 4px | Flat | Sans + mono accents (Inter + JetBrains Mono) |

These defaults are starting points — the user's later answers override.

### Q4. Border radius vibe (pick one)

> "Pick a shape language for the brand:
> 1. Sharp (0–2px) — serious, financial, technical
> 2. Soft (8–14px) — friendly, modern, default
> 3. Very rounded (20px+) — playful, consumer
> 4. Fully pill (9999px on small elements) — bold, distinctive"

Translate to a radius scale in the dembrandt-shaped `rounded` block. No `status` field inside it — v2 base keys never carry per-token status; the user-supplied confidence is recorded in `extraction_status` only:

```yaml
rounded:
  none: 0px
  xs: <half of base>
  sm: <base>
  md: <1.5x base>
  lg: <2x base>
  xl: <3x base>
  full: 9999px
```

For "soft" base = 8px; for "sharp" base = 2px; for "very rounded" base = 12px; for "pill" base = 16px.

### Q5. Typography vibe (pick one or specify)

> "What kind of typography?
> 1. System sans (Inter is the modern default)
> 2. Serif display + sans body (e.g. Fraunces + Inter — editorial)
> 3. Mono accents (e.g. JetBrains Mono for code, Inter for body — developer-tool feel)
> 4. Custom paid font — name it, I'll set fallbacks"

Record the answer in the top-level `fonts` frontmatter block (the v2 home for resolved font names — see [../design-md-format.md](../design-md-format.md)). For option 4, ask the font name and fill it like this; for options 1–3, fill the same block from the vibe defaults:

```yaml
fonts:
  display-name: "<user's font>"
  display-fallback-stack: "'<user's font>', system-ui, sans-serif"
  display-open-source-substitute: "<closest Google Fonts match — ../references/google-fonts-substitutes.md>"
  display-licensing: "Paid — specified by user"      # or "Google Fonts" for options 1–3
  body-name: "Inter"
  body-fallback-stack: "'Inter', system-ui, sans-serif"
  body-licensing: "Google Fonts"
```

### Q6. Shadow vibe (pick one)

> "How should depth feel?
> 1. Flat — no shadows anywhere (Linear, Apple, Airbnb)
> 2. One subtle tier — cards lift on hover only (most modern marketing sites)
> 3. Layered — multiple shadow tiers, more dimensional (older SaaS look)"

Translate into the body's `## Elevation & Depth` prose section — v2 has no `elevation` frontmatter key. Flat = omit the section entirely (keys and sections with no evidence stay absent, per the evidence-only rule) and note the deliberate flatness in Do's and Don'ts. One-tier = write a concrete default inline, e.g. `box-shadow: 0 1px 3px rgba(0,0,0,0.04), 0 4px 12px rgba(0,0,0,0.06)` on cards, hover-lift only. Layered = three tiers of increasing spread (`sm`, `md`, `lg`). Either way the values are defaults, not observations — flag them in `extraction_status.tokens`' parenthetical or as a `Known Gaps` row.

### Q7. Reference site (optional)

> "Is there any website that captures the vibe you want — even loosely? Paste a URL.
> This is optional but high-leverage — I can use it to refine my defaults."

If the user provides one, treat it as a "soft Path A": run a quick dembrandt `get_color_palette` extraction on it as a sanity check and use the dominant color family to refine the palette generated from Q1–Q2. Don't override the user's primary color from Q1, but pull a couple of complementary tints if the reference site has a clear ramp. Same call note as Q1: `get_color_palette` is async by default — prefer `sync: true` for this single-page check, or one `get_job_status` poll. If dembrandt isn't set up, don't install it for this — the reference-site refinement is skippable; keep the URL in `Known Gaps` for a future Path A run.

### Q8. Logo URL (optional)

> "If you have a logo URL handy, paste it. I'll reference it in the DESIGN.md so downstream tools can render branded headers."

If provided, record it in the `assets` frontmatter block per the schema in [../design-md-format.md](../design-md-format.md) (Path D does no download — the public URL is the canonical reference anyway):

```yaml
assets:
  status: extracted-by-user
  logo_wordmark:
    public_url: "<the URL the user pasted>"
    public_url_note: "Canonical reference for production use. Softr cannot resolve project-relative paths."
    local_backup: null
    local_backup_note: "Not downloaded in Path D — upload manually to Softr's media library if self-hosting."
    format: "<SVG | PNG | WebP | JPG — from the URL>"
    usage: "Use as <img src='<public_url>' alt='<Brand>' /> in nav lockups. Do not rebuild the wordmark in CSS."
```

If the user gives nothing, omit the `assets` block entirely and add a `Known Gaps` row instead ("logo not captured — Q&A path; paste a URL when one exists").

## Synthesis

After all 8 answers:

1. Map mood + radius + shadow + typography vibes to default values.
2. Override defaults with explicit user answers (Q1, Q2, Q5 specific font name).
3. Populate the dembrandt-shaped frontmatter keys from the results (see [../design-md-format.md](../design-md-format.md)): `colors` roles (`primary` from Q1, `secondary` from Q2 when given, `surface` / `on-surface` from the mood defaults), `typography` tokens, `spacing`, `rounded` — plus the skill-owned `fonts` block from Q5. Keys with no answer and no sensible default stay absent, not invented.
4. Generate the `description` paragraph (the "atmosphere" section) from the mood choice plus any free-text the user added.
5. Generate the `Voice & Copy Register` section with a placeholder: "User did not provide hero copy during Q&A. Match microcopy to the chosen mood: `<mood>`."
6. Mark `extraction_status` (section-level flags only, per [../references/confidence-flags.md](../references/confidence-flags.md)):

```yaml
extraction_status:
  tokens: extracted-by-user (Q&A hexes; radius/shadow defaults inferred from vibe answers)
  fonts: inferred (from vibe answers — extracted-by-user when Q5 named an exact font)
  voice_and_mood: extracted (from Q&A mood answers — verify with hero copy when available)
  app_patterns: scaffolded
```

Add `assets: extracted-by-user (URL pasted in Q8)` when Q8 produced a logo.

7. Add a `Known Gaps` section noting that this is a Q&A-derived kit and recommending iteration with Path A on a real reference site once one exists.

## Constraints

- Never run more than the 8 questions. If the user gets impatient or skips, accept the defaults rather than asking more.
- Never produce more than one DESIGN.md from a single Q&A run.
- Always allow "skip" or "I don't know" — fall back to defaults silently and flag the affected sections in `extraction_status` (itemize in `Known Gaps`).
- Never invent a font name from imagination. If Q5 returns "I don't know", default to Inter in the `fonts` block and mark `extraction_status.fonts: needs-verification`.

## Reference example

After a complete Q&A run, the resulting DESIGN.md should look like a smaller version of the [northwind-studio-DESIGN.md](../examples/northwind-studio-DESIGN.md) reference example (v2: dembrandt-shaped frontmatter, section-level flags) — with `extracted-by-user` on `extraction_status.tokens` for the Q1/Q2 colors and `inferred` on `extraction_status.fonts`. Never per-token flags.
