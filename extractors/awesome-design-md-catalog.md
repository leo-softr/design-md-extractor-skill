# Path B — awesome-design-md Catalog

Voltagent's [awesome-design-md](https://github.com/VoltAgent/awesome-design-md) repository hosts ~71 ready-made `DESIGN.md` files for popular brands: Airbnb, Stripe, Linear, Notion, Vercel, Tesla, Spotify, Apple, BMW, Cursor, Figma, Mistral, Replicate, Stripe, and many more. Each file is also published at `https://getdesign.md/<brand>/design-md`.

If the user's target brand is in the catalog, this is the **fastest path** — no extraction needed, no Apify costs, no MCPs required. The skill fetches the file, validates it against the schema, and saves it locally.

## When to use Path B

The skill should suggest Path B before Path A when:

- The user names a well-known brand: "Make a DESIGN.md like Airbnb" / "Use Stripe's design system."
- The user pastes a URL that matches a catalog entry domain: `https://airbnb.com`, `https://stripe.com`, etc.
- The user explicitly asks for a "pre-built" or "ready-made" brand kit.

Always check the catalog first — a hit saves 90 seconds and $0.08 vs. running the full Apify pipeline.

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
3. **Validate against the schema** at [../design-md-format.md](../design-md-format.md). Required fields: `version`, `name`, `description`, `colors`, `typography`. Required sections: Overview, Colors, Typography.
4. **Add `extraction_status`.** Catalog files don't always include this field. Add one with all sections marked `extracted` (since the catalog is curated).
5. **Save.** Write to `./DESIGN.md` in the project folder.
6. **Skip the verification prompt.** Catalog files are pre-validated; no `needs-verification` flags should fire.
7. **Generate `custom-code-header.html`** as in Step 7 of the main `SKILL.md` workflow.

## Edge cases

| Edge case | Handling |
|---|---|
| Brand not in catalog | Tell the user, suggest Path A (URL extraction) instead. |
| Brand has multiple sub-brands (e.g. BMW vs. BMW M) | Ask which one. Each sub-brand has its own catalog file. |
| User wants to customize the catalog file | Save the catalog file as-is, then ask if they want to override specific tokens. Update those in place — mark them `extracted-by-user`. |
| Catalog file is older than the brand's current site | Note this in the `Known Gaps` section: "Catalog file dated `<date>`. May not reflect recent brand updates." |
| The `getdesign.md/<slug>/design-md` URL 404s | Fall back to the GitHub raw URL. If both fail, treat as "not in catalog" and suggest Path A. |

## What this path does NOT do

- Does not run Apify or Exa. No MCP calls.
- Does not modify the brand description or voice section. Catalog content is taken at face value.
- Does not generate `custom-code-header.html` from scratch — it derives the snippet from the catalog file's frontmatter (Step 7 of `SKILL.md`).

## Documentation

- Catalog README: <https://github.com/VoltAgent/awesome-design-md/blob/main/README.md>
- Design system index: <https://getdesign.md>
- Format spec: <https://stitch.withgoogle.com/docs/design-md/format>
