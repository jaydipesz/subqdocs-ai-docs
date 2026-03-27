# Skill: Codebase Audit

**Canonical audit procedure:** When running inside CW2, CW2 steps supersede this skill's steps. Use this skill only for standalone audits outside the MASTER workflow.

## Purpose
Systematically identify reusable code, existing patterns, and relevant infrastructure before planning a new feature, preventing wasted effort from rebuilding what already exists and from assuming libraries without checking.

## When to Use
- Starting a new feature that touches an existing domain
- Needing to determine which validation library or form library to use
- Checking whether a model, repository, or component already covers part of a requirement

## Steps

1. Identify the target domain keyword from the feature description (e.g., "billing", "patient", "visit", "scheduling").
2. Run `grep -r "<domain>" src/sequelize/models/ --include="*.ts" -l` to find models with the domain keyword in filename or content.
3. For each model found: open the file, record table name, `paranoid` flag (check `@Table` decorator), `organization_id` column presence, and all column names with their DataTypes.
4. Open `src/sequelize/models/index.ts`. Verify each found model is in the `models` array. Flag any unregistered model as **UNREGISTERED — will silently fail**.
5. Run `grep -r "<domain>" src/sequelize/repository/ --include="*.ts" -l`. For each repository: open the file, record exported function names and their query patterns.
6. Run `grep -r "<domain>" src/modules/ --include="*.ts" -l`. Open the closest module's controller, validation, and route files.
7. Confirm the backend validation library in the codebase complies with the global `production-rules.md`.
8. Open `src/utils/enum.ts`. Search for enums whose names or values relate to this domain. Record enum name and all values.
9. Run `grep -r "<domain>" src/components/ --include="*.tsx" -l`. For each component: open the file, record the Props interface and what the component renders.
10. Search `src/redux/` or `src/store/` for slices containing the domain keyword. Record: slice name, state shape, actions.
11. Confirm the frontend form library in the codebase complies with the priorities set in the global `production-rules.md`. Record which one to use.
12. Open `src/components/common/svg/Svg.tsx`. Search for icon names that match elements in the Figma design. Record matching icon names.
13. Open `src/constants/routePath.tsx`. Search for route entries in the same domain area. Record route key, path, and routeType.
14. Compile all findings into Backend Audit and Frontend Audit sections with file paths on every line.

## Rules

- ALWAYS open actual files to confirm — never assume a library from the file name or directory structure.
- NEVER report a function or component as "reusable" without opening it and verifying the signature/props match.
- ALWAYS verify model registration in `models/index.ts` — unregistered models are the #1 silent failure.
- ALWAYS include the full file path in every finding for traceability.
- NEVER skip the adjacent-module validation/form library check — guessing the library causes mixed-library violations.

## Output

A structured AUDIT REPORT with:
1. **Backend Audit** — models (path, paranoid, org_id, columns), repositories (path, functions), modules (path, coverage), validation library (exact import), enums (name, values)
2. **Frontend Audit** — components (path, props, renders), Redux slices (path, state shape), form library (exact import), icons (names from Svg.tsx), routes (key, path, routeType)

Each finding includes its file path.

## Example

```
Backend Audit:
- Model: PatientVisit (src/sequelize/models/patient_visit.model.ts)
    paranoid: true, organization_id: yes
    Columns: id, patient_id, visit_date, visit_time, end_time, status, organization_id
    Registered in models/index.ts: ✓
- Repository: (src/sequelize/repository/patient_visit.repository.ts)
    Exports: getPatientVisitRepo, getPatientVisitsRepo, createPatientVisitRepo, updatePatientVisitRepo
- Validation library: Joi
    Import: `import Joi from 'joi'` (src/modules/patient-visit/validation_schema/patient-visit.validation.ts)
- Enum: VisitStatus = { SCHEDULED: 'Scheduled', COMPLETED: 'Completed', CANCELLED: 'Cancelled' }

Frontend Audit:
- Component: VisitCard (src/components/pages/Dashboard/VisitCard.tsx)
    Props: { visit: PatientVisit, onClick: () => void }
    Renders: card with patient name, visit date, status badge
- Form library: Formik + Yup
    Import: `import { Formik } from 'formik'` (src/components/pages/Dashboard/ScheduleVisit.tsx)
- Icon: CalendarIcon, ClockIcon — match Figma calendar and time icons
- Route: SCHEDULE_VISIT { path: '/schedule-visit', routeType: 'authenticate' }
```
