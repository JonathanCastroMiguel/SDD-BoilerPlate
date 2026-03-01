---
category: Command
description: Create OpenSpecs change from a previously saved canonical
  enriched snapshot and archive it.
name: "ai-specs: Handoff US"
tags:
- ai-specs
- handoff
- opsx
- archive
---

# ai-specs:handoff-us (With Auto-Archive)

Input: snapshot name (without extension)

Example: /ai-specs:handoff-us companies-20260301-1405

## Steps

1️⃣ Load snapshot from:

drafts/enriched/`<snapshot>`{=html}.md

2️⃣ Pass its FULL content verbatim to:

opsx:new

Rules: - Do NOT re-enrich. - Do NOT read from Notion. - Do NOT modify
the snapshot. - Use the file content exactly as-is.

3️⃣ If `opsx:new` completes successfully:

-   Create directory if missing: drafts/enriched/\_archived/

-   Move the snapshot file to:

    drafts/enriched/\_archived/`<snapshot>`{=html}.md

4️⃣ Console Output

After successful archive, print:

Snapshot archived to: drafts/enriched/\_archived/`<snapshot>`{=html}.md

If `opsx:new` fails: - Do NOT move the file. - Leave it in
drafts/enriched/ for retry.

This guarantees: - Deterministic artifact generation - Clean active
drafts directory - Full traceability of past enrichments
