---
name: Agent Node Implementation and Workflow Integration
description: >
  Mandatory skill for adding, modifying, or removing clinical agent nodes
  within the SubQDocs clinical documentation pipeline. Covers node implementation (agent_nodes/),
  workflow registration (stateGraphWorkflow.ts), multi-prompt orchestration rules,
  continue-recording / re-run patterns, and chatbot edit journey integration
  (preValidationPrompt.ts, outputStructures.ts, sectionInstructionPrompt.ts).
  Depends on Skill 19 (prompt rules) and Skill 21 (pipeline prompt integration).
---

# Skill 20 — Agent Node Implementation and Workflow Integration

> **When to load:** Before creating, editing, or reviewing ANY file inside
> `src/common/latest-agents/agent_nodes/` or modifying `src/common/latest-agents/workflow/stateGraphWorkflow.ts`.
>
> **Depends on:** Skill 19 (Universal Medical Prompt Rules), Skill 21 (Pipeline Prompt Integration).

---

## 1. Agent Node Architecture (LangGraph Integration)

SubQDocs uses **LangGraph** for clinical documentation. Each "section" (HPI, Exam, Plan, etc.) is an independent **Node** in the workflow.

### 1.1 Node Function Signature
Every node MUST follow this signature:

```typescript
import { State } from "@common/latest-agents/workflow/stateGraphWorkflow";

export const myAgentNode = async (state: State) => {
    const { transcriptionData, transcriptId, visit_id, doctor_id, organization_id } = state;
    try {
        // 1. Implementation logic (LLM invocation, DB queries)
        // 2. State update
        return { MySectionData: [/* your result */] };
    } catch (error) {
        // 3. Error Handling
        await handleNodeError("myAgentNode", error, state, ["TRANSCRIPT_STATUS"]);
        return { myAgentNode: { failed: true } };
    }
};
```

---

## 2. Implementation Checklist — The Node (`agent_nodes/`)

### 2.1 Boilerplate & Safety
- [ ] **Standard Imports:** Include `parse()` from `@utils/common.utils` and `logger` from `@utils/logger`.
- [ ] **Organization ID:** Always include `organization_id = state.organization_id` in repository queries (Rule 1).
- [ ] **Socket Status:** Use `emitNodeStatus` from `@common/latest-agents/utils/centralizedSocket` to update the frontend (e.g., `emitNodeStatus("TRANSCRIPT_STATUS", state, "SUCCESS", result)`).
- [ ] **Error Handling:** Use `handleNodeError` to log failures and notify the system via socket.
- [ ] **Memory Management:** Ensure `chatHistory` is passed to `invokeWithValidationAndRetry` if the node requires context.

### 2.2 LLM & Validation
- [ ] **Factory Prompt:** Use a prompt factory from `src/common/latest-agents/prompts/` (Skill 19 + Skill 21).
- [ ] **Retry Loop:** Use `invokeWithValidationAndRetry` with a dual-mode schema from `validation_schema/`.
- [ ] **Payload Sanitization:** Use `removeHtmlFromObject` when passing previous HTML results back to the LLM as input.

### 2.3 Continue-Recording / Re-Run Support

When a node supports **continue recording** (`isContinueRecording`) or **re-runs** (`PastDataDetails`), the agent node must:

#### 2.3.1 Import and Append `ReRunExtraInstructions`

```typescript
import { ReRunExtraInstructions } from "@common/latest-agents/prompts/reRunExtraInstructions";

const prompt = PastDataDetails
  ? `${dynamicPrompt} /n ${ReRunExtraInstructions}`
  : dynamicPrompt;
```

`ReRunExtraInstructions` provides universal merge rules:
- Preserve ALL old data — no deletion, no overwriting unless explicitly contradicted
- Generate from scratch if `OLD_GENERATED_DATA` is null/empty
- Merge new data into existing structure

#### 2.3.2 Build Dual-Path Payloads

