# Path B — awesome-design-md Catalog

Voltagent's [awesome-design-md](https://github.com/VoltAgent/awesome-design-md) repository hosts ~71 ready-made `DESIGN.md` files for popular brands: Airbnb, Stripe, Linear, Notion, Vercel, Tesla, Spotify, Apple, BMW, Cursor, Figma, Mistral, Replicate, and many more. Each file is also published at `https://getdesign.md/<brand>/design-md`.

If the user's target brand is in the catalog, this is the **fastest path** — no extraction needed, no browser crawl, no MCPs required. The skill fetches the file, validates it, and runs it through the same assembly steps as every other path: the catalog content becomes the base layer (untouched), and the skill's banner and augmentation layers are appended around it (`SKILL.md` Steps 4–4b).

## When to use Path B

The skill should suggest Path B before Path A when:

- The user names a well-known brand: "Make a DESIGN.md like Airbnb" / "Use Stripe's design system."
- The user pastes a URL that matches a catalog entry domain: `https://airbnb.com`, `https://stripe.com`, etc.
- The user explicitly asks for a "pre-built" or "ready-made" brand kit.

Always check the catalog first — a hit skips the multi-page browser extraction entirely (a couple of minutes saved, and no browser install needed).

## How to fetch from the catalog

Each entry is published at a predictable URL. Two formats:

```
https://getdesign.md/<brand>/design-md
https://github.com/VoltAgent/awesome-design-md/blob/main/<brand>/DESIGN.md
```

Where `<brand>` is the lowercase, hyphenated brand name (e.g. `airbnb`, `stripe`, `the-verge`, `bmw-m`).

Fetch the file with whatever HTTP-fetching tool the client has available. The raw content is markdown.

## Catalog index (as of last check)

To suggest Path B accurately, the skill should know what's in the catalog. The categories below are stable; the brand list grows over time.

| Category | Sample brands |
|---|---|
| AI & LLM Platforms | Claude, Cohere, ElevenLabs, Mistral AI, Ollama, Replicate, RunwayML, Together AI, VoltAgent, xAI |
| Developer Tools & IDEs | Cursor, Expo, Lovable, Raycast, Superhuman, Vercel, Warp |
| Backend, Database & DevOps | ClickHouse, Composio, HashiCorp, MongoDB, PostHog, Sanity, Sentry, Supabase |
| Productivity & SaaS | Cal.com, Intercom, Linear, Mintlify, Notion, Resend, Zapier |
| Design & Creative Tools | Airtable, Clay, Figma, Framer, Miro, Webflow |
| Fintech & Crypto | Binance, Coinbase, Kraken, Mastercard, Revolut, Stripe, Wise |
| E-commerce & Retail | Airbnb, Meta, Nike, Shopify, Starbucks |
| Media & Consumer Tech | Apple, IBM, NVIDIA, Pinterest, PlayStation, SpaceX, Spotify, The Verge, Uber, Vodafone, WIRED |
| Automotive | BMW, BMW M, Bugatti, Ferrari, Lamborghini, Renault, Tesla |

For the authoritative current list, fetch <https://github.com/VoltAgent/awesome-design-md/blob/main/README.md>.

## Workflow

1. **Resolve the brand.** Take the user's input (URL or brand name) and resolve to a catalog slug.
   - URL like `https://airbnb.com` → slug `airbnb`.
   - Name like "Linear" → slug `linear-app` (some brands have qualified slugs — check the catalog README).
   - Don't guess slugs you can't verify. If unsure, fetch the catalog README first to find the slug.
2. **Fetch the file.** Try `https://getdesign.md/<slug>/design-md` first. If 404, try the GitHub raw URL.
3. **Validate the fetched file** against [../design-md-format.md](../design-md-format.md), using the same pragmatic rule as its "Validating an uploaded DESIGN.md" section. Required frontmatter: `name`, `description`, `colors`, `typography` — reject only if one is missing or `colors` values aren't valid CSS colors. Body: at least one prose section; list (don't silently fill) anything the v2 anatomy has that the catalog file lacks. `version` is **not** required on the fetched file — the skill adds `version: 2` itself during augmentation.
4. **Accept the catalog content as the base layer.** Catalog files follow the same Google DESIGN.md format as dembrandt output, so the frontmatter keys generally map one-to-one already; where naming differs, map them into the dembrandt-shaped keys without reshaping the values. The catalog's token content, description, and voice are taken at face value — the skill's layers are appended **around** the catalog content, never rewriting it.
5. **Hand off to `SKILL.md` Steps 4 + 4b.** Assemble `./DESIGN.md` exactly as the main workflow specifies: the preamble banner at the very top (required on every generated DESIGN.md, catalog path included — `<source>` is the catalog entry, e.g. "awesome-design-md catalog entry `stripe`"), then the catalog base, then the appended frontmatter keys (`version: 2`, `target_platform`, `downstream_skill`, `tech_stack`, `fonts`, `extraction_status`) and the appended body sections — including the Application Patterns scaffold from Step 4b, which is never skipped (catalog files don't carry app patterns either).
6. **Set `extraction_status`** (`SKILL.md` Step 5). Use `tokens: extracted (catalog: <entry>)` and `app_patterns: scaffolded`; set the other keys as applicable — e.g. `fonts: extracted (catalog)` when the file names real font families, `voice_and_mood: extracted (catalog)` when its description/overview carries usable brand voice.
7. **Skip the verification prompt** — this is Path-B-specific. Catalog files are curated and pre-validated, so no `needs-verification` flags should fire on the catalog content itself. (If augmentation surfaces a genuine gap — e.g. no resolvable fonts — flag it normally and let `SKILL.md` Step 6 run.)
8. **Generate `custom-code-header.html`** as in Step 7 of the main `SKILL.md` workflow.

## Edge cases

| Edge case | Handling |
|---|---|
| Brand not in catalog | Tell the user, suggest Path A (URL extraction) instead. |
| Brand has multiple sub-brands (e.g. BMW vs. BMW M) | Ask which one. Each sub-brand has its own catalog file. |
| User wants to customize the catalog file | Assemble the augmented `DESIGN.md` first (catalog base untouched), then ask which tokens to override. Update the values in place — no per-token status fields in v2 — and note the overrides in `extraction_status` (e.g. `tokens: extracted (catalog: stripe; primary overridden by user)`). |
| Catalog file is older than the brand's current site | Note this in the `Known Gaps` section: "Catalog file dated `<date>`. May not reflect recent brand updates." |
| The `getdesign.md/<slug>/design-md` URL 404s | Fall back to the GitHub raw URL. If both fail, treat as "not in catalog" and suggest Path A. |

## What this path does NOT do

- Does not run dembrandt. No extraction, no MCP calls.
- Does not modify the brand description or voice section. Catalog content is taken at face value — the augmentation layers are appended around it, never into it.
- Does not skip `SKILL.md` Steps 4/4b. Only extraction (Step 3's crawl) and the verification prompt (Step 6) are bypassed; the banner, augmentation frontmatter, and Application Patterns scaffold are added like on every other path.
- Does not generate `custom-code-header.html` from scratch — it derives the snippet from the assembled DESIGN.md's frontmatter (catalog tokens plus the augmented `fonts` block; Step 7 of `SKILL.md`).

## Documentation

- Catalog README: <https://github.com/VoltAgent/awesome-design-md/blob/main/README.md>
- Design system index: <https://getdesign.md>
- Format spec: <https://stitch.withgoogle.com/docs/design-md/format>
