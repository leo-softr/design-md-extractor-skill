# Building Design MD Skill

A Claude Skill for extracting a brand design system into a portable `DESIGN.md` file. The output is the canonical artifact any downstream skill (Softr Vibe Coding block generator, Webflow generator, custom React skill, human designer) can consume.

## What this skill does

Captures a target brand's design tokens (colors, typography, spacing, shadows, voice, framework signals) into a single `DESIGN.md` file in your project folder. The file is the canonical artifact: any downstream tool (a Softr Vibe Coding block generator, a Webflow generator, a custom React skill, a human designer) can consume it.

The skill supports four input paths:

| Path | When to use |
|---|---|
| **A. URL extraction** | You have a target website. The skill runs an Apify + Exa pipeline to pull colors, fonts, framework, and voice. |
| **B. awesome-design-md catalog** | Your target brand is one of the ~71 already documented at <https://github.com/VoltAgent/awesome-design-md>. Fastest path. |
| **C. Manual upload** | You already have a `DESIGN.md` (from another tool, hand-written, or generated previously). The skill validates and accepts it. |
| **D. Guided Q&A** | You have a brand in your head but nothing written. The skill asks 8 questions and synthesizes a minimal `DESIGN.md`. |

The skill always produces two files:

- `./DESIGN.md` — the brand artifact
- `./custom-code-header.html` — a snippet to paste into Softr's `Settings → Custom Code → Code inside header` for app-wide brand inheritance

## Install

Drop the `building-design-md/` folder into your skills directory:

| Agent | Path |
|---|---|
| Claude Code | `~/.claude/skills/` or `./.claude/skills/` |
| Cursor | `~/.cursor/skills/` |
| GitHub Copilot | `~/.copilot/skills/` or `./.github/skills/` |
| Gemini CLI | `~/.gemini/skills/` |

Restart your client. Verify by typing `/building-design-md` (or asking the AI "extract brand from a URL"). The skill should appear in the slash-command menu or trigger automatically on relevant requests.

Alternatively, on Claude.ai (web), upload the bundled `.zip` directly:
1. Open Claude.ai → Settings → Customize → Skills.
2. Click `+` → `+ Create skill` (or `+ Upload skill`, depending on your version).
3. Select the zip file. Claude validates the structure and installs.

## Prerequisites

For full functionality (Path A):

1. **Apify MCP** connected to your client. Free tier works. See [extractors/apify-mcp-install.md](extractors/apify-mcp-install.md).
2. **Exa MCP** connected (optional but recommended for voice/mood capture). See [extractors/exa-mcp.md](extractors/exa-mcp.md).
3. **One-time approval** of `apify/web-scraper` in the Apify Console. See [extractors/apify-actor-approval.md](extractors/apify-actor-approval.md). The skill will prompt you when the approval is needed.

Paths B, C, and D have no MCP prerequisites.

## Usage

Three common invocation patterns:

**Auto-trigger (recommended):**
```
> Extract the brand from https://airbnb.com and save it as DESIGN.md.
```

**Slash command:**
```
> /brand-extraction
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
├── design-md-format.md               # The DESIGN.md schema and field inventory
├── extractors/                       # Path-specific tooling
│   ├── apify-pipeline.md             # Path A — the tiered Apify+Exa pipeline
│   ├── apify-mcp-install.md          # One-time Apify MCP setup
│   ├── apify-actor-approval.md       # One-time apify/web-scraper approval
│   ├── exa-mcp.md                    # Exa MCP install + usage
│   ├── awesome-design-md-catalog.md  # Path B — pre-built catalog
│   └── guided-qa.md                  # Path D — 8-question Q&A
├── references/                       # General references
│   ├── intake-flow.md                # Gate question + path selection
│   ├── confidence-flags.md           # extracted / inferred / needs-verification
│   ├── tailwind-class-trap.md        # Why class-count analysis lies on Tailwind sites
│   ├── google-fonts-substitutes.md   # Fallback table for licensed brand fonts
│   └── custom-code-header.md         # Generating the Softr Custom Code snippet
└── examples/
    └── northwind-studio-DESIGN.md        # A real working extraction
```

## What this skill is *not*

- **Not** a UI generator. This skill never produces JSX, React components, or HTML pages.
- **Not** a website cloner. It captures tokens, not pixel-perfect replicas.
- **Not** a substitute for human design judgment. Confidence flags exist because extractors are imperfect — review them.

## Companion skills

This skill is built to feed downstream tools. The intended companion is the `softr-vibe-coding` skill, which reads the `DESIGN.md` produced here and generates Softr Vibe Coding blocks that match the brand. Other consumers (Webflow generators, hand-rolled React, Figma plugins) can read the same `DESIGN.md` if they understand the format from <https://stitch.withgoogle.com/docs/design-md> or <https://github.com/VoltAgent/awesome-design-md>.

## License

MIT.
