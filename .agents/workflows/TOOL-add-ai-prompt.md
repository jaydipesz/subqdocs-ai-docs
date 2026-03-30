---
description: Generate a new AI/LLM prompt for the SubQDocs clinical documentation pipeline. Enforces mandatory data-gathering before writing, ensures prompt adheres to Skill 19, and validates the output end-to-end.
---

# TOOL — Add AI Prompt

> **Trigger:** User asks to "create a prompt", "add an AI prompt", "new LLM prompt",
> "write a system prompt", "edit a prompt", or any similar request for a new/modified agent prompt.

---

## Pre-requisite: Load Skill 19

Before ANY work begins, load and read `docs/skills/19-prompt-engineering.md`.
Every rule in Skill 19 is **mandatory** for this workflow.

---

## Phase 0A — Mandatory Information Gathering (DO NOT SKIP)

**You MUST NOT write any code until the user provides ALL FOUR of the following items.** If any are missing, ask for them explicitly before proceeding.

### Required Inputs from the User

| # | Input | What to ask for | Example |
|---|-------|-----------------|---------|
| 1 | **The Input** | What data/variables will be passed to the prompt? List every input the prompt will receive (transcript, past data, format settings, templates, reference codes, etc.). | `"This prompt receives: (1) a <transcript> of a doctor-patient visit, (2) <PAST_DATA> with previously documented medications, (3) formatSettings with tone and format."` |
| 2 | **Expected Output & Schema** | The exact TypeScript interface or JSON schema the LLM must return. Every field, its type, and a description of its content. | `{ "medications_html": string, "new_medications_html": string }` |
| 3 | **Expected Behavior** | What the LLM actually needs to calculate, extract, classify, or transform. Be specific about the clinical logic. | `"Extract all past and current medications from the transcript. Separate newly prescribed medications from previously documented ones. Include strength, frequency, and instructions."` |
| 4 | **Extra Instructions** | Any domain-specific medical edge cases, exclusion rules, or special behaviors unique to this prompt. | `"Do NOT include over-the-counter supplements unless the doctor explicitly prescribes them. If a medication is discontinued, mark its status as 'Discontinued'."` |

### Gathering Protocol

1. If the user provides all 4 items in their initial request → proceed to Phase 1.
2. If any items are missing → ask a **single, consolidated question** listing all missing items. Do not ask one at a time.
3. If the user says "just use defaults" or equivalent → inform them that Items 1-3 are non-negotiable and request them. Item 4 can default to "None".

---

## Phase 0B — Edit Mode (When Modifying an Existing Prompt)

If the user asks to **edit** an existing prompt (not create a new one), use this reduced protocol instead of Phase 0A:

### Required Inputs for Edits

| # | Input | What to ask for |
|---|-------|-----------------|
| 1 | **Which prompt file** | Exact file path or section name (e.g., "cancer history prompt"). |
| 2 | **What to change** | Specific rules to add, remove, or modify. |
| 3 | **Why** | The clinical scenario, bug, or user feedback that motivated the change. |

### Edit Safety Rules

1. **Read the full prompt** before making changes (Production Rule 22).
2. **Never remove anti-hallucination guards** unless replacing them with stronger ones.
3. **Preserve the factory function signature** — do not change `(formatSettings, extraUserInstruction)` parameters for pipeline prompts.
4. **Assess if JSON output schema changes are needed.** Changing field names or types is a **BREAKING CHANGE** that requires updating:
   - The validation schema in `validation_schema/<section>.schema.ts`
   - The agent node in `agent_nodes/<section>.ts`
   - The `outputStructures.ts` entry (if in edit journey)
   - The `preValidationPrompt.ts` section list (if in edit journey)
5. **After editing:** Run the applicable Skill 19 checklist (§8, and §18 if pipeline) and Phase 5A dry-run.

### After Gathering → Skip to Phase 1 (Research), then Phase 2 (but edit the existing file instead of creating a new one), then Phase 5.

---

## Phase 0C — Determine Prompt Type (Layer Decision Gate)

After gathering inputs (Phase 0A or 0B), classify the prompt:

| If the prompt... | Then apply... | Phases to follow |
|-----------------|---------------|------------------|
| Processes transcripts, uses `formatSettings`, will be wired into `stateGraphWorkflow.ts` | **Layer 1 + Layer 2** (full pipeline pattern) | All phases |
| Is a standalone medical utility (lab analysis, risk scoring, drug interaction check, clinical classification, etc.) | **Layer 1 only** (universal rules) | Phases 0A → 0C → 1 → 2 → 5 → 5A (skip Phases 3, 4) |

### What This Controls Downstream

