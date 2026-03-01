---
name: "ai-specs: Enrich US"
description: Enrich a user story from Jira, Notion, or manual text. Deterministic scope + design metadata. Design structure is resolved at apply-time (Live Design Mode).
category: Command
tags: [ai-specs, enrich, user-story, mcp]
---

Enrich a User Story so it is implementation-ready and aligned with project standards.

Input ($ARGUMENTS) can be:
- A Jira issue key or URL
- A Notion page URL or page id
- Raw markdown/text
- A Base User Story block

---

## Steps

### 0) Validate input

If empty or unclear, ask for:
- Jira issue key/url
- Notion page url/id
- Raw user story text

---

### 1) Identify input type

Classify as:
- Jira
- Notion
- Base User Story
- Manual

If ambiguous, ask one clarifying question.

---

### 2) Retrieve source content

#### Jira
- Fetch title, description, acceptance criteria.
- Normalize into Base User Story block.

#### Notion
- Fetch page title and requirement-related blocks.
- If the page belongs to a database, attempt to read these boolean properties:
  - Backend (BE)
  - Frontend (FE)
  - Design Linked
- Extract any Figma URLs containing `node-id=`.
- Normalize into Base User Story block.

#### Manual
- Convert into Base User Story format.

---

### 3) Ensure Base User Story format

# Base User Story
Source: <Jira|Notion|Manual>
Reference: <url|key|N/A>

## Title
## Problem / Context
## Desired Outcome
## Acceptance Criteria (raw)
## Constraints / Notes

---

### 4) Deterministic Scope & Design Detection

Initialize:

scope:
  backend: false
  frontend: false

design-linked: false

#### 4.1 Scope from Notion properties (preferred)

If Notion boolean properties exist:

- scope.backend = value of "Backend (BE)"
- scope.frontend = value of "Frontend (FE)"
- design-linked = value of "Design Linked"

Checkbox values ALWAYS take priority over content inference.

#### 4.2 Fallback (only if properties missing)

If Backend/Frontend properties are missing:

- Ask the user:
  "Does this feature include backend, frontend, or both?"

If Design Linked property is missing:

- If Figma URLs with `node-id=` are found OR a `## Design References` section exists:
  - design-linked = true

If both property and content exist and contradict:
- Property value wins.

---

### 5) Produce Enriched User Story (STRICT FORMAT)

You MUST output the following header exactly in this structure:

# Enriched User Story

design-linked: <true|false>

scope:
  backend: <true|false>
  frontend: <true|false>

source: <Jira|Notion|Manual>
reference: <url|key|N/A>

---


Additional STRICTNESS rules:

- Output booleans MUST be exactly `true` or `false` (lowercase).
- The scope block MUST be exactly:
  scope:
    backend: <true|false>
    frontend: <true|false>
- The metadata block MUST appear near the top, immediately after `# Enriched User Story`.
- NEVER inline metadata (e.g., `design-linked: true | source: Notion | scope: FE + BE`).
- NEVER collapse scope into text.
- If the story language is Spanish, content can be Spanish, but the metadata block MUST remain in this strict format.

Then include:


## Context
## Goals / Non-goals
## Backend Requirements (if scope.backend == true)
## Frontend Requirements (if scope.frontend == true)
## Data / Entities (if applicable)
## API / Interfaces (if applicable)
## Acceptance Criteria
## Non-Functional Requirements
## Assumptions
## Open Questions
## Definition of Done

Rules:

- NEVER inline metadata like: `design-linked: true | scope: FE + BE`
- NEVER collapse scope into text.
- ALWAYS output the YAML-style block exactly as shown.
- If scope.backend == false, omit Backend Requirements section.
- If scope.frontend == false, omit Frontend Requirements section.
- Tests are governed by project standards (do not use scope booleans for tests).

---

### 6) Design References (Light Mode, Deterministic)

If design-linked == true:

Append EXACTLY:

## Design References

Figma File:
<main figma file url if available>

Referenced Nodes:
- <full node-id URL 1>
- <full node-id URL 2>

Rules:

- ALWAYS include full URLs containing `node-id=`
- DO NOT summarize nodes as "aside (1:3)" without the URL
- DO NOT extract layout structure here
- Layout will be resolved at `/opsx:apply` time via Figma MCP

---

### 7) Source Update Behavior

#### Jira
Ask:
"Update the Jira ticket with the enhanced version? (yes/no)"
Default: yes

If updating:
- Append enhanced content.
- Move status from "To refine" → "Pending refinement validation" (if possible).

#### Notion
Ask:
"Update the Notion page with the enhanced version? (yes/no)"
Default: yes

If updating:
- Append enhanced content.
- If Status property exists, move to "Pending refinement validation" (safe mode).

Never fail if status transition not possible.

---

---

### 8) Output Validation & Auto-Repair (MANDATORY)

Before returning your final answer, validate that your output satisfies ALL of the following:

1) Contains the strict metadata block (Step 5) with:
   - `design-linked: true|false` on its own line
   - `scope:` YAML block with `backend:` and `frontend:` booleans
   - `source:` and `reference:`
2) Does NOT contain inline metadata patterns like:
   - `design-linked: true | source: ... | scope: ...`
3) If `design-linked: true`, contains a `## Design References` section with:
   - `Figma File:` URL (if available)
   - `Referenced Nodes:` list
   - Each node MUST be a FULL URL containing `node-id=...`

If ANY validation fails:
- You MUST re-write the Enriched User Story so it passes validation.
- You MUST NOT ask the user for permission to reformat.
- You MAY ask the user for missing inputs ONLY if required:
  - If `design-linked: true` but you have zero node-id URLs anywhere in source, ask for at least one Figma node URL with `node-id=...`.
  - Otherwise, repair formatting using only existing information.


### 9) Output

Always return:

1) Base User Story
2) Enriched User Story
3) Summary line:
   - Source: ...
   - Scope: backend=<true|false>, frontend=<true|false>
   - Design linked: true/false
   - Updated: yes/no/N/A
   - Status changed: yes/no/N/A
