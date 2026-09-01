# Application Patterns Stubs (Softr)

Scaffolds for app patterns that cannot be extracted from a marketing-source brand audit. The skill appends these to every generated `DESIGN.md` (Step 4b) using the **actual brand tokens** captured in the frontmatter — they're starting recipes, not blanks.

**Softr-specific.** All patterns below assume Softr's stack:

- **Components**: shadcn/ui imported from `@/components/ui/<name>` (Button, Card, Dialog, Input, Select, AlertDialog, Skeleton, Badge, etc.)
- **Styling**: Tailwind CSS classes inline
- **Icons**: `lucide-react` (`import { Filter, UserCog, ... } from "lucide-react"`)
- **Toasts**: `sonner` (`import { toast } from "sonner"`)
- **Date formatting**: `date-fns` (`import { format } from "date-fns"`)
- **Data hooks**: `@/lib/datasource` (`useRecords`, `useRecordCreate`, `useRecordUpdate`, `q.select`)
- **User context**: `@/lib/user` (`useCurrentUser`)
- **Language**: modern TypeScript compiles (`?.`, `??`, arrows, generics — verified live 2026-08-25); named React imports only (`import { useState } from "react"`), no `import React from 'react'`
- **Field access**: `record.fields.aliasName`, never `record.aliasName`
- **Outer wrapper** (every block): `<div className="container py-0"><div className="content">...</div></div>` — or a deliberate full-bleed layout
- **Token vocabulary**: the `{colors.x}`/`{typography.x}`/`{rounded.x}` placeholders below use illustrative names; substitute the file's actual token names at generation time — dembrandt vocabulary is `{colors.primary}` / `{colors.on-surface}` / `{colors.surface}` / `{typography.headline-md}` / `{typography.label-lg}` / `{rounded.lg}` / `{rounded.full}`

Downstream `softr-vibe-coding` writes against this exact stack. The stubs below name shadcn components and lucide icons explicitly so the handoff is unambiguous.

## When to load this file

Always, in Step 4b, after the main DESIGN.md prose has been written. The skill substitutes `{colors.x}`, `{typography.x}`, `{rounded.x}`, etc. with the file's actual token names, then appends the rendered section to the prose body **after the `## Do's and Don'ts` section, before `## Known Gaps in This Extraction`** (per the v2 anatomy in [../design-md-format.md](../design-md-format.md)).

## Substitution rules

When the skill loads this file:

1. Replace `{colors.primary}` etc. with the file's actual token **names** from the `colors` block (keep the `{...}` reference form — downstream tools resolve it); inline a raw hex only when no matching role exists (rule 4).
2. Replace `{rounded.x}` etc. with the actual radius token names from the `rounded` block.
3. Replace `{typography.x}` etc. with the actual typography token names.
4. If a referenced token is missing from the extraction (e.g., the brand has no `surface-lavender-pale`), substitute the closest available token (e.g., the lightest extracted surface) and add a note in the stub: `# fallback: extracted has no surface-lavender-pale, using {colors.surface}`.
5. Mark each rendered stub with a `status: "scaffolded"` line inside its own body section, and set `extraction_status.app_patterns: scaffolded` in the frontmatter — **never add stub tokens to the frontmatter** (the base token blocks are dembrandt-owned and never restructured).
6. Keep the "Refine when you build:" hints intact — they're guidance for the team, not metadata.

## Section to append

Render and append the following section verbatim (after substitution) to the prose body of the DESIGN.md, after the `## Do's and Don'ts` section and before `## Known Gaps in This Extraction`. **The appendable region ends at the closing `---` after the last stub** — the `## Next step` and `## Evolving this file` sections further below are templates: the first is agent guidance for SKILL.md Step 8 (not appended), the second is the canonical text for the DESIGN.md's own appended `## Evolving this file` section (append it minus the future-skill note).

---

## Application Patterns (scaffolded)

The patterns below are common across most app UIs (modals, status pills, page headers, empty states, etc.) but do not appear on a typical marketing source. Each is **scaffolded** using the brand tokens captured above, so it ships with brand-aligned defaults out of the box. Refine each one when you build the screen that uses it — these are starting points, not finished specs.

### Block scaffolding (Softr — every block needs this)

Every Vibe Coding block must wrap its content in Softr's standard container/content pattern, otherwise the block won't size correctly inside the Softr page:

```jsx
<div className="container py-X">
  <div className="content">
    {/* block content goes here */}
  </div>
</div>
```

