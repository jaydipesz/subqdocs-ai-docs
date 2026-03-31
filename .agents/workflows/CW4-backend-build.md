---
description: CW4 — Implement the backend in strict dependency order and produce an API contract.
---

_Formats: [WORKFLOW-FORMAT.md](./WORKFLOW-FORMAT.md)._

# CW4 — Backend Implementation

## Purpose
Implement the backend in strict dependency order, verify it compiles, and produce an API contract for the frontend.

## Standalone Inputs
- **IMPLEMENTATION PLAN** — output of CW3
- **AUDIT REPORT** — output of CW2
- `AUTH_LEVEL` — `all-staff` / `admin-only` / `patient-facing` / `dual-auth`

## Trigger Condition
After CW3 is confirmed by the user.

## Steps

1. Load `docs/skills/14-structured-backend-implementation.md`. Follow its steps 1–9 using the IMPLEMENTATION PLAN and AUDIT REPORT as inputs.

2. **Verification:** Run `npx tsc --noEmit` in `subqdocs-backend/`. Fix every type error before proceeding. Do not continue until exit code 0.

3. Open the 2 closest modules in `src/modules/` to the one just created (by alphabetical proximity). Compare error message phrasing and match before writing error strings.

4. Produce the **API CONTRACT DOCUMENT**.

## Output
**API CONTRACT DOCUMENT** with this format per endpoint:
```
[METHOD] [PATH]
  Auth: [middleware chain]
  Body/Query: { field: type, ... }
  Response: { field: type, ... }
  Errors: [list of error cases and HTTP status codes]
```
End with: **"Backend complete. API contract produced. Confirm to begin frontend."**

## Done Condition
`tsc --noEmit` passes. User replies **"confirmed"**. Do not proceed to CW5 until confirmation is received.

## Skills Required
- `docs/skills/14-structured-backend-implementation.md`
- `docs/skills/17-database-migration-safety.md`
