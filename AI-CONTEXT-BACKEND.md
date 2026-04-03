# SubQDocs — Backend Context

> **Parent:** [`AI-CONTEXT.md`](AI-CONTEXT.md) — Architecture, Domain Rules, Never Do  
> **Skills:** [`docs/skills/INDEX.md`](docs/skills/INDEX.md)

---

# Stack

Node.js + Express + TypeScript + Sequelize (PostgreSQL). See `subqdocs-backend/package.json` for full versions.

Key libraries and why they matter:

| Library | Why it matters to an agent |
|---|---|
| `sequelize-typescript` | Decorator-based models — `@Table`, `@Column`, `@ForeignKey` |
| `bullmq` + `redis` | Background job processing — see [`docs/skills/07-add-background-job.md`](docs/skills/07-add-background-job.md) |
| `@langchain/langgraph` | AI agent state graph orchestration |
| `@deepgram/sdk` | Audio transcription |
| `multer` | File upload — routes need body-parser bypass in `app.ts` |
| `node-cron` | Scheduled tasks — must call `.start()` in `server.ts` |
| `joi` | Request validation (see Rule 11) |

---

# Convention-Critical Paths

```
subqdocs-backend/src/
├── server.ts                      # Route registration + cron starts + worker init
├── app.ts                         # Express setup, CORS, body parsers, Multer bypass list
├── socket.ts                      # ALL socket.io logic — handlers go here
├── sequelize/models/              # One file per DB table (122 models)
├── sequelize/models/index.ts      # Model registration — new models MUST be added here
├── sequelize/migrations/          # Sequelize CLI migrations (345 files)
├── sequelize/repository/          # Query wrappers: getXRepo(), findAllX()
├── modules/<name>/                # Feature modules — route/controller/validation/service/jobs
├── common/utils/generalResponse   # THE response helper (see Rule 6)
├── common/utils/s3/s3.ts          # S3 upload functions — import from here
├── middlewares/auth.middleware.ts  # JWT decode → req.user
└── config/index.ts                # ENV variable exports
```

---

# Conventions

## Module Structure

Every module under `src/modules/<name>/` follows:
```
routes/<name>.route.ts        # Factory function (see Rule 5)
controller/<name>.controller.ts
validation_schema/<name>.validation.ts
services/<name>.service.ts    # (optional) extracted business logic
jobs/<name>.cron.ts           # (optional) cron job
```

## Models (`src/sequelize/models/`)
- One file per table: `snake_case.model.ts` → `class PascalCaseModel`
- Registry: New models **MUST** be registered in `models/index.ts` — without this, queries silently fail.

---

# Transaction Strategy

To ensure data integrity as per Rule 27, follow this managed transaction pattern:

1.  **Initiation**: Always prefer **Managed Transactions** in Services (or Controllers for simple tasks).
    ```ts
    await sequelize.transaction(async (transaction) => {
        // use transaction object in repository calls
        await repo.create({ ... }, { transaction });
    });
    ```
2.  **Passing**: Always pass the `transaction` object in the options `{ transaction: t }` to repositories.

# Fast-Track Update Pattern

To reduce database and logging overhead:
- Before calling `repository.update(...)`, perform a shallow comparison between the incoming data and the existing database record.
- **Rule**: If the data hasn't changed, return early with the original record without hitting the database.

## Repositories
- Pure functions wrapping Sequelize queries — keeps controllers clean
- Not mandatory for all models but preferred for reused queries

## Migrations
- Generate: `npm run migrate:create -- <name>`
- Run: `npm run migrate`
- Undo last: `npm run migrate:undo`

## Validation
- Primary: `joi` (see Rule 11) via `validationMiddleware`
- Some modules use `express-validator` or `zod` — match what adjacent modules use

## Logging
- Always follow Rule 13 (see `@utils/logger`)
- ⚠️ Body logger in `app.ts` currently logs `JSON.stringify(body)` for non-webhook requests — PHI risk

## Error Handling
- Follow Rules 2 & 6: use `parse(model)` and `generalResponse`
- Sentry notified on every caught exception

## Body Parsing Exceptions
- **Webhook routes** (`/webhooks/stripe`, `/efax/webhook/westfax`): skip all body parsing — need raw body for signature verification
- **Multer routes**: skip `express-form-data` in `app.ts` — Multer handles its own stream

## Audit Logging
- Table: `audit_logs` — actions: `DELETE`, `AUTO_DELETION` — entity types: `NOTE`, `TRANSCRIPT`
- Auto-deletion cron: `autoDeleteTranscriptsCronJob` in `src/modules/patient-transcript/jobs/`