`container` and `content` are Softr-recognized class names that constrain the block to the app's max-width and apply page-level padding. The `py-X` value (`py-3` for detail-page blocks, `py-8` for admin-page blocks with a lavender wrapper) controls vertical breathing room. **No block ships without this wrapper.**

For admin pages (lists, dashboards), wrap the inner content in a soft brand-tinted container:

```jsx
<div className="container py-8">
  <div className="content">
    <div className="rounded-[30px] p-8" style={{ backgroundColor: LIGHT_BG }}>
      {/* admin content */}
    </div>
  </div>
</div>
```

Where `LIGHT_BG` is `{colors.surface-soft}` (or the lightest extracted brand surface).

**Refine when you build:** the first block.

### Page Header

Top-of-page lockup used at the top of every admin / detail page.

- Icon square: 44×44, `{rounded.md}` corners, `{colors.primary-deep}` background, white lucide icon at 20px.
- Title: h2, `{typography.display-md}` (or `text-2xl`), font-weight 600, color `{colors.ink}`.
- Description: `{typography.body-sm}`, color muted-foreground, `mt-1 ml-14` (aligned past the icon).

**Refine when you build:** the first screen that has a header.

### Dialogs / Modals

Four-part anatomy. The dialog content uses `p-0 overflow-hidden` so each part renders flush against the rounded corners.

- **dialog-content**: `{rounded.xl}` corners, `sm:max-w-lg` for standard, 90vw × 90vh for media lightboxes.
- **dialog-header**: white strip, 20px / 24px padding, 1px hairline border-bottom. 40×40 `{rounded.md}` icon square in `{colors.primary-deep}` with white lucide icon. DialogTitle (`text-lg font-semibold {colors.ink}`) and DialogDescription (`text-xs text-muted-foreground mt-0.5`). Use `DialogDescription`, not a plain `<p>`.
- **dialog-body**: `{colors.surface-soft}` (or lavender wash, whatever the lightest brand surface is), 16px / 24px padding. **Always** apply `max-h-[60vh] overflow-y-auto` so short viewports scroll the body instead of pushing the footer offscreen.
- **dialog-footer**: subtle gray-tint strip, 16px / 24px padding, 1px hairline border-top. `flex justify-end gap-2`.

**Refine when you build:** the first modal — most likely a confirmation dialog or an edit form.

### Alert Dialogs (confirmation flows)

Use shadcn `<AlertDialog>` (`@/components/ui/alert-dialog`) for actions that need explicit acknowledgement (delete, archive, irreversible operations).

- 40×40 `{rounded.md}` icon tile in the action's accent tint:
  - Destructive: `{colors.destructive}15` background + `{colors.destructive}` icon.
  - Positive: `{colors.primary}10` background + `{colors.primary}` icon.
- AlertDialogTitle (`text-base font-semibold {colors.ink}`), AlertDialogDescription (`text-sm text-muted-foreground`).
- Cancel = `{components.button-secondary}`. Confirm:
  - Destructive flow: `{colors.destructive}` background + white text.
  - Positive flow: `{components.button-primary}`.
- Cannot be dismissed by clicking outside.

**Refine when you build:** the first destructive action.

### Lightbox (media viewer)

Full-bleed modal for previewing documents, images, PDFs.

- DialogContent at `90vw × 90vh`, `{rounded.xl}` corners, `p-0 overflow-hidden flex flex-col`.
- Header: 12 / 20px padding, white. Filename left (`text-sm font-semibold truncate {colors.primary}`), page indicator right (`1 / N` `text-xs muted-foreground`, only when stack > 1).
- Body: `flex-1 bg-gray-50 overflow-hidden relative`. Image: `<img object-contain>` centered. PDF: `<iframe>`. Other: muted File icon + "Preview not available" + `{components.button-primary}` "Open in new tab".
- Navigation arrows (only when stack > 1): `h-10 w-10 rounded-full bg-white/95 shadow-md`, absolute `left-3` / `right-3` at vertical center.

**Refine when you build:** any document or attachment list.

### Pill family

Four pill variants with distinct roles. Don't conflate them.

