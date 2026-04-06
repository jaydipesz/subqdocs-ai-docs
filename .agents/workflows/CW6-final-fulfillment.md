---
description: CW6 — Final fulfillment audit against requirements and technical plan (Human-Out-The-Loop).
---

# CW6 — Fulfillment Audit

## Purpose
Ensure the final implemented feature fully delivers on every requirement, business rule, and technical task defined in the initial Intake and the Implementation Plan. This phase performs a final quality scan to catch any "technical debt" (logs, untyped vars) and verify the code is ready for PR.

## Standalone Inputs
- **Global Inputs** (from Master Intake: `FEATURE_DESCRIPTION`, `BUSINESS_RULES`, `AUTH_LEVEL`, `REAL_TIME`, `FILE_UPLOAD`)
- **IMPLEMENTATION PLAN** — output of CW3
- **FIGMA ANALYSIS REPORT** — output of CW1 (for screen/state mapping)
- **UI VERIFICATION REPORT** — output of CW5A

## Trigger Condition
Immediately after CW5A completes. Mandatory.

## HOTL (Human-Out-The-Loop) Verification Protocol

This workflow is designed to execute autonomously. Do not ask for confirmation unless a **Plan Deviation** or **Logical Conflict** is discovered.

### Step 1: Quality Violation Scan
1. Load `docs/skills/18-standard-violation-audit.md`. 
2. Scan every new or modified file from CW4 (Backend) and CW5 (Frontend).
3. **Auto-Remediation:** For every **Violation** found (e.g. `console.log`, missing `organization_id`, `: any` where a type exists), fix the code immediately. 
4. Do not report fixed violations as blockers; list them in the "Actions Taken" column of the final report.

### Step 2: Traceability Audit (Requirement vs. Evidence)
For each item in the following categories, locate the specific file and line number(s) where it is implemented:

| Category | Source | Requirement to Trace |
|---|---|---|
| **Intake Rules** | Master Intake | Every specified `BUSINESS_RULE`, `AUTH_LEVEL`, `REAL_TIME`, `FILE_UPLOAD` |
| **Technical Tasks** | CW3 Plan | Every numbered "Step-by-Step" implementation task |
| **UX Coverage** | CW1 Specs | Every screen name and state (loading, error, empty) |

### Step 3: Type Safety & Build Check
1. Run `npx tsc --noEmit` in both `subqdocs-backend/` and `subqdocs-frontend/`.
2. Every type error must be fixed before proceeding.

### Step 4: Final Assessment
- If **100% Traceability** is achieved and **0 Violations** remain: Pass.
- If a requirement was **omitted** or **changed**: Flag as a **Plan Deviation** and stop for human review.

## Output
**FULFILLMENT REPORT** with this format:

### 1. Requirement Traceability Matrix
| Requirement / Task | Evidence (File + Line) | Status | Action Taken/Note |
|---|---|---|---|
| Rule: [Name] | [Path:Line] | ✅ PASS | |
| Task: [Name] | [Path:Line] | ✅ PASS | |
| State: [Name] | [Path:Line] | ✅ PASS | |

### 2. Standard Compliance Log
| File | Violation Fixed | Detail |
|---|---|---|
| `...` | `console.log` | Removed 2 occurrences |
| `...` | `organization_id`| Critical: Added missing scope check |

### 3. Final Build Status
- Backend `tsc`: [PASS / FAIL]
- Frontend `tsc`: [PASS / FAIL]

## Done Condition
100% Traceability achieved. All violations auto-fixed. Build passes. Feature is ready for merge.

## Skills Required
- `docs/skills/18-standard-violation-audit.md`
- `docs/skills/14-structured-backend-implementation.md` (Testing section)
- `docs/skills/15-structured-frontend-implementation.md` (Rules 1-4)
