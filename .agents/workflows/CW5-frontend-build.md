---
description: CW5 — Implement the frontend matching Figma exactly using the API contract.
---

# CW5 — Frontend Implementation

## Purpose
Implement the frontend to match the Figma design exactly, using the API contract as the single source of truth for data types and endpoints.

## Standalone Inputs
- **FIGMA ANALYSIS REPORT** — output of CW1
- **API CONTRACT DOCUMENT** — output of CW4
- **AUDIT REPORT** — output of CW2

## Trigger Condition
After CW4 is confirmed by the user. If `PARTIAL_SCOPE` is `backend-first`, **skip CW5 and CW5A entirely** — the master workflow terminates after CW4.

## Steps

Before writing any code, re-read the **API CONTRACT DOCUMENT**. Every interface, API function, and validation rule must come from that document.

1. Load `docs/skills/15-structured-frontend-implementation.md`. Follow its steps 1–8 using the FIGMA ANALYSIS REPORT, API CONTRACT DOCUMENT, and AUDIT REPORT as inputs.

### Verification
2. Run `npx tsc --noEmit` in `subqdocs-frontend/`. Fix every type error before proceeding.
3. Open 2–3 adjacent pages. Compare toast and error message phrasing and match.
4. Produce the **COMPONENT-TO-FIGMA MAPPING**.

## Output
**COMPONENT-TO-FIGMA MAPPING** with this format:
```
| Component         | File Path                        | Figma Screen    | Node ID  | States Implemented              |
|-------------------|----------------------------------|-----------------|----------|---------------------------------|
| InvoiceListPage   | src/components/.../InvoiceList   | Invoice List    | 42:1500  | loading, empty, error, success  |
```

## Done Condition
`tsc --noEmit` passes. Mapping table produced. Hand off to CW5A immediately — no user confirmation gate (CW5A is mandatory).

## Skills Required
- `docs/skills/15-structured-frontend-implementation.md`
