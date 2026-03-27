# Skill: Frontend Implementation

**Prerequisite:** Load skills `02-add-frontend-route`, `05-add-redux-state`, and `11-add-api-service-call` first. This skill adds orchestration-level concerns; those skills provide the file-by-file reference code.

## Purpose
Ensure frontend features are implemented in the correct dependency order with build verification, preventing the most common integration failures: mismatched API types, missing states, mixed form libraries, and broken routing.

## When to Use
- Implementing a new frontend page with multiple components
- Running CW5 — Frontend Implementation
- Adding 3+ related components that connect to new API endpoints

## Steps

1. Open the API contract or backend endpoint definitions provided as input. For each endpoint, create a TypeScript interface matching the response shape. Export all interfaces from a single types file in the feature directory.
2. Create API service functions following skill `11-add-api-service-call`. One function per endpoint. Name as `verbNoun`. Every function must use the typed HTTP wrapper — verify by checking the import matches the pattern in the audit report.
3. If Redux is needed (client-side UI state only), follow skill `05-add-redux-state`. Never store API response data in Redux — use `useQuery` for that.
4. Build each component as `React.FC<Props>`. Define the `Props` interface above the component. For each component:
   a. Open the FIGMA ANALYSIS REPORT and find the matching screen by name.
   b. Read the Visual Spec section for spacing, typography, and colors.
   c. Lookup the global `tailwind.config.js` in `subqdocs-frontend/`. If a matching design token (color, font, spacing) exists, ALWAYS use the configured Tailwind utility class instead of hardcoding arbitrary values.
   d. Implement the layout to match the Figma hierarchy (flex direction, gap, padding, alignment).
   e. Implement every state from the State Matrix for this screen. If a state is listed as "—" (not designed), implement a sensible default and note it in the mapping table.
5. For forms: ALWAYS use the library mandated by the global `production-rules.md`. Define the validation schema as a `const` outside the component. Copy validation rules from the matching backend validation schema exactly — field names, types, required/optional, and constraints must match.
6. For pages: wrap data fetching in `useQuery` with a descriptive query key (e.g., `['invoices', { patientId }]`). Wrap mutations in `useMutation` with `onSuccess` that invalidates the relevant query keys and `onError` that shows a toast to the user.
7. For modals: place in `src/components/common/modal/`. On close, reset form values and clear validation errors.
8. For routing: follow skill `02-add-frontend-route`. Use `lazyWithRetry()`, set `routeType`, and add `navigatePath` for dynamic routes.
9. Run `npx tsc --noEmit` in `subqdocs-frontend/`. Fix all errors.
10. Open 2–3 adjacent pages. Match toast message phrasing (e.g., "Created successfully" vs "Invoice created").
11. Produce the COMPONENT-TO-FIGMA MAPPING table.

## Rules

- NEVER use `React.lazy()` — always use `lazyWithRetry()` (prevents white screens on deploy).
- NEVER store server-fetched data in Redux — use `useQuery` (prevents stale data bugs).
- NEVER hardcode route strings — always reference `routePath.tsx` constants (prevents broken navigation).
- ALWAYS add `onError` with user-visible feedback to every `useMutation` (prevents silent failures).
- ALWAYS provide empty-state UI for every list/table — show an illustration and message, not a blank area.
- NEVER mix form libraries in the same feature — use the one identified in the AUDIT REPORT.
- ALWAYS run `tsc --noEmit` after all files are created — partial checks produce false passes.

## Core UI Implementation Rules

### 1. Mutation UI State
ALWAYS disable submit buttons during active mutations to prevent duplicate requests.
**Example:** `<button disabled={mutation.isPending}>Submit</button>`

### 2. Layout Stability
ALWAYS prefer Tailwind `gap` utilities within flex/grid containers over manual `margin` spacing between sibling elements.
**Example:** `className="flex flex-col gap-4"`
**Avoid:** Space created via manual margin stacking (e.g., `mb-4`, `mt-4` on list items).

