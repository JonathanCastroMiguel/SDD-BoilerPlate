---
category: Command
description: Enrich a user story and persist a canonical snapshot for
  deterministic handoff.
name: "ai-specs: Enrich US"
tags:
- ai-specs
- enrich
- user-story
- snapshot
---

# ai-specs:enrich-us (Enterprise Snapshot Mode)

After generating the Enriched User Story:

## 1️⃣ Persist Canonical Snapshot (MANDATORY)

You MUST:

1.  Generate a slug from the feature title (kebab-case).
2.  Generate timestamp: YYYYMMDD-HHMM.
3.  Save ONLY the canonical enriched block (between BEGIN/END markers,
    or the strict enriched section) to:

drafts/enriched/`<slug>`{=html}-`<timestamp>`{=html}.md

The file MUST contain:

# Enriched User Story

design-linked: \<true\|false\> scope: backend: \<true\|false\> frontend:
\<true\|false\> source: `<source>`{=html} reference:
`<reference>`{=html}

...rest of enriched content...

No Base User Story. No formatting transformations.

## 2️⃣ Notion Update (Readable + Canonical)

When updating Notion:

A)  Append the readable enriched section (formatted for humans).

B)  Immediately below it, append:

Heading: ENRICHED (CANONICAL --- DO NOT EDIT)

Then insert a CODE BLOCK containing EXACTLY the canonical enriched
snapshot text (verbatim, no rewriting).

You MUST NOT: - Rephrase - Summarize - Remove URLs - Modify indentation

The code block must match the saved draft file 1:1.

## 3️⃣ Console Output

After saving the draft, print:

Saved enriched draft:
drafts/enriched/`<slug>`{=html}-`<timestamp>`{=html}.md

Next step for developer: Run /ai-specs:handoff-us
`<slug>`{=html}-`<timestamp>`{=html}
