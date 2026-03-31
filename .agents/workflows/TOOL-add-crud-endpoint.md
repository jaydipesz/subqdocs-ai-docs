---
description: Add a new CRUD endpoint to an existing backend module (controller + validation + route registration)
---

1. Read `docs/skills/01-backend-module-scaffolding.md` (steps 6–8) and `docs/skills/10-secure-patient-data-query.md`.

2. Ask the user for:
   - Which existing module (e.g. `patient`, `patient_visit`, `prescription`)
   - HTTP method and path (e.g. `GET /patient/:patient_id/lab-results`)
   - Request body/query fields and their validation rules
   - Whether `organizationMemberMiddleware` is needed (yes if path has `patient_id` or `visit_id`)

3. Add the Joi/express-validator schema to the module's `validation_schema/` file.

4. Add the controller function in the module's `controller/` file:
   - Wrap in try/catch
   - Scope queries by `organization_id: loggedInUser.organization_id`
   - Call `parse(result)` before returning
   - Return via `generalResponse()`

5. Add the route in the module's `routes/` file inside the existing factory function:
   - Apply middleware chain: `authMiddleware`, `organizationMemberMiddleware` (if needed), `trackDeviceLog`, `userActivityMiddleware`, `validationMiddleware(schema, source)`

6. Add the corresponding frontend service function in `subqdocs-frontend/src/api/<domain>Services.ts` using the typed axios wrapper. Follow `docs/skills/11-frontend-api-integration.md`.

7. Validate:
// turbo
```bash
cd subqdocs-backend && npx tsc --noEmit
```

8. Report the endpoint path and the files modified.
