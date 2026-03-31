---
name: Agent Node Implementation and Workflow Integration
description: >
  Mandatory skill for adding, modifying, or removing clinical agent nodes
  within the SubQDocs clinical documentation pipeline. Covers node implementation (agent_nodes/),
  workflow registration (stateGraphWorkflow.ts), and chatbot edit journey
  integration (preValidationPrompt.ts, outputStructures.ts).
---

# Skill 20 — Agent Node Implementation and Workflow Integration

> **When to load:** Before creating, editing, or reviewing ANY file inside
> `src/common/latest-agents/agent_nodes/` or modifying `src/common/latest-agents/workflow/stateGraphWorkflow.ts`.

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
- [ ] **Factory Prompt:** Use a prompt factory from `src/common/latest-agents/prompts/` (Skill 19).
- [ ] **Retry Loop:** Use `invokeWithValidationAndRetry` with a dual-mode schema from `validation_schema/`.
- [ ] **Payload Sanitization:** Use `removeHtmlFromObject` when passing previous HTML results back to the LLM as input.

### 2.3 Continue-Recording / Re-Run Support
If the node supports appending to an existing note:
- [ ] **Input Check:** Detect `isContinueRecording` or `PastDataDetails`.
- [ ] **Instruction Append:** Combine the clinical prompt with `ReRunExtraInstructions`.
- [ ] **Merge Logic:** Use the standard `<OLD_GENERATED_DATA>` and `<EXTENDED_TRANSCRIPT>` payload structure (Skill 19 §20).

---

## 3. Registration Checklist — The Workflow (`stateGraphWorkflow.ts`)

### 3.1 State Annotation
Nodes must append their output to the shared state.
- [ ] **Annotation Key:** Use a plural/Detail key (e.g., `MySectionDetail`).
- [ ] **Reducer:** Use `{ reducer: stateConcatenation }` to ensure history is preserved and parallel results don't overwrite each other.

```typescript
MySectionDetail: Annotation<any[]>({ reducer: stateConcatenation }),
```

### 3.2 Workflow Wiring
- [ ] **Node Registration:** Use `withFailureHandler` to wrap the node function.
    ```typescript
    .addNode("myNode", withFailureHandler("myNode", myAgentNode))
    ```
- [ ] **Edge Wiring:**
    - `.addEdge("__start__", "myNode")` if it's an entry point.
    - `.addEdge("myNode", "finalize")` to aggregate results.
    - `.addConditionalEdges(...)` if the node controls branching.

---

## 4. Chatbot Edit Journey Integration (Optional)

If the new section should be editable via the "Refine" chatbot:
- [ ] **`preValidationPrompt.ts`**: Register the section name (English) and its sub-keys.
- [ ] **`outputStructures.ts`**: Add a blank JSON template to `EditJsonStructures`.
- [ ] **`sectionInstructionPrompt.ts`**: Map the section UI name to its **legacy static prompt** constant.
- [ ] **Validation Schema**: Ensure the schema in `validation_schema/` supports both initial generation and refinement.

---

## 5. Common Mistakes to Avoid

| Mistake | Correct Pattern |
|---------|-----------------|
| Forgetting `organization_id` in repo calls | `repoFunc({ where: { id, organization_id: state.organization_id } })` |
| Using raw `state.myKey = value` | Return an object `{ myKey: value }` — LangGraph handles the state merge. |
| Overwriting state in parallel nodes | Always use `{ reducer: stateConcatenation }` for clinical outputs. |
| Not emitting socket status | Frontend spinners will hang forever. Use `emitNodeStatus`. |
| Missing `withFailureHandler` | A single node failure will crash the entire workflow process. |

---

## 6. Verification Plan

1. **Dry-Run State Mutation:** Verify the node returns the exact key defined in `StateAnnotation`.
2. **Failure Injection:** Throw an error in the node and confirm `handleNodeError` logs it correctly.
3. **Socket Capture:** Use the `TRANSCRIPT_STATUS` channel to verify the frontend receives the update.
