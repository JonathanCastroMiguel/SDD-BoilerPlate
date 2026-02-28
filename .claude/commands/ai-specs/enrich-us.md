---
name: "ai-specs: Enrich US"
description: Enrich a user story from Jira, Notion, or manual text. For Jira and Notion inputs, update the source by default unless the user opts out. If Figma references are present, enrich with design context and set design-linked: true.
category: Command
tags: [ai-specs, enrich, user-story, mcp, figma]
---

Enrich a User Story so a developer can execute autonomously.

Input ($ARGUMENTS) can be:
- A Jira issue key or URL (e.g., PROJ-123)
- A Notion page URL or page id
- A raw markdown/text user story
- A "Base User Story" markdown block (recommended)

---

## Steps

### 0) Validate input

If $ARGUMENTS is empty or too vague, ask the user to provide one of:
- Jira issue key/url
- Notion page url/id
- Raw user story text (or a Base User Story block)

---

### 1) Identify input type

Classify $ARGUMENTS as one of:
- Jira reference
- Notion reference
- Base User Story block
- Manual text

If ambiguous, ask one clarifying question.

---

### 2) Retrieve source content

#### Jira
- Fetch title, description, acceptance criteria, relevant comments.
- Construct a Base User Story block.

#### Notion
- Fetch page title and requirement-related blocks.
- Construct a Base User Story block.

#### Base User Story / Manual
- Use as source of truth.
- If manual, convert into Base User Story format.

---

### 3) Ensure Base User Story block

Ensure exactly one block:

# Base User Story
Source: <Jira|Notion|Manual>
Reference: <issue key/url or notion url/id or "N/A">

## Title
## Problem / Context
## Desired Outcome
## Acceptance Criteria (raw)
## Constraints / Notes

---

### 4) Enrich the User Story

Produce an implementation-ready document:

# Enriched User Story

The enriched story MUST include:

design-linked: <true|false>
source: <Jira|Notion|Manual>
reference: <issue key/url or notion url/id or "N/A">

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
- Keep it actionable and specific.
- Do not invent tools/stack details unless already known via project context.
- If there are contradictions or missing requirements, ask a short clarifying question.

---

### 4.1) Design-aware enrichment (Figma)

If the input content (Notion/Jira/Manual/Base US) contains a "Design References" section or any Figma URL(s):

#### 4.1.A Detect Design References
Look for a section like:

## Design References

Figma File:
<url>

Node(s):
- <url with node-id=...>
- ...

Components:
- <component name>
- ...

Parse:
- Figma file URL (required when present)
- Node URLs (preferred, most deterministic)
- Component names (optional fallback)

Set:
- design-linked: true

If no design references exist:
- design-linked: false
- Skip the rest of this step.

#### 4.1.B Inspect via Figma MCP (if authenticated)
If Figma MCP is authenticated:
- Inspect the referenced nodes/components using Figma MCP.
- Retrieve minimum useful implementation detail:
  - Node/component name(s)
  - Component variants/states (if available)
  - Key layout constraints / notes (as available)

If Figma MCP is NOT authenticated:
- Do not fail.
- Add a note indicating Figma is not authenticated.

#### 4.1.C Add "Design Integration" section
In the Enriched User Story, add:

## Design Integration
- Figma File: <url or "N/A">
- Nodes:
  - <node url> (Name: <...>, Notes: <...>)
- Components:
  - <component name> (Variants/States: <...>)
- Implementation Notes:
  - <bullet points that connect design constraints to implementation>

Never modify Figma content automatically.

---

### 5) Source Update Behavior

#### Jira inputs

- Ask: "Update the Jira ticket with the enhanced version? (yes/no)"
- Default behavior: update unless user says "no"

If updating:
1) Append after original content.
2) Use H2 headings: [original] and [enhanced].
3) Keep formatting readable.
4) If status is "To refine", move to "Pending refinement validation".

---

#### Notion inputs

- Ask: "Update the Notion page with the enhanced version? (yes/no)"
- Default behavior: update unless user says "no"

If updating:
1) Append enhanced content below original.
2) Add section headings: [original] and [enhanced].
3) Preserve existing formatting.

Status update (safe mode):
- If the page belongs to a database AND has a property named:
  - Status
  - State
  - Workflow
- Then move it to:
  - "Pending refinement validation"
  - or the closest matching value.
- If no such property exists, do nothing.

Never fail if status cannot be updated.

---

### 6) Output

Always return:
1) Base User Story block
2) Enriched User Story
3) Summary line:
   - Source: Jira/Notion/Manual
   - Design linked: true/false
   - Updated: yes/no/N/A
   - Status changed: yes/no/N/A