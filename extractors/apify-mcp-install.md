# Apify MCP — Install Walkthrough

Required for Path A (URL extraction). One-time setup; takes about 60 seconds.

## Step 1 — Create an Apify account

Sign up at <https://apify.com>. The free tier is sufficient for this skill — typical extractions cost ~$0.08 on free tier.

## Step 2 — Generate an API token

In the Apify Console:

1. Click your profile menu (top right).
2. Settings → Integrations → API tokens.
3. Click **Create new token**.
4. Copy the token. It starts with `apify_api_`.

**Never commit the token to git.** Keep it in your MCP client config only.

## Step 3 — Add the MCP config to your client

Pick the flavor that matches how you've been adding MCPs.

### Hosted MCP (recommended)

Single-line install — uses Apify's hosted HTTP MCP endpoint:

```json
{
  "mcpServers": {
    "apify": {
      "url": "https://mcp.apify.com",
      "headers": { "Authorization": "Bearer YOUR_APIFY_TOKEN" }
    }
  }
}
```

Replace `YOUR_APIFY_TOKEN` with the token from Step 2.

### Local stdio MCP (alternative)

Runs the MCP server locally on your machine via npx:

```json
{
  "mcpServers": {
    "apify": {
      "command": "npx",
      "args": [
        "-y", "@apify/actors-mcp-server",
        "--actors",
        "automation-lab/css-color-extractor,automation-lab/css-stats-analyzer,misterkhan/website-tech-stack-scanner,apify/website-content-crawler,apify/web-scraper"
      ],
      "env": { "APIFY_TOKEN": "YOUR_APIFY_TOKEN" }
    }
  }
}
```

The `--actors` flag whitelists which actors the MCP exposes. The five listed cover the full pipeline. Without the flag, the MCP exposes Apify's entire 10,000+ actor catalog and clutters the tool list.

## Step 4 — Restart your client

Close and reopen Claude Code / Cursor / Cowork / whichever client you use. The new MCP server loads on startup.

## Step 5 — Confirm install

Ask the LLM: "Search Apify actors for 'css color'." If a list comes back, you're connected. Alternatively, call:

```
mcp__Apify__search-actors({"keywords": "css color"})
```

If the call succeeds and returns actors like `automation-lab/css-color-extractor`, the install worked.

## Common install issues

| Symptom | Cause | Fix |
|---|---|---|
| `"Apify MCP server not found"` | Config not picked up | Restart the client. If still failing, check the config file path. |
| `"Authentication failed"` or 401 errors | Wrong token, expired token, or wrong header format | Regenerate token in Apify Console. Confirm `Authorization: Bearer <token>` for hosted, or `APIFY_TOKEN` env var for local. |
| `"Memory limit exceeded"` on actor calls | Default request exceeds free tier | Pass `callOptions: { memory: 1024 }` on each call. |
| `"This Actor requires full access to your account"` on `apify/web-scraper` | One-time actor approval needed | Follow [apify-actor-approval.md](apify-actor-approval.md). |

## Apify documentation

- Apify MCP overview: <https://docs.apify.com/platform/integrations/mcp>
- Hosted MCP endpoint: <https://mcp.apify.com>
- API tokens: <https://console.apify.com/settings/integrations>
