---
description: CW5A — Pixel-perfect comparison between code and Figma.
---

_Formats: [WORKFLOW-FORMAT.md](./WORKFLOW-FORMAT.md)._

# CW5A — UI Verification

## Purpose
Verify every frontend component against the Figma source, fix all mismatches, and produce a verification report before the feature is declared complete.

## Standalone Inputs
- `FIGMA_URL` — Figma file or frame URL
- **FIGMA ANALYSIS REPORT** — output of CW1
- **COMPONENT-TO-FIGMA MAPPING** — output of CW5

## Trigger Condition
Immediately after CW5 completes. Mandatory. Cannot be skipped.

## Steps

1. Open the Figma file using Figma MCP with the file key from `FIGMA_URL`.
2. Load the **COMPONENT-TO-FIGMA MAPPING** table.
3. For each component row in the mapping table, navigate to the Figma node using the Node ID column, then load `docs/skills/16-pixel-perfect-ui-verification.md` and follow its steps 4–12 (Layout, Elements, Typography, Colors, Icons, States, Interactions).

4. **Mismatch fix protocol:** For every mismatch found in skill steps 4–12:
   - Record: component name, Figma screen name, Figma element, expected value, actual value.
   - Fix the code immediately.
   - Re-verify the fix matches Figma before moving to the next component.

5. **Verification:** Run `npx tsc --noEmit` in `subqdocs-frontend/` after fixes. Do not produce **Output** until exit code 0.

## Output
**UI VERIFICATION REPORT** with this format per component:
```
Component: [name]
Figma Screen: [name] ([node ID])

| Category     | Status | Detail                                              |
|--------------|--------|-----------------------------------------------------|
| Layout       | PASS   |                                                     |
| Spacing      | FAIL   | Gap 12px expected, 8px found → fixed to 12px        |
| Elements     | PASS   |                                                     |
| Typography   | PASS   |                                                     |
| Colors       | PASS   |                                                     |
| Icons        | PASS   |                                                     |
| States       | PASS   |                                                     |
| Interactions | PASS   |                                                     |
```

## Done Condition
Every component in the mapping table verified. All mismatches fixed. Report produced. User replies **"confirmed"**. Feature is complete.

## Skills Required
- `docs/skills/16-pixel-perfect-ui-verification.md`
- `docs/skills/12-figma-design-extraction.md`
