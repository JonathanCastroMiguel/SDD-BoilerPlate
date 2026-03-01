---
name: "ai-specs: Enrich US"
description: Enrich a user story from Jira, Notion, or manual text. Uses Notion scope properties (BE/FE/Design Linked) when available. Design structure is resolved at apply-time (Live Design Mode).
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

### 4) Scope & Design Detection

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

Checkbox values always take priority over content inference.

#### 4.2 Fallback inference (only if properties missing)

If Backend/Frontend properties are missing:

- Ask the user:
  "Does this feature include backend, frontend, or both?"

If Design Linked property is missing:

- If a section titled "## Design References" exists OR any Figma URL is present:
  - design-linked = true

If both property and content exist and contradict:
- Property value wins.

---

### 5) Enrich the User Story

Produce:

# Enriched User Story

design-linked: <true|false>

scope:
  backend: <true|false>
  frontend: <true|false>

source: <Jira|Notion|Manual>
reference: <url|key|N/A>

## Context
## Goals / Non-goals
## Functional Requirements
## Acceptance Criteria
## Data / Entities (if applicable)
## API / Interfaces (if applicable)
## Implementation Notes
## Testing Plan
## Non-Functional Requirements
## Assumptions
## Open Questions
## Definition of Done

Rules:
- Be specific and implementation-ready.
- Do not invent tech stack details.
- Follow project standards where available.
- Tests are NOT controlled by scope booleans — they are governed by project standards.

---

### 6) Design References (Light Mode)

If design-linked is true:

Append:

## Design References

Figma File:
<url if available>

Referenced Nodes:
- <node-url-1>
- <node-url-2>

Note:
Design structure and layout will be retrieved live during the implementation phase (opsx:apply) via Figma MCP. No structural assumptions are frozen at enrichment time.

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

### 8) Output

Always return:

1) Base User Story
2) Enriched User Story
3) Summary line:
   - Source: ...
   - Scope: backend=<true|false>, frontend=<true|false>
   - Design linked: true/false
   - Updated: yes/no/N/A
   - Status changed: yes/no/N/A
