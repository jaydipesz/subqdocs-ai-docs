# Skill: Add Redux State (Frontend)

**When NOT to use this:** The state comes from an API call → use TanStack Query. The state is local to one component (modal open, active tab, input value) → use `useState`. The state is needed only during a single render cycle → use `useRef` or a local variable.

---

## Decision table: where does this state belong?

| State type | Example | Storage |
|---|---|---|
| Server-fetched data | Patient list, visit details | TanStack Query (`useQuery`) |
| Auth session + token | `user`, `token` | Redux → persisted |
| Applied list filters | `patientFilter`, `scheduleVisitFilter` | Redux → persisted |
| Toast queue | Error/success messages | Redux → persisted (`toastSlice`) |
| Active recording state | Recording session metadata | Redux → NOT persisted |
| Modal open, active tab | `isOpen`, `selectedTab` | `useState` |

---

## Step 1 — Create the slice

`src/redux/ducks/myFeature.ts`:
```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface MyFeatureState {
    activeId: number | null;
}

const initialState: MyFeatureState = { activeId: null };

const myFeatureSlice = createSlice({
    name: 'myFeature',
    initialState,
    reducers: {
        setActiveId: (state, action: PayloadAction<number | null>) => {
            state.activeId = action.payload;
        },
        resetMyFeature: () => initialState,  // ← always add a reset action
    },
});

export const { setActiveId, resetMyFeature } = myFeatureSlice.actions;
export default myFeatureSlice.reducer;
```

---

## Step 2 — Register in store.ts

`src/redux/store.ts`:
```ts
import myFeatureReducer from './ducks/myFeature';

const rootReducer = combineReducers({
    // ...existing slices
    myFeature: myFeatureReducer,
});
```

---

## Step 3 — Persistence (conscious decision per slice)

Add to the `persistConfig.whitelist` in `src/redux/store.ts` **only if** the state must survive a page refresh:

```ts
const persistConfig = {
    key: 'root',
    storage,
    whitelist: ['user', 'toast', 'patientFilter', 'breadCrumb', 'deviceId', 'scheduleVisitFilter', 'myFeature'],
};
```

**Do not persist:** loading flags, recording state, transient error state. If the persisted shape changes between deploys, users get a hydration error.

---

## Step 4 — Use in a component

```tsx
import { useSelector, useDispatch } from 'react-redux';
import { RootState, AppDispatch } from '../../../redux/store';
import { setActiveId } from '../../../redux/ducks/myFeature';

const dispatch = useDispatch<AppDispatch>();   // ← typed, not base useDispatch()
const { activeId } = useSelector((state: RootState) => state.myFeature);

dispatch(setActiveId(5));
```

---

## Step 5 — Dispatch outside a component (Axios interceptor, utility functions)

```ts
// src/redux/dispatch/myFeature.dispatch.ts
import { store } from '../store';
import { setActiveId } from '../ducks/myFeature';

export const dispatchSetActiveId = (id: number | null) => store.dispatch(setActiveId(id));
```

Exact codebase precedent — from `src/api/axios.ts`:
```ts
import { dispatchToast } from '../redux/dispatch/toast.dispatch';
dispatchToast({ message: 'Error occurred', type: 'error' });
```

---

## PHI risk: logout reset

Persisted slices retain the **previous user's data** when a new user logs in on the same browser. Every new slice must be reset on logout. Find the logout action in the user slice and add:

```ts
dispatch(resetMyFeature());
```

---

## Checklist
- [ ] State is not API-fetched (TanStack Query handles that)
- [ ] Slice has a `reset` action
- [ ] Reducer registered in `rootReducer` in `store.ts`
- [ ] Persistence decision is explicit — added to whitelist only if refresh-survival is required
- [ ] `useDispatch<AppDispatch>()` used (not base `useDispatch()`)
- [ ] Dispatch helper created if used outside a component
- [ ] Slice reset called on user logout
