# Skill: Migration Safety

## Purpose
Prevent data loss, partial state failures, and deployment issues caused by unsafe database migrations on populated tables. This skill ensures migrations are robust, fully reversible, and safe to execute in a production environment.

## When to Use
- Writing a new database migration
- Altering an existing table (adding, renaming, modifying, or dropping columns)
- Planning migrations during CW3 — Implementation Roadmap
- Reviewing a migration before execution or merging during CW4 — Backend Build

## Steps

1. Open the migration file. Determine if `up` creates a new table (`createTable`) or alters an existing one (`addColumn`, `changeColumn`, `removeColumn`, `renameColumn`).
2. If **new table**: verify every column has an explicit `type`. Verify `id` is `autoIncrement` + `primaryKey`. Verify `created_at`, `updated_at`, and `deleted_at` (if using paranoia/soft-deletes) are appropriately included.
3. If **altering an existing table** — for each column being added:
   a. Read the `allowNull` value. If `false` (NOT NULL), read the `defaultValue`.
   b. If `allowNull: false` AND no `defaultValue` is set: **flag as MIGRATION RISK**. Existing rows cannot satisfy this constraint and the migration will crash on deployment.
   c. **Fix:** Either set `allowNull: true` or provide a `defaultValue` that is valid for existing data.
4. If **renaming a column**: search `src/sequelize/` (repositories, models, etc.) for the old column name. Every single reference must be updated in the same commit.
5. If **dropping a column**: search the entire `src/` directory for references to the column name. Ensure no application code relies on it. If references exist, the migration will break runtime queries. Fix the code prior to or alongside the migration.
6. **Transactions for Multiple Operations:** If `up` or `down` performs multiple operations (e.g., adding several columns, updating multiple tables), wrap them in a **managed transaction**. This guarantees that if one operation fails, the entire migration rolls back, preventing partial schema states.
7. Verify the `down` function:
   a. If `up` adds columns → `down` must `removeColumn` each one.
   b. If `up` creates a table → `down` must `dropTable`.
   c. If `up` adds an ENUM type → `down` must drop the ENUM type *after* removing the relevant columns (`DROP TYPE IF EXISTS "enum_<table>_<column>"`).
   d. If `down` is empty or only contains a comment: **flag as ROLLBACK BLOCKED**. This migration cannot be reversed safely.
8. Verify every column `type` in the migration matches the model definition:
   - `DataType.STRING(255)` in model → `Sequelize.STRING(255)` in migration (never use `Sequelize.STRING` without a length if defined in the model).
   - `DataType.INTEGER` in model → `Sequelize.INTEGER` in migration.
   - `DataType.ENUM(...)` values must match exactly between the model and the migration.
9. Test the migration:
   - Run `npm run migrate` in your local development environment. If it fails, read the error, fix the migration, and re-run.
   - Run `npm run migrate:undo` to verify that the `down` function works seamlessly.
   - Re-run `npm run migrate` to restore the state.

## Rules

- NEVER add a `NOT NULL` column without a `defaultValue` to a table that may have existing rows — this will cause a deployment crash.
- ALWAYS use a transaction when performing multiple operations in `up` or `down` to prevent partial migrations.
- ALWAYS write a complete `down` function — an empty `down` function breaks the ability to rollback a release.
- ALWAYS match migration column types to model column types exactly — mismatches can cause subsequent silent data truncation or validation errors.
- NEVER drop or rename a column without searching for and removing code references first — this causes runtime query failures in production.
- ALWAYS test both `migrate` and `migrate:undo` — a `down` function that runs but doesn't fully reverse changes causes state drift.
- NEVER add an ENUM column without subsequently dropping the ENUM type in the `down` function — leftover types block future migration re-runs.

## Output

A safe migration file containing:
- New columns on existing tables strictly defined with `allowNull: true` or a valid `defaultValue`.
- All multiple operations wrapped securely in a managed transaction.
- A complete `down` function that fully and cleanly reverses `up`.
- Column types that represent an exact match to the model.
- ENUM cleanup in `down` (if applicable).

## Examples

### Single Operation Example
```javascript
// SAFE — single operation, nullable column on existing table with ENUM
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn('patient_visits', 'billing_status', {
      type: Sequelize.ENUM('pending', 'billed', 'paid'),
      allowNull: true,
    });
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.removeColumn('patient_visits', 'billing_status');
    await queryInterface.sequelize.query('DROP TYPE IF EXISTS "enum_patient_visits_billing_status"');
  },
};
```

### Multiple Operations Example (Managed Transaction Required)
```javascript
// SAFE — multiple operations wrapped in a transaction
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.sequelize.transaction(async (transaction) => {
      await queryInterface.addColumn('patient_visits', 'insurance_provider', {
        type: Sequelize.STRING(255),
        allowNull: true,
      }, { transaction });

      await queryInterface.addColumn('patient_visits', 'copay_amount', {
        type: Sequelize.DECIMAL(10, 2),
        allowNull: false,
        defaultValue: 0.00,
      }, { transaction });
    });
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.sequelize.transaction(async (transaction) => {
      await queryInterface.removeColumn('patient_visits', 'insurance_provider', { transaction });
      await queryInterface.removeColumn('patient_visits', 'copay_amount', { transaction });
    });
  },
};
```

### Unsafe Example (DO NOT USE)
```javascript
// UNSAFE — will crash on deploy if patient_visits has existing rows, lack of transaction, and missing rollback
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn('patient_visits', 'billing_status', {
      type: Sequelize.ENUM('pending', 'billed', 'paid'),
      allowNull: false,  // ← MIGRATION RISK: no defaultValue, existing rows will fail
    });
  },
  down: async () => {
    // ← ROLLBACK BLOCKED: empty down function
  },
};
```
