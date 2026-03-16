---
name: "ai-specs: Init Brownfield"
description: Adopt an existing codebase into the SDD workflow. Discovers stack, documents architecture, generates technical and functional baselines.
category: Workflow
tags: [standards, brownfield, bootstrap, baseline, adoption]
---

# ai-specs:init-brownfield — Adopt an Existing System

This is the adoption command for existing (brownfield) projects.
It analyzes the current codebase and produces everything needed to start working with the SDD/OpenSpec workflow on top of what already exists.

**It populates or updates two baselines:**

- `ai-specs/specs/*` — Technical baseline (standards, data model, API spec, dev guide)
- `openspec/specs/*` — Functional baseline (capabilities and behavior specs)

**It does NOT:**

- Modify existing source code
- Restructure the project
- Delete anything

---

## Narrative

The command follows four explicit phases:

1. **Discover** — Understand the current state of the system
2. **Normalize** — Generate technical standards that reflect reality
3. **Baseline** — Capture functional capabilities as OpenSpec specs
4. **Prepare** — Leave the project ready for incremental SDD changes

---

## Phase 1: Discover Current State

### 1.1 Validate template availability

Check that both template files exist:

- `ai-specs/specs/templates/backend-standards-template.mdc`
- `ai-specs/specs/templates/frontend-standards-template.mdc`

If any template is missing → FAIL immediately, show missing path, stop.

### 1.2 Optional Integrations (MCP)

Same as init-greenfield: offer Notion, Jira, Figma MCP auth if available.
If any MCP is authenticated, use it to gather existing project context (architecture docs, existing tickets, design systems).

### 1.3 Automated stack detection

Before asking the user anything, analyze the codebase to detect:

**Package/dependency files** (read and parse):
- `package.json`, `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`
- `requirements.txt`, `pyproject.toml`, `Pipfile`
- `go.mod`, `Cargo.toml`, `Gemfile`, `pom.xml`, `build.gradle`
- `composer.json`, `.csproj`, `*.sln`

**Configuration files** (detect conventions):
- `.eslintrc*`, `.prettierrc*`, `biome.json`, `deno.json`
- `tsconfig.json`, `jsconfig.json`
- `Dockerfile`, `docker-compose.yml`
- `.github/workflows/*`, `.gitlab-ci.yml`, `Jenkinsfile`
- `.env`, `.env.example`
- `openapi.yaml`, `openapi.json`, `swagger.*`

**Source structure** (detect architecture):
- Map top-level directories under `src/`, `app/`, `lib/`, `server/`, `client/`, `frontend/`, `backend/`, `packages/`
- Identify architectural patterns (MVC, DDD layers, feature-based, etc.)
- Detect monorepo vs single repo

**Existing tests** (detect testing approach):
- Look for `__tests__/`, `test/`, `tests/`, `spec/`, `cypress/`, `e2e/`, `playwright/`
- Identify frameworks from config files or imports

**Database/ORM** (detect data layer):
- Migration files, schema files, Prisma schema, TypeORM entities, Sequelize models, Drizzle config
- Database connection strings in config (type only, never capture credentials)

**API layer** (detect API style):
- Route files, controller files, resolver files (GraphQL)
- Existing OpenAPI/Swagger specs
- REST vs GraphQL vs gRPC indicators

### 1.4 Present findings and confirm

Present the detected stack as a structured summary:

```
## Detected Stack

### Backend
- Language: TypeScript (Node.js 20)
- Framework: Express 4.18
- Architecture: Feature-based (src/features/*)
- Auth: Passport.js (JWT strategy)
- Validation: Zod

### Data
- Database: PostgreSQL (detected from docker-compose)
- ORM: Prisma 5.x
- Migrations: Prisma Migrate (prisma/migrations/)

### API
- Style: REST
- OpenAPI: Yes (docs/openapi.yaml)
- Error format: { error: string, code: number }
- Versioning: URL prefix (/api/v1/)

### Frontend
- Framework: React 18 + Vite
- State: Zustand
- Routing: React Router v6
- Styling: Tailwind CSS
- Components: Radix UI

### Testing
- Unit: Vitest
- E2E: Playwright
- Coverage: ~62% (estimated from test file count)

### Tooling
- Lint: ESLint (flat config)
- Format: Prettier
- CI: GitHub Actions
- Monorepo: No
```

Use AskUserQuestion to confirm:
- "Is this accurate? Anything to correct or add?"

For anything NOT detected, ask follow-up questions (same as init-greenfield would, but only for gaps).

### 1.5 Architecture deep-dive

After stack confirmation, analyze and present:

**Module/capability map:**
- List the main modules, features, or bounded contexts found
- For each: brief description of what it appears to do (based on file/folder names, route names, model names)
- Estimated size (small/medium/large based on file count)

Example:
```
## Detected Modules

1. **auth** — Authentication and session management (src/features/auth/)
   Routes: /api/v1/auth/login, /register, /refresh, /logout
   Models: User, Session
   Size: Medium (~15 files)

2. **products** — Product catalog and search (src/features/products/)
   Routes: /api/v1/products (CRUD), /search
   Models: Product, Category
   Size: Large (~25 files)

3. **orders** — Order processing (src/features/orders/)
   Routes: /api/v1/orders (CRUD), /checkout
   Models: Order, OrderItem
   Size: Medium (~18 files)

4. **notifications** — Email and push notifications (src/features/notifications/)
   Routes: internal only (no public API)
   Models: Notification, NotificationPreference
   Size: Small (~8 files)
```

Use AskUserQuestion:
- "Is this module map correct? Any modules I missed or misidentified?"
- "Are there any modules that are deprecated, legacy, or should be excluded from specs?"

---

## Phase 2: Normalize Standards

