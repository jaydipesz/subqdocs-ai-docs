# SubQDocs — Backend conventions

> **Parent:** [`../project/overview.md`](../project/overview.md) — Product, architecture, domain rules, never-do  
> **Skills:** [`../../skills/INDEX.md`](../../skills/INDEX.md)

## Knowledge base (`subqdocs-backend`)

Distributed KB (hub, `src/common`, `src/sequelize`, priority module deep docs): **[`../subqdocs-backend/INDEX.md`](../subqdocs-backend/INDEX.md)**

---

# Stack

Node.js + Express + TypeScript + Sequelize (PostgreSQL). See `subqdocs-backend/package.json` for full versions.

Key libraries and why they matter:

| Library | Why it matters to an agent |
|---|---|
| `sequelize-typescript` | Decorator-based models — `@Table`, `@Column`, `@ForeignKey` |
| `bullmq` + `redis` | Background job processing — see [`../../skills/07-bullmq-background-jobs.md`](../../skills/07-bullmq-background-jobs.md) |
| `@langchain/langgraph` | AI agent state graph orchestration |
| `@deepgram/sdk` | Audio transcription |
| `multer` | File upload — routes need body-parser bypass in `app.ts` |
| `node-cron` | Scheduled tasks — must call `.start()` in `server.ts` |
| `joi` | Primary request validation (some modules use `express-validator` or `zod`) |

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
├── common/utils/generalResponse   # THE response helper — all responses go through this
├── common/utils/s3/s3.ts          # S3 upload functions — import from here
├── middlewares/auth.middleware.ts  # JWT decode → req.user
└── config/index.ts                # ENV variable exports
```

---

# Conventions

## Module Structure

Every module under `src/modules/<name>/` follows:
```
routes/<name>.route.ts        # Factory function returning Router
controller/<name>.controller.ts
validation_schema/<name>.validation.ts
services/<name>.service.ts    # (optional) extracted business logic
jobs/<name>.cron.ts           # (optional) cron job
```

## Models (`src/sequelize/models/`)
- One file per table: `snake_case.model.ts` → `class PascalCaseModel`
- `@Table({ paranoid: true })` on patient/clinical data (soft delete)
- Type interfaces in `models/types/<model>.model.type.ts`
- Enums in `src/common/utils/enum.ts`
- Password hashing: Argon2 in `@BeforeCreate/@BeforeUpdate` hook on User model
- **New models must be registered in `models/index.ts`** — without this, queries silently fail

## Repositories
- Pure functions wrapping Sequelize queries — keeps controllers clean
- Not mandatory for all models but preferred for reused queries

## Migrations
- Generate: `npm run migrate:create -- <name>`
- Run: `npm run migrate`
- Undo last: `npm run migrate:undo`

## Validation
- Most modules: `joi` via `validationMiddleware(schema, 'body'|'query'|'params')`
- Some modules use `express-validator` or `zod` — match what adjacent modules use

## Logging
- Via `logger` from `@utils/logger` (Winston with daily rotation)
- **LLM (`invokeEngine` + `conditionallyCallLLMEngine`):** `invokeEngine` logs stream/API failures at **`logger.debug`** / **`logger.warn`** (not `error`) so Sentry is not flooded. **`logger.error`** for LLM retry flows is reserved for a **single** terminal line: `LLM orchestrator: all retries exhausted — terminal failure` in `conditionallyCallLLMEngine` when the retry loop gives up; per-attempt issues use **`logger.warn`** in the orchestrator catch.
- **Stale Sentry issues** from old `[invokeEngine]` / duplicate timeout messages: after deploy, bulk-resolve with `subqdocs-backend/scripts/resolve-sentry-llm-log-noise-issues.sh` (requires `SENTRY_AUTH_TOKEN` with `event:write`; see script header). Or multi-select and **Resolve** in the Sentry UI.
- ⚠️ Body logger in `app.ts` currently logs `JSON.stringify(body)` for non-webhook requests — PHI risk

## Error Handling
- Throw `new HttpException(status, message, data, toast)` from controllers
- `error.middleware.ts` catches and calls `generalResponse`
- Sentry notified on every caught exception

## Body Parsing Exceptions
- **Webhook routes** (`/webhooks/stripe`, `/efax/webhook/westfax`): skip all body parsing — need raw body for signature verification
- **Multer routes**: skip `express-form-data` in `app.ts` — Multer handles its own stream

## Audit Logging
- Table: `audit_logs` — actions: `DELETE`, `AUTO_DELETION` — entity types: `NOTE`, `TRANSCRIPT`
- Auto-deletion cron: `autoDeleteTranscriptsCronJob` in `src/modules/patient-transcript/jobs/`