| Aspect | Layer 1 Only | Layer 1 + Layer 2 |
|--------|-------------|-------------------|
| Function signature | Any export pattern (factory or static) | Must use `get<Name>Prompt(formatSettings, extraUserInstruction)` |
| Output format | Any valid JSON | HTML inside JSON with 4 format variants |
| Invocation method | Any LLM call method | `invokeWithValidationAndRetry` |
| Pipeline wiring | Not needed | Register in `stateGraphWorkflow.ts` |
| Checklist | §8 only | §8 + §18 |

---

## Phase 1 — Research Existing Patterns

### Strategy: Match by Domain, then by Output Shape

**Step 1 — Match by section domain:**

| If the new prompt is for... | Read these reference prompts |
|-----------------------------|------------------------------|
| A clinical section (HPI, Exam, ROS, History) | `hpiChiefAllergiesPrompt.ts`, `examPrompt.ts`, `reviewOfSystemPrompt.ts` |
| Coding / billing | `cptCodePrompt.ts`, `billingTablePrompt.ts`, `icdCodePrompt.ts` |
| Chatbot / edit journey | `chatBotPrompt.ts`, `preValidationPrompt.ts`, `editJourneyPrompt.ts` |
| Validation / post-processing | `postValidationPrompt.ts`, `preValidationPrompt.ts` |
| Template processing | `suggestedTemplatesPrompt.ts`, `generateTemplatePrompt.ts` |

**Step 2 — Match by output shape:**

| If the output is... | Follow this pattern |
|---------------------|---------------------|
| `{ summary, html }` (two fields) | `cancerHistoryPrompt.ts`, `socialHistoryPrompt.ts` |
| `{ singleField }` (one field) | `examPrompt.ts`, `reviewOfSystemPrompt.ts` |
| Array of objects `[ { title, content, ... } ]` | `impression&planPrompt.ts`, `billingTablePrompt.ts` |
| Classification / routing object | `preValidationPrompt.ts`, `processAndLabelUserInputPrompt.ts` |

**Step 3 — Read the agent node + validation schema** for the matched prompt (not just the prompt file itself):
```
src/common/latest-agents/agent_nodes/<matched>.ts
src/common/latest-agents/validation_schema/<matched>.schema.ts
```

**Step 4 — Check `outputStructures.ts`** for any existing entry for this section.

---

## Phase 2 — Write the Prompt File

Create the new prompt file at:
```
src/common/latest-agents/prompts/<sectionName>Prompt.ts
```

### Mandatory Structure (from Skill 19)

The prompt MUST follow this exact section order:

```
1. Role & Persona          — "You are a [clinical role] specializing in [domain]."
2. Input Declaration       — List every data source with XML tag names.
3. Extraction / Task Rules — What to extract, what to exclude, perspective rules.
4. Anti-Hallucination      — At minimum: exclusion list + omission-over-default.
5. Format & Personalization— formatRules object + PERSONALIZED SETTINGS block.
6. Instruction Priority    — Pre-Processing > Personalized > Template > Transcript.
7. JSON Output Schema      — Exact schema from user's Item 2.
8. Strict Rules Footer     — Final "Do NOT" constraints.
```

### Token Budget Check (from Skill 19 §12)

After writing the prompt, estimate the token count:
- < 16KB → ✅ proceed
- 16KB–32KB → ⚠️ review for deduplication
- \> 32KB → 🔴 split using `sections_list` pattern (see `hpiChiefAllergiesPrompt.ts`)

### Implementation Checklist

- [ ] Export as factory function: `export const get<Name>Prompt = (formatSettings, extraUserInstruction) => { ... }`
- [ ] Include `formatRules` object with all 4 format variants (`paragraph`, `bullet points`, `extended paragraph`, `short bullet points`)
- [ ] Tone uses `formatSettings?.tone ?? "Professional"`
- [ ] Format uses `formatSettings?.<sectionKey>?.toLowerCase() ?? "bullet points"`
- [ ] Custom instructions sandboxed with ALLOWED/PROTECTED pattern
- [ ] "Do NOT infer, assume, or fabricate" is explicitly stated
- [ ] "The final note MUST not include any placeholders" is explicitly stated
- [ ] JSON schema at the end matches the user's provided schema exactly
- [ ] All HTML tag closure rules documented
- [ ] Family history / patient-only exclusion rules (if applicable)
- [ ] PHI firewall in place (no static PHI in prompt text)
- [ ] Applicable edge cases from Skill 19 §13 addressed

---

## Phase 3 — Write / Update the Agent Node

If a new agent node is needed, create it at:
```
src/common/latest-agents/agent_nodes/<sectionName>.ts
```

