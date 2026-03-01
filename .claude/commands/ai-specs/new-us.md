
IMPORTANT OUTPUT CONTRACT:
- You MAY display the Base User Story (original) for traceability.
- You MUST then append the Enriched User Story wrapped in the exact markers:
  <!-- BEGIN_ENRICHED_USER_STORY -->
  # Enriched User Story
  ... (strict metadata + content) ...
  <!-- END_ENRICHED_USER_STORY -->
- Do NOT output `## [Enriched User Story]`. Use the exact H1 heading.
- Do NOT alter the Base User Story content when copying it.


---
name: "ai-specs: new-us"
description: Start from Jira, Notion, or manual input. Enrich the user story and optionally create an OpenSpecs change.
category: Command
tags: [ai-specs, opsx, enrich, mcp]
---

Create a new User Story from Jira, Notion, or manual input.  
Enrich it using `ai-specs: enrich-us`.  
Then optionally execute `opsx:new` using the enriched story as the ONLY input.

---

## Steps

### 0. Detect available MCP integrations

Determine which integrations are authenticated:
- Atlassian (Jira)
- Notion

Rules:
- Only offer sources that are authenticated.
- If none are authenticated, default to Manual input.

---

### 1. Choose input source

Ask the user:

Do you want to start from:
- Jira issue (if connected)
- Notion page (if connected)
- Manual input

If the selected source requires authentication and authentication is missing:
- Prompt the user to authenticate via /mcp.
- Offer the option to continue using Manual Input instead.

If integration access fails after authentication,
automatically fall back to Manual Input.

---

### 2. Collect and normalize into Base User Story

#### Jira (if selected)

Ask for issue key or URL.

Retrieve:
- Title
- Description
- Acceptance criteria (if present)
- Only relevant comments (optional)

#### Notion (if selected)

Ask for page URL or ID.

Retrieve:
- Page title
- Requirement-related blocks
- Acceptance criteria
- Constraints

#### Manual (if selected)

Ask user to paste raw user story text.

---

### 3. Normalize into required format

Produce exactly one markdown block:

# Base User Story
Source: <Jira|Notion|Manual>
Reference: <issue key/url or notion url/id or "N/A">

## Title
<short title>

## Problem / Context
<what is happening and why it matters>

## Desired Outcome
<what success looks like>

## Acceptance Criteria (raw)
- ...

## Constraints / Notes
- ...

---

### 4. Enrich the User Story

Execute:

ai-specs: enrich-us

Input:
The Base User Story markdown block.

Rules:
- The output must be an implementation-ready Enriched User Story.

---

### 5. Decide Next Action

Ask the user:

Do you want to:

- Stop here (PO mode) and keep only the Enriched User Story
- Continue and create an OpenSpecs change (run opsx:new)

Rules:
- If the user chooses "Stop here", end the command.
- If the user chooses "Continue", proceed to Step 6.
- If the user provides no explicit answer, default to Stop here.

If stopping:
- Inform the user they can later run:

  opsx:new

  and paste the Enriched User Story manually.

---

### 6. Start OpenSpecs Change (Delivery Mode)

Execute:

opsx:new

Input:
The Enriched User Story from Step 4.

Rules:
- Do not modify the enriched story before passing it.
- Follow opsx:new instructions exactly.
- Stop where opsx:new tells you to stop.

---

### 7. Report

Display:
- Selected source
- Confirmation that enrichment ran
- Whether OpenSpecs change was created
- If created: change name/path
- The next recommended action