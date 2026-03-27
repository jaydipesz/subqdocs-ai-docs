# Skill: Add an API Service Call (Frontend)

**When NOT to use this:** Calling the Python AI service — use `pythonApiPost` (already in `axios.ts`). Socket-based server communication — use skill 06.

---

## Files involved
```
subqdocs-frontend/src/api/
  axios.ts              ← typed wrappers — import from here, never raw axios
  <domain>Services.ts   ← one file per domain (patientServices, CalendarServices, etc.)
```

---

## Wrappers: never use raw axios

| Wrapper | Use for |
|---|---|
| `axiosGet<T>(url, params?)` | GET |
| `axiosPost<T>(url, { data })` | POST JSON |
| `axiosPut<T>(url, { data })` | PUT JSON |
| `axiosPatch<T>(url, { data })` | PATCH JSON |
| `axiosDelete<T>(url, params?)` | DELETE |
| `axiosPostFormData<T>(url, formData)` | File upload (multipart) |
| `pythonApiPost<T>(url, data)` | POST to Python AI service |

All wrappers auto-attach `Authorization`, `x-timezone`, `X-Device-Info` headers. Raw `axios.get()` skips all of these.

---

## Step 1 — Add to the correct service file

Find the existing domain file (e.g., `patientServices.ts`, `CalendarServices.ts`) or create one:

```ts
// src/api/myFeatureServices.ts
import { axiosGet, axiosPost, axiosPut, axiosDelete } from './axios';

export const getMyFeatures = async (params?: { page?: number; limit?: number }) =>
    axiosGet<MyFeatureListResponse>('/my-feature', params);

export const getMyFeatureById = async (id: number) =>
    axiosGet<MyFeatureResponse>(`/my-feature/${id}`);

export const createMyFeature = async (data: CreateMyFeaturePayload) =>
    axiosPost<MyFeatureResponse>('/my-feature', { data });

export const updateMyFeature = async (id: number, data: UpdateMyFeaturePayload) =>
    axiosPut<MyFeatureResponse>(`/my-feature/${id}`, { data });

export const deleteMyFeature = async (id: number) =>
    axiosDelete(`/my-feature/${id}`);
```

From `src/api/patientServices.ts` (exact):
```ts
export const getPatientById = async (patientId: number) =>
    axiosGet<PatientResponse>(`/patient/${patientId}`);

export const updatePatient = async (patientId: number, data: PatientUpdatePayload) =>
    axiosPut<PatientResponse>(`/patient/${patientId}`, { data });

export const uploadPatientAttachment = async (patientId: number, formData: FormData) =>
    axiosPostFormData<AttachmentResponse>(`/patient/${patientId}/attachments`, formData);
```

---

## Step 2 — File upload service function

```ts
export const uploadMyFeatureFile = async (visitId: number, file: File) => {
    const formData = new FormData();
    formData.append('file', file);
    formData.append('visit_id', String(visitId));
    // ← Never set Content-Type manually — axios sets the multipart boundary
    return axiosPostFormData<UploadResponse>(`/my-feature/${visitId}/upload`, formData);
};
```

---

## Step 3 — Use with TanStack Query

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { getMyFeatureById, updateMyFeature } from '../../../api/myFeatureServices';

const queryClient = useQueryClient();

const { data, isLoading } = useQuery({
    queryKey: ['my-feature', id],
    queryFn: () => getMyFeatureById(id),
    enabled: !!id,
});

const mutation = useMutation({
    mutationFn: (payload: UpdateMyFeaturePayload) => updateMyFeature(id, payload),
    onSuccess: () => {
        queryClient.invalidateQueries({ queryKey: ['my-feature', id] });
    },
    onError: (error) => {
        // Axios interceptor auto-toasts, but handle UI state (e.g., reset, error label) here
    },
});
```

You **must** include an `onError` callback in every `useMutation` (per Rule 16) to handle local UI state (like resetting a spinner), even though the Axios response interceptor auto-toasts the error globally.

---

## Step 4 — Suppress the auto-toast (when needed)

```ts
export const silentCheck = async (id: number) =>
    axiosGet<MyFeatureResponse>(`/my-feature/${id}`, undefined, { skipToast: true });
```

Use when the call is a background health check or when you're handling the error state visually yourself.

---

## Response shape from backend

All backend responses use `generalResponse()`:
```ts
{
    responseData: T;
    message: string;
    toast: boolean;
    response_type: 'success' | 'error';
}
```

The typed wrappers unwrap `responseData` — your service function already returns `T`. Do not access `.responseData` in the component.

---

## Checklist
- [ ] Uses `axiosGet`/`axiosPost`/etc., not raw `axios`
- [ ] Return type generic is specified: `axiosGet<MyType>(...)`
- [ ] File uploads use `axiosPostFormData` with no manual `Content-Type`
- [ ] Function placed in `src/api/<domain>Services.ts`
- [ ] Component uses `useQuery` or `useMutation` (not direct `async/await` in event handlers)
- [ ] `queryClient.invalidateQueries()` called on successful mutations
- [ ] Custom `onError` added to every mutation for UI state management (Rule 16)
