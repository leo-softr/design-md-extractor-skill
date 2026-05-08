# Generating `custom-code-header.html`

The skill produces two artifacts. The second one is `custom-code-header.html` — a small HTML/CSS snippet the user pastes into Softr's `Settings → Custom Code → Code inside header` so brand tokens are loaded once at the app level and inherited by every block.

This file is a Softr-specific output. Other downstream consumers (Webflow, custom React) may not need it. The skill should still produce it whenever a custom design is configured, since it's harmless if unused.

## Why app-level (not block-level)

The Softr Vibe Coding block automatically inherits the app's global theme — accent color, fonts, border radius — from `Settings → Theme` and from custom CSS loaded via `Settings → Custom Code`. Loading brand tokens at the app level once means:

- Every block in the app uses the same brand without duplicating token declarations.
- Google Fonts load once per page session, not per block.
- The user can update the brand in one place and every block follows.

The block-level fallback (injecting `<link>` via `useEffect`) is documented in the downstream Softr Vibe Coding skill and only used when the user can't or won't touch app settings.

## Snippet structure

The full snippet has three parts:

```html
<!-- 1. Google Fonts preconnect + load -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=<font-name>:wght@<weights>&display=swap" rel="stylesheet">

<!-- 2. Brand tokens as CSS custom properties + applied font/color rules -->
<style>
  :root {
    --brand-primary: <hex>;
    --brand-primary-pressed: <hex>;
    --brand-ink: <hex>;
    --brand-canvas: <hex>;
    --brand-destructive: <hex>;
    --brand-radius-sm: <px>;
    --brand-radius-md: <px>;
  }
  html, body {
    font-family: <font-stack> !important;
    color: var(--brand-ink) !important;
    background-color: var(--brand-canvas) !important;
  }
  h1, h2, h3, h4, h5, h6 {
    font-family: <display-font-stack> !important;
    font-weight: 600 !important;
    color: var(--brand-ink) !important;
  }
</style>
```

The `!important` flags on the `html, body` and `h1–h6` rules are mandatory for Softr — Softr's theme CSS loads after this snippet and would otherwise override the brand fonts on built-in surfaces (navigation, login forms, account pages, list blocks). See Step 5 for the full explanation.

## Generating from DESIGN.md

Read `./DESIGN.md` and synthesize the snippet:

### Step 1 — Identify the font(s) to load

Look at the `typography` block. For each font:

- If `font-*-name` is on Google Fonts (Inter, Manrope, Public Sans, Fraunces, etc.), generate a `<link>` for it. Pull the weights actually used from the hierarchy table (display weights + body weights).
- If `font-*-name` is a paid foundry font (PPNeueMontreal, Söhne, etc.), use the `font-*-open-source-substitute` instead. Add a comment noting the substitution.
- If both display and body fonts are the same Google Fonts entry, generate one `<link>` with combined weights.

### Step 2 — Build the Google Fonts URL

Format:

```
https://fonts.googleapis.com/css2?family=<Font+Name>:wght@<weights>&display=swap
```

Example for Inter at weights 400, 500, 600, 700:

```
https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap
```

For multiple fonts, use one `<link>` per font (cleaner than the combined `&family=` query syntax).

### Step 3 — Define CSS custom properties

For each token in the DESIGN.md `colors` block, add a `--brand-*` variable on `:root`. Naming convention: convert `colors.foo-bar` to `--brand-foo-bar`.

```yaml
# DESIGN.md
colors:
  primary: "#e82d42"
  primary-pressed: "#ce2439"
  ink: "#120b0c"
  canvas: "#ffffff"
  destructive: "#ef4444"
```

becomes:

```css
:root {
  --brand-primary: #e82d42;
  --brand-primary-pressed: #ce2439;
  --brand-ink: #120b0c;
  --brand-canvas: #ffffff;
  --brand-destructive: #ef4444;
}
```

Include the four required colors at minimum (`primary`, `ink`, `canvas`, `destructive`) plus any additional brand-significant colors (tint ramp, surface variants, accents). Skip colors the brand uses but doesn't ship as a token (e.g. one-off hex values used only on a single page).

### Step 4 — Define the radius and spacing variables (optional but recommended)

If the DESIGN.md has a `rounded` block with `extracted` or `extracted-by-user` status, include the dominant radius values:

```css
:root {
  --brand-radius-sm: 8px;
  --brand-radius-md: 14px;
  --brand-radius-lg: 20px;
  --brand-radius-full: 9999px;
}
```

Skip if `rounded.status` is `inferred` or `needs-verification` — let downstream tools use their defaults rather than locking inferred values into the global CSS.

### Step 5 — Set the body, heading, and background with `!important`

