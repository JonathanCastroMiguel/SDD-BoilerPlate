---
category: Command
description: Enrich a user story, preserve Figma node URLs verbatim, and
  persist a canonical snapshot for deterministic handoff.
name: "ai-specs: Enrich US"
tags:
- ai-specs
- enrich
- user-story
- snapshot
- figma
---

# ai-specs:enrich-us (Enterprise Snapshot Mode v3)

## Core Contract (MANDATORY)

Your output MAY include the Base User Story for traceability. However,
the canonical enriched spec MUST be deterministic, machine-parseable,
and must preserve design URLs.

### Canonical section markers

You MUST wrap the canonical enriched content with the exact markers:

```{=html}
<!-- BEGIN_ENRICHED_USER_STORY -->
```
# Enriched User Story

... (strict metadata + content) ...
`<!-- END_ENRICHED_USER_STORY -->`{=html}

Rules: - `# Enriched User Story` MUST be an H1 immediately after BEGIN
marker. - Do NOT use brackets in headings. - The metadata block MUST be
valid YAML with 2-space indentation.

### Strict metadata block (MANDATORY)

Immediately after the H1, output this block with valid YAML:

design-linked: \<true\|false\> scope: backend: \<true\|false\> frontend:
\<true\|false\> source: \<Notion\|Jira\|Manual\> reference:
`<url-or-id>`{=html}

Booleans MUST be lowercase `true|false`.

## Design URL Preservation (MANDATORY)

If `design-linked: true` and the Base User Story contains Figma node
URLs (any URLs containing `node-id=`), then the canonical enriched spec
MUST include a `## Design References` section with:

Figma File: `<figma file url>`{=html}

Referenced Nodes: - \<FULL figma node url 1 containing node-id=...\> -
\<FULL figma node url 2 containing node-id=...\> ...

IMPORTANT: - You MUST copy the node URLs verbatim from the Base User
Story (do NOT replace with "Node 1:3" text). - You MUST NOT omit,
shorten, or "pretty print" URLs. - You MAY additionally include a
human-friendly "Figma Reference" section (Node 1:3 --- Sidebar, etc.),
but the canonical URLs section is required.

If `design-linked: true` but there are NO `node-id=` URLs in the Base
User Story, ask the user for at least one node URL.

## 1️⃣ Persist Canonical Snapshot (MANDATORY)

After producing the canonical enriched section:

1.  Generate slug from the feature title (kebab-case).
2.  Generate timestamp: YYYYMMDD-HHMM.
3.  Save ONLY the canonical enriched content (the text between BEGIN/END
    markers, inclusive) to:

drafts/enriched/`<slug>`{=html}-`<timestamp>`{=html}.md

No Base User Story. No formatting transformations.

## 2️⃣ Notion Update (CANONICAL ONLY, FAST)

When updating Notion, do NOT render a readable version.

Instead, append only:

1)  A heading: ENRICHED (CANONICAL --- DO NOT EDIT)

2)  A single CODE BLOCK whose contents are EXACTLY the canonical
    enriched content between BEGIN/END markers (verbatim). This must
    match the saved draft file 1:1.

You MUST NOT: - Rephrase - Summarize - Remove URLs - Modify indentation

## 3️⃣ Console Output

After saving the draft, print:

Saved enriched draft:
drafts/enriched/`<slug>`{=html}-`<timestamp>`{=html}.md

Next step for developer: Run /ai-specs:handoff-us
`<slug>`{=html}-`<timestamp>`{=html}
