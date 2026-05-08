# Apify `web-scraper` — One-Time Approval

The `apify/web-scraper` actor (used in Tier 2b for computed-style extraction) requires a one-time human approval before it can run on a free-tier Apify account. The same applies to several other browser-rendering actors (`apify/puppeteer-scraper`, `apify/playwright-scraper`).

This is normal. The approval persists across all future MCP calls — once you do it, you never see this error again.

## When this approval is needed

The skill detects this failure mode automatically. The error message is:

```
This Actor requires full access to your account.
You must approve its permissions before running it.
```

The error appears on the first call to any actor in this category. Other actors used by this skill (`automation-lab/css-color-extractor`, `automation-lab/css-stats-analyzer`, `misterkhan/website-tech-stack-scanner`, `apify/website-content-crawler`) do not require this step.

## Approval workflow

1. Open <https://console.apify.com> in a browser.
2. Sign in with the same account whose API token is configured in your MCP.
3. Navigate to the actor — for example: <https://console.apify.com/actors/apify~web-scraper>.
4. Click **Try for free** or **Run** (the button labels vary).
5. Apify presents a permission dialog — usually "Allow this Actor to run on your account."
6. Click **Allow**.
7. Optionally, click **Save & Start** to test-run the actor once with default inputs (this confirms the approval took effect).

The whole process takes about 30 seconds.

## Retry after approval

Return to your MCP client and retry the failing call. It should now succeed. The skill should detect the success and continue the pipeline normally.

If it still fails with the same error, wait 60 seconds (Apify's permission state has a small propagation delay) and retry once more. If it fails after that, the approval did not save — repeat steps 3–6 above.

## Why this exists

Apify's permission model treats browser-rendering actors as higher-risk because they execute arbitrary JavaScript in a real browser. Free-tier accounts must explicitly approve them. Paid plans grant blanket approval. From the user's perspective, this is a one-time click — but a click that has to happen in the web console, not via MCP.

## Skill behavior when this error fires

When this error is seen:

1. Stop the pipeline. Do not retry automatically more than once.
2. Surface this exact instruction to the user:

   > "I need to run `apify/web-scraper` for high-confidence font extraction, but it needs a one-time approval. Please:
   >
   > 1. Open <https://console.apify.com/actors/apify~web-scraper>
   > 2. Click 'Try for free' or 'Run'
   > 3. Click 'Allow' on the permission dialog
   > 4. Reply 'done' here and I'll retry."

3. Wait for the user's confirmation.
4. Retry the call once. If it succeeds, continue. If it fails again, fall back to Tier 3 with `needs-verification` flags on typography.