- **status-badge** — Standalone, prominent. `{colors.primary-tint-soft}` background + `{colors.primary-pressed}` text. Used for "Active", "New", etc. on hero cards.
- **pill-status-inline** — In-card, compact (32px tall, content-sized). `{colors.primary-deep}10` background + `{colors.primary-deep}` text. Used inline for status select + arbitrator/owner tags. With shadcn SelectTrigger, pass `size="sm"`.
- **pill-summary-count** — Top-of-page counts ("12 Active", "3 Pending"). All pills in the group share the same `{colors.primary-deep}10` background — never per-status colors at the summary level. Internal hierarchy: count is `font-bold`, label is `font-medium opacity-80`. Total pill uses `{colors.primary-deep}15` (slightly more saturated) and a leading icon. Sort by count descending.
- **pill-email-avatar** — Email or person tag. `{colors.primary-deep}08` background + 24px solid-NAVY avatar circle on the left with first-letter initial in white. Replace shadcn `Badge variant="secondary"` (which renders washed-out and fails contrast).

**Refine when you build:** the first list view that shows status, the first dashboard with summary counts, or the first place an email or person is rendered.

### Section card (page-level)

White card with a tinted header strip — used for content sections inside detail pages.

- Outer: `bg-white {rounded.xl} border border-{colors.hairline}/100 shadow-sm`.
- Header strip: `#FAFBFC` background, 16px / 24px padding, 1px hairline border-bottom. 32×32 `{rounded.md}` icon square (accent-color-tinted bg + accent-color icon) + uppercase eyebrow label (`text-sm font-bold uppercase tracking-wider`, accent color).
- Body: 16px / 24px padding.
- Stack with `mb-6` (24px) between sections. **Do not** put `mb-6` on the last card in a block — the outer wrapper's bottom padding handles the buffer; doubling produces 32–40px gaps that look weirdly bigger than the within-block gaps.

**Refine when you build:** the first detail page with multiple content sections.

### Form field card

Wraps a label + input inside a modal whose body uses `{colors.surface-soft}` (lavender canvas).

- `bg-white {rounded.lg} border border-{colors.hairline}/200 p-4`.
- Multiple cards stack with `space-y-2`.
- Without this, bare inputs blend into the lavender body and lose hierarchy.

**Refine when you build:** the first form-in-modal (e.g., "Add X", "Edit X").

### Search input + filter row + reset button

Standard pattern for any list view that has filters.

- **Search input**: `flex-1` wrapper, `Search` lucide icon absolute-positioned `left-3.5 top-1/2 -translate-y-1/2`, Input class `pl-10 h-11 {rounded.pill} border-{colors.hairline}/200 bg-white shadow-sm`.
- **Filter selects**: shadcn `<Select>` (`@/components/ui/select`). **Never** native `<select>` — Softr's surfaces inherit OS chrome and the native control opens an OS-native dark context menu that breaks brand. 44px tall, `{rounded.pill}`, leading lucide icon (Filter, UserCog, etc.) at `left-3.5 z-10`, `pl-10 pr-3`. minWidth ~220px. Use `!h-11` (Tailwind important modifier) to override shadcn's default `data-[size=default]:h-9` rule.
- **Reset button**: always rendered, `disabled={!hasActiveFilters}` so the layout doesn't shift when filters become active. `{components.button-secondary}` styling + FilterX icon. Don't conditionally mount/unmount — that produces a layout shift the eye notices.

**Refine when you build:** the first list view with filters.

### Eyebrow labels

Two sizes for two roles:

- **Section eyebrow** (in section-card headers): `text-sm font-bold uppercase tracking-wider`, accent color.
- **In-card eyebrow** (above field values, inside subgroups): `text-xs font-semibold uppercase tracking-wider`, accent color.

The size signals hierarchy: bigger = section, smaller = subgroup within a section.

**Refine when you build:** the first section card with subgroups.

### Meta rows

Inline strip of labeled facts inside a card header (e.g., "Filed On · Status · Owner").

- `flex items-center gap-x-8 gap-y-3 flex-wrap`.
- Vertical hairline separators between items: `h-10 w-px bg-{colors.hairline}/100`, hidden on mobile (`hidden sm:block`).
- Each item: small uppercase label on top (`text-xs font-medium uppercase tracking-wider muted-foreground`), value below (`text-sm font-semibold {colors.ink}`).
- For status, prepend a small colored dot: `h-2.5 w-2.5 rounded-full` with the status's color.

**Refine when you build:** the first detail page that shows multiple meta facts.

### Empty / loading / error states

Every data-driven block must handle four states with consistent treatment:

- **Loading** — Skeleton shapes matching the final card layout. Use `{rounded.xl}` (the actual card radius) so the skeleton previews real geometry.
  - Page-header icon: `Skeleton h-10 w-10 {rounded.md}`.
  - Page-header title: `Skeleton h-8 w-48`.
  - Search/filter: `Skeleton h-10 ... rounded-full` (`flex-1` for search, `w-32` for filter).
  - List card: `Skeleton h-28 w-full {rounded.xl}`.
  - Section card: `Skeleton h-48 w-full {rounded.xl}`.
- **Error** — Centered icon + heading + retry button. Icon: 64×64 rounded-full with `{colors.destructive}15` background + `{colors.destructive}` icon (AlertCircle). Heading uses `{colors.ink}`. Retry is `{components.button-primary}`.
- **Empty (no data)** — Centered icon + heading + description. Icon: 80×80 `{rounded.lg}` with `linear-gradient(135deg, {colors.primary} → {colors.primary-pressed})` background + white icon. The gradient is reserved for empty-state icons; don't use it elsewhere.
- **Empty after filtering** — Same shape as empty, but heading reads "No matching X" and description suggests adjusting filters. Always include a Reset / Clear Filters action.

**Refine when you build:** the first data-driven block.

### Document / file icons

Color the file icon by type when rendering a list of attachments:

- PDF: `{colors.accent-amber}` (or whatever amber/yellow the brand has) + `{...}15` background.
- Image (jpg/png/gif/webp): `{colors.primary}` + `{colors.primary}10` background.
- Generic file: `{colors.primary-deep}` + `{colors.primary-deep}10` background.

NEVER use destructive red/orange for PDFs. Most brands have no separate red, and PDF is not an error state.

**Refine when you build:** the first list of file attachments.

### Interaction states

Consistent transitions and state feedback:

- **Hover (cards)**: `hover:shadow-md hover:border-{colors.hairline}/200 transition-all duration-200`.
- **Hover (filled buttons)**: `hover:opacity-90`.
- **Hover (outline buttons)**: `hover:shadow-sm`.
- **Focus (text inputs)**: `focus:border-{colors.primary}/50 focus:ring-{colors.primary}/50`.
- **Disabled**: `disabled:opacity-50 disabled:cursor-not-allowed`.
- **Loading button**: every async-action button must enter this state during the mutation:
  - Add `disabled` until resolved.
  - Show `Loader2` with `animate-spin` (`h-4 w-4`) before the label.
  - Swap label to gerund: "Save" → "Saving…", "Submit" → "Submitting…".
  - Restore on success/error.

**Refine when you build:** the first interactive button or form. Most apps will share these defaults.

### Toast / notifications

Use `sonner` (`import { toast } from "sonner"`) for mutation feedback:

- `toast.success("X updated")` after successful mutations.
- `toast.error("Couldn't update X", { description: err.message })` on failure.
- Always call `refetch()` before showing the success toast so the UI reflects the new state when the toast appears.

**Refine when you build:** the first mutation flow.

---

## Next step — `softr-vibe-coding`

This DESIGN.md is now ready for the second skill in the workflow. To build the actual Softr blocks:

1. Paste `./custom-code-header.html` into your Softr app: **Settings → Custom Code → Code inside header**, save and publish.
2. Run the `softr-vibe-coding` skill with a request like "build me a Softr block for X" or "create a claims dashboard." It auto-reads `./DESIGN.md`, applies the brand tokens, and uses the `Application Patterns` scaffolds above as starting points.

`softr-vibe-coding` writes against the exact tech stack documented in this file's `tech_stack` frontmatter — shadcn/ui at `@/components/ui/<name>`, Tailwind classes inline, lucide-react icons, sonner toasts, date-fns formatting, and the `useRecords` / `useRecordCreate` / `useRecordUpdate` hooks from `@/lib/datasource`. There's no translation step between this file and that skill.

## Evolving this file

Every section above is marked `status: "scaffolded"`. As the team builds real Softr blocks:

1. The first time you build a block that uses a scaffolded pattern, refine the scaffold to match what you actually shipped (real values, not the brand-token defaults).
2. Promote the refined token from `status: "scaffolded"` to `status: "extracted"` (it's now extracted from your real implementation, not from the marketing source — but extracted nonetheless).
3. Update the prose section to reflect the real pattern, not the scaffold.

When you've built 2–3 blocks and refined the relevant scaffolds, the file will have shifted from "brand foundation + scaffolds" to "real working Softr design system." That's the goal.

A future companion skill (`building-design-md-refine`) will automate steps 1–3 by reading the block JSX in your project and proposing refinements. Until that exists, the team does it manually.