### Agent Node Checklist

- [ ] Imports the prompt factory function from the prompts directory
- [ ] Defines a **validation schema** in `validation_schema/<section>.schema.ts` using the dual-mode pattern:
  ```typescript
  export function get<Section>Schema(options: { type: "validation" | "llm" }) {
    const coreSchema = { /* JSON Schema */ };
    if (options.type === "validation") return coreSchema;
    return { type: "json_schema", name: "<section>", schema: coreSchema };
  }
  ```
- [ ] Calls `invokeWithValidationAndRetry` with:
  - `prompt`: the generated prompt string
  - `transcriptionData`: the user's data payload (XML-tagged format)
  - `validationSchema`: `get<Section>Schema({ type: "validation" })`
  - `responseFormat`: `get<Section>Schema({ type: "llm" })`
  - `maxRetries`: 2–3
  - `modelName`: appropriate model selection
  - `isPayloadSet`: true
- [ ] Uses `emitNodeStatus()` for socket updates: `STARTED`, `SUCCESS`, `FAILED`
- [ ] Uses `handleNodeError()` for centralized error handling
- [ ] Returns data in `{ <Key>Detail: [parsedResult] }` format for state concatenation
- [ ] Handles the `isContinueRecording` / `PastDataDetails` pattern for re-runs

---

## Phase 4 — Update Supporting Files

### 4A — Pipeline Wiring (if this prompt is part of the documentation pipeline)

Follow Skill 19 §11 — Pipeline Integration:

1. **`stateGraphWorkflow.ts`:**
   - Import the agent node
   - Add state annotation with `stateConcatenation` reducer
   - Register node with `withFailureHandler`
   - Wire edges (usually `→ finalize`)
   - Connect to the appropriate routing condition

2. **Verify parallel vs sequential placement:**
   - Clinical section nodes (HPI, ROS, Exam, histories) run in **parallel**
   - Billing/coding nodes run **sequentially** after impression&plan
   - Your node must be wired accordingly

### 4B — Edit Journey Integration (if this section should be editable via chatbot)

1. **`preValidationPrompt.ts`** → Add to `MAIN SECTIONS AND THEIR SUB-SECTIONS`:
   ```
   - "MY_SECTION": ["my_field_1", "my_field_2"]
   ```
2. **`outputStructures.ts`** → Add to `EditJsonStructures`:
   ```typescript
   MY_SECTION: `{ "my_field_1": "...", "my_field_2": "..." }`
   ```
3. **`editJourney` agent node** → Register the section handler

---

## Phase 5 — Verify

1. **Type-check** — Run `npx tsc --noEmit` to ensure no TypeScript errors.
2. **Prompt review** — Re-read the final prompt against the Skill 19 checklist (§8).
3. **Schema cross-check** — Verify the validation schema exactly matches the JSON output schema in the prompt text.

---

## Phase 5A — Dry-Run Validation

> This phase catches clinical logic errors that type-checking cannot.

1. **Create 3 synthetic test transcripts** (NO real PHI):

   | # | Scenario | Purpose |
   |---|----------|---------|
   | 1 | **Happy path** | All expected fields present in transcript | 
   | 2 | **Sparse** | Most data missing — tests omission rules |
   | 3 | **Adversarial** | Family history mixed in, colloquial terms, abbreviations — tests exclusion + translation rules |

2. **For each test transcript:**
   - Mentally trace the prompt's extraction rules
   - Predict the exact JSON output
   - Verify the predicted output matches the Joi/JSON Schema

3. **If any prediction reveals a gap** → go back to Phase 2 and add the missing rule before proceeding.

4. **Document the 3 test scenarios** as code comments in the agent node file for future regression reference.

---

## Quick Reference — What to Ask the User

### For New Prompts

If a user says _"Create a new prompt for X"_, respond with:

> Before I write the prompt, I need 4 things from you:
>
> 1. **Input:** What data/variables will this prompt receive? (transcript, past data, templates, reference codes, etc.)
> 2. **Output Schema:** What's the exact JSON structure the LLM should return? (TypeScript interface or JSON example)
> 3. **Behavior:** What should the LLM extract, calculate, or transform? (The clinical logic in plain English)
> 4. **Edge Cases:** Any special medical rules, exclusions, or domain-specific instructions? (or "None")

### For Editing Existing Prompts

If a user says _"Edit the X prompt"_ or _"Fix the X prompt"_, respond with:

> To modify the prompt safely, I need 3 things:
>
> 1. **Which prompt?** The file name or section name.
> 2. **What to change?** The specific rules to add, remove, or modify.
> 3. **Why?** The clinical scenario or bug that triggered this change.
