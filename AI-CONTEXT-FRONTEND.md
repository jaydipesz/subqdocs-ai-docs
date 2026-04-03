# SubQDocs — Frontend Context

> **Parent:** [`AI-CONTEXT.md`](AI-CONTEXT.md) — Architecture, Domain Rules, Never Do  
> **Skills:** [`docs/skills/INDEX.md`](docs/skills/INDEX.md)

---

# Stack

React 18 + Vite + TypeScript. See `subqdocs-frontend/package.json` for full versions.

Key libraries and why they matter:

| Library | Why it matters to an agent |
|---|---|
| `@reduxjs/toolkit` + `redux-persist` | Global state (15 persisted slices) |
| `@tanstack/react-query` | Server-fetched data (see Rule 16) |
| `@mui/material` + `@mantine/core` | MUI is **Primary**; Mantine is **Secondary/Specialized** |

---

# UI Library Hierarchy

To avoid mixed styles and ensure consistency:

1.  **MUI (Primary)**: Use for 95% of components including all inputs, buttons, sidebars, and structural layout.
2.  **Mantine (Secondary)**: 
    - Use for the `MantineProvider` (root).
    - Use for complex widgets and hooks (**`useDisclosure`**, **`RichTextEditor`**) not provided by MUI.
    - Avoid using Mantine buttons/inputs if an MUI equivalent exists.

# Modal Management

The project uses a **Centralized Modal Pattern**:

1.  **Providers**: `ModalProvider` in `App.tsx` manages global modal visibility.
2.  **Usage**: 
    - Trigger modals via the custom hook: `const { openModal } = useModal()`.
    - Features: `GlobalModal` component renders the active modal from context.
    - Standard: Feature-specific modals live in `src/components/common/modal/`.
| `formik` + `yup` | Primary forms (see Rule 11) |
| `tailwindcss` | Utility CSS for layout and spacing |
| `socket.io-client` | Real-time via singleton in `src/services/socket.ts` |
| `react-big-calendar` | Calendar grid in Dashboard |

---

# Convention-Critical Paths

```
subqdocs-frontend/src/
├── constants/routePath.tsx        # Route definitions (see Rule 8)
├── components/common/svg/Svg.tsx  # SVG icons (see Rule 9)
├── api/axios.ts                   # Typed wrappers — never use raw axios
├── redux/store.ts                 # All slices registered here
├── redux/ducks/                   # One slice per concern
├── redux/dispatch/                # Dispatch helpers for outside-component use
├── helper/laxywithRetry.ts        # See Rule 15
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
- **Server state:** follow Rule 16 (TanStack Query)
- **Global client state:** Redux (persisted via `redux-persist`)
- **Local:** `useState` / `useReducer`
- Outside-component Redux: dispatch helpers in `src/redux/dispatch/`

## Components
- Components are `React.FC<Props>` with explicit Props interface
- Each page: own folder with `index.tsx`
- Modals live in `src/components/common/modal/[Feature]Modal.tsx`

## Forms
- **Primary:** `Formik` + `Yup` (see Rule 11)
- **Secondary:** `react-hook-form`
- **Rule:** Never mix the two in the same component.

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
