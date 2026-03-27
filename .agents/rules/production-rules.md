---
trigger: always_on
description: Mandatory rules for PHI, API, and frontend patterns.
---

# Workspace Rules

1. **[CANONICAL — referenced by Skills 01, 10]** Every query on `patients`, `patient_visits`, or any clinical table MUST include `organization_id: loggedInUser.organization_id` in the WHERE clause — omitting it leaks PHI across tenants.
2. Never return raw Sequelize model instances from a controller — call `parse(model)` from `@utils/common.utils` before passing data to `generalResponse()`.
3. Never hard-delete rows in `patients` or `patient_visits` — both use `paranoid: true`; use the repository `deleteData` function which sets `deleted_at`.
4. Never expose `password`, `otp`, `pin`, `token`, `secret_2fa`, `optum_password`, or `optum_username` in any API response, log output, socket event, or email subject.
5. Backend route files must export a factory function (`(): Router => { ... }`) — plain `Router()` exports are not picked up by `server.ts`.
6. All controller responses must go through `generalResponse(res, data, message, type, toast, status)` — never call `res.json()` or `res.send()` directly.
7. `visit_time` and `end_time` are stored as UTC `TIME` strings (`HH:MM:SS`) — always convert with `getFormattedVisitDateAndTime()` on write and `convertUTCToLocalTime()` with `x-timezone` header on read.
8. Frontend routes exist only in `src/constants/routePath.tsx` — adding a route anywhere else bypasses auth guards.
9. Frontend SVG icons exist only in `src/components/common/svg/Svg.tsx` — never inline SVGs in component files.
10. Before any operation listed in `docs/skills/INDEX.md`, load the matching skill file from `docs/skills/` by its numbered filename — it contains the project-specific checklist that prevents the most common mistakes for that operation.

# Code Quality Rules

11. **Library Standards:** ALWAYS use `Joi` for backend schema validation. For frontend forms, ALWAYS use `Formik` + `Yup` as the 1st priority standard; `react-hook-form` is the 2nd priority only if strictly necessary.
12. Never use `process.env.X` directly in application code — always access environment variables through `config/index.ts`.
13. Never use `console.log`, `console.warn`, `console.error`, or `console.info` — always use `logger` from `@utils/logger`.
14. Never include PHI (patient name, first name, last name, DOB, SSN, diagnosis, medication, phone, email, address) in any log statement. See `AI-CONTEXT.md` § PHI Fields for the complete list.
15. Frontend lazy imports must use `lazyWithRetry()` from `helper/laxywithRetry.tsx` — never use `React.lazy()` directly.
16. Server-fetched data must not be stored in Redux — use TanStack Query / `useQuery` for server state. Redux is only for client-side UI state.
17. Every `useMutation` must have an `onError` callback to handle UI state (e.g., stopping spinners) even though the global interceptor auto-toasts the error — silent mutation failures are unacceptable.
18. BullMQ job payloads must not contain `password`, `token`, `otp`, `pin`, `secret_2fa`, `optum_password`, or raw Sequelize model instances — job data is serialized to Redis.
19. Never `await sendEmail(...)` — call it fire-and-forget. Awaiting blocks the HTTP response and converts SMTP failures into 500 errors for the user.
20. Never add a `NOT NULL` column without a `defaultValue` to an existing table — existing rows will violate the constraint and crash the migration in production. Load `docs/skills/17-migration-safety.md` before writing ALTER TABLE migrations.
21. Prefer Sequelize bulk operations (`bulkCreate`, `bulkUpdate`, `bulkDestroy`, `update` with `WHERE IN`) whenever handling multiple records, and avoid iterative single-row database queries inside loops unless business logic explicitly requires per-record processing.

# Global Rules

22. Read the relevant file before editing it — never generate code based on assumptions about its current content.
23. Run the build or type-check after structural changes to catch errors before reporting success.
24. When a task touches multiple files, make all edits before testing — partial edits produce misleading errors.
25. If a task is ambiguous, ask one clarifying question rather than guessing and producing wrong output.