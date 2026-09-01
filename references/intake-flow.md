# Intake Flow — Gate Question and Path Selection

This file expands on Steps 0–2 in `SKILL.md`. Load when the user's intent is unclear and the gate or path question needs disambiguation.

## The shape of the intake

```
Step 0 — Existing DESIGN.md? ─────────────► load it / replace it / update flags
                │
                ▼ (if no existing file)
Step 1 — Apply a custom design? (yes/no) ──► no  → write minimal DESIGN.md, exit
                │ yes
                ▼
Step 2 — Which path? (A/B/C/D)
   A. URL extraction       → load extractors/dembrandt-pipeline.md
   B. Catalog              → load extractors/awesome-design-md-catalog.md
   C. Upload               → load design-md-format.md (validate)
   D. Guided Q&A           → load extractors/guided-qa.md
```

## Phrasing the gate question

The skill should always ask the gate question the same way (SKILL.md holds the canonical short form; this expanded phrasing is equivalent):

> "Do you want to apply a custom design to this project? (yes / no)
>
> Saying **no** is a real first-class choice — many projects use the downstream tool's default style. I won't pressure you into customization."

Variations to accept as "no":

- "no" / "nope" / "skip"
- "default" / "use defaults"
- "Softr default" / "platform default"
- silence / no response within reasonable timeout

Variations to accept as "yes":

- "yes" / "yeah" / "sure"
- a URL pasted alone (treat as "yes, Path A" — go straight to Step 2A)
- a brand name from the awesome-design-md catalog (treat as "yes, Path B")
- a `.md` file upload (treat as "yes, Path C")

If the user pastes a URL during the gate question, do not ask Step 2 separately — they've signaled both intent and path. Confirm: "Got it — running URL extraction on `<url>`. Starting now." Then proceed.

## Phrasing the path question

If the user said "yes" but didn't indicate a path, ask Step 2 (SKILL.md holds the canonical short form; this expanded phrasing is equivalent):

> "How would you like to provide the design?
>
> **A.** Extract from a website URL with dembrandt (recommended for real brands you have access to).
> **B.** Use a pre-built file from the awesome-design-md catalog (fastest if your brand is in the catalog — Airbnb, Stripe, Linear, Notion, Tesla, ~70 others).
> **C.** Upload an existing DESIGN.md file.
> **D.** Guided Q&A — I'll ask 8 questions about your brand."

Map user responses:

| User said | Path |
|---|---|
| "URL" / "extract from <url>" / pastes a URL | A |
| "catalog" / "Stripe-like" / pastes a `getdesign.md` URL | B |
| "I have a file" / "upload" / drops a `.md` file | C |
| "Q&A" / "I don't have a site yet" / "I don't have anything" | D |
| "I don't know" / unclear | Recommend B if a brand was named, otherwise A |

## Skipping ahead

If the user's first message contains enough signal, you can skip questions:

- "Extract Airbnb's brand" → go to Path B with slug `airbnb`. Skip both gate and path questions.
- "Build a brand kit for `https://example.com`" → go to Path A with that URL.
- "I want a design system from scratch — playful, blue accent" → go to Path D, skip gate question.

When skipping, briefly confirm what you're doing:

> "Running [Path X] on [target]. This will take ~1 minute per crawled page for Path A (3–5 minutes for the recommended 3–5-page crawl), ~5 seconds for Path B/C, ~2 minutes for Path D. Reply 'cancel' if you want to choose differently."

## Disambiguation traps

A few user inputs are ambiguous and need clarification:

| User input | Possible interpretations | Resolution |
|---|---|---|
| "Use the same brand as my last project" | Path C (load existing DESIGN.md from another folder) or Step 0 (use the DESIGN.md already in this folder) | Check the current project folder first. If empty, ask "which project's DESIGN.md should I copy?" |
| "Make me a brand like Apple's" | Path B (Apple is in the catalog) or Path A (extract from apple.com fresh) | Default to Path B since the catalog is faster. Note Apple is in the catalog and offer Path A as alternative if user wants the latest. |
| "My brand is just blue and white" | Path D with minimal answers, or Path A on the user's current site if they have one | Ask: "Do you have a website to extract from, or should we run the 8-question Q&A?" |

## Output of the intake step

After Steps 0–2, the skill should have:

- A confirmed gate answer (yes/no).
- If yes, a confirmed path letter (A/B/C/D).
- The input artifact for that path (URL / catalog slug / file path / first Q&A answer).

That's enough to proceed to Step 3 in `SKILL.md`. Do not start extraction or Q&A before all three are confirmed.
