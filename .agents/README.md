# `.agents/` — Antigravity workflows and production rules

**Last reviewed:** 2026-03-31 · **Doc set:** 1

Entry point for **Antigravity-oriented** playbooks and **non-negotiable** workspace rules. Cursor users may still read these for alignment; execution is typically via Antigravity workflow triggers.

Step numbering, verification gates, and `// turbo` usage: [`workflows/WORKFLOW-FORMAT.md`](./workflows/WORKFLOW-FORMAT.md).

---

## Hard rules first

- **[`rules/production-rules.md`](./rules/production-rules.md)** — PHI, `organization_id`, Sequelize `parse`, Joi, Formik, and other production constraints referenced by implementation and review workflows.
- **[`docs/kb/project/overview.md`](../docs/kb/project/overview.md)** — Product context and PHI-related expectations; keep in sync with implementation decisions.

---

## Workflow index

| File | Trigger / use when | Primary outputs |
|------|----------------------|-----------------|
| [`WORKFLOW-FORMAT.md`](./workflows/WORKFLOW-FORMAT.md) | Editing or authoring workflows | Numbering, verification, turbo conventions |
| [`FEATURE-IMPLEMENTATION-MASTER.md`](./workflows/FEATURE-IMPLEMENTATION-MASTER.md) | End-to-end feature from Figma to verified frontend | Phased artifacts (specs → roadmap → implementation → UI sync) per workflow gates |
| [`TOOL-pr-review.md`](./workflows/TOOL-pr-review.md) | `"Review PR #[number]"` or equivalent in Antigravity | Structured PR review (findings, severity, follow-ups) |
| [`TOOL-add-backend-module.md`](./workflows/TOOL-add-backend-module.md) | New table/module from migration through route registration | Migration, model, repo, validation, controller, route, `server.ts` wiring |
| [`TOOL-add-crud-endpoint.md`](./workflows/TOOL-add-crud-endpoint.md) | New CRUD on an existing backend module | Controller + validation + route registration |
| [`TOOL-add-db-column.md`](./workflows/TOOL-add-db-column.md) | New column on an existing table | Migration + model + types |
| [`TOOL-add-file-upload.md`](./workflows/TOOL-add-file-upload.md) | File upload with S3 | Multer route + S3 upload + signed URL persistence |
| [`TOOL-add-socket-event.md`](./workflows/TOOL-add-socket-event.md) | Real-time event | Backend handler + frontend listener |
| [`TOOL-add-frontend-page.md`](./workflows/TOOL-add-frontend-page.md) | New screen | Route + API service + optional Redux |
| [`TOOL-add-background-job.md`](./workflows/TOOL-add-background-job.md) | Async work off the request path | BullMQ queue + worker + registration |
| [`TOOL-add-email-template.md`](./workflows/TOOL-add-email-template.md) | Transactional email | Handlebars template + `sendEmail` integration |
| [`CW1-figma-to-specs.md`](./workflows/CW1-figma-to-specs.md) | After Figma URL + feature description | Structured Figma analysis report |
| [`CW2-pattern-audit.md`](./workflows/CW2-pattern-audit.md) | Before large implementation | Pattern audit / gap list against codebase |
| [`CW3-implementation-roadmap.md`](./workflows/CW3-implementation-roadmap.md) | After specs + audit, before coding | Implementation plan and sequencing |
| [`CW4-backend-build.md`](./workflows/CW4-backend-build.md) | Backend phase of a specced feature | Backend implementation + API contract |
| [`CW5-frontend-build.md`](./workflows/CW5-frontend-build.md) | Frontend phase against API contract | Frontend matching design + contract |
| [`CW5A-final-ui-sync.md`](./workflows/CW5A-final-ui-sync.md) | Final polish pass | Pixel/behavior sync vs Figma |

---

## Bootstrap and context

Decision order, default bootstrap, and master-workflow steps are defined once in **[`docs/kb/ai-documentation-standards.md`](../docs/kb/ai-documentation-standards.md)** (mirror FEATURE-IMPLEMENTATION and Cursor alignment).

---

## Links

- [`docs/README.md`](../docs/README.md) — workspace documentation map and read order.
- [`docs/kb/README.md`](../docs/kb/README.md) — knowledge base hub.
- [`docs/kb/ai-documentation-standards.md`](../docs/kb/ai-documentation-standards.md) — single source for AI context, `@` paths, and bootstrap order.
- [`docs/skills/INDEX.md`](../docs/skills/INDEX.md) — procedural skills index.
