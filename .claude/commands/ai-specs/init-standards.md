---
name: "ai-specs: Init Standards"
description: Generate definitive backend and frontend standards from templates using the project's tech stack
category: Workflow
tags: [standards, templates, bootstrap]
---

Generate missing definitive standards files for this repository using the provided templates and the project's actual tech stack.

Goal:
Create (or update if empty) the following files:

- ai-specs/specs/backend-standards.mdc
- ai-specs/specs/frontend-standards.mdc

Templates:

- ai-specs/specs/templates/backend-standards-template.mdc
- ai-specs/specs/templates/frontend-standards-template.mdc

---

## Steps

### 1. Validate template availability

Check that both template files exist.

If any template is missing:

- FAIL immediately
- Show the missing path
- Do not continue

---

### 2. Collect project stack information

Use the AskUserQuestion tool to gather the necessary stack details.

Minimum required information:

Backend:
- Language
- Framework
- Architecture pattern
- Validation approach
- Authentication strategy
- Background jobs (if any)

Data:
- Database
- ORM / Query builder
- Migration strategy

API:
- REST or GraphQL
- OpenAPI version (if applicable)
- Error response format
- Versioning strategy

Testing:
- Unit testing framework
- Integration / e2e framework
- Coverage expectations

Frontend:
- Framework
- State management approach
- Routing
- Styling solution
- Component library (if any)

Tooling:
- Linting
- Formatting
- CI/CD
- Monorepo or single repo

If any answer is vague or missing, ask follow-up questions.

---

### 3. Generate backend standards

Create:

ai-specs/specs/backend-standards.mdc

Rules:

- Use the template for STRUCTURE ONLY (headings and section order).
- Do NOT copy template example content verbatim.
- Replace generic placeholders with stack-specific rules.
- Provide concrete conventions and commands.
- Do not invent tools not confirmed by the user.

Must include:

- Folder structure conventions
- API patterns
- Data access patterns
- Error handling conventions
- Authentication rules
- Logging conventions
- Testing rules
- Migration strategy
- Definition of Done for backend changes

---

### 4. Generate frontend standards

Create:

ai-specs/specs/frontend-standards.mdc

Rules:

- Use the template for STRUCTURE ONLY.
- Do NOT copy example content verbatim.
- Provide stack-specific conventions.

Must include:

- Component structure conventions
- State management patterns
- Styling conventions
- Accessibility baseline
- Testing strategy
- Definition of Done for frontend changes

---

### 5. Report results

Display a clear summary:

- Which files were created
- Which templates were used
- The confirmed stack assumptions

---

## Output On Success

## Standards Initialized

Created:
- ai-specs/specs/backend-standards.mdc
- ai-specs/specs/frontend-standards.mdc

Templates used:
- ai-specs/specs/templates/backend-standards-template.mdc
- ai-specs/specs/templates/frontend-standards-template.mdc

Stack configuration has been applied to both standards files.

---

## Output On Error (Missing Template)

## Init Standards Failed

Missing template:
- <path>

Add the template file and retry.

---

## Guardrails

- Templates define STRUCTURE ONLY (never copy example content).
- Never invent stack details — always confirm with the user.
- Keep standards concrete and actionable.
- Do not overwrite existing standards without explicit confirmation.
