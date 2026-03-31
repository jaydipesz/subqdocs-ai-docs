---
description: CW1 — Extract every screen, state, and interaction from Figma.
---

_Formats: [WORKFLOW-FORMAT.md](./WORKFLOW-FORMAT.md)._

# CW1 — Figma Analysis

## Purpose
Extract every screen, state, field, interaction, and visual detail from the Figma design into a structured report that drives all downstream implementation.

## Standalone Inputs
- `FIGMA_URL` — Figma file or frame URL 
- `FEATURE_DESCRIPTION` — 2–3 sentence description of what the feature does and why
- `TARGET_SCREENS` — which Figma screens are in scope (by name or node ID). "All" if the entire file is one feature

## Trigger Condition
Always runs first in the master workflow.

## Steps

1. Extract the file key and root node ID from `FIGMA_URL`.
2. Open the Figma file via Figma MCP using `mcp_figma_view_node` with the file key and root node ID.
3. List every top-level frame. If `TARGET_SCREENS` is not "All", filter to only the screens listed in `TARGET_SCREENS`. Write a one-sentence purpose for each in-scope screen. Note any out-of-scope screens as **"OUT OF SCOPE — not analyzed"**.
4. For each screen, enumerate every UI state visible in the design: empty, loading, error, success, disabled, partial data. If a state is not designed, note it as **missing from Figma**.
5. For each screen, list every user interaction: what element is interactive, what action it triggers (navigation, API call, modal open, form submit, toggle, delete).
6. For each screen, extract every data field as a row: field name, input type (text, number, date, select, checkbox, radio, file, tel), required/optional, and any validation rule visible in the design (max length, format hint, error message text).
7. For each screen, document every list, table, or card — record column names, rendered fields, sort indicators, and inline actions.
8. For each screen, document every filter, sort, search input, and pagination control — record their position and connected data.
9. For each screen, document every file upload area, image display, and date picker — record accepted formats if visible.
10. Document navigation: which element links to which screen, route parameters implied by the URL structure, and conditional visibility (e.g., button only shown for admin).
11. Identify role/permission signals: staff-only elements, patient-facing views, org-scoped data filters.
12. Document layout structure per screen: flex/grid direction, gaps (in px), padding, alignment.
13. Document typography per element type: font size (px), font weight (number), line height, text color (hex).
14. Document color usage: map each color hex to its semantic meaning (primary action, error state, success state, disabled, border, background).
15. Identify reusable components: elements that appear on multiple screens with the same structure. Group them and list where each is used.
16. If any screen, state, field, or visual detail is ambiguous — **stop and ask**. Do not infer intent. Do not proceed until the ambiguity is resolved.

17. **Verification:** Confirm every in-scope screen from `TARGET_SCREENS` appears in the report with node IDs. Do not produce **Output** until this check passes.

## Output
**FIGMA ANALYSIS REPORT** containing these exact sections:
1. Screen Inventory — name, purpose, node ID per screen
2. Field Catalog — name, type, required, validation per field per screen
3. State Matrix — which states exist per screen, which are missing from design
4. Interaction Map — element → action per screen
5. Visual Spec — typography scale, color map (hex → semantic), spacing tokens
6. Component Hierarchy — reusable elements and where they appear
7. Navigation Flow — screen-to-screen links and route parameters

## Done Condition
Report is complete. All ambiguities resolved. User replies **"confirmed"**. Do not proceed to CW2 until confirmation is received.

## Skills Required
- `docs/skills/12-figma-design-extraction.md`
