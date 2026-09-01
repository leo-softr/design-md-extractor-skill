# Building Design MD Skill

A Claude Skill for extracting a brand design system into a portable `DESIGN.md` file. Since v2.0.0 the extraction engine is **[dembrandt](https://github.com/dembrandt/dembrandt)** (MIT) — a real-browser design-token extractor — and this skill layers on everything raw extraction can't provide: voice & copy register, resolved font names, logo assets, Softr app-pattern scaffolds, and an app-level custom-code snippet.

## What this skill does

Captures a target brand's design tokens (colors, typography, spacing, radii, shadows, components) plus voice and assets into a single `DESIGN.md` file in your project folder. The base of the file is dembrandt's own DESIGN.md output (Google's DESIGN.md draft format), kept verbatim; the skill's augmentation layers make it Softr-ready.

The skill supports four input paths:

| Path | When to use |
|---|---|
| **A. URL extraction (dembrandt)** | You have a target website. dembrandt renders it in a real browser and reads computed styles — colors, typography, spacing, components — across a multi-page crawl. |
| **B. awesome-design-md catalog** | Your target brand is one of the ~71 already documented at <https://github.com/VoltAgent/awesome-design-md>. Fastest path. |
| **C. Manual upload** | You already have a `DESIGN.md` (from another tool, hand-written, or generated previously). The skill validates, accepts, and augments it. |
| **D. Guided Q&A** | You have a brand in your head but nothing written. The skill asks 8 questions and synthesizes a minimal `DESIGN.md`. |

The skill always produces two files (plus `./assets/` on Path A):

- `./DESIGN.md` — the brand artifact
- `./custom-code-header.html` — a snippet to paste into Softr's `Settings → Custom Code → Code inside header` for app-wide brand inheritance

## Install

### Claude Code — one-line install with auto-updates (recommended)

```bash
npx building-design-md@latest init
```

This installs the skill into `~/.claude/skills/building-design-md/` and adds a `SessionStart` hook to `~/.claude/settings.json` so the skill auto-updates to the latest published version on every Claude Code session. No manual `git pull` needed.

Requires Node.js 18+.

### Other agents (manual install)

Drop the `building-design-md/` folder into your skills directory:

| Agent | Path |
|---|---|
| Claude Code | `~/.claude/skills/` or `./.claude/skills/` |
| Cursor | `~/.cursor/skills/` |
| GitHub Copilot | `~/.copilot/skills/` or `./.github/skills/` |
| Gemini CLI | `~/.gemini/skills/` |

Get the folder via `git clone https://github.com/leo-softr/design-md-extractor-skill.git` or by downloading the repo ZIP.

Restart your client. Verify by typing `/building-design-md` (or asking the AI "extract brand from a URL"). The skill should appear in the slash-command menu or trigger automatically on relevant requests.

Alternatively, on Claude.ai (web), upload the bundled `.zip` directly:
1. Open Claude.ai → Settings → Customize → Skills.
2. Click `+` → `+ Create skill` (or `+ Upload skill`, depending on your version).
3. Select the zip file. Claude validates the structure and installs.

## Prerequisites

For full functionality (Path A), dembrandt — no accounts, no API keys:

```bash
claude mcp add --transport stdio dembrandt -- npx -y --package dembrandt@latest dembrandt-mcp
npx -y dembrandt@latest install-browser   # one-time: fetches the Chromium dembrandt drives
```

The `@latest` tag makes every server launch resolve the newest dembrandt release. No MCP? The skill falls back to the dembrandt CLI (`npx -y dembrandt@latest <url> --design-md --crawl 5`) — same engine, same-session. Setup details and failure handling: [extractors/dembrandt-pipeline.md](extractors/dembrandt-pipeline.md).

Paths B, C, and D have no prerequisites.

## Usage

Three common invocation patterns:

**Auto-trigger (recommended):**
```
> Extract the brand from https://example.com and save it as DESIGN.md.
```

**Slash command:**
```
> /building-design-md
> [skill prompts for path and source]
```

**Iteration on existing DESIGN.md:**
```
> Update the DESIGN.md in this project — the font name was wrong.
```

The skill always ends with a written `DESIGN.md` in the project folder and a prompt to run any downstream skill (e.g. a Softr block generator) once you're happy with the brand.

## File structure

```
building-design-md/
├── SKILL.md                          # Main skill prompt (entry point for the LLM)
├── README.md                         # This file
├── design-md-format.md               # The DESIGN.md schema: dembrandt base + Softr augmentation layers
├── extractors/                       # Path-specific tooling
│   ├── dembrandt-pipeline.md         # Path A — dembrandt setup, extraction, fonts, voice, logo, failures
│   ├── awesome-design-md-catalog.md  # Path B — pre-built catalog
│   └── guided-qa.md                  # Path D — 8-question Q&A
├── references/                       # General references
│   ├── intake-flow.md                # Gate question + path selection
│   ├── app-patterns-stubs.md         # Step 4b Application Patterns scaffold (Softr stack)
│   ├── confidence-flags.md           # extracted / inferred / needs-verification / scaffolded / partial
│   ├── google-fonts-substitutes.md   # Fallback table for licensed brand fonts
│   └── custom-code-header.md         # Generating the Softr Custom Code snippet
└── examples/
    └── northwind-studio-DESIGN.md    # Fictional reference example (v2 format)
```

## What this skill is *not*

- **Not** a UI generator. This skill never produces JSX, React components, or HTML pages.
- **Not** a website cloner. It captures tokens, not pixel-perfect replicas — and only from sites you own or have permission to analyze.
- **Not** a substitute for human design judgment. Confidence flags exist because extraction is imperfect — review them.

## Companion skill — `softr-vibe-coding`

This skill is the **brand-foundation half** of the pipeline:

```
New client → building-design-md (dembrandt → DESIGN.md + layers) → softr-vibe-coding (DESIGN.md → blocks) → shipped Softr app
```

The intended downstream is the [`softr-vibe-coding`](https://github.com/leo-softr/Softr-Vibe-Coding-Block-Claude-Skill) skill, which reads the `DESIGN.md` produced here and generates brand-aligned Softr Vibe Coding blocks (custom TSX/JSX components) without re-asking about colors, fonts, or component patterns. Its own Step 1 can also generate a quick *raw* dembrandt DESIGN.md mid-build — run this skill instead when the project deserves the full foundation (voice, assets, scaffolds, custom-code snippet).

**Install both for the full workflow** (one-line each):

```bash
npx building-design-md@latest init
npx softr-vibe-coding@latest init
```

Both auto-update on every Claude Code session.

Other consumers (Webflow generators, hand-rolled React, Figma plugins) can read the same `DESIGN.md` — the base layer follows the format from <https://stitch.withgoogle.com/docs/design-md> / <https://github.com/VoltAgent/awesome-design-md>, and the Softr-specific additions are clearly separated (see [design-md-format.md](design-md-format.md)).

## License

MIT.