```typescript
const payload = PastDataDetails
  ? `
    <OLD_GENERATED_DATA>
      ${JSON.stringify(removeHtmlFromObject(PastDataDetails?.[0]?.my_field))}
    </OLD_GENERATED_DATA>
    <OLD_TRANSCRIPT>${JSON.stringify(oldTranscript)}</OLD_TRANSCRIPT>
    <EXTENDED_TRANSCRIPT>
      ${JSON.stringify(isContinueRecording ? newTranscript : cleanedTranscript)}
    </EXTENDED_TRANSCRIPT>
  `
  : `
    <transcript>${JSON.stringify(cleanedTranscript)}</transcript>
    <past_data>${JSON.stringify(pastData)}</past_data>
  `;
```

#### 2.3.3 Extract Section Settings

Every agent node extracts `formatSettings` / `extraUserInstruction` from `sectionSettingsData` using:

```typescript
import { getSectionSettingsForName } from "@utils/common.utils";

const section_setting = getSectionSettingsForName(sectionSettingsData, "Section Display Name");
const formatSettings = {
  my_section: section_setting.format,
  tone: section_setting.tone,
};
const extraUserInstruction = {
  my_section: section_setting.custom_instructions,
};
```

> The first argument is `sectionSettingsData` from the workflow state. The second argument is the **human-readable section name** (e.g., `"Cancer History"`, `"Impression and Plan"`, `"Review of System"`).

---

## 3. Registration Checklist — The Workflow (`stateGraphWorkflow.ts`)

### 3.1 The 3 Workflow Graphs

All workflows are defined in `src/common/latest-agents/workflow/stateGraphWorkflow.ts`:

| Graph | Variable | Purpose | Entrypoint |
|-------|----------|---------|------------|
| **Main workflow** | `workflow` | Full transcript processing pipeline | `executeWorkflow()` |
| **Partial workflow** | `partialWorkFlow` | Template-based re-runs | `executePartialWorkflow()` |
| **Continue workflow** | `newWorkflow` | Chatbot edits, corrections, validation | `newExecuteWorkflow()` |

### 3.2 State Annotation
Nodes must append their output to the shared state.
- [ ] **Annotation Key:** Use a plural/Detail key (e.g., `MySectionDetail`).
- [ ] **Reducer:** Use `{ reducer: stateConcatenation }` to ensure history is preserved and parallel results don't overwrite each other.

```typescript
MySectionDetail: Annotation<any[]>({ reducer: stateConcatenation }),
```

### 3.3 Adding a New Node — Step by Step

1. **Import** the agent node function at the top of `stateGraphWorkflow.ts`:
   ```typescript
   import { MyNode } from "@common/latest-agents/agent_nodes/myNode";
   ```

2. **Add a state annotation** if the node produces new output:
   ```typescript
   MyNodeDetail: Annotation<any>({ reducer: stateConcatenation }),
   ```

3. **Register the node** with a failure handler:
   ```typescript
   .addNode("myNode", withFailureHandler("myNode", MyNode))
   ```

4. **Wire it** to finalize (or to a downstream node):
   ```typescript
   .addEdge("myNode", "finalize")
   ```

5. **Connect the route** that triggers it — usually inside `suggestedTemplatesRoute`, `partialWorkflowStartRoute`, or as a parallel branch from `correctionAndLabel`.

**Placement guidance:**
- **Clinical section nodes** (HPI, Exam, ROS, etc.) → parallel branches from `correctionAndLabel`
- **Billing / coding nodes** → sequential chain after `impression&plan`

### 3.4 Edge Wiring Options
- [ ] **Node Registration:** Use `withFailureHandler` to wrap the node function.
    ```typescript
    .addNode("myNode", withFailureHandler("myNode", myAgentNode))
    ```
- [ ] **Edge Wiring:**
    - `.addEdge("__start__", "myNode")` if it's an entry point.
    - `.addEdge("myNode", "finalize")` to aggregate results.
    - `.addConditionalEdges(...)` if the node controls branching.

---

## 4. Multi-Prompt Orchestration

### 4.1 Pipeline Flow (from `stateGraphWorkflow.ts`)

