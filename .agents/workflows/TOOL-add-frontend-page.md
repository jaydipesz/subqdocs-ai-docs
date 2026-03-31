---
description: Add a new frontend page with route, API service, and optional Redux state
---

_Formats: [WORKFLOW-FORMAT.md](./WORKFLOW-FORMAT.md)._

1. Read `docs/skills/02-frontend-route-registration.md` and `docs/skills/11-frontend-api-integration.md`.

2. Ask the user for:
   - Page name (e.g. `LabResults`)
   - URL path (e.g. `/lab-results` or `/lab-results/:id`)
   - Route type: `authenticate` | `un-authenticate` | `public` | `admin`
   - Backend endpoints it will call (if any exist yet)
   - Whether it needs global Redux state

3. Create the page component at `subqdocs-frontend/src/components/pages/<PageName>/index.tsx`.

4. Add the lazy import at the top of `src/constants/routePath.tsx`:
   ```tsx
   const PageName = lazyWithRetry(() => import('../components/pages/<PageName>'));
   ```

5. Add the route entry to the `ROUTES` object in `src/constants/routePath.tsx`. Include `navigatePath` if the route has URL params.

6. If backend endpoints exist, add service functions in the appropriate `src/api/<domain>Services.ts` file using the typed wrappers from `src/api/axios.ts`.

7. If global Redux state is needed, follow `docs/skills/05-redux-state-management.md`:
   - Create the slice in `src/redux/ducks/<feature>.ts`
   - Register in `src/redux/store.ts`
   - Decide on persistence

8. **Verification:** Run `npx tsc --noEmit` in `subqdocs-frontend/`. Do not report complete until exit code 0.
// turbo
```bash
cd subqdocs-frontend && npx tsc --noEmit
```

9. Report the list of files created/modified.
