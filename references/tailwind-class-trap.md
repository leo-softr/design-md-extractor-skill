# The Tailwind Class-Name Trap

A documented failure mode that the skill must always guard against on Tailwind-based sites. The pipeline's Tier 2a (HTML class-count analysis) can produce a confidently wrong reading of typography because **Tailwind utility class names are arbitrary labels, not semantic descriptions of the font they reference**.

## The trap, in one sentence

A site that uses the class `font-serif` 200+ times can be entirely sans-serif. The Tailwind config controls what `font-serif` resolves to, and that resolution can be anything.

## How the trap fires

In `tailwind.config.js` (or the CSS-native v4 equivalent):

```js
// tailwind.config.js
module.exports = {
  theme: {
    fontFamily: {
      // Brand maps "serif" utility class to a SANS font:
      serif: ['PPNeueMontreal', 'Arial', 'sans-serif'],
      sans: ['KitSans', 'Arial', 'sans-serif']
    }
  }
}
```

Once compiled, the rendered page contains markup like:

```html
<h1 class="font-serif text-4xl">Become the business everyone wants to beat</h1>
```

Static analysis of the HTML (Tier 2a) sees `font-serif` and concludes "the brand is using a serif typeface for headlines." But `getComputedStyle(h1).fontFamily` returns `PPNeueMontreal, Arial, sans-serif` — a sans-serif typeface.

This is what happened on a real B2B agency site we ran the pipeline against (Astro + Sanity + Tailwind stack). The class-count analysis returned `font-serif: 209 uses, font-sans: 3 uses`. Confident "serif-dominant" reading. Wrong by 100%.

## Why brands do this

Naming `font-serif` to a sans typeface is unusual but common enough to be a real risk. Reasons:

- **Migration baggage.** A brand migrated from a serif font to a sans-serif one and never renamed the utility class.
- **Tailwind config inertia.** The default Tailwind config has `serif` and `sans` slots. Designers keep those slot names even when they pick non-conforming fonts.
- **Custom font slot remapping.** Some brands use `font-serif` as a "display" alias and `font-sans` as a "body" alias, regardless of whether either is actually serif.

The trap is documented in modern Tailwind/Astro/Next.js theming guides but easy to miss when extracting from outside.

## How to avoid the trap

**Always run Tier 2b before locking typography decisions on Tailwind sites.**

Tier 2b uses `apify/web-scraper` with a `pageFunction` that reads `getComputedStyle(el).fontFamily` for representative selectors. Computed styles are the rendered truth — they reflect what the browser actually displays, post-Tailwind-resolution.

If Tier 2b can't run (the apify/web-scraper approval is missing and the user declines to grant it), the skill must:

1. Still record the class-count signal — it's not useless, just incomplete.
2. Mark `typography_specific_fonts: needs-verification` explicitly.
3. Surface the verification prompt with: "I couldn't confirm the exact font name. The Tailwind utility classes suggest it might be a `<class-implied>` typeface, but those classes can be misleading on this kind of site. Paste the font name from your brand guide."

Never claim the brand is "X-dominant" based on class counts alone. Always hedge.

## Detecting Tailwind use

The `misterkhan/website-tech-stack-scanner` actor (Tier 1.3) reports Tailwind in its detected technologies. If Tailwind is in the result, this trap applies. If Tailwind is not in the result but `font-*` utility classes are common in the HTML body, treat the site as Tailwind-like and apply the same caution.

Other sites where this trap applies:

- Tailwind-CSS-based sites generally (Astro, Next.js with Tailwind, modern Webflow with Tailwind extension).
- Sites using utility-first CSS frameworks like UnoCSS, WindiCSS — same trap, same mitigation.
- Custom design systems with semantic-named utility classes (`heading-font`, `body-font`) — same family of problem.

This trap does **not** apply to sites that use:

- Pure CSS with `@font-face` declarations and `font-family` rules tied to specific selectors (`h1`, `body`, `.card-title` etc.) — Tier 1.2 (`automation-lab/css-stats-analyzer`) catches these.
- CSS custom properties on `:root` that resolve to font names (`--font-display: 'Söhne'`) — Tier 2b's CSS-variable extraction catches these.

## What to write in DESIGN.md when this trap is caught

Document it explicitly in the `Typography` prose section:

> **Important trap caught by Tier 2b.** Tailwind class-count analysis showed `font-serif` used 209 times vs. `font-sans` 3 times, suggesting a serif-dominant brand. That was misleading — the `tailwind.config.js` aliases the `font-serif` utility class to a sans-serif font (`PPNeueMontreal`). The class name is just a label; only the computed style tells the truth. **Lesson: always run Tier 2b before locking typography decisions.** Class names lie.

Including this note keeps downstream consumers honest and trains future versions of this skill to expect the trap.

## Reference example

See [../examples/northwind-studio-DESIGN.md](../examples/northwind-studio-DESIGN.md) for a complete reference DESIGN.md showing the schema (note: the example uses a single-typeface system so it does not itself reproduce this specific trap — the trap is documented standalone here).
