---
description: Add a new database column to an existing table (migration + model + type update)
---

1. Read `docs/skills/17-database-migration-safety.md` for the NOT NULL / defaultValue safety rules.

2. Ask the user for:
   - Table name / model name (e.g. `patient_visits` / `PatientVisit`)
   - Column name, type, nullable, default value
   - Whether it needs a frontend type update too

3. Generate the migration:
// turbo
```bash
cd subqdocs-backend && npm run migrate:create -- add-<column_name>-to-<table_name>
```

4. Edit the generated migration file in `src/sequelize/migrations/` — follow the `addColumn` pattern in skill 17 (type, allowNull, defaultValue in `up`; `removeColumn` in `down`).

5. Run the migration:
// turbo
```bash
cd subqdocs-backend && npm run migrate
```

6. Add the column to the Sequelize model in `src/sequelize/models/<table_name>.model.ts` with the matching decorator.

7. Add the field to the type interface in `src/sequelize/models/types/<table_name>.model.type.ts`. If it's optional, add `?` to the interface field. Update `RequiredAttributesType` only if the column is NOT NULL and has no default.

8. If the column is used in API responses, update the relevant frontend TypeScript interface in `subqdocs-frontend/src/types/`.

9. Validate:
// turbo
```bash
cd subqdocs-backend && npx tsc --noEmit
```

10. Report the migration file name and all files modified.
