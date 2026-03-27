# Skill: Add a Frontend Route

**When NOT to use this:** Adding a modal, drawer, or tab inside an existing page (no route needed). Adding a redirect — use `navigate()` instead.

---

## Files involved
```
subqdocs-frontend/src/
  constants/routePath.tsx          ← only file where routes are registered
  helper/laxywithRetry.tsx         ← use this, not React.lazy()
  components/pages/<Feature>/index.tsx
```

---

## Step 1 — Lazy-import the page component

At the top of `src/constants/routePath.tsx`, alongside the other lazy imports:

```tsx
import { lazyWithRetry } from '../helper/laxywithRetry';

const MyFeaturePage = lazyWithRetry(() => import('../components/pages/MyFeature'));
```

Do **not** use `React.lazy()` directly. `lazyWithRetry` adds automatic retry on chunk-load failure, preventing white screens when a new bundle is deployed.

---

## Step 2 — Add the route entry

Inside the `ROUTES` object in `src/constants/routePath.tsx`:

```tsx
MY_FEATURE: {
    path: '/my-feature',
    routeType: 'authenticate',
    headerName: 'My Feature',
    breadCrumb: 'My Feature',
    showHeader: true,
    showFooter: false,
    element: <MyFeaturePage />,
},
```

For routes with URL params, add `navigatePath`:
```tsx
MY_FEATURE_DETAIL: {
    path: '/my-feature/:id',
    routeType: 'authenticate',
    headerName: 'My Feature Detail',
    breadCrumb: 'My Feature Detail',
    element: <MyFeatureDetailPage />,
    navigatePath: (id: string | number) => `/my-feature/${id}`,
},
```

---

## routeType → guard mapping

| routeType | Guard | Use for |
|---|---|---|
| `'authenticate'` | AuthGuard | All logged-in staff pages |
| `'un-authenticate'` | UnauthGuard | Login, signup, forgot-password |
| `'public'` | None | Chatbot widget page, public intake forms |
| `'admin'` | AdminGuard | Admin-only settings |

---

## Step 3 — Navigate programmatically

```tsx
import { useNavigate } from 'react-router-dom';
import { ROUTES } from '../constants/routePath';

const navigate = useNavigate();
navigate(ROUTES.MY_FEATURE_DETAIL.navigatePath(entityId));
```

Never interpolate paths manually (`` `/my-feature/${id}` ``) — it breaks if the path key ever changes.

---

## Exact codebase reference

From `src/constants/routePath.tsx`:
```tsx
const PatientRecord = lazyWithRetry(() => import('../components/pages/Dashboard/PatientRecord'));

PATIENT_RECORD: {
    path: '/patient-record/:patient_id',
    routeType: 'authenticate',
    headerName: 'Patient Record',
    breadCrumb: 'Patient Record',
    showHeader: true,
    showFooter: false,
    element: <PatientRecord />,
    navigatePath: (patient_id: string | number) => `/patient-record/${patient_id}`,
},
```

---

## Checklist
- [ ] Lazy import uses `lazyWithRetry`, not `React.lazy()`
- [ ] Route defined in `ROUTES` in `src/constants/routePath.tsx` (no other file)
- [ ] `routeType` matches the intended audience
- [ ] Dynamic routes have `navigatePath`
- [ ] All programmatic navigation uses `ROUTES.X.navigatePath()` or `ROUTES.X.path`
