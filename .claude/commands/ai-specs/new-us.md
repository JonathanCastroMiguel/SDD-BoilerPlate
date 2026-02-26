---
name: "ai-specs: new-us"
description: Start a new OpenSpecs change from Jira, Notion, or manual input. Enrich the user story before running opsx:new.
category: Command
tags: [ai-specs, opsx, enrich, mcp]
---

Create a new OpenSpecs change by first capturing a User Story from Jira, Notion, or manual input.  
Then enrich it using `ai-specs: enrich-us`.  
Finally, execute `opsx:new` using the enriched story as the ONLY input.

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

### 5. Start OpenSpecs Change

Execute:

opsx:new

Input:
The Enriched User Story from Step 4.

Rules:
- Do not modify the enriched story before passing it.
- Follow opsx:new instructions exactly.
- Stop where opsx:new tells you to stop.

---

### 6. Report

Display:
- Selected source
- Confirmation that enrichment ran
- Change name/path created by opsx:new
- The next recommended action