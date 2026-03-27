---
description: Scaffold a complete backend module (migration → model → repo → validation → controller → route → server.ts registration)
---

// turbo-all

1. Read `docs/skills/01-add-backend-module.md` for the full pattern.

2. Ask the user for:
   - Table name (snake_case plural, e.g. `lab_results`)
   - Column definitions (name, type, nullable, unique)
   - Whether it needs `organization_id` scoping (almost always yes for clinical data)
   - Whether it needs `paranoid: true` (yes for any patient/clinical data)

3. Generate the migration file:
```bash
cd subqdocs-backend && npm run migrate:create -- add-<table_name>
```

4. Edit the generated migration in `src/sequelize/migrations/` — add all columns from step 2.

5. Run the migration:
```bash
cd subqdocs-backend && npm run migrate
```

6. Create the model file at `src/sequelize/models/<table_name>.model.ts`.

7. Create the type interface at `src/sequelize/models/types/<table_name>.model.type.ts`.

8. Register the model in `src/sequelize/models/index.ts` — add it to the models array.

9. Create the repository at `src/sequelize/repository/<table_name>.repository.ts`.

10. Create the validation schema at `src/modules/<module_name>/validation_schema/<module_name>.validation.ts`.

11. Create the controller at `src/modules/<module_name>/controller/<module_name>.controller.ts`.

12. Create the route file at `src/modules/<module_name>/routes/<module_name>.route.ts` — must be a factory function.

13. Register the route in `src/server.ts` — add `ModuleRoute()` to the `apiRoutes` array.

14. Validate by running:
```bash
cd subqdocs-backend && npx tsc --noEmit
```

15. Report the list of files created/modified.
