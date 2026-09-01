# Path A — URL extraction with dembrandt

Path A drives [dembrandt](https://github.com/dembrandt/dembrandt) (MIT, npm package `dembrandt`, verified against v0.30.0): a real-browser design-token extractor. It renders the page in Chromium and reads **computed styles from the live DOM** — which is the only honest source. Class names are arbitrary labels, not semantic descriptions (observed in the wild: a site using a `font-serif` utility class 209 times whose rendered typography was 100% sans — only the computed style told the truth). Never lock typography or color conclusions from class names, framework defaults, or CSS source text alone.

Only extract sites the user owns or has permission to analyze — for contracted client work, the client's own website qualifies. Respect robots.txt and the site's ToS. Canvas/WebGL-rendered sites cannot be analyzed (no DOM to read).

## Setup

**MCP server (preferred):**

```bash
claude mcp add --transport stdio dembrandt -- npx -y --package dembrandt@latest dembrandt-mcp
npx -y dembrandt@latest install-browser   # one-time — the MCP server does NOT auto-install the browser
```

The `@latest` tag makes each launch resolve the newest release (bare `npx` reuses its cache). A server added mid-session connects on the **next** session; the CLI below is the same-session path. Requires Node.js 18+.

**CLI fallback (no MCP):**

```bash
npx -y dembrandt@latest install-browser              # once (CLI self-heals a missing browser since 0.30.0)
npx -y dembrandt@latest <url> --json-only --crawl 5  # extraction JSON to stdout
npx -y dembrandt@latest <url> --design-md --crawl 5  # base document → output/<domain>/DESIGN.md (cwd-relative — copy it)
```

**When dembrandt is not available at all** (MCP not connected, npx blocked): stop — do not improvise an extraction. Give the user the exact setup commands above as a numbered list ending with "Reply 'done' and I'll retry." Retry once after they confirm. If it still fails, offer Paths B / C / D instead and flag `extraction_status.tokens: needs-verification`.

## The extraction

1. **`get_design_tokens({ url, pages: 5 })`** → `{ job_id, status: "queued" }`. Always crawl multiple pages (`pages: 3`–`5`, max 20) — merged multi-page extraction is markedly stronger, and single-page noise is real (a live 1-page run produced 46 near-duplicate typography tokens and a white-on-white button sample). `paths: ["/pricing", "/about"]` names pages explicitly; `sitemap: true` discovers them (alone it takes up to 20 pages). Other options: `slow` (JS-heavy SPAs), `mobile`, `darkMode`/`wcag` (these two exist only on `get_design_tokens`/`get_color_palette`), `cookie`/`header` (authenticated staging), `noSandbox` (Docker/CI).
2. **Poll `get_job_status(job_id)`** until `completed` (~60s/page). The completed response embeds the **full extraction JSON** — keep it: Steps 3b and 4 mine keys the rendered document omits or truncates (`logo`, `logoInstances`, `favicons`, `gradients`, `shadows`, `motion`, `frameworks`, `iconSystem`, `breakpoints`). Don't re-send the payload to other tools — they all take `job_id`. Completed jobs live 1 hour.
   - **If the job fails**: timeout → retry once with `slow: true`; bot protection / Cloudflare → CLI with `--browser=firefox` (`npx -y dembrandt@latest install-browser firefox` first); `Browser launch failed` → run install-browser, retry.
3. **`get_findings(job_id)`** — contrast/consistency lint of the extraction. Feed anything severe into `Known Gaps` and the Step 6 verification prompt.
4. **`generate_design_md(job_id)`** — returns the base DESIGN.md **as text; it writes no file**. This verbatim output is the base layer of the file (see [design-md-format.md](../design-md-format.md)) — the skill's augmentation happens around it, never inside it (one exception: the `description` mood-append).

**Honesty rule:** if extraction fails end-to-end after the remediations above, stop the pipeline and tell the user plainly. Do not synthesize a DESIGN.md from imagination — suggest Path D (guided Q&A) instead. For a *single* missing token where the rest extracted fine, sensible defaults are acceptable **only with a flag**: 8px radius, white canvas, near-black text `#1a1a1a`, Inter, one-tier shadow — each marked in `extraction_status` / `Known Gaps` as `needs-verification`, resolved in the Step 6 prompt.

## Resolving real font names (the computed-fontFamily trap)

Dembrandt's `typography.*.fontFamily` is a **computed value** — when a site loads a custom font, the computed stack can surface a generic fallback (`ui-sans-serif`) while the real brand font hides in the body's **Font URLs** list (direct `.woff2` links). Resolve the actual family names from those URLs (filenames usually name the face, e.g. `Nohemi-SemiBold.woff2`) and the live site; record them in the `fonts` frontmatter block. If the font is paid/licensed, pick a substitute via [google-fonts-substitutes.md](../references/google-fonts-substitutes.md).

When a font name cannot be confirmed, hedge — never claim "the brand is X-dominant" from weak signals. Record the weak signal, mark `extraction_status.fonts: needs-verification`, and ask in the Step 6 prompt: "I couldn't confirm the exact font name — paste it from your brand guide, or confirm my guess." When extraction *catches* a misleading signal (a lying class name, a fallback-masked font), write the catch into the file's Typography prose so downstream consumers stay honest.

## Voice & copy register (not part of dembrandt)

The extractions this pipeline runs contain no voice data — the MCP server exposes no voice option, and the CLI's undocumented opt-in `--voice` flag is not used here (verified live + source, v0.30.0). Capture voice yourself by reading the live site's key pages (homepage, pricing, about) directly:

- **Hero copy**: quote the first 1–2 brand-direct sentences.
- **CTA pattern**: the action verbs on buttons ("Get started" vs "Book a demo" vs "Reserve").
- **Pain-point framing**: direct / empathetic / technical / playful.
- **Trust signals**: logos, numbers, testimonials, certifications.

Synthesize a 2–3 sentence register paragraph for the `Voice & Copy Register` section. Calibration examples of the expected output:

> *"Confident but friendly. Short declarative sentences, direct second person, no jargon. CTAs are invitations, not commands — 'Schedule a free discovery session', never 'BUY NOW'."*

> *"Editorial and restrained. Long-form sentences, third person, occasional French loanwords. CTAs are quiet — 'Reserve' or 'Continue'."*

Status semantics: `extracted` when the register is built from readable page copy; `inferred` when guessed from incomplete text; `needs-verification` when nothing usable was fetched. Voice is optional — on failure, skip silently with the flag set and let the Step 6 prompt offer a manual hero-copy paste.

## Logo & assets (from the extraction JSON)

The logo comes from the extraction's `logo` / `logoInstances` / `favicons` keys — source URL, dimensions, `type` (`wordmark` / `logomark` / `combination`), safe zone — no separate scrape needed. Prefer `wordmark`; treat `logomark` as the icon mark, never the primary asset. Apply SKILL.md Step 3b's selection heuristics (SVG → PNG-transparent → WebP → JPG; wordmark over icon), download with `curl -sSL -o <path> <url>` into `./assets/` (`mkdir -p` first), and record both `public_url` and `local_backup` per the schema in [design-md-format.md](../design-md-format.md#assets-path-a-only).

Failure modes:

| Symptom | Do |
|---|---|
| No usable logo in the extraction JSON | Check `logoInstances` and `favicons` before giving up; a CSS `background-image` logo may only appear there |
| `curl` returns 403 / zero bytes | Retry once with `-A "Mozilla/5.0"` and a `Referer: <site url>` header; if still blocked, ask the user to download it manually |
| Multiple plausible logos | Save the **wordmark** only (largest image whose alt/filename carries the brand name) — never auto-save the icon mark as the primary asset |
| Logo is a sprite sheet | Don't crop programmatically — record the URL with `status: needs-verification` and surface it to the user |

Hand everything — base document, extraction JSON, findings, resolved fonts, voice paragraph, assets — to SKILL.md Step 4 for assembly.
