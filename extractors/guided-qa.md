# Path D — Guided Q&A

When the user has no website, no catalog match, and no existing `DESIGN.md` to upload — only a brand "in their head" — Path D builds a minimal `DESIGN.md` from 8 targeted questions.

This produces a Tier 1 brand kit (the seven required tokens plus a voice paragraph). Tier 2 fields (typography hierarchy, component recipes, spacing scale, shadow rules) are populated with sensible defaults marked `inferred`. The user can iterate later by re-running Path A on a future reference site or uploading a more complete file.

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

If the user provides a URL instead of a hex, run a single Apify `automation-lab/css-color-extractor` call on it and pull the most-used color. Confirm the hex with the user before continuing.

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

Translate to a radius scale in the `rounded` block:

```yaml
rounded:
  status: "extracted-by-user"
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

If they pick option 4, ask the font name and add it to the YAML with `font-display-licensing: "Paid font specified by user"`. Set the open-source fallback to Inter or whatever Google Fonts substitute is closest.

### Q6. Shadow vibe (pick one)

> "How should depth feel?
> 1. Flat — no shadows anywhere (Linear, Apple, Airbnb)
> 2. One subtle tier — cards lift on hover only (most modern marketing sites)
> 3. Layered — multiple shadow tiers, more dimensional (older SaaS look)"

Translate to the `elevation` block. Flat = empty shadow rules. One-tier = the standard recipe in [../design-md-format.md](../design-md-format.md). Layered = three shadow tiers (`sm`, `md`, `lg`).

### Q7. Reference site (optional)

> "Is there any website that captures the vibe you want — even loosely? Paste a URL.
> This is optional but high-leverage — I can use it to refine my defaults."

If the user provides one, treat it as a "soft Path A": run the Apify color extractor on it as a sanity check and use the dominant color family to refine the palette generated from Q1–Q2. Don't override the user's primary color from Q1, but pull a couple of complementary tints if the reference site has a clear ramp.

### Q8. Logo URL (optional)

> "If you have a logo URL handy, paste it. I'll reference it in the DESIGN.md so downstream tools can render branded headers."

Save to the YAML as `logo_url`. Mark `extracted-by-user` if provided, omit the field if not.

## Synthesis

After all 8 answers:

1. Map mood + radius + shadow + typography vibes to default values.
2. Override defaults with explicit user answers (Q1, Q2, Q5 specific font name).
3. Generate the `description` paragraph (the "atmosphere" section) from the mood choice plus any free-text the user added.
4. Generate the `Voice & Copy Register` section with a placeholder: "User did not provide hero copy during Q&A. Match microcopy to the chosen mood: `<mood>`."
5. Mark `extraction_status`:

```yaml
extraction_status:
  colors: extracted-by-user
  typography: extracted-by-user
  voice_and_mood: inferred (from mood selection — verify with hero copy when available)
  spacing_radius_shadows: inferred (from vibe selections)
```

6. Add a `Known Gaps` section noting that this is a Q&A-derived kit and recommending iteration with Path A on a real reference site once one exists.

## Constraints

- Never run more than the 8 questions. If the user gets impatient or skips, accept the defaults rather than asking more.
- Never produce more than one DESIGN.md from a single Q&A run.
- Always allow "skip" or "I don't know" — fall back to defaults silently and flag the affected tokens.
- Never invent a font name from imagination. If Q5 returns "I don't know", default to Inter and mark `font-display-name: needs-verification`.

## Reference example

After a complete Q&A run, the resulting DESIGN.md should look like a smaller version of the [northwind-studio-DESIGN.md](../examples/northwind-studio-DESIGN.md) example, with `inferred` flags on the typography hierarchy and `extracted-by-user` on the Q1/Q2 colors.
