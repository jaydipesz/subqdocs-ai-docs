# SubQDocs Skills Index

An agent should load a skill before acting when any trigger phrase below matches the task.

> **Matching rule:** If the user's request contains ANY of the listed trigger phrases as a substring (case-insensitive), load the corresponding skill BEFORE acting. When in doubt, load the skill — false positives (loading an unnecessary skill) are cheaper than false negatives (missing a critical checklist).

> **Depends on** means: if the operation requires creating files covered by the depended skill, load that skill too. It does NOT mean the depended skill must be loaded for every invocation.

---

| # | File | Agent trigger phrases | Depends on | What it prevents |
|---|---|---|---|---|
| 01 | [add-backend-module.md](./01-add-backend-module.md) | "add a new backend endpoint", "create a new API route", "add a new database table", "create a new module", "new backend feature" | — | Unprotected routes, cross-tenant data access, missing model registration, hard-delete of clinical data |
| 02 | [add-frontend-route.md](./02-add-frontend-route.md) | "add a new page", "create a new route", "new frontend page", "add a view", "register a page component" | — | Auth guards bypassed, white screens from stale chunks, broken navigation when paths change |
| 03 | [visit-time-handling.md](./03-visit-time-handling.md) | "visit time", "visit_time", "end_time", "schedule a visit", "timezone", "appointment time" | 01 | Date shifting by hours for non-UTC users, filtering on the wrong column, broken end_time on reschedule |
| 04 | [s3-file-upload.md](./04-s3-file-upload.md) | "upload a file", "upload to S3", "store a file", "upload audio", "upload PDF", "upload attachment", "eFax upload" | 01 | Broken multipart streams, expired URLs served to users, eFax files in wrong bucket, missing `uploaded_at` |
| 05 | [add-redux-state.md](./05-add-redux-state.md) | "add global state", "persist state across pages", "share state between components", "add a Redux slice", "add to Redux store" | — | Previous user's PHI visible to next user via persisted state, stale server data double-stored in Redux |
| 06 | [socket-events.md](./06-socket-events.md) | "real-time update", "socket event", "emit event", "live status", "push notification", "Socket.io", "visit status update" | 01 | Events silently dropped due to wrong room name format, handler outside connection block, listener accumulation on re-render |
| 07 | [add-background-job.md](./07-add-background-job.md) | "background job", "queue a job", "process asynchronously", "scheduled task", "cron job", "offload processing", "BullMQ worker" | 01, 06 | Duplicate job execution from short `lockDuration`, swallowed errors marking failed jobs as success, cron not started |
| 08 | [send-email.md](./08-send-email.md) | "send an email", "email notification", "send invitation", "forgot password email", "notify user by email", "email template" | — | SMTP failure becoming a 500 to the user (from awaiting sendEmail), PHI in email subjects, broken links in non-production environments |
| 09 | [add-svg-icon.md](./09-add-svg-icon.md) | "add an icon", "new icon", "SVG icon", "add a SVG", "icon component" | — | Icons scattered across 50+ component files with no central control, hardcoded colors that break theming |
| 10 | [query-patient-data.md](./10-query-patient-data.md) | "query patient", "get patient", "fetch visit", "patient data", "filter patients", "patient list", "delete patient" | 01 | PHI cross-tenant leaks (missing org_id scope), hard-delete of patient records, raw Sequelize instances in responses |
| 11 | [add-api-service-call.md](./11-add-api-service-call.md) | "call the backend", "API call", "fetch from backend", "add a service function", "HTTP request from frontend", "add an API service" | 02, 05 | Missing auth/timezone headers from raw axios, broken multipart uploads from manual Content-Type, stale cache after mutations |
| 12 | [figma-extraction.md](./12-figma-extraction.md) | "extract from Figma", "Figma analysis", "read Figma design", "parse Figma file" | — | Inferred fields, missing states, skipped screens, approximate colors |
| 13 | [codebase-audit.md](./13-codebase-audit.md) | "audit codebase", "what exists for this domain", "reusable components", "existing models" | — | Assumed libraries, missed reusable code, unregistered models |
| 14 | [backend-implementation.md](./14-backend-implementation.md) | "implement backend", "create endpoints", "build API", "new module implementation" | 01 | Wrong implementation order, missing model registration, empty down migrations |
| 15 | [frontend-implementation.md](./15-frontend-implementation.md) | "implement frontend", "build UI", "create pages", "implement components" | 02, 05, 11 | React.lazy instead of lazyWithRetry, server state in Redux, mixed form libraries |
| 16 | [ui-verification.md](./16-ui-verification.md) | "verify against Figma", "check UI matches design", "compare with Figma", "UI audit" | 12 | Silent icon substitution, missed states, approximate colors, missing elements |
| 17 | [migration-safety.md](./17-migration-safety.md) | "alter table", "add column", "migration risk", "safe migration", "existing table" | 01 | NOT NULL without default on existing rows, empty down function, type mismatch |
| 18 | [violation-audit.md](./18-violation-audit.md) | "check for violations", "best practice audit", "code quality check", "find anti-patterns" | 13 | Replicated violations, false positives, missed console.log or PHI in logs |
