---
category: Command
description: Create OpenSpecs change from a previously saved canonical
  enriched snapshot.
name: "ai-specs: Handoff US"
tags:
- ai-specs
- handoff
- opsx
---

# ai-specs:handoff-us

Input: snapshot name (without extension)

Example: /ai-specs:handoff-us companies-20260301-1405

## Steps

1.  Load file from: drafts/enriched/`<snapshot>`{=html}.md

2.  Pass its FULL content verbatim to:

opsx:new

3.  Do NOT re-enrich.
4.  Do NOT read from Notion.
5.  Do NOT modify the snapshot.

This guarantees deterministic artifact generation from the original
enriched specification.