```
START → lanTranslation → correctionAndLabel
  ├─→ [parallel clinical nodes] ────────────────→ finalize
  │     ├── hpiCheifAllergies
  │     ├── reviewOfSystem
  │     ├── exam
  │     ├── skinHistory / socialHistory / cancerHistory
  │     ├── medicationHistory / prescription
  │     ├── personalNoteAgent
  │     └── visitSummary → visitSnapshot
  ├─→ suggestedTemplates → impression&plan → ICD mapping
  │     → checkProceedWithIcdMapping → billing → photoMetaData → finalize
  └─→ finalize → END
```

### 4.2 Rules for Cascading Prompts

1. **Upstream nodes set state, downstream nodes read it.** Never have a prompt directly read another prompt's raw output — use the shared `State` object.
2. **State annotations use `stateConcatenation` reducer** — your node's output is **appended**, not replaced. Return data in the expected `{ KeyDetail: [result] }` format.
3. **Don't duplicate extraction logic.** If HPI already extracts the chief complaint, don't re-extract it in impression&plan. Read it from state instead.
4. **Shared context flows through `cleanedTranscript`** — all clinical nodes receive the same cleaned transcript from the `correctionAndLabel` step.
5. **Parallel nodes cannot depend on each other.** Nodes wired in parallel (all the clinical section nodes) execute concurrently and cannot read each other's output. Only `finalize` and downstream sequential nodes can aggregate their results.
6. **The finalize node** aggregates all parallel results and writes to the database. Your node's output shape must be compatible with it.

---

## 5. Chatbot Edit Journey Integration (Optional)

If the new section should be editable via the "Refine" chatbot:

1. **`preValidationPrompt.ts`** → Add the section name + sub-sections to the `MAIN SECTIONS AND THEIR SUB-SECTIONS` list.
2. **`outputStructures.ts`** → Add the section's JSON template to `EditJsonStructures`.
3. **`sectionInstructionPrompt.ts`** → Add the section to the `SectionInstructions` map. This map pairs each section name with its **legacy static prompt** export (e.g., `CancerHistoryPrompt`):
   ```typescript
   MY_SECTION: `## **HIGH PRIORITY EDIT INSTRUCTION** ${editInstruction}\n ${MySectionPrompt}`
   ```
   > **Note:** `SectionInstructions` uses the legacy `export const` prompts, not the factory functions. If you create a new section, you must also export a static version (or keep the existing legacy export) for this map.
4. **Validation schema** → Place in `src/common/latest-agents/validation_schema/<section>.schema.ts`. Export a function that returns both a `"validation"` schema and an `"llm"` (structured output) schema.

---

## 6. Common Mistakes to Avoid

| Mistake | Correct Pattern |
|---------|-----------------|
| Forgetting `organization_id` in repo calls | `repoFunc({ where: { id, organization_id: state.organization_id } })` |
| Using raw `state.myKey = value` | Return an object `{ myKey: value }` — LangGraph handles the state merge. |
| Overwriting state in parallel nodes | Always use `{ reducer: stateConcatenation }` for clinical outputs. |
| Not emitting socket status | Frontend spinners will hang forever. Use `emitNodeStatus`. |
| Missing `withFailureHandler` | A single node failure will crash the entire workflow process. |
| Parallel nodes reading each other's output | Parallel nodes execute concurrently — only `finalize` and downstream sequential nodes can aggregate. |
| Skipping `ReRunExtraInstructions` for re-runs | Continue-recording will overwrite old data instead of merging. |
| Forgetting legacy static export for edit journey | `sectionInstructionPrompt.ts` requires `export const` prompt, not factory function. |

---

## 7. Verification Plan

1. **Dry-Run State Mutation:** Verify the node returns the exact key defined in `StateAnnotation`.
2. **Failure Injection:** Throw an error in the node and confirm `handleNodeError` logs it correctly.
3. **Socket Capture:** Use the `TRANSCRIPT_STATUS` channel to verify the frontend receives the update.
4. **Re-Run Path:** If supporting continue-recording, verify merge behavior with both fresh and existing data paths.
5. **Edit Journey:** If integrated, verify the chatbot can identify, route to, and re-generate the section.
