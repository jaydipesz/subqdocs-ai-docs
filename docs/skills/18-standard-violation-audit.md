# Skill: Violation Audit

## Purpose
Detect and log best-practice violations in existing code before building on top of it, preventing the most common replication failure: copying a pattern that works but violates project standards, then having to rewrite everything after review.

## When to Use
- Running CW2 — Pattern Audit (best practice audit section)
- Before extending an existing module — checking whether its patterns are safe to replicate
- Reviewing code quality after implementation

## Steps

### TypeScript Violations
1. Search each file for `: any` and `: unknown`. For each occurrence, check if a comment on the same or previous line explains why. Flag if no explanation exists.
2. Search for `export const` and `export function` declarations. Check if a return type annotation is present after the parameter list. Flag if missing on exported functions.
3. Search for `!` (non-null assertion operator, pattern: `variable!.` or `variable!)`). Check if the assertion could be replaced with an `if` null check or optional chaining. Flag if a safe alternative exists.
4. Search for ` as ` (type cast). Check if the cast suppresses a genuine type mismatch (e.g., `response.data as User` where response.data is `unknown`). Flag if the underlying type should be fixed instead.

### Backend Violations
5. Search for `process.env.` outside of `src/config/`. Flag every occurrence — all env access must go through `config/index.ts`.
6. Search for backtick strings containing SQL keywords (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `JOIN`, `WHERE`) and `Sequelize.literal(`. Flag in repositories and controllers — raw SQL bypasses model validation.
7. Search for `Model.findAll`, `Model.findOne`, `Model.create`, `Model.update`, `Model.destroy` in controller files. Flag every occurrence — controllers must call repositories, not Sequelize directly.
8. Open each migration file in `src/sequelize/migrations/`. Read the `down` function. Flag if the function body is empty, contains only a comment, or is missing entirely.
9. Check each model file's `@Column` decorators. Flag any column missing an explicit `DataType` argument (e.g., `@Column` without `@Column(DataType.STRING)`).
10. Search for `async` function declarations in controllers and services. Check if the function body has a `try/catch` block or if the route applies error-handling middleware. Flag if neither exists.
11. Search for `console.log`, `console.warn`, `console.error`, `console.info` across all `src/` files. Flag every occurrence — use `logger` from `@utils/logger`.
12. Search log statements (`logger.info`, `logger.error`, `logger.warn`) for PHI field names: `patient_name`, `first_name`, `last_name`, `date_of_birth`, `dob`, `ssn`, `social_security`, `diagnosis`, `medication`, `phone`, `email`, `address`. Flag any occurrence.
13. Search for `res.json(`, `res.send(`, `res.status(` in controllers. Flag if `generalResponse()` is not used instead.
14. Search controller and service files for `findOne`, `findAll`, `getAll`, `.get(` calls. For each call touching a clinical table (`patients`, `patient_visits`, and their related tables), check if `organization_id` is in the WHERE clause. Flag any missing `organization_id` as **CRITICAL — cross-tenant data access risk**.
15. Search for string literals that appear 3+ times across different files in the same module. Flag where an enum in `enum.ts` should replace them.

### Frontend Violations
16. Search for `axios.get`, `axios.post`, `axios.put`, `axios.delete`, `axios.patch`, `axios(` in `src/components/` and `src/pages/`. Flag every occurrence — must use the typed HTTP wrapper.
17. Search for `React.lazy(` across all frontend files. Flag every occurrence — must use `lazyWithRetry()`.
18. Open each Redux slice. Check if any reducer or extra reducer stores API response data (patterns: `state.data = action.payload`, `state.items = action.payload.data`). Flag — server state belongs in `useQuery`.
19. Open each page/component that renders a list or table. Check for the empty state: is there a conditional render when the array is empty? Flag if the component renders nothing or a bare empty container when data is `[]`.
20. Check for mixed form libraries: importing both `formik` and `react-hook-form` in the same component is a violation. Either is allowed independently as per Rule 11 (Primary: Formik, Secondary: RHF).
21. Search for string literals that look like route paths (`'/patient'`, `'/dashboard'`, `'/visit'`, etc.) outside of `routePath.tsx`. Flag every occurrence.
22. Search for `useEffect` where the body calls a setState with data from a `useQuery` result or API response (pattern: `useEffect(() => { setX(queryData) }, [queryData])`). Flag — this double-stores server state.
23. Search for `useMutation({` without `onError` in the options object. Flag every occurrence — mutations without error handling silently fail.
24. Check each component's Props type. Flag if typed as `any`, `Record<string, any>`, or if the component accepts props but has no explicit interface/type definition.

### Compile Results
25. For each violation found, record: file path, line number, category (TS/Backend/Frontend), violation type (short name), and a one-line description.
26. Classify each finding as either **"violation — do not replicate"** or **"pattern to follow"** based on surrounding code context.

## Rules

- NEVER skip a file identified in the audit — check every one, even if it's in a well-maintained module.
- ALWAYS include file path AND line number in every violation entry — vague locations waste debugging time.
- NEVER replicate a detected violation in new code — document it and use the correct pattern.
- ALWAYS read 5 lines of surrounding context before flagging — some patterns (e.g., `as` casts) are legitimate in specific contexts.
- NEVER report a violation without verifying it exists at the reported line — stale line numbers destroy trust in the audit.
- ALWAYS separate "violation to flag" from "existing pattern to follow" in the output — mixing them causes confusion.

## Output

A violation log with two sections:

**Violations — Do Not Replicate:**
| File | Line | Category | Violation | Description |
|------|------|----------|-----------|-------------|

**Patterns — Safe to Follow:**
| File | Line | Pattern | Description |
|------|------|---------|-------------|

## Example

```
Violations — Do Not Replicate:
| File                                              | Line | Category | Violation                | Description                                |
|---------------------------------------------------|------|----------|--------------------------|--------------------------------------------|
| src/modules/billing/controller/billing.ctrl.ts    | 42   | Backend  | console.log              | Uses console.log instead of logger         |
| src/modules/billing/controller/billing.ctrl.ts    | 58   | Backend  | generalResponse bypassed | Uses res.json() directly                   |
| src/components/pages/Billing/InvoiceList.tsx      | 12   | Frontend | React.lazy               | Should use lazyWithRetry                   |
| src/redux/slices/billingSlice.ts                  | 30   | Frontend | Server state in Redux    | state.invoices = action.payload.data       |
| src/modules/billing/controller/billing.ctrl.ts    | 15   | TS       | Untyped any              | req.body typed as any, no comment          |

Patterns — Safe to Follow:
| File                                              | Line | Pattern              | Description                              |
|---------------------------------------------------|------|----------------------|------------------------------------------|
| src/modules/billing/validation_schema/billing.ts  | 3    | Joi validation       | import Joi from 'joi' — use this library |
| src/modules/billing/controller/billing.ctrl.ts    | 71   | Error message style  | "Invoice not found" — match this phrasing|
```
