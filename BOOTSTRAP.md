# 🚀 BOOTSTRAP GUIDE

## Using This Repository as a Greenfield Template

This repository is designed to be reused as a **governed development
starter kit**.

It provides:

-   OpenSpecs workflow (`/opsx:*`)
-   AI governance layer (`/ai-specs:*`)
-   Deterministic standards system
-   Documentation enforcement
-   Spec-Driven Development (SDD)

This guide explains how to bootstrap a new project using this template.

------------------------------------------------------------------------

# 1️⃣ Create a New Project from This Template

Option A (Recommended): - Mark this repository as a GitHub Template. -
Click **"Use this template"** to create a new repo.

Option B: - Clone this repository. - Remove its `.git` folder. -
Initialize a new Git repository.

------------------------------------------------------------------------

# 2️⃣ Clean Project-Specific Content

Keep:

-   `.claude/`
-   `ai-specs/`
-   `openspec/`
-   `CLAUDE.md`
-   `.gitignore`
-   `README.md`
-   `BOOTSTRAP.md`

Remove (if present):

-   Old project source code
-   Previous domain-specific models
-   Old CI config (if not reusable)
-   Example APIs or dummy data

The governance and workflow system must remain intact.

------------------------------------------------------------------------

# 3️⃣ Initialize Standards (MANDATORY)

If these files do not exist:

-   `ai-specs/specs/backend-standards.mdc`
-   `ai-specs/specs/frontend-standards.mdc`

Run:

/ai-specs:init-standards

You will be prompted for:

-   Backend language & framework
-   Architecture pattern
-   Database & ORM
-   API style (REST / GraphQL)
-   Testing stack
-   Frontend framework
-   Tooling & CI strategy

This generates deterministic, stack-specific standards.

Without this step, development is blocked by design.

------------------------------------------------------------------------

# 4️⃣ Start Spec-Driven Development

Create your first change:

/opsx:new

Follow the lifecycle strictly:

1.  Define specs
2.  Apply change
3.  Verify artifacts
4.  Update documentation
5.  Archive change

Do not bypass the workflow.

------------------------------------------------------------------------

# 5️⃣ Documentation Rules

All documentation lives under:

ai-specs/specs/

Templates live under:

ai-specs/specs/templates/

Templates define structure only. Never copy template example content
verbatim.

Documentation is enforced automatically during:

/opsx:archive

------------------------------------------------------------------------

# 6️⃣ Optional Hardening

For stricter governance, consider:

-   Enforcing non-empty standards validation
-   Adding CI validation for api-spec.yml changes
-   Requiring archive before merge
-   Disallowing direct commits to main

------------------------------------------------------------------------

# 🎯 Final Principle

This repository is not a code scaffold.

It is a governed development system.

Standards first. Specs first. Code second.
