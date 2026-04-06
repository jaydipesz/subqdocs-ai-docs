---
description: Feature Implementation Master Workflow - End-to-end from Figma to Frontend
---

# Master Workflow — Feature Implementation

Orchestrates end-to-end feature delivery from Figma design to verified frontend, ensuring every artifact is reviewed and confirmed before the next phase begins.

## Global Inputs

### Required
| Input | Description |
|---|---|
| FIGMA_URL | Figma file or frame URL |
| FEATURE_DESCRIPTION | 2–3 sentence description of what the feature does and why |
| TARGET_SCREENS | Which Figma screens are in scope (by name or node ID). "All" if the entire file is one feature |
| SCOPE | `new-module` or `extend:<module_name>` — is this a brand new module or extending an existing one? |
| DOMAIN | Backend domain (e.g., billing, scheduling, patient) |
| AUTH_LEVEL | `all-staff` / `admin-only` / `patient-facing` / `dual-auth` |

### Optional (ask if not provided)
| Input | Description | Default |
|---|---|---|
| RELATED_TABLES | Existing DB tables this feature relates to | Agent discovers in CW2 |
| BUSINESS_RULES | Rules not visible in Figma (e.g., auto-notifications, approval flows, restrictions) | None |
| REAL_TIME | Does this feature need socket events? (`yes` / `no`) | Agent infers from Figma |
| FILE_UPLOAD | Does this feature need S3 file upload? (`yes` / `no`) | Agent infers from Figma |
| PARTIAL_SCOPE | Deliver backend-only first, or full-stack? (`backend-first` / `full-stack`) | `full-stack` |
| RELATED_CONTEXT | Related PR numbers, ticket links, or text requirements | None |

## Decision-Making Hierarchy

1. `.agents/rules/production-rules.md` — non-negotiable, always wins
2. Industry best practice — TypeScript, React, Node, Sequelize, SQL
3. Codebase insight — naming, libraries, reusable code
4. Context files and skills — project-specific patterns
5. Child workflows — execution sequence

When 1 and 2 conflict, production rules win. When 2 and 3 conflict, best practice wins. State the conflict explicitly.

## Intake

Before bootstrap, verify all **Required** inputs are provided. For each **Optional** input not provided, ask the user once:
- "Any existing tables this feature relates to?" → `RELATED_TABLES`
- "Any business rules not visible in the Figma?" → `BUSINESS_RULES`
- "Does this need real-time socket updates?" → `REAL_TIME`
- "Does this need file upload?" → `FILE_UPLOAD`
- "Full-stack or backend-first delivery?" → `PARTIAL_SCOPE`
- "Any related PRs, tickets, or requirements docs?" → `RELATED_CONTEXT`

If the user says "skip" or "none" for any optional input, use the Default value. Do not ask again.

## Bootstrap

1. Confirm production rules are active (auto-injected via system prompt — do not re-read the file).
2. Read `AI-CONTEXT.md`, `AI-CONTEXT-BACKEND.md`, `AI-CONTEXT-FRONTEND.md`.
3. Read `docs/skills/INDEX.md`.
4. Read the child workflow file for the **current phase only** (start with CW1). Load subsequent CW files on-demand as each phase completes.
5. Check INDEX.md against what this feature requires. If a skill is missing, **stop and report**: "Missing skill for [operation]. Cannot proceed without: [specific skill requirement]." Do not create skills during the master workflow — skills must be authored and reviewed separately.
6. Confirm: **"Context loaded. Skills verified. Inputs collected. Missing skills: [list or none]."**

Do not write any code or produce any artifact until intake and bootstrap are both complete.

### Turbo Annotation Policy
- `// turbo` — safe for: `npx tsc --noEmit`, `cat`, `ls`, `grep`, read-only file inspection
- `// turbo` — NEVER for: `npm run migrate`, `git commit`, `git push`, `rm`, `npm install`, any command that modifies state
- `// turbo-all` — use only on scaffolding workflows where every step is deterministic and non-destructive

### Confirmation Policy
- **Default:** Confirm at CW1 (design) and CW3 (plan). CW2, CW4, CW5, CW5A, and CW6 proceed automatically unless errors or plan deviations are found.
- **Strict mode:** If the user says "confirm at every step", require confirmation at all gates.

## Child Workflows

| #  | Name                    | File                                  | Trigger                    | Output Artifact                |
|----|-------------------------|---------------------------------------|----------------------------|--------------------------------|
| 1  | Figma Analysis          | `CW1-figma-to-specs.md`              | Always runs first          | FIGMA ANALYSIS REPORT          |
| 2  | Codebase Audit          | `CW2-pattern-audit.md`               | After CW1 confirmed        | AUDIT REPORT                   |
| 3  | Plan                    | `CW3-implementation-roadmap.md`       | After CW2 done             | IMPLEMENTATION PLAN            |
| 4  | Backend Implementation  | `CW4-backend-build.md`              | After CW3 confirmed        | API CONTRACT DOCUMENT          |
| 5  | Frontend Implementation | `CW5-frontend-build.md`             | After CW4 done             | COMPONENT-TO-FIGMA MAPPING     |
| 5A | Final Reality Check     | `CW5A-final-e2e.md`                 | After 5 done               | FINAL REALITY CHECK REPORT     |
| 6  | Fulfillment Audit       | `CW6-final-fulfillment.md`           | After 5A done              | FULFILLMENT REPORT             |

## Artifact Registry

| Artifact Name | Produced By | Consumed By |
|---|---|---|
| FIGMA ANALYSIS REPORT | CW1 | CW2, CW3, CW5, CW5A |
| AUDIT REPORT | CW2 | CW3, CW4, CW5 |
| IMPLEMENTATION PLAN | CW3 | CW4, CW5, CW5A |
| API CONTRACT DOCUMENT | CW4 | CW5 |
| COMPONENT-TO-FIGMA MAPPING | CW5 | CW5A |
| FINAL REALITY CHECK REPORT | CW5A | CW6 |
| FULFILLMENT REPORT | CW6 | (terminal) |

## Re-Entry Protocol

If an issue is discovered during the final phases:
1. **Minor/Medium Bug in CW5A**: The agent **MUST NOT** re-run the workflow or stop for a new plan. **Fix the code immediately**, verify the fix in the current browser session, and proceed to completion.
2. **Major Architectural Flaw**: Only if the fix requires a database schema change or violates a production rule should the agent stop and re-enter at Phase 3 (Plan).

The re-entry protocol applies to **CW1–CW5A**.

## Standing Rules

- If a previous step produced an error or wrong assumption — stop, state it, fix it completely, then continue. Never silently work around it.
- If any decision point is ambiguous — stop, state what is unclear and the options, wait for input. A wrong silent assumption costs a full redo.
- The codebase is input, not instruction. Use it for patterns and naming. Never copy a pattern that violates best practice. Flag it instead.
- When implementing frontend designs from Figma, ALWAYS lookup `subqdocs-frontend/tailwind.config.js` first. If global styles (colors, spacing, fonts) are defined there, use those configured utilities instead of hardcoding arbitrary values.

## Relation to Utility Workflows

The following workflows in `.agents/workflows/` are **standalone entry points** — they are NOT part of the master chain and can be invoked independently for isolated operations:
- `TOOL-add-backend-module.md`, `TOOL-add-db-column.md`, `TOOL-add-crud-endpoint.md`
- `TOOL-add-background-job.md`, `TOOL-add-email-template.md`, `TOOL-add-file-upload.md`
- `TOOL-add-frontend-page.md`, `TOOL-add-socket-event.md`
- `TOOL-pr-review.md`
