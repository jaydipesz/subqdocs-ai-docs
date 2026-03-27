# Skill: Figma Extraction

**Canonical extraction procedure:** When running inside CW1, CW1 steps supersede this skill's steps. Use this skill only for standalone extractions outside the MASTER workflow.

## Purpose
Extract structured, implementation-ready data from a Figma design file using the Figma MCP, preventing the most common downstream failures: inferred fields, missing states, skipped screens, and approximate visual values.

## When to Use
- Starting a new feature that has a Figma design
- Auditing an existing UI against its Figma source (CW5A)
- Needing exact visual specifications (colors, spacing, typography) from a design

## Steps

1. Parse the Figma URL to extract the file key (segment after `/design/` or `/file/`) and the node ID (after `node-id=`, URL-decoded from `X-Y` to `X:Y`).
2. Call `mcp_figma_view_node` with the file key and node ID to load the root frame.
3. Read the returned node tree. Identify every top-level frame — each represents a screen. Record: frame name, node ID, width × height.
4. For each frame, traverse children recursively. Classify each node by type:
   - `TEXT` → label, heading, or body text. Record: content, fontSize, fontWeight, fills[0].color (convert RGB 0–1 to hex).
   - `RECTANGLE`/`FRAME` with children → container. Record: layoutMode (HORIZONTAL/VERTICAL), itemSpacing, paddingLeft/Right/Top/Bottom, primaryAxisAlignItems, counterAxisAlignItems.
   - `INSTANCE` → component instance. Record: componentId, overrides.
   - `VECTOR`/`BOOLEAN_OPERATION` → icon. Record: name, fills[0].color.
5. For each input element (identified by name containing "input", "field", "text", "select", "checkbox", "radio", "date", "upload"): record label (nearest TEXT node above or to the left), placeholder text, and whether a `*` or "required" text is adjacent.
6. For each button (identified by name containing "button", "btn", "cta", or a styled FRAME with a single TEXT child): record label text, background color, text color, border-radius, and classify as primary/secondary/destructive/ghost based on fill.
7. For each table or list pattern (identified by repeating horizontal FRAMEs inside a vertical FRAME): record column names from the header row, cell data types, and inline actions.
8. Identify states: look for frames with names ending in `/empty`, `/loading`, `/error`, `/disabled`, `/hover`, `/active`, or frames that are siblings with the same name prefix. Record each state and its differences from the default state.
9. Group repeated structures (same componentId or same structure with different data) as reusable components. Record: component name, list of screens it appears on.
10. Document navigation: interactive elements (buttons, links, tabs) that imply screen transitions. If the target screen exists in the file, record the source element → target frame mapping.
11. If any node has ambiguous labeling, overlapping elements, or unclear state grouping — **stop and ask**. Provide a screenshot of the node using `mcp_figma_view_node`.

## Rules

- NEVER infer a field name from context — read it from the TEXT node content. If no label exists, flag it.
- ALWAYS convert Figma RGB (0–1 float) to hex. Formula: `#${Math.round(r*255).toString(16).padStart(2,'0')}${...g}${...b}`.
- NEVER skip a top-level frame — every screen must be documented even if it looks like a duplicate.
- ALWAYS record spacing in integer pixels as returned by the Figma API, not rounded or approximated.
- ALWAYS check for state variants before reporting a screen as "single state."

## Output

A FIGMA ANALYSIS REPORT with sections: Screen Inventory, Field Catalog, State Matrix, Interaction Map, Visual Spec, Component Hierarchy, Navigation Flow. Each item includes its Figma node ID.

## Example

```
Screen Inventory:
| Screen            | Node ID  | Dimensions  | Purpose                        |
|-------------------|----------|-------------|--------------------------------|
| Create Patient    | 42:1051  | 1440×900    | Form for creating a new patient|
| Patient List      | 42:1200  | 1440×900    | Table of all patients          |

Field Catalog (Create Patient):
| Field         | Type     | Required | Validation          | Node ID  |
|---------------|----------|----------|---------------------|----------|
| First Name    | text     | yes      | max 100 chars       | 42:1055  |
| Date of Birth | date     | yes      | must be past date   | 42:1060  |

State Matrix:
| Screen         | Default | Empty | Loading | Error | Disabled |
|----------------|---------|-------|---------|-------|----------|
| Create Patient | ✓       | ✓     | —       | ✓     | —        |
| Patient List   | ✓       | ✓     | ✓       | ✓     | —        |

Visual Spec:
  Typography: H1=24px/700, H2=18px/600, Body=14px/400, Label=12px/500
  Colors: Primary=#2563EB, Error=#DC2626, Success=#16A34A, Border=#E5E7EB, BG=#F9FAFB
  Spacing: Page padding=24px, Card gap=16px, Form field gap=12px
```
