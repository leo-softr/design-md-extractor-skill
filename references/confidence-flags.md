# Confidence Flags — How to Mark Extraction Status

Every section in `DESIGN.md`'s `extraction_status` block carries a confidence flag. The flag tells downstream consumers (humans, LLMs, other skills) how much to trust the values. Always set one per section — never leave one blank.

**Where flags live in v2:** at the **section level** in `extraction_status`, plus on skill-owned blocks (`fonts`, `assets`, `tech_stack`, Application Patterns stubs). The dembrandt base tokens (`colors`, `typography`, `spacing`, `rounded`, `components`) never carry per-token `status` fields — they are observed values by definition, and restructuring them would break the base shape downstream tools parse. Specific doubts about individual base tokens go in `Known Gaps` prose instead.

## The flags

### `extracted`

The value was observed directly from the source with high confidence.

**Use when:**

- Tokens came from a dembrandt extraction (they are computed-style observations by definition — the whole `tokens` section is `extracted` whenever Path A completed).
- Font names were resolved from the extraction's Font URLs (real webfont files), not guessed.
- The voice paragraph quotes hero/CTA copy read directly from the live site.
- The logo came from the extraction JSON's `logo` key and downloaded successfully.
- Tokens came from a user-uploaded `DESIGN.md` (Path C) and validated against the schema.

**Trust level:** High. Downstream consumers should use these as-is without prompting the user.

### `inferred`

The value was derived from indirect signals, not observed directly. May be wrong.

**Use when:**

- Fonts were guessed from a vibe answer in guided Q&A ("clean and modern" → Inter) rather than observed.
- The brand-mood sentences in `description` were synthesized without explicit brand language about mood.
- The register was guessed from incomplete page text (voice three-way rule: `extracted` from readable copy / `inferred` from fragments / `needs-verification` from nothing).

**Trust level:** Medium. The verification prompt (Step 6 in `SKILL.md`) should ask the user to confirm or override.

### `needs-verification`

The value could not be captured. The skill must prompt the user before locking the DESIGN.md.

**Use when:**

- A font name couldn't be confirmed (computed `fontFamily` was a generic fallback and the Font URLs didn't resolve it either).
- Logo URL not captured (nothing plausible in `logo`/`logoInstances`/`favicons`, or the download stayed blocked).
- The extraction failed outright and the user declined Path D.
- A missing single token was filled with the standard fallback defaults (8px radius, white canvas, near-black `#1a1a1a` text, Inter, one-tier shadow — see [../extractors/dembrandt-pipeline.md](../extractors/dembrandt-pipeline.md)): defaults ship only with this flag, resolved in the Step 6 prompt.
- A user-uploaded `DESIGN.md` (Path C) is missing required fields.
- dembrandt's `get_findings` flagged something severe the user hasn't resolved (e.g. a WCAG contrast failure on the primary).

**Trust level:** None until verified. Downstream consumers should treat the value as a placeholder and prompt the user before using it.

### `scaffolded`

Application Patterns stubs generated in Step 4b using brand-token defaults. Not present in the source, not yet refined by the team — intentional placeholders that say "starting point, revisit when building." `scaffolded` sections do **not** trigger the Step 6 verification prompt.

## Section-level rollup

```yaml
extraction_status:
  tokens: extracted (dembrandt v0.30.0, 5-page crawl of example.com)
  fonts: extracted (resolved from Font URLs)
  voice_and_mood: extracted (hero + pricing copy read from the live site)
  assets: needs-verification (no plausible logo in the extraction — ask user)
  app_patterns: scaffolded
```

| Section value | Meaning |
|---|---|
| `extracted` | Everything in this section was observed directly. |
| `partial` | Some observed, some inferred or missing. Itemize the gaps in `Known Gaps`. |
| `inferred` | Defaults or guesses applied; user should confirm. |
| `needs-verification` | Essentially empty — mandatory prompt. |
| `scaffolded` | Intentional stubs (Application Patterns only). |

Annotate the flag with a short parenthetical saying *how* the value was obtained — future readers (and the Step 6 prompt) rely on it.

## What to do when flags fire

After Step 4 (assembly) in `SKILL.md`, count the flags:

- **Nothing below `extracted`** (ignoring `scaffolded`) → skip to Step 7 (custom-code-header). No prompt needed.
- **Any `inferred` or `partial`** → optional verification prompt. Phrase as "I made these assumptions — confirm or override".
- **Any `needs-verification`** → mandatory verification prompt. Phrase as "I couldn't capture these — paste from your brand guide or say 'skip'".

The verification prompt should be **one consolidated message**, not a series of questions. Group flagged items by section. Always offer "skip" — some users will accept the placeholder and resolve later.

Example:

> "I extracted tokens and voice fully. Three things I couldn't fully confirm:
>
> - **Display font name** (computed styles showed only `ui-sans-serif`; the webfont files suggest 'Nohemi' — status: needs-verification). Confirm, or paste the name from your brand guide.
> - **Radius scale** (site showed almost no rounded corners; recorded only `sm: 2px` — status: partial). Confirm that's intentional brand flatness.
> - **Logo URL** (status: needs-verification). Paste the URL or say 'skip'.
>
> Reply with corrections, or say 'looks good' to lock these in as-is."

After the user replies, edit `DESIGN.md` in place. Update both the values *and* the flags. An item the user confirms or supplies moves to `extracted-by-user`. An item the user says "skip" on stays at its current flag.

## The implicit flag — `extracted-by-user`

When the user confirms or overrides a value during Step 6, mark it `extracted-by-user`. This distinguishes "automated extraction succeeded" from "human filled the gap" without lying about the source. Downstream consumers treat both as high-confidence.

```yaml
fonts:
  display-name: "Söhne"   # extracted-by-user (verified by user during Step 6 prompt)
```
