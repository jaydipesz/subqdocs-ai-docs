# Skill: UI Verification

## Purpose
Systematically compare every implemented frontend component against its Figma source and fix all discrepancies, preventing the most common UI failures: wrong spacing, approximate colors, missing states, and silently substituted icons.

## When to Use
- After completing frontend implementation for a feature (CW5A — Final UI Sync)
- When a component visually doesn't match the design and needs a systematic check
- During a design review to verify Figma fidelity

> **Figma API fallback:** If the Figma MCP tool is unavailable (API down, token expired, file unshared), ask the user to provide exported screenshots or a PDF of the relevant screens before proceeding. Do not skip the design verification step.

## Steps

1. Extract the file key and node ID from the Figma URL. Call `mcp_figma_view_node` with the file key and the node ID of the first screen to verify.
2. Open the COMPONENT-TO-FIGMA MAPPING table. Identify every component row.
3. For each component row, open the component source file and the Figma node (by node ID from the mapping table).
4. **Layout check:** Read the Figma frame's `layoutMode` (HORIZONTAL/VERTICAL), `itemSpacing`, `paddingLeft/Right/Top/Bottom`. Compare to the component's flex/grid direction, `gap`, `padding`. Verify that standard Tailwind spacing tokens are used over arbitrary pixel values (e.g., `gap-4` instead of `gap-[16px]`). Record any mismatch.
5. **Element check:** Count every child node in the Figma frame (TEXT, INSTANCE, RECTANGLE, VECTOR, IMAGE). Count every JSX element in the component. If counts differ, identify which element is missing or extra.
6. **Typography check:** For each TEXT node in Figma, read `fontSize`, `fontWeight`, `textAlignHorizontal`. Compare to the component's CSS/Tailwind for that text element, ensuring configured typography classes from `tailwind.config.js` are used. Record any mismatch.
7. **Color check:** For each visible FRAME/RECTANGLE, read `fills[0].color` and convert RGB(0–1) to hex. Compare to the component's `backgroundColor`, `color`, `borderColor`. **Crucially, verify that if a matching color token exists in `tailwind.config.js`, the component uses the built-in Tailwind class (e.g., `text-primary-500`) instead of a hardcoded hex value (e.g., `text-[#22C55E]`).** Record any mismatch. Check state-dependent colors (hover, active, disabled, error) separately.
8. **Icon check:** For each VECTOR/BOOLEAN_OPERATION node in Figma, read its `name`. Search `Svg.tsx` for an icon with that name or semantic equivalent. If no match exists: do NOT substitute — document it as **MISSING ICON: [figma name] — no equivalent in Svg.tsx**.
9. **State check:** Open the FIGMA ANALYSIS REPORT State Matrix for this screen. For each state marked ✓, navigate to the corresponding Figma state frame (frame with name suffix `/empty`, `/loading`, `/error`, etc.). Compare the state's visual appearance to what the component renders for that state. Record any mismatch.
10. **Interaction check:** For each interactive element (button, link, input, form), verify:
    - Buttons have `onClick` handlers that call the correct function.
    - Forms have `onSubmit` connected to the correct `useMutation`.
    - Navigation calls use `ROUTES.X.path` or `ROUTES.X.navigatePath()` from `routePath.tsx`.
    - Inputs have `onChange` connected to form state.
11. For each mismatch found in steps 4–10:
    a. Record: `{ component, figmaScreen, figmaElement, expected, actual }`.
    b. Edit the component file to fix the mismatch.
    c. Re-read the fixed line to confirm it now matches the Figma value.
12. After all components are verified, compile the verification results into the report.

## Rules

- NEVER skip a component in the mapping table — every row must be verified.
- NEVER silently substitute an icon — if `Svg.tsx` doesn't have it, flag it as MISSING ICON.
- ALWAYS verify every state in the State Matrix, not just the default/happy path.
- ALWAYS fix mismatches immediately per component — do not batch fixes across components.
- NEVER approximate colors — compare exact hex values from Figma API, not visual approximations.
- ALWAYS flag hardcoded hex codes or arbitrary `[pixel]` spacing values if a matching global token exists in `tailwind.config.js`.
- ALWAYS re-verify a fix before moving to the next component — a bad fix costs a second pass.

## Output

A UI VERIFICATION REPORT with one table per component showing pass/fail per category, mismatch details, and fix applied.

## Example

```
Component: InvoiceCard
Figma Screen: Invoice List (42:1520)

| Category     | Status | Detail                                                       |
|--------------|--------|--------------------------------------------------------------|
| Layout       | PASS   |                                                              |
| Spacing      | FAIL   | gap: expected 12px (Figma itemSpacing=12), actual 8px → fixed|
| Elements     | PASS   | 6/6 elements match                                           |
| Typography   | PASS   |                                                              |
| Colors       | FAIL   | Status badge bg: expected #22C55E, actual #10B981 → fixed    |
| Icons        | PASS   | 2/2 icons found in Svg.tsx                                   |
| States       | PASS   | default ✓, disabled ✓                                        |
| Interactions | PASS   | onClick → navigate(ROUTES.INVOICE_DETAIL.navigatePath(id))   |

Component: InvoiceListPage
Figma Screen: Invoice List (42:1500)

| Category     | Status | Detail                                                       |
|--------------|--------|--------------------------------------------------------------|
| Layout       | PASS   |                                                              |
| Elements     | FAIL   | Filter dropdown missing → added StatusFilter component       |
| Icons        | FAIL   | MISSING ICON: filter-funnel — no equivalent in Svg.tsx       |
| States       | PASS   | loading ✓, empty ✓, error ✓, success ✓                      |
```
