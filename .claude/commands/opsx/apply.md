---
name: "OPSX: Apply"
description: Implement tasks from an OpenSpec change (Experimental). If design-linked is true, fetch design structure live from Figma MCP before implementing UI tasks.
category: Workflow
tags: [workflow, artifacts, experimental, figma, mcp]
---

Implement tasks from an OpenSpec change.

**Input**: Optionally specify a change name (e.g., `/opsx:apply add-auth`). If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

---

## Steps

### 1. Select the change

If a name is provided, use it. Otherwise:
- Infer from conversation context if the user mentioned a change
- Auto-select if only one active change exists
- If ambiguous, run `openspec list --json` to get available changes and use the **AskUserQuestion tool** to let the user select

Always announce: "Using change: <name>" and how to override (e.g., `/opsx:apply <other>`).

---

### 2. Check status to understand the schema

```bash
openspec status --change "<name>" --json
```

Parse the JSON to understand:
- `schemaName`: The workflow being used (e.g., "spec-driven")
- Which artifact contains the tasks (typically "tasks" for spec-driven, check status for others)

---

### 3. Get apply instructions

```bash
openspec instructions apply --change "<name>" --json
```

This returns:
- Context file paths (varies by schema)
- Progress (total, complete, remaining)
- Task list with status
- Dynamic instruction based on current state

Handle states:
- If `state: "blocked"` (missing artifacts): show message, suggest using `/opsx:continue`
- If `state: "all_done"`: congratulate, suggest archive
- Otherwise: proceed to implementation

---

### 4. Read context files

Read the files listed in `contextFiles` from the apply instructions output.

---

### 4.1 Detect design-linked and design references (Live Design Mode)

Scan the loaded context files for:

- `design-linked: true`

If found, enable **Live Design Mode**.

In Live Design Mode, extract design references from the context:
- Look for a `## Design References` section (preferred).
- Parse:
  - `Figma File:` URL (optional if node URLs exist)
  - `Referenced Nodes:` list of node URLs containing `node-id=...`

Rules:
- If Live Design Mode is enabled but no node-id URLs are found:
  - Pause and ask the user to provide at least one Figma node URL with `node-id`.
  - Do NOT implement UI tasks until a node-id is available.
- Do not assume design structure from prose alone.

---

### 4.2 Fetch design snapshot via Figma MCP (Live Design Mode)

If Live Design Mode is enabled AND Figma MCP is authenticated:

For each referenced node:
- Inspect the node via Figma MCP.
- Retrieve best-effort structure:
  - Node name and type
  - Auto-layout direction (vertical/horizontal/none)
  - Item spacing and padding (if available)
  - Direct children in visual order
  - Component instances and variant info (if resolvable)

Build a **Design Snapshot** in-memory (do not write back to Figma/Notion by default):

## Design Snapshot (from Figma MCP)
Figma File: <...>

Node: <...> (node-id=...)
- Auto Layout: <...>
- Spacing/Padding: <...>
- Children (visual order):
  - <child 1>
  - <child 2>
  - <child 3>
- Components/Variants:
  - <...>

If Figma MCP is not authenticated:
- Keep Live Design Mode enabled.
- Warn that UI fidelity may be lower.
- Ask the user whether to proceed or to authenticate Figma via `/mcp`.

---

### 5. Show current progress

Display:
- Schema being used
- Progress: "N/M tasks complete"
- Remaining tasks overview
- Dynamic instruction from CLI
- If Live Design Mode is enabled: show "Live Design Mode: ON" and list the referenced node-ids

---

### 6. Implement tasks (loop until done or blocked)

For each pending task:
- Show which task is being worked on
- Make the code changes required
- Keep changes minimal and focused
- Mark task complete in the tasks file: `- [ ]` → `- [x]`

#### 6.1 UI tasks must follow design snapshot (Live Design Mode)

If Live Design Mode is ON and the task impacts UI (frontend views/components/styles):
- Use the Design Snapshot as the primary source of truth.
- Do not approximate layout from prose if it conflicts with the snapshot.
- If there is a mismatch between tasks/specs and the snapshot:
  - Pause and propose updating artifacts OR clarifying which is authoritative.
- If a component is referenced in the snapshot but not in codebase:
  - Create a wrapper component to match the design structure.
- Prefer component composition that mirrors the Figma hierarchy (avoid flattening).

Non-UI tasks (API, services, backend) proceed normally.

Pause if:
- Task is unclear → ask for clarification
- Implementation reveals a design issue → suggest updating artifacts
- Error or blocker encountered → report and wait for guidance
- User interrupts

---

### 7. On completion or pause, show status

Display:
- Tasks completed this session
- Overall progress: "N/M tasks complete"
- If all done: suggest archive
- If paused: explain why and wait for guidance

---

## Output During Implementation

```
## Implementing: <change-name> (schema: <schema-name>)

Live Design Mode: ON
Referenced node-ids: <...>

Working on task 3/7: <task description>
[...implementation happening...]
✓ Task complete
```

## Output On Completion

```
## Implementation Complete

**Change:** <change-name>
**Schema:** <schema-name>
**Progress:** 7/7 tasks complete ✓

### Completed This Session
- [x] Task 1
- [x] Task 2
...

All tasks complete! You can archive this change with `/opsx:archive`.
```

## Output On Pause (Issue Encountered)

```
## Implementation Paused

**Change:** <change-name>
**Schema:** <schema-name>
**Progress:** 4/7 tasks complete

### Issue Encountered
<description of the issue>

**Options:**
1. <option 1>
2. <option 2>
3. Other approach

What would you like to do?
```
---

## Guardrails

- Keep going through tasks until done or blocked
- Always read context files before starting
- If task is ambiguous, pause and ask before implementing
- Keep code changes minimal and scoped
- Update task checkbox immediately after completing each task
- Pause on errors, blockers, or unclear requirements

Live Design Mode guardrails:
- If `design-linked: true`, do not implement UI tasks without at least one node-id URL.
- Prefer the Design Snapshot over prose when implementing UI.
- Never modify Figma.
- Do not write snapshot back to Notion unless explicitly requested.

---

## Fluid Workflow Integration

This skill supports the "actions on a change" model:

- Can be invoked anytime
- Allows artifact updates if implementation reveals design issues
- Not phase-locked
