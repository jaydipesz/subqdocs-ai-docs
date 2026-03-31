---
description: CW3 — Create a complete implementation plan before writing code.
---

# CW3 — Plan

## Purpose
Produce a complete, reviewable implementation plan covering backend and frontend before any code is written.

## Standalone Inputs
- **FIGMA ANALYSIS REPORT** — output of CW1
- **AUDIT REPORT** — output of CW2
- `SCOPE` — `new-module` or `extend:<module_name>`
- `AUTH_LEVEL` — `all-staff` / `admin-only` / `patient-facing` / `dual-auth`
- `BUSINESS_RULES` — rules not visible in Figma (if provided)
- `REAL_TIME` — `yes` / `no` — whether socket events are needed
- `FILE_UPLOAD` — `yes` / `no` — whether S3 file upload is needed
- `PARTIAL_SCOPE` — `backend-first` / `full-stack`

## Trigger Condition
After CW2 is confirmed by the user.

## Steps

### Backend Plan
1. If `SCOPE` is `new-module`, plan a full module scaffold. If `extend:<module_name>`, plan additions to the existing module.
2. List new models: table name, every column (name, DataType, allowNull, defaultValue), paranoid flag, `organization_id` requirement.
3. List migrations: new table or alter existing.
   - If altering: for each new column state whether it is nullable or has a default.
   - Flag any `NOT NULL` column without a default on an existing table as **MIGRATION RISK**. Apply `migration-safety` skill.
4. List repositories: function name, parameters, return type, query description.
5. List validation schemas: one schema per endpoint, field name + rule per field (from FIGMA ANALYSIS REPORT field catalog).
6. If `BUSINESS_RULES` was provided, add validation rules or controller guards for each business rule.
7. List controllers: HTTP method, path, request shape, response shape, error cases.
8. List route registrations: exact middleware chain per route in order. Use the middleware chain determined by `AUTH_LEVEL` in the AUDIT REPORT.
9. List `server.ts` changes: new imports and `apiRoutes` additions.
10. If `REAL_TIME` is `yes`: list socket events — event name, payload shape, room.
11. If `FILE_UPLOAD` is `yes`: list S3 strategy — bucket, key pattern, accepted MIME types, max size.

### Frontend Plan
12. If `PARTIAL_SCOPE` is `backend-first`, note that frontend will be implemented in a later cycle and skip steps 13–18.
13. List new TypeScript interfaces: interface name, fields, source (which planned backend endpoint).
14. List new API service functions: function name (`verbNoun`), HTTP method, path, request/response types.
15. List Redux changes: new slice or extended slice, initial state shape, actions, store registration, persist config changes.
16. List new components: file path, props interface, which Figma screen it maps to (by screen name and node ID).
17. List new pages: file path, route path, data dependencies (which query keys).
18. List modals: file path, trigger action, form fields.
19. List `routePath.tsx` entry: key, path, routeType, headerName, element component, navigatePath if dynamic.

### Deviations
20. List every place the plan differs from existing codebase patterns found in the AUDIT REPORT.
21. For each deviation: state the existing pattern (with file path), your pattern, and why yours is better.

## Output
**IMPLEMENTATION PLAN** with these exact sections: Backend Plan, Frontend Plan, Deviations. End with: **"Plan complete. Confirm to begin implementation."**

## Done Condition
User replies **"confirmed"**. Do not proceed to CW4 until confirmation is received.

## Skills Required
- `docs/skills/17-database-migration-safety.md`
