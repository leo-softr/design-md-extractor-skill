# Exa MCP — Install + Usage

Exa is the recommended companion to Apify for capturing **voice and copy register**. Exa returns clean markdown content from any URL — it strips HTML, CSS, and tags, so it cannot extract design tokens, but it is the cleanest way to capture the *language* of a brand.

In the pipeline, Exa runs in Tier 1.4 alongside the Apify actors and feeds Tier 1.5 (voice synthesis).

## Install

### Step 1 — Create an Exa account

Sign up at <https://exa.ai>. The free tier covers ~1,000 queries per month, well above what this skill needs.

### Step 2 — Generate an API key

In the Exa dashboard:

1. Click your profile menu.
2. **API Keys**.
3. **Create new key**.
4. Copy the key.

### Step 3 — Add the MCP config

```json
{
  "mcpServers": {
    "exa": {
      "url": "https://mcp.exa.ai/mcp",
      "headers": { "x-api-key": "YOUR_EXA_API_KEY" }
    }
  }
}
```

### Step 4 — Restart your client

The MCP loads on startup.

### Step 5 — Confirm install

Ask the LLM: "Use Exa to fetch <https://example.com>." If clean markdown comes back, you're connected.

## Usage in the pipeline

Call once with the target URL:

```
mcp__exa__web_fetch_exa({
  "urls": ["https://target-site.com"],
  "maxCharacters": 5000
})
```

You can also fetch multiple pages in one call (e.g. homepage + about + services) by passing multiple URLs in the `urls` array.

The response is plain markdown. Read it for:

- **Hero copy** — the first 2–4 sentences of brand-direct messaging. Quote 1–2 of these in the DESIGN.md `Voice & Copy Register` section.
- **CTA pattern** — the action verbs the brand uses (e.g. "Schedule a free discovery session" vs. "Get started" vs. "Book a demo").
- **Pain-point framing** — how the brand describes the customer's problem. This signals the register: empathetic / technical / playful / direct.
- **Trust signals** — testimonials, logos, "5-star" claims, named clients.

Synthesize a 2–3 sentence paragraph capturing the register. Examples:

- *"Confident but friendly, slightly playful, customer-success-driven, never corporate-stiff. Uses real customer phrasing in pain-point quotes. CTAs are direct ('Schedule a free discovery session')."*
- *"Editorial and restrained. Sentence-case headlines, generous whitespace, photography over typography. CTAs are quiet — 'Reserve' or 'Continue', never 'Get started now'."*

Mark `voice_and_mood: extracted` in the DESIGN.md `extraction_status` if Exa returned readable content. Mark `inferred` if you had to guess the register from incomplete text. Mark `needs-verification` if Exa returned no content.

## Limitations to know

- **Exa returns no CSS, no fonts, no colors.** It is a content extractor, not a token extractor. Do not use it to fill any of the `colors`, `typography`, `rounded`, `spacing`, or `elevation` fields.
- **Exa's character cap defaults to 3000 per page.** Pass `maxCharacters: 5000` (or higher) if you need more context. Cap at 10000 to avoid context pollution.
- **Exa strips `<head>` content.** No `<link>` tags, no `<meta>` tags, no Google Fonts URLs. Use Apify Tier 2b for those.

## When Exa fails

If Exa returns an empty response or an error:

1. Skip Tier 1.4 silently — voice synthesis is optional.
2. Mark `voice_and_mood: needs-verification` in `extraction_status`.
3. The Step 6 verification prompt will offer the user a chance to paste hero copy manually.

Do not retry Exa — its failures are usually persistent (rate limit, blocked URL).

## Documentation

- Exa MCP: <https://docs.exa.ai/reference/mcp>
- Exa API: <https://docs.exa.ai>
