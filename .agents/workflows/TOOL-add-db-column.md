---
description: Add a new database column to an existing table (migration + model + type update)
---

1. Ask the user for:
   - Table name / model name (e.g. `patient_visits` / `PatientVisit`)
   - Column name, type, nullable, default value
   - Whether it needs a frontend type update too

2. Generate the migration:
// turbo
```bash
cd subqdocs-backend && npm run migrate:create -- add-<column_name>-to-<table_name>
```

3. Edit the generated migration file in `src/sequelize/migrations/`:
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

4. Run the migration:
// turbo
```bash
cd subqdocs-backend && npm run migrate
```

5. Add the column to the Sequelize model in `src/sequelize/models/<table_name>.model.ts` with the matching decorator.

6. Add the field to the type interface in `src/sequelize/models/types/<table_name>.model.type.ts`. If it's optional, add `?` to the interface field. Update `RequiredAttributesType` only if the column is NOT NULL and has no default.

7. If the column is used in API responses, update the relevant frontend TypeScript interface in `subqdocs-frontend/src/types/`.

8. Validate:
// turbo
```bash
cd subqdocs-backend && npx tsc --noEmit
```

9. Report the migration file name and all files modified.
