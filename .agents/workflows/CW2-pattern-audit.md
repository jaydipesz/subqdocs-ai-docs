---
description: CW2 — Audit the codebase for reusable patterns and best-practice violations.
---

_Formats: [WORKFLOW-FORMAT.md](./WORKFLOW-FORMAT.md)._

# CW2 — Codebase Audit

## Purpose
Identify reusable code, existing patterns, and best-practice violations in the files this feature will touch, before any planning begins.

## Standalone Inputs
- **FIGMA ANALYSIS REPORT** — output of CW1
- `DOMAIN` — backend domain (e.g., billing, scheduling, patient)
- `SCOPE` — `new-module` or `extend:<module_name>`
- `RELATED_TABLES` — existing DB tables this feature relates to (if provided)
- `AUTH_LEVEL` — `all-staff` / `admin-only` / `patient-facing` / `dual-auth`
- `BUSINESS_RULES` — rules not visible in Figma (if provided)

## Trigger Condition
After CW1 is confirmed by the user.

## Steps

### Backend Audit
1. If `SCOPE` is `extend:<module_name>`, open `src/modules/<module_name>/` first. If `SCOPE` is `new-module`, skip to step 2.
2. Search `src/sequelize/models/` for models whose columns overlap with fields in the FIGMA ANALYSIS REPORT. If `RELATED_TABLES` was provided, also open those models directly. For each match, record: file path, table name, `paranoid` flag, `organization_id` presence.
3. Search `src/sequelize/repository/` for repository functions that query those models. Record: file path, function name, what it queries.
4. Search `src/modules/` for modules in the `DOMAIN`. Open the controller, route, and validation files. Record: file path, what each covers.
5. Open the validation file of the closest module found in step 4. Record the exact library used (Joi, Zod, express-validator) and the import statement.
6. Open `src/utils/enum.ts`. List every enum semantically relevant to this feature by name and values.
7. For each model found in step 2, confirm it is registered in `src/sequelize/models/index.ts`. Flag any unregistered model.
8. Record `AUTH_LEVEL` and determine the middleware chain: `all-staff` → `authMiddleware`, `admin-only` → `authMiddleware` + admin check, `patient-facing` → `authOrIntakeSessionMiddleware`, `dual-auth` → `authOrIntakeSessionMiddleware`.
9. If `BUSINESS_RULES` was provided, list each rule and map it to the backend component that will enforce it (validation, controller logic, cron, or socket event).

### Frontend Audit
10. Search `src/components/` for components whose names or rendered elements match UI patterns in the FIGMA ANALYSIS REPORT. Record: file path, props interface, what it renders.
11. Search `src/redux/` or `src/store/` for slices in the `DOMAIN`. Record: file path, slice name, what state it holds.
12. Open the form component of the closest adjacent feature. Record the exact form library used (Formik, React Hook Form) and its import statement.
13. Open `src/components/common/svg/Svg.tsx`. List every icon whose name or shape matches icons in the FIGMA ANALYSIS REPORT.
14. Open `src/constants/routePath.tsx`. List every route entry in the same domain area.

### Best Practice Audit
15. For each file found in steps 1–14, run the `violation-audit` skill (`docs/skills/18-standard-violation-audit.md`). Limit to: only the TypeScript violations (steps 1–4) and the category-specific violations (Backend steps 5–14 for backend files, Frontend steps 15–23 for frontend files). Do not cross-apply categories.
16. Log every violation found using the violation-audit output format: file, line, category, violation type, description.
17. Do not replicate any logged violation in new code.

18. **Verification:** Confirm the **AUDIT REPORT** has all three sections (Backend Audit, Frontend Audit, Violation Log) with file paths where applicable. Do not produce **Output** until complete.

## Output
**AUDIT REPORT** with these exact sections:
1. Backend Audit — models, repositories, modules, validation library, enums, paranoid/org_id status (each with file path)
2. Frontend Audit — components, Redux slices, form library, icons, routes (each with file path)
3. Violation Log — table of all violations found (file, line, category, violation type, description)

## Done Condition
Audit report is complete. User replies **"confirmed"**. Do not proceed to CW3 until confirmation is received.

## Skills Required
- `docs/skills/13-reusable-codebase-audit.md`
- `docs/skills/18-standard-violation-audit.md`
