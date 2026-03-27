# SubQDocs — Frontend Context

> **Parent:** [`AI-CONTEXT.md`](AI-CONTEXT.md) — Architecture, Domain Rules, Never Do  
> **Skills:** [`docs/skills/INDEX.md`](docs/skills/INDEX.md)

---

# Stack

React 18 + Vite + TypeScript. See `subqdocs-frontend/package.json` for full versions.

Key libraries and why they matter:

| Library | Why it matters to an agent |
|---|---|
| `@reduxjs/toolkit` + `redux-persist` | Global state — 15 persisted slices in `store.ts` |
| `@tanstack/react-query` | All server-fetched data — never duplicate in Redux |
| `@mui/material` + `@mantine/core` | Two component libraries coexist — MUI is primary |
| `formik` + `yup` | Primary form library — preferred for new forms |
| `tailwindcss` | Utility CSS for layout and spacing |
| `socket.io-client` | Real-time via singleton in `src/services/socket.ts` |
| `react-big-calendar` | Calendar grid in Dashboard |

---

# Convention-Critical Paths

```
subqdocs-frontend/src/
├── constants/routePath.tsx        # THE only file for route definitions
├── components/common/svg/Svg.tsx  # THE only file for SVG icons (6000+ lines)
├── api/axios.ts                   # Typed wrappers — never use raw axios
├── redux/store.ts                 # All slices registered here
├── redux/ducks/                   # One slice per concern
├── redux/dispatch/                # Dispatch helpers for outside-component use
├── helper/laxywithRetry.ts        # Use instead of React.lazy()
└── main.tsx                       # PDF.js worker lock + Sentry init
```

---

# Conventions

## API Layer (`src/api/`)
- All calls go through typed wrappers: `axiosGet<T>()`, `axiosPost<T>()`, `axiosPut<T>()`, `axiosPatch<T>()`, `axiosDelete<T>()`, `axiosPostFormData<T>()`
- Request interceptor auto-attaches: `Authorization`, `x-timezone`, `X-Device-Info`
- Response interceptor: `401` → auto-logout; `404` → redirect to `/` (with exceptions)
- `skipToast: true` suppresses automatic error toasts

## State Management
- **Server state:** TanStack Query (`useQuery`, `useMutation`) — cache invalidation on mutations
- **Global client state:** Redux (persisted via `redux-persist`)
- **Local:** `useState` / `useReducer`
- Outside-component Redux: dispatch helpers in `src/redux/dispatch/`

## Components
- Components are `React.FC<Props>` with explicit Props interface
- Each page: own folder with `index.tsx`
- Modals live in `src/components/common/modal/[Feature]Modal.tsx`

## Forms
- **Preferred (use first):** Formik + Yup
- **Alternative:** React Hook Form + Yup (or Zod) — used in some modules
- Never mix the two in the same component

## TypeScript
- `@` path alias resolves to `src/`
- Generic Axios wrappers: `axiosGet<MyResponseType>()`

## Naming
- **Files:** `PascalCase` for components/pages, `camelCase` for utilities/services
- **SVG icons:** `PascalCaseIconSVG` (e.g., `DeleteIconSVG`)
- **Redux selectors:** `current*` (e.g., `currentUser`)
- **API functions:** `verbNoun` (e.g., `getPatientById`)

## PDF.js
- Worker configured in `main.tsx` before React mounts
- Local worker on `localhost`, CDN on deployed envs
- `workerSrc` locked with `Object.defineProperty` — never touch without understanding load order
