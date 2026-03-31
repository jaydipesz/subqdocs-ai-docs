---
description: Add a new database column to an existing table (migration + model + type update)
---

_Formats: [WORKFLOW-FORMAT.md](./WORKFLOW-FORMAT.md)._

1. Ask the user for:
   - Table name / model name (e.g. `patient_visits` / `PatientVisit`)
   - Column name, type, nullable, default value
   - Whether it needs a frontend type update too

2. Read `docs/skills/17-database-migration-safety.md` for NOT NULL, defaults, down migrations, and risky patterns before writing SQL.

3. Generate the migration:
// turbo
```bash
cd subqdocs-backend && npm run migrate:create -- add-<column_name>-to-<table_name>
```

4. Edit the generated migration file in `src/sequelize/migrations/`:
   ```js
   module.exports = {
     up: async (queryInterface, Sequelize) => {
       await queryInterface.addColumn('<table_name>', '<column_name>', {
         type: Sequelize.<TYPE>,
         allowNull: <true|false>,
         defaultValue: <value|null>,
       });
     },
     down: async (queryInterface) => {
       await queryInterface.removeColumn('<table_name>', '<column_name>');
     },
   };
   ```

5. Run the migration:
// turbo
```bash
cd subqdocs-backend && npm run migrate
```

6. Add the column to the Sequelize model in `src/sequelize/models/<table_name>.model.ts` with the matching decorator.

7. Add the field to the type interface in `src/sequelize/models/types/<table_name>.model.type.ts`. If it's optional, add `?` to the interface field. Update `RequiredAttributesType` only if the column is NOT NULL and has no default.

8. If the column is used in API responses, update the relevant frontend TypeScript interface in `subqdocs-frontend/src/types/`.

9. **Verification:** Run `npx tsc --noEmit` in `subqdocs-backend/`. Do not report complete until exit code 0.
// turbo
```bash
cd subqdocs-backend && npx tsc --noEmit
```

10. **Verification:** If `subqdocs-frontend/` types were updated in step 8, run `npx tsc --noEmit` in `subqdocs-frontend/` until exit code 0.

11. Report the migration file name and all files modified.
