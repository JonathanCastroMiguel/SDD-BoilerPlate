---
name: "ai-specs: Enrich US"
description: Enrich a user story from Jira, Notion, or manual text. If Figma references are present, mark design-linked: true and defer structural extraction to apply phase.
category: Command
tags: [ai-specs, enrich, user-story, mcp, figma]
---

Enrich a User Story so a developer can execute autonomously.

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
- Fetch title and requirement-related blocks.
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

### 4) Enrich the User Story

Produce:

# Enriched User Story

design-linked: <true|false>
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
- Do not invent tech stack.
- Ask clarification if needed.

---

## 4.1) Design Reference Detection (Light Mode)

If the input contains:

- A section titled "## Design References"
OR
- Any Figma URL

Then:

- Set `design-linked: true`
- Extract:
  - Figma File URL (if present)
  - Node URL(s) with node-id
  - Component names (if listed)

Do NOT:
- Extract layout hierarchy
- Extract spacing/padding
- Freeze structure
- Create Layout Contract

If no Figma references exist:
- Set `design-linked: false`

---

### 4.2) Add Design Reference Section (Light)

If design-linked is true, append:

## Design References

Figma File:
<url or inferred from node>

Referenced Nodes:
- <node-url-1>
- <node-url-2>

Note:
Design structure and layout must be retrieved directly from Figma MCP during the implementation phase (opsx:apply). No structural assumptions are frozen at enrichment time.

---

### 5) Source Update Behavior

#### Jira
Ask:
"Update the Jira ticket with the enhanced version? (yes/no)"
Default: yes

If updating:
- Append enhanced section
- Move status from "To refine" → "Pending refinement validation" (if possible)

---

#### Notion
Ask:
"Update the Notion page with the enhanced version? (yes/no)"
Default: yes

If updating:
- Append enhanced content
- Move Status property to "Pending refinement validation" (if exists)

Never fail if status transition not possible.

---

### 6) Output

Always return:

1) Base User Story
2) Enriched User Story
3) Summary line:
   - Source: ...
   - Design linked: true/false
   - Updated: yes/no/N/A
   - Status changed: yes/no/N/A