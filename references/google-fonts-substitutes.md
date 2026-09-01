# Google Fonts Substitutes for Licensed Brand Fonts

When a brand uses a paid foundry font that isn't on Google Fonts, downstream consumers (especially Softr blocks rendered in a runtime that doesn't load licensed fonts) need an open-source fallback. This table maps the most common licensed brand fonts to the closest Google Fonts substitute.

Use this table when synthesizing the `display-open-source-substitute` and `body-open-source-substitute` fields in the DESIGN.md `fonts` frontmatter block.

## Substitute table

### Sans-serif

| Licensed font | Foundry / source | Google Fonts substitute | Notes |
|---|---|---|---|
| **Söhne / Söhne Mono** | Klim Type | **Inter** | Closest tonally; Inter is a clean grotesque. |
| **Neue Haas Grotesk** | Linotype | **Inter** or **Public Sans** | Both are revivals of the Helvetica family. |
| **PPNeueMontreal** (Neue Montreal) | Pangram Pangram | **Inter** or **Manrope** | Inter is the safest single match. |
| **GT America** / **GT Walsheim** | Grilli Type | **Manrope** or **Mulish** | Both are warm geometric sans options. |
| **Founders Grotesk** | Klim Type | **Plus Jakarta Sans** | Slightly more rounded, similar weight progression. |
| **Aktiv Grotesk** | Dalton Maag | **Public Sans** | Both are neutral grotesques. |
| **Söhne Schmal** (condensed) | Klim Type | **Barlow Condensed** | |
| **Calibre** | Klim Type | **Inter** | |
| **Suisse Int'l** | Swiss Typefaces | **Inter** or **Hind** | |
| **Untitled Sans** | Klim Type | **Inter** | |
| **TWK Lausanne** | Weltkern | **Public Sans** | |
| **Geist Sans** | Vercel | **Geist** (now on Google Fonts) | Self-publishing, available open. |
| **Airbnb Cereal** | Airbnb (proprietary) | **Inter** | Documented in Airbnb's DESIGN.md. |
| **Vodafone** (custom) | Vodafone (proprietary) | **Inter** | Heavy condensed weights — use Inter Bold + tracking adjustment. |
| **NouvelR** | Renault (proprietary) | **DM Sans** | NouvelR is rounded and humanist. |
| **Apple SF Pro** | Apple (proprietary) | System fallback first (`-apple-system, BlinkMacSystemFont`) → **Inter** | SF Pro renders natively on Apple devices. |

### Serif

| Licensed font | Foundry / source | Google Fonts substitute | Notes |
|---|---|---|---|
| **Tiempos** | Klim Type | **Fraunces** | Both have generous x-height and editorial feel. |
| **Söhne Breit** | Klim Type | **Inter** with extended tracking (sans-serif, included for completeness) | |
| **Reckless** | Displaay | **Playfair Display** or **DM Serif Display** | High-contrast display serifs. |
| **Domaine Display** | Klim Type | **Playfair Display** | |
| **Caslon** family | Various | **Libre Caslon Text** | Open Caslon revival. |
| **Söhne Mono** | Klim Type | **JetBrains Mono** (mono, included for completeness) | |
| **Charter** | Bitstream | **Source Serif** | Both are utility serifs. |

### Monospace

| Licensed font | Foundry / source | Google Fonts substitute | Notes |
|---|---|---|---|
| **Söhne Mono** | Klim Type | **JetBrains Mono** | |
| **Berkeley Mono** | Berkeley Graphics | **JetBrains Mono** or **IBM Plex Mono** | |
| **GT America Mono** | Grilli Type | **JetBrains Mono** | |
| **MD IO** | Mass Driver | **Geist Mono** | |
| **Operator Mono** | Hoefler & Co | **JetBrains Mono** with italic variant | |

### Display / specialty

| Licensed font | Foundry / source | Google Fonts substitute | Notes |
|---|---|---|---|
| **Cabinet Grotesk** | Indian Type Foundry | **Public Sans Bold** or **Inter Black** | Cabinet has very heavy weights. |
| **PP Editorial New** | Pangram Pangram | **Fraunces** | Both are revival serifs with editorial flavor. |
| **Söhne Buch** | Klim Type | **Spectral** or **Source Serif** | |
| **Klim Calibre** | Klim Type | **DM Sans** | |

## Choice rules

1. **Prefer Inter** for any sans-serif fallback unless the brand is specifically condensed, geometric, or rounded.
2. **Prefer Fraunces** for any serif fallback unless the brand is high-contrast display (use Playfair) or specifically a Caslon revival (use Libre Caslon).
3. **Prefer JetBrains Mono** for any monospace fallback.
4. **Prefer the system stack** when the brand uses Apple's SF Pro — list `-apple-system, BlinkMacSystemFont, system-ui` first, then fall back to Inter.
5. **Match weight availability.** If the brand uses ultra-heavy weights (Cabinet, Inter Black), confirm the Google Fonts substitute has those weights before recommending.

## Multi-font fallback chains

When a brand uses two licensed fonts (e.g. PPNeueMontreal for headings + KitSans for body), recommend a single substitute that works for both unless they're tonally different. Inter usually serves both heading and body roles well; if the brand specifies a serif heading + sans body, use Fraunces + Inter.

```yaml
# Two paid fonts → single Inter substitute (most B2B sites)
fonts:
  display-open-source-substitute: "Inter"
  body-open-source-substitute: "Inter"

# Serif display + sans body → split substitute
fonts:
  display-open-source-substitute: "Fraunces"
  body-open-source-substitute: "Inter"
```

## When the licensed font isn't in this table

For obscure fonts not listed above:

1. Try a Google search for the font name + "Google Fonts alternative" — the design community often documents these.
2. If unsure, default to **Inter** for sans, **Fraunces** for serif, **JetBrains Mono** for monospace.
3. Note in the `Known Gaps` section: "Open-source substitute is a best-effort guess; cross-check the actual font via the Font URLs in the dembrandt extraction (the webfont filenames usually name the face), then verify the substitute looks similar."

## Reference: how this gets used

The DESIGN.md `fonts` frontmatter block should always include both the licensed name and the substitute (never write these into the `typography` block — that's dembrandt-owned base and is never restructured):

```yaml
fonts:
  display-name: "PPNeueMontreal"
  display-fallback-stack: "'PPNeueMontreal', Arial, sans-serif"
  display-open-source-substitute: "Inter"
  display-licensing: "Paid — Pangram Pangram"
```

Downstream consumers (Softr, Webflow, etc.) decide whether to load the licensed font or fall back to the substitute based on what's available in their runtime.
