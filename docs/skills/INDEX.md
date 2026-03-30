# SubQDocs Skills Index

An agent should load a skill before acting when any trigger phrase below matches the task.

> **Matching rule:** If the user's request contains ANY of the listed trigger phrases as a substring (case-insensitive), load the corresponding skill BEFORE acting. When in doubt, load the skill — false positives (loading an unnecessary skill) are cheaper than false negatives (missing a critical checklist).

> **Depends on** means: if the operation requires creating files covered by the depended skill, load that skill too. It does NOT mean the depended skill must be loaded for every invocation.

---

| # | File | Agent trigger phrases | Depends on | What it prevents |
|---|---|---|---|---|
| # | File | Agent trigger phrases | Depends on | What it prevents |
|---|---|---|---|---|
| 01 | [01-backend-module-scaffolding.md](./01-backend-module-scaffolding.md) | "add a new backend endpoint", "create a new API route", "add a new database table", "create a new module", "new backend feature" | — | Unprotected routes, cross-tenant data access, missing model registration, hard-delete of clinical data |
| 02 | [02-frontend-route-registration.md](./02-frontend-route-registration.md) | "add a new page", "create a new route", "new frontend page", "add a view", "register a page component" | — | Auth guards bypassed, white screens from stale chunks, broken navigation when paths change |
| 03 | [03-utc-time-management.md](./03-utc-time-management.md) | "visit time", "visit_time", "end_time", "schedule a visit", "timezone", "appointment time" | 01 | Date shifting by hours for non-UTC users, filtering on the wrong column, broken end_time on reschedule |
| 04 | [04-s3-multipart-upload.md](./04-s3-multipart-upload.md) | "upload a file", "upload to S3", "store a file", "upload audio", "upload PDF", "upload attachment", "eFax upload" | 01 | Broken multipart streams, expired URLs served to users, eFax files in wrong bucket, missing `uploaded_at` |
| 05 | [05-redux-state-management.md](./05-redux-state-management.md) | "add global state", "persist state across pages", "share state between components", "add a Redux slice", "add to Redux store" | — | Previous user's PHI visible to next user via persisted state, stale server data double-stored in Redux |
| 06 | [06-realtime-socket-events.md](./06-realtime-socket-events.md) | "real-time update", "socket event", "emit event", "live status", "push notification", "Socket.io", "visit status update" | 01 | Events silently dropped due to wrong room name format, handler outside connection block, listener accumulation on re-render |
| 07 | [07-bullmq-background-jobs.md](./07-bullmq-background-jobs.md) | "background job", "queue a job", "process asynchronously", "scheduled task", "cron job", "offload processing", "BullMQ worker" | 01, 06 | Duplicate job execution from short `lockDuration`, swallowed errors marking failed jobs as success, cron not started |
| 08 | [08-transactional-email-service.md](./08-transactional-email-service.md) | "send an email", "email notification", "send invitation", "forgot password email", "notify user by email", "email template" | — | SMTP failure becoming a 500 to the user (from awaiting sendEmail), PHI in email subjects, broken links in non-production environments |
| 09 | [09-centralized-svg-icons.md](./09-centralized-svg-icons.md) | "add an icon", "new icon", "SVG icon", "add a SVG", "icon component" | — | Icons scattered across 50+ component files with no central control, hardcoded colors that break theming |
| 10 | [10-secure-patient-data-query.md](./10-secure-patient-data-query.md) | "query patient", "get patient", "fetch visit", "patient data", "fetch data", "patient list", "delete patient" | 01 | PHI cross-tenant leaks (missing org_id scope), hard-delete of patient records, raw Sequelize instances in responses |
| 11 | [11-frontend-api-integration.md](./11-frontend-api-integration.md) | "call the backend", "API call", "fetch from backend", "add a service function", "HTTP request from frontend", "add an API service" | 02, 05 | Missing auth/timezone headers from raw axios, broken multipart uploads from manual Content-Type, stale cache after mutations |
| 12 | [12-figma-design-extraction.md](./12-figma-design-extraction.md) | "extract from Figma", "Figma analysis", "read Figma design", "parse Figma file" | — | Inferred fields, missing states, skipped screens, approximate colors |
| 13 | [13-reusable-codebase-audit.md](./13-reusable-codebase-audit.md) | "audit codebase", "what exists for this domain", "reusable components", "existing models" | — | Assumed libraries, missed reusable code, unregistered models |
| 14 | [14-structured-backend-implementation.md](./14-structured-backend-implementation.md) | "implement backend", "create endpoints", "build API", "new module implementation" | 01 | Wrong implementation order, missing model registration, empty down migrations |
| 15 | [15-structured-frontend-implementation.md](./15-structured-frontend-implementation.md) | "implement frontend", "build UI", "create pages", "implement components" | 02, 05, 11 | React.lazy instead of lazyWithRetry, server state in Redux, mixed form libraries |
| 16 | [16-pixel-perfect-ui-verification.md](./16-pixel-perfect-ui-verification.md) | "verify against Figma", "check UI matches design", "compare with Figma", "UI audit" | 12 | Silent icon substitution, missed states, approximate colors, missing elements |
| 17 | [17-database-migration-safety.md](./17-database-migration-safety.md) | "alter table", "add column", "migration risk", "safe migration", "existing table" | 01 | NOT NULL without default on existing rows, empty down function, type mismatch |
| 18 | [18-standard-violation-audit.md](./18-standard-violation-audit.md) | "check for violations", "best practice audit", "code quality check", "find anti-patterns" | 13 | Replicated violations, false positives, missed console.log or PHI in logs |
| 19 | [19-prompt-engineering.md](./19-prompt-engineering.md) | "create a prompt", "add an AI prompt", "new LLM prompt", "write a system prompt", "edit a prompt", "prompt engineering" | — | Hallucinated medical data, missing anti-hallucination guards, raw placeholders in output, broken JSON schema, PHI in static prompt text |
