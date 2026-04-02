---
description: End-to-end lifecycle management (create/edit/optimize) for AI clinical agent nodes. Handles prompt engineering, validation schemas, LangGraph node implementation, and pipeline wiring. Ensures compliance with Skills 19, 21, and 20.
---

# TOOL — Manage Clinical Agent Node

> **Trigger:** "Add/edit clinical section", "modify agent node", "update exam logic",
> "create a prompt", "optimize hpi prompt", "fix transcription node", "new clinical node".

---

## Pre-requisite — Load Skills

Load the applicable skills before ANY work:

| Skill | When to load | File |
|-------|-------------|------|
| **Skill 19** (Universal Medical Prompt Rules) | Always | `docs/skills/19-medical-prompt-rules.md` |
| **Skill 21** (Pipeline Prompt Integration) | If prompt is part of the SubQDocs pipeline | `docs/skills/21-pipeline-prompt-integration.md` |
| **Skill 20** (Agent Node + Workflow Wiring) | If creating/editing an agent node or wiring | `docs/skills/20-agent-node-implementation.md` |

---

## Phase 0 — Mandatory Information Gathering (DO NOT SKIP)

**Scenario A: New Node/Prompt**
Ask for these 4 inputs (consolidate into ONE question):
1. **The Input:** All data/variables (e.g. transcript, past data, format settings).
2. **Output Schema:** Exact JSON structure/fields.
3. **Behavior:** Specific clinical extraction/transformation logic.
4. **Extra Instructions:** Exclusions or edge cases (or "None").

**Scenario B: Edit/Modify Existing**
Ask for these 3 inputs:
1. **Target:** File path or section name (e.g. "Exam", "HPI").
2. **Change:** Rules to add, logic shifts, or schema updates.
3. **Context:** Clinical scenario, bug, or feedback motivating the change.

---

## Phase 1 — Research & Reference Patterns

**Step 1 — Match by domain (Path: `subqdocs-backend/src/common/latest-agents/prompts/`):**

| Domain | Reference prompts |
|--------|-------------------|
| Clinical section (HPI, Exam, ROS, History) | `hpiChiefAllergiesPrompt.ts`, `examPrompt.ts`, `reviewOfSystemPrompt.ts` |
| Coding / billing | `cptCodePrompt.ts`, `billingTablePrompt.ts`, `icdCodePrompt.ts` |
| Chatbot / edit journey | `chatBotPrompt.ts`, `preValidationPrompt.ts`, `editJourmeyPrompt.ts` (note: actual spelling has 'm') |
| Validation / post-processing | `postValidationPrompt.ts`, `preValidationPrompt.ts` |
| Template processing | `suggestedTemplatesPrompt.ts`, `generateTemplatePrompt.ts` |

**Step 2 — Match by output shape:**
- `{ summary, html }` → `cancerHistoryPrompt.ts`, `socialHistoryPrompt.ts`
- `{ singleField }` → `examPrompt.ts`, `reviewOfSystemPrompt.ts`
- `[ { array_of_objects } ]` → `impression&planPrompt.ts`, `billingTablePrompt.ts`

**Step 3 — Read the full chain:**
Check `agent_nodes/<name>.ts` + `validation_schema/<name>.schema.ts` for the reference you chose.

> [!WARNING]
> **Check Annotation Spelling:** Some nodes in `stateGraphWorkflow.ts` have inconsistent spelling (e.g., `HPICheifAllergies` vs `hpiChiefAllergiesPrompt`). Always verify the `Annotation` key before writing new wiring.

---

## Phase 2 — Write/Modify the Prompt File

Path: `subqdocs-backend/src/common/latest-agents/prompts/<sectionName>Prompt.ts`

1. **Section Order (Skill 19 §2):** Persona → Input Declaration → Extraction Rules → Anti-Hallucination → Format Block → Priority Hierarchy → JSON Schema → Strict Footer.
2. **Signature (Skill 21 §2):** Use the factory function `get<SectionName>Prompt(formatSettings, extraUserInstruction)`.
3. **Checklist:** Verify against **Skill 19 §8** (Universal) and **Skill 21 §8** (Pipeline).

---

## Phase 3 — Implement Node, Schema, and Wiring

> **Follow Skill 20 completely.** Use `subqdocs-backend/src/common/latest-agents/` as root.

1. **Validation Schema:** Create/update `validation_schema/<name>.schema.ts` using the dual-mode ("validation"/"llm") pattern. **Must match prompt's JSON structure exactly.**
2. **Agent Node:** Create/update `agent_nodes/<name>.ts`. Include `org_id` scoping, `emitNodeStatus` for frontend, and `ReRunExtraInstructions` for continue-recording.
3. **Wiring:** Register in `workflow/stateGraphWorkflow.ts`.
    - Add to `StateAnnotation` with `{ reducer: stateConcatenation }`.
    - Use `withFailureHandler` in `.addNode()`.
    - Wire `.addEdge()` according to data flow (Parallel for clinical, Sequential for billing).

---

## Phase 4 — Manual Key Audit & Build

1. **Manual Key Audit:** Explicitly compare the prompt's `JSON Output Schema` string block against the keys in `validation_schema`. Ensure no plural/singular or casing mismatches (e.g. `medical_history` vs `medicalHistory`).
2. **Build:** Run `npx tsc --noEmit` from `subqdocs-backend`.
3. **Registration:** If this is a new clinical section, ensure it is added to the `MAIN SECTIONS` list in `preValidationPrompt.ts` and `outputStructures.ts` (Skill 20 §5).

---

## Phase 5 — Clinical Audit & Documentation

1. **Create 3 Test Scenarios** (synthetic data, NO PHI):
    - **Happy Path:** All fields present.
    - **Sparse Path:** Missing data (verify omission vs hallucination rules).
    - **Adversarial:** Mixed history/contradictions (verify exclusion rules).
2. **Trace:** Mentally (or via script) run the inputs through the prompt rules.
3. **MANDATORY DOCUMENTATION:** Add the test scenarios and expected results as a `/* TEST CASES */` commentary block at the end of the **agent node** or **prompt file** for future audit and regression testing.

---

## Quick Reference — Triggers

| Action | Triggers |
|--------|----------|
| **Create** | "Add section", "new agent node", "create prompt" |
| **Modify** | "Edit hpi", "fix exam logic", "optimize prompt", "update schema" |
| **Pipeline** | "Transcription pipe", "LangGraph wiring", "stateGraph update" |