Generate technical standards that **reflect the existing codebase**, not a theoretical ideal.

### 2.1 Generate backend standards

Create: `ai-specs/specs/backend-standards.mdc`

Rules:
- Use the template for STRUCTURE ONLY (headings and section order)
- Do NOT copy template example content verbatim
- **Document what IS, not what SHOULD BE** — standards must match the existing codebase
- If the codebase uses a pattern (e.g., feature-based folders), document that as the standard
- If the codebase is inconsistent in some area, note it and propose a convention (marked as PROPOSED)
- Provide concrete conventions and commands based on existing tooling

### 2.2 Generate frontend standards

Create: `ai-specs/specs/frontend-standards.mdc`

Same rules as backend: document existing reality, flag inconsistencies as PROPOSED.

### 2.3 Generate/update supporting documentation

If these files don't exist, generate them from codebase analysis:

- `ai-specs/specs/data-model.md` — Document existing models/entities/schemas
- `ai-specs/specs/api-spec.yml` — Generate from existing routes (or adopt existing OpenAPI spec if found)
- `ai-specs/specs/development_guide.md` — Setup, workflows, deployment based on existing scripts/CI

If any of these already exist in the project (e.g., an existing OpenAPI spec), **adopt and reference them** rather than generating from scratch. Ask the user if they want to keep the existing location or copy into ai-specs/specs/.

---

## Phase 3: Baseline Functional Specs

Generate OpenSpec specs that capture the current functional behavior of the system.

### 3.1 Generate capability specs

For each module/capability identified in Phase 1.5, create:

`openspec/specs/<capability>/spec.md`

Format (following OpenSpec conventions):

```markdown
# <Capability Name> Specification

## Purpose
<Brief description of what this capability does in the system>

## Requirements

### Requirement: <Name>
The system <SHALL|MUST|SHOULD> <behavior description>.

#### Scenario: <Scenario name>
- **GIVEN** <precondition>
- **WHEN** <action>
- **THEN** <expected outcome>
```

Rules:
- **Use LITE spec format** — short, behavior-first requirements
- **Only spec observable behavior** — what the API does, what the UI shows, not internal implementation
- **Derive from routes, models, and tests** — use existing tests as scenario sources when available
- **Mark confidence levels:**
  - Requirements derived from tests or explicit API contracts → high confidence (no marker needed)
  - Requirements inferred from code reading → mark with `<!-- inferred -->`
  - Requirements guessed from naming/structure → mark with `<!-- estimated -->`
- **Don't over-spec** — it's better to have 3 solid requirements than 15 guesses
- **Group by capability**, not by technical layer

### 3.2 Present baseline summary

Show a summary of generated specs:

```
## Functional Baseline Generated

| Capability | Requirements | Scenarios | Confidence |
|---|---|---|---|
| auth | 4 | 8 | High (from tests) |
| products | 6 | 12 | Medium (from routes + models) |
| orders | 5 | 9 | Medium (from routes) |
| notifications | 2 | 3 | Low (inferred from code) |

Total: 17 requirements, 32 scenarios across 4 capabilities
```

Use AskUserQuestion:
- "Review the generated specs? I can show any capability in detail."
- "Any capabilities that need more detail or corrections?"

---

## Phase 4: Prepare for Future Changes

### 4.1 Update openspec/config.yaml

Update the config with discovered project context:

```yaml
schema: spec-driven

context: |
  Tech stack: <detected stack summary>
  Architecture: <detected pattern>
  Project type: brownfield (adopted <date>)
  Key conventions: <brief list>
```

### 4.2 Verify project readiness

Run a quick checklist:

- [ ] `ai-specs/specs/backend-standards.mdc` exists and has content
- [ ] `ai-specs/specs/frontend-standards.mdc` exists and has content
- [ ] `ai-specs/specs/data-model.md` exists
- [ ] `ai-specs/specs/api-spec.yml` exists
- [ ] `openspec/specs/` has at least one capability spec
- [ ] `openspec/config.yaml` has project context

### 4.3 Report results

Display a final summary:

```
## Brownfield Adoption Complete

### Technical Baseline (ai-specs/specs/)
- ✓ backend-standards.mdc (from detected Express/TypeScript stack)
- ✓ frontend-standards.mdc (from detected React/Vite stack)
- ✓ data-model.md (4 models documented)
- ✓ api-spec.yml (23 endpoints documented)
- ✓ development_guide.md (setup + CI documented)

### Functional Baseline (openspec/specs/)
- ✓ auth/spec.md (4 requirements, 8 scenarios)
- ✓ products/spec.md (6 requirements, 12 scenarios)
- ✓ orders/spec.md (5 requirements, 9 scenarios)
- ✓ notifications/spec.md (2 requirements, 3 scenarios)

### Ready For
- /ai-specs:new-us → create new user stories
- /opsx:new → create changes with delta specs
- /opsx:explore → investigate the codebase

### Recommendations
- [ ] Review generated specs and correct any inaccuracies
- [ ] Add scenarios for critical business logic not captured
- [ ] Consider running /opsx:explore on the most complex module
```

---

## Guardrails

- **Never modify source code.** This command only generates documentation and spec files.
- **Never invent stack details.** If detection fails, ask the user.
- **Document what IS, not what SHOULD BE.** Standards must reflect the actual codebase. Improvements go in future changes.
- **Prefer accuracy over completeness.** A small accurate baseline beats a large inaccurate one.
- **Mark uncertainty.** Use confidence markers on inferred specs.
- **Respect existing documentation.** If the project already has docs (OpenAPI, README, etc.), adopt them rather than regenerating.
- **Don't overwrite existing specs.** If `openspec/specs/` already has content, ask before modifying.
- Templates define STRUCTURE ONLY (never copy example content).
