# Skill: Backend Implementation

**Prerequisite:** Load skill `01-backend-module-scaffolding` first. This skill adds orchestration-level concerns; skill 01 provides the file-by-file reference code.

## Purpose
Ensure backend features are implemented in the correct dependency order with build verification, preventing the most common integration failures: unregistered models, missing migrations, broken type contracts.

## When to Use
- Implementing a new backend module with multiple files (model, repo, controller, route)
- Running CW4 — Backend Build
- Adding 3+ related endpoints to an existing module

## Steps

1. Open the implementation plan provided as input. For each model listed, follow skill `01-backend-module-scaffolding` steps 1–9 in order.
2. Before creating any migration, run the `17-database-migration-safety` skill against the planned column changes.
3. After creating all migrations, run `npm run migrate`. If any migration fails, fix it before creating repositories.
4. After creating all repositories, run `npx tsc --noEmit`. Fix type errors before creating controllers.
5. For validation schemas, use the standard library mandated by the global `production-rules.md`. Use the established import pattern from adjacent endpoints if needed.
6. For services, extract any logic that involves more than one repository call or conditional branching. Controllers should have at most: parse input → call service/repo → return `generalResponse()`.
7. For error messages, open 2–3 adjacent controllers in the same domain. Copy the phrasing pattern (e.g., "Patient not found" vs "No patient found with this ID").
8. After all files are created, run `npx tsc --noEmit` again. Zero errors required.
9. Produce the API CONTRACT DOCUMENT. For each endpoint list: method, path, middleware chain, request schema (field + type + required), response shape (field + type), error cases (condition + HTTP status + message).

## Rules

- NEVER skip the migration-safety check on altered tables — a broken migration blocks the entire deployment pipeline.
- ALWAYS run `tsc --noEmit` after completing all files — partial type checking produces false passes.
- ALWAYS delegate individual file creation to skill `01-backend-module-scaffolding` — do not reinvent patterns.
- NEVER write a controller that directly imports a Sequelize model — always go through a repository.
- ALWAYS match error message phrasing to adjacent modules — inconsistent messages confuse the frontend team.
- NEVER proceed past a failed migration — fix it before creating dependent code.

## Selective Sequelize Transaction Usage

When implementing database logic using Sequelize:

Use managed transactions ONLY when business integrity depends on multiple writes succeeding together OR concurrent updates may cause incorrect state.

**Apply transactions when:**
- multiple dependent inserts/updates/deletes occur
- booking, wallet, inventory, counters, or status transitions exist
- cascading updates must stay consistent
- a record is read before being modified (read → validate → update flow)

**Apply row-level locking ONLY when:**
- preventing duplicate updates from concurrent requests
- preventing double booking / double spending
- modifying stock, counters, or availability
- updating the same row after validation logic

**Example safe pattern:**
```javascript
await sequelize.transaction(async (t) => {
  const record = await Model.findOne({
    where: { /* ... */ },
    transaction: t,
    lock: t.LOCK.UPDATE
  });

  // validate state

  await record.update({ /* ... */ }, { transaction: t });
});
```

**Do NOT use transactions for:**
- simple SELECT queries
- single independent UPDATE/INSERT
- logging or analytics writes
- non-critical background writes

Prefer managed transactions over manual commit/rollback.

## Sequelize Query & Include Optimizations

Always select only required attributes in Sequelize queries.

**Avoid:**
```javascript
Model.findAll()
```

**Prefer:**
```javascript
Model.findAll({
  attributes: ['id', 'name', 'status'] // Specify required fields
})
```

**When using `include` relations:**
- ALWAYS specify `attributes` inside `include`
- AVOID nested includes unless strictly required
- AVOID eager loading large relations in list endpoints

## Transaction Context Propagation

Ensure transaction context propagation across repository/service layers.

If a transaction object exists:
- pass `{ transaction: t }` to every Sequelize query (including within `findOne`, `update`, `create`, etc.)
- forward the transaction through helper/repository calls
- NEVER start nested transactions unless explicitly required

**Pattern:**
```typescript
async function repoMethod(payload: any, options?: { transaction?: Transaction }) {
  // Always reuse existing transaction when provided
  return await Model.create(payload, { 
    transaction: options?.transaction 
  });
}
```

## Output

Working backend module with all files following skill 01 patterns, plus an API CONTRACT DOCUMENT:

## Example

```
API CONTRACT DOCUMENT

POST /api/billing/invoice
  Auth: authMiddleware, organizationMemberMiddleware, trackDeviceLog, userActivityMiddleware
  Validation: validationMiddleware(createInvoiceSchema, 'body')
  Body: { patient_id: number (required), items: Array<{ description: string, amount: number }> (required) }
  Response: { id: number, patient_id: number, total: number, status: 'draft', created_at: string }
  Errors:
    - Patient not found → 404
    - Validation failed → 400
    - Unauthorized → 401

GET /api/billing/invoices
  Auth: authMiddleware
  Query: { patient_id?: number, status?: 'draft'|'sent'|'paid', page?: number, limit?: number }
  Response: { rows: Invoice[], count: number }
  Errors:
    - Unauthorized → 401
```

---

## Testing Conventions

### Test file location and naming
- Backend test files go alongside the module: `src/modules/<module>/__tests__/<module>.test.ts`
- Name test files to match what they test: `patient.controller.test.ts`, `invoice.service.test.ts`
- Shared test utilities go in `src/__tests__/helpers/`

### What to test
| Layer | What to test | What NOT to test |
|-------|-------------|-----------------|
| Controller | HTTP status codes, response shape, validation rejection, auth guards | Internal Sequelize behavior |
| Service | Business logic, multi-step operations, edge cases | Framework internals |
| Repository | Complex queries with filters, pagination, soft-delete behavior | Simple CRUD wrappers |

### Test rules
- NEVER use production database credentials in tests — use a separate test database or mock the repository layer
- NEVER include real PHI in test fixtures — use realistic but fake data (e.g., "Jane Doe", DOB "1990-01-01")
- ALWAYS clean up created test data in `afterEach` or `afterAll` hooks
- ALWAYS mock external services (email, S3, Stripe) — never make real API calls in tests
- ALWAYS scope test queries by a test-specific `organization_id` to prevent pollution
