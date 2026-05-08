# Confidence Flags — How to Mark Token Status

Every token in `DESIGN.md` carries a confidence flag. The flag tells downstream consumers (humans, LLMs, other skills) how much to trust the value. There are three flags. Always set one — never leave it blank.

## The three flags

### `extracted`

The value was pulled directly from the source with high confidence.

**Use when:**

- Color came from `automation-lab/css-color-extractor` with usage_count > 0.
- Font name came from `apify/web-scraper` Tier 2b computed-style pass.
- Voice paragraph synthesized from Exa content where hero copy was directly quoted.
- Token came from a user-uploaded `DESIGN.md` (Path C) and validated against schema.
- Tech stack came from `misterkhan/website-tech-stack-scanner` and matched a known fingerprint.

**Trust level:** High. Downstream consumers should use these as-is without prompting the user.

### `inferred`

The value was derived from indirect signals, not from the source directly. May be wrong.

**Use when:**

- Typography hierarchy sizes (display-xl, display-lg, etc.) — derived from typical Astro/Webflow patterns when CSS extractors didn't return them directly.
- Border radius scale — when the dominant radius value came from class counts (`rounded-md` used 80x → infer `md: 12px`) but specific values weren't computed.
- Shadow recipe — when shadow tier was inferred from "modern marketing site" patterns.
- Spacing scale — when defaulted to Tailwind's 4px grid because no explicit scale was extracted.
- Brand mood description — when inferred from voice synthesis without explicit brand language about mood.

**Trust level:** Medium. The verification prompt (Step 6 in `SKILL.md`) should ask the user to confirm or override.

### `needs-verification`

The value could not be captured. The skill must prompt the user before locking the DESIGN.md.

**Use when:**

- Specific font names couldn't be extracted (e.g. Tier 2b not run because `apify/web-scraper` approval missing, or page returned no font info).
- Logo URL not captured.
- Component-level recipes (button-primary, card, etc.) not directly extracted — only generic patterns inferred.
- A user-uploaded `DESIGN.md` (Path C) is missing required fields.
- A specific `extraction_status` section returned nothing useful.

**Trust level:** None until verified. Downstream consumers should treat the value as a placeholder and prompt the user before using it.

## Section-level flags

`extraction_status` rolls up token flags into section-level summaries:

```yaml
extraction_status:
  colors: complete                              # all colors extracted
  framework: complete                           # tech stack scanner returned matches
  voice_and_mood: complete                      # Exa returned content + voice synthesized
  typography_specific_fonts: complete           # Tier 2b ran successfully
  spacing_radius_shadows: needs-verification    # Tier 1+2 didn't capture, defaults inferred
```

| Section value | Meaning |
|---|---|
| `complete` | Every token in this section is `extracted`. |
| `partial` | Some tokens are `extracted`, others are `inferred` or `needs-verification`. List the gaps in `Known Gaps`. |
| `inferred` | All tokens are `inferred` — defaults applied; user should confirm. |
| `needs-verification` | All tokens flagged; section is essentially empty. |

Do not use `complete` if any token in the section is below `extracted`.

## Setting the right flag — examples

**Color extracted directly from CSS:**
```yaml
primary: "#e82d42"  # extracted from CSS — usage_count: 4, properties: background-color, border-color, color
```

**Font extracted via computed styles:**
```yaml
font-display-name: "PPNeueMontreal"  # extracted via Tier 2b getComputedStyle on h1
```

**Typography hierarchy with mixed confidence:**
```yaml
display-xl:
  fontFamily: "PPNeueMontreal, Arial, sans-serif"  # extracted
  fontSize: 48px                                    # inferred — typical for Astro h1
  fontWeight: 600                                   # inferred from Tailwind class counts
  lineHeight: 1.1                                   # inferred
```

In a case like this, mark the parent token `inferred` because the size/weight/leading are guesses. The font family alone being extracted doesn't lift the whole entry.

**Token completely missing:**
```yaml
spacing:
  status: "needs-verification"
  base: 16px      # default — verify
  lg: 24px        # default — verify
  section: 64px   # default — verify
```

Always set `status: needs-verification` at the top of the parent block when the entire scale is filler.

## What to do when flags fire

After Step 4 (synthesis) in `SKILL.md`, count the flags:

- **0 flags below `extracted`** → skip to Step 7 (custom-code-header). No prompt needed.
- **Any `inferred` flags** → optional verification prompt. Phrase as "I made these assumptions — confirm or override".
- **Any `needs-verification` flags** → mandatory verification prompt. Phrase as "I couldn't extract these — paste from your brand guide or say 'skip'".

The verification prompt should be **one consolidated message**, not a series of questions. Group flagged tokens by section. Always offer "skip" as an option — some users will accept the placeholder and resolve later.

Example:

> "I extracted colors and voice fully. Three things I couldn't fully confirm:
>
> - **Specific serif font name** (currently Inter as fallback, status: needs-verification). Tier 2b extraction was skipped because the actor wasn't approved. Want me to retry, or paste the font name from your brand guide?
> - **Border radius scale** (defaulted to 8px / 14px / 20px, status: inferred). Confirm or paste your scale.
> - **Logo URL** (status: needs-verification). Paste the URL or say 'skip'.
>
> Reply with corrections, or say 'looks good' to lock these in as-is."

After the user replies, edit `DESIGN.md` in place. Update both the token values *and* the flags. A token the user confirms moves from `inferred` to `extracted-by-user`. A token the user provides a new value for moves to `extracted-by-user` with the new value. A token the user says "skip" on stays at its current flag.

## The fourth (implicit) flag — `extracted-by-user`

When the user confirms or overrides a value during Step 6, mark it `extracted-by-user`. This distinguishes between "automated extraction succeeded" and "human filled in the gap" without lying about the source. Downstream consumers can treat both equally as high-confidence.

```yaml
font-display-name: "Söhne"  # extracted-by-user (verified by user during Step 6 prompt)
```