### 3. List Rendering
All mapped lists MUST include stable, unique keys based on data IDs.
**Example:** `key={item.id}`
**Never:** Do NOT use the array `index` as a key.

### 4. React Query Keys
Query keys MUST strictly follow an array structure containing the resource name and a params object or ID: `['resource-name', paramsObject]`.
**Examples:**
- `['invoices']`
- `['invoices', { patientId }]`
- `['invoice', invoiceId]`
**Never:** Do NOT use string-only dynamic keys (e.g., `` `invoices-${patientId}` ``).

## Desktop & Laptop Responsive Fidelity Rules (Figma-Aligned)

Responsive styling MUST remain consistent across all laptop and desktop screen sizes while matching Figma layout constraints.

**Follow these rules:**

1. **Desktop First:** Treat the desktop layout as the primary source of truth unless tablet/mobile variants are explicitly defined in Figma.
2. **Standard Breakpoints:** Support these breakpoints using your Tailwind config:
   - `xsm` (360px)
   - `sm` (640px)
   - `md` (768px)
   - `lg` (1024px) → laptop baseline
   - `xl` (1280px) → standard desktop baseline
   - `2xl` (1536px) → large desktop baseline
   Layouts MUST remain visually stable across `lg` → `xl` → `2xl`. Avoid layout shifts between these breakpoints unless Figma specifically defines them.
3. **Container Layouts:** Use container-based constraints instead of percentage widths on desktop screens.
   - **Prefer:** `container`, `max-w-*`, `mx-auto`
   - **Avoid:** `w-[72%]`, `w-[83%]` unless explicitly required by the Figma design.
4. **Typography Tokens:** Typography MUST use Tailwind tokens instead of pixel values.
   - **Allowed:** `text-xs`, `text-sm`, `text-base`, `text-lg`, `text-xl`, `text-2xl`
   - **Avoid:** `text-[13px]`, `text-[15px]` unless Figma explicitly requires a non-token gap.
5. **Direct Typography Mapping:** Standardize text sizes strictly:
   - 12px → `text-xs`
   - 14px → `text-sm`
   - 16px → `text-base`
   - 18px → `text-lg`
   - 20px → `text-xl`
   - 24px → `text-2xl`
   If a strict mismatch exists, **extend `tailwind.config.js`** instead of using arbitrary arbitrary `[]` values.
6. **Spacing Scale:** Spacing MUST rigorously use the Tailwind spacing scale (`p-*`, `gap-*`, `mt-*`).
   - **Avoid:** `mt-[13px]`, `gap-[11px]` unless requested by Figma.
7. **Typography Stability:** Maintain an identical typography scale across `lg`, `xl`, and `2xl` unless Figma explicitly changes typography between desktop variants.
8. **Fidelity Verification:** Verify responsive fidelity visually after implementation:
   - Check layout at **1024px, 1280px, and 1536px**.
   - Ensure no wrapping regressions, spacing collapses, typography overflows, or alignment drifts.
   - If responsive behavior isn't explicitly defined in Figma, keep the layout stable across desktop breakpoints without introducing scaling on your own.

## Output

Complete frontend code following skills 02/05/11, plus a COMPONENT-TO-FIGMA MAPPING table.

## Example

```
COMPONENT-TO-FIGMA MAPPING:
| Component           | File Path                                     | Figma Screen    | Node ID  | States Implemented              |
|---------------------|-----------------------------------------------|-----------------|----------|---------------------------------|
| InvoiceListPage     | src/components/pages/Billing/InvoiceList.tsx  | Invoice List    | 42:1500  | loading, empty, error, success  |
| InvoiceCard         | src/components/pages/Billing/InvoiceCard.tsx  | Invoice List    | 42:1520  | default, disabled               |
| CreateInvoiceModal  | src/components/common/modal/CreateInvoice.tsx | Create Invoice  | 42:1600  | default, submitting, error      |
```