Add rules applying the body font stack to `html, body`, the display font stack to `h1–h6`, and the brand canvas as the page background on `html, body`. **All three properties require `!important`** because Softr's theme CSS loads after this snippet — see the box below.

```css
html, body {
  font-family: 'Inter', -apple-system, system-ui, sans-serif !important;
  color: var(--brand-ink) !important;
  background-color: var(--brand-canvas) !important;
}
h1, h2, h3, h4, h5, h6 {
  font-family: 'Inter', -apple-system, system-ui, sans-serif !important;
  font-weight: 600 !important;
  color: var(--brand-ink) !important;
}
```

Use `font-body-fallback-stack` for the body rule and `font-display-fallback-stack` for the heading rule (or the open-source substitute followed by the system stack if the source font is paid). If body and display use the same font, both rules can reference the same family.

The `background-color` `!important` flag is what makes the brand canvas extend to the entire page — Softr's theme defaults the body to white. Without `!important`, the canvas variable is set on `:root` but no element actually paints with it on Softr's chrome (header strip between Softr nav and the first block, page background outside the block's max-width). Users see white margins around their cream-canvas blocks instead of a unified brand surface.

#### Policy: `!important` on every declaration in this file — including custom properties

CSS does not have a global flag that makes every declaration in a `<style>` block `!important`. Each property has to be tagged individually. Given that:

**Apply `!important` to every line in `custom-code-header.html`. No exceptions.** That includes:

- Every applied-style property in `html, body { ... }` (`font-family`, `color`, `background-color`, anything you set)
- Every applied-style property in `h1, h2, h3, h4, h5, h6 { ... }` (`font-family`, `font-weight`, `letter-spacing`, `line-height`, `color`, `font-size` — every line)
- **Every CSS custom property on `:root`** (`--brand-primary: #xxx !important;` and friends)

The first two categories obviously need it (Softr's theme CSS loads after and competes for the same selectors). The custom-property case is the one people skip — and it's the one users notice as inconsistent. Custom properties on `:root` rarely have competition in practice, but adding `!important` is uniform-defensive at zero cost: if Softr ever introduces a brand-namespaced custom property of their own, or the user adds a second custom-code snippet later, or a third-party Softr plugin sets a `--brand-*` value, the brand still wins.

The philosophy: **anything in this file is brand-intentional. The brand wins.** Treat `!important` as the default policy of the file, not a per-line judgment call.

Add a top-of-file marker comment so the policy is auditable at a glance:

```html
<!-- Policy: every declaration in this file is brand-intentional. !important is applied
     to every property, including CSS custom properties on :root, so brand values always
     win against Softr's theme CSS (which loads after this snippet) and against any future
     custom-code additions. CSS doesn't have a global "make everything important" flag;
     each property has to be tagged explicitly. -->
```

Generated example:

```css
:root {
  --brand-primary: #e82d42 !important;
  --brand-canvas: #ffffff !important;
  /* ...every other --brand-* line gets !important too */
}

html, body {
  font-family: 'Inter', -apple-system, system-ui, sans-serif !important;
  color: var(--brand-ink) !important;
  background-color: var(--brand-canvas) !important;
}

h1, h2, h3, h4, h5, h6 {
  font-family: 'Inter', -apple-system, system-ui, sans-serif !important;
  font-weight: 600 !important;
  color: var(--brand-ink) !important;
  letter-spacing: -0.005em !important;
}
```

The `<link>` tags for Google Fonts are unaffected by this policy — they don't take `!important`. They load `@font-face` declarations, which then become available globally.

#### Why `!important` is mandatory here

> **Softr's theme CSS loads after this custom header**, which means any brand font / color rule you set on `html`, `body`, or `h1–h6` gets overridden by Softr's theme defaults on every Softr-rendered surface — navigation, login forms, account pages, list blocks, etc. With `!important`, the brand wins on those native surfaces.
>
> **Important caveat — the `!important` rules do NOT reach inside Vibe Coding blocks.** Vibe Coding blocks render inside a shadow DOM (or similarly-isolated subtree). The `html, body` and `h1–h6` selectors don't match anything inside the shadow root because those elements live in the parent document. So the `!important` rules in this header file apply to:
>
> - ✅ Softr's native surfaces (navigation, login, account, footer, list blocks)
> - ❌ Vibe Coding block content (must apply brand fonts/colors at the block's own outer wrapper)
>
> What DOES cross the shadow DOM boundary:
>
> - ✅ **CSS custom properties** (`--brand-*` on `:root`) — these pierce the shadow boundary, so `var(--brand-primary)` works inside Vibe Coding blocks
> - ✅ **`@font-face` declarations** loaded via the Google Fonts `<link>` tags — the font files are downloaded into the document and are available everywhere, including inside shadow DOM. Vibe Coding blocks just need to **apply** them via inline `fontFamily` on their outer wrapper.
>
> Bottom line: this header file is necessary but not sufficient for full brand application. The `softr-vibe-coding` skill's blocks must also set `fontFamily` (and any other non-inheriting brand defaults) inline on each block's outermost `<div>`.
>
> Per the policy above, mark **every declaration in this file** `!important`, including CSS custom properties on `:root`. The custom-property case is uniform-defensive — no expected competition today, but cheap insurance against future custom-code additions or theme changes.
>
> If the downstream tool is not Softr (e.g., the snippet is being adapted for a custom React app or a static site), `!important` is optional but harmless.

### Step 6 — Add a header comment

Open the snippet with a comment naming the brand and the source DESIGN.md timestamp:

```html
<!-- Brand tokens for <Brand Name> -->
<!-- Generated by brand-extraction skill from DESIGN.md on 2026-05-06 -->
<!-- Paste into Softr → Settings → Custom Code → Code inside header -->
```

This helps future maintainers understand where the snippet came from.

## Complete worked example — example.com

Given the [northwind-studio-DESIGN.md](../examples/northwind-studio-DESIGN.md), the generated snippet would be:

```html
<!-- Brand tokens for Northwind Studio -->
<!-- Generated by brand-extraction skill from DESIGN.md on 2026-05-06 -->
<!-- Paste into Softr → Settings → Custom Code → Code inside header -->

<!-- Policy: every declaration in this file is brand-intentional. !important is applied
     to every property, including CSS custom properties on :root, so brand values always
     win against Softr's theme CSS (which loads after this snippet) and against any future
     custom-code additions. CSS doesn't have a global "make everything important" flag;
     each property has to be tagged explicitly. -->

<!-- Google Fonts: Inter (open-source substitute for paid PPNeueMontreal and KitSans) -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">

<style>
  :root {
    /* Brand voltage */
    --brand-primary: #e82d42 !important;
    --brand-primary-pressed: #ce2439 !important;
    --brand-primary-tint-md: #ed5566 !important;
    --brand-primary-tint-xs: #f6a6af !important;

    /* Text */
    --brand-ink: #120b0c !important;
    --brand-body: #57534e !important;

    /* Surface */
    --brand-canvas: #ffffff !important;
    --brand-surface-pink: #fff6f7 !important;
    --brand-surface-peach: #fff0eb !important;
    --brand-surface-cream: #fff6e9 !important;

    /* Semantic */
    --brand-destructive: #ef4444 !important;
    --brand-on-primary: #ffffff !important;
  }

  html, body {
    font-family: 'Inter', -apple-system, system-ui, sans-serif !important;
    color: var(--brand-ink) !important;
    background-color: var(--brand-canvas) !important;
  }
  h1, h2, h3, h4, h5, h6 {
    font-family: 'Inter', -apple-system, system-ui, sans-serif !important;
    font-weight: 600 !important;
    color: var(--brand-ink) !important;
  }
</style>
```

## Constraints

- **Always use `!important` on the `html, body` and `h1–h6` rules** when the target is Softr. Softr's theme CSS loads after this snippet and overrides any non-`!important` font/color rule on those selectors. Without the flag, brand fonts only apply inside Vibe Coding blocks (shadow DOM) — not in navigation, login, account, or any other Softr-rendered surface. CSS custom properties (`--brand-*`) on `:root` do NOT need `!important` — they don't compete for specificity.
- **Never include API keys or tokens** in the snippet. It will be exposed in the user's HTML head.
- **Never load licensed fonts** from URLs the brand doesn't actually serve them at. Use Google Fonts substitutes for any paid foundry fonts.
- **Keep the snippet under 100 lines.** Softr's Custom Code field has practical length limits and pasting a wall of CSS slows down theme inheritance.
- **Don't generate the snippet for `style: premium-default` runs.** When the user opted out of custom design (gate question = "no"), the snippet is unnecessary — write a one-line comment instead: `<!-- No custom brand applied. Softr defaults will be used. -->`.

## Where this gets pasted

After Step 7 in `SKILL.md`, tell the user:

> "The Custom Code snippet is at `./custom-code-header.html`. To apply your brand across all Softr blocks in this app:
>
> 1. Open your Softr app.
> 2. Go to **Settings → Custom Code → Code inside header**.
> 3. Paste the contents of `custom-code-header.html`.
> 4. Save and publish.
>
> Now any block you generate (with the Softr Vibe Coding skill or otherwise) will inherit these tokens automatically."
