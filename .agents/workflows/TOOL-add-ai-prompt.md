---
description: Generate a new AI/LLM prompt for the SubQDocs clinical documentation pipeline. Enforces mandatory data-gathering before writing, ensures prompt adheres to Skill 19, and validates the output end-to-end.
---

# TOOL — Add AI Prompt

> **Trigger:** User asks to "create a prompt", "add an AI prompt", "new LLM prompt",
> "write a system prompt", "edit a prompt", or similar.

---

## Pre-requisite

Load and read `docs/skills/19-prompt-engineering.md` before ANY work. All Skill 19 rules are **mandatory**.

---

## Phase 0A — Mandatory Information Gathering (DO NOT SKIP)

**MUST NOT write code until ALL FOUR inputs are provided.** Ask explicitly for any missing items.

| # | Input | What to ask for | Example |
|---|-------|-----------------|------------|
| 1 | **The Input** | All data/variables passed to the prompt (transcript, past data, format settings, templates, etc.) | `"Receives: (1) <transcript>, (2) <PAST_DATA> with prior medications, (3) formatSettings with tone/format."` |
| 2 | **Output Schema** | Exact TypeScript interface or JSON schema. Every field, type, and description. | `{ "medications_html": string, "new_medications_html": string }` |
| 3 | **Behavior** | What the LLM must extract, classify, or transform. Specific clinical logic. | `"Extract past/current medications. Separate new from prior. Include strength, frequency, instructions."` |
| 4 | **Extra Instructions** | Domain-specific edge cases, exclusions, or special behaviors. | `"Exclude OTC supplements unless explicitly prescribed. Mark discontinued meds as 'Discontinued'."` |

### Gathering Protocol

1. All 4 provided → proceed to Phase 1.
2. Any missing → ask a **single consolidated question** listing all missing items.
3. "Use defaults" → Items 1-3 are non-negotiable. Item 4 can default to "None".

---

## Phase 0B — Edit Mode (Modifying an Existing Prompt)

Use this instead of Phase 0A when editing:

| # | Input | What to ask for |
|---|-------|-----------------|
| 1 | **Which prompt** | File path or section name |
| 2 | **What to change** | Rules to add, remove, or modify |
| 3 | **Why** | Clinical scenario, bug, or feedback motivating the change |

### Edit Safety Rules

1. Read the full prompt before editing (Production Rule 22).
2. Never remove anti-hallucination guards unless replacing with stronger ones.
3. Preserve factory function signature `(formatSettings, extraUserInstruction)`.
4. Schema field changes = **BREAKING CHANGE** → update: `validation_schema/<section>.schema.ts`, `agent_nodes/<section>.ts`, `outputStructures.ts`, `preValidationPrompt.ts`.
5. After editing: run Skill 19 checklists (§8 + §18 if pipeline) and Phase 5A dry-run.

**After Gathering →** Phase 1 → Phase 2 (edit existing file) → Phase 5 + Phase 5A.

---

## Phase 0C — Prompt Type (Layer Decision Gate)

| If the prompt... | Apply... | Phases |
|-----------------|----------|--------|
| Processes transcripts, uses `formatSettings`, wired into `stateGraphWorkflow.ts` | **Layer 1 + 2** (full pipeline) | All phases |
| Standalone medical utility (lab analysis, risk scoring, etc.) | **Layer 1 only** | 0A → 0C → 1 → 2 → 5 → 5A (skip 3, 4) |

| Aspect | Layer 1 Only | Layer 1 + 2 |
|--------|-------------|-------------|
| Signature | Any export | `get<Name>Prompt(formatSettings, extraUserInstruction)` |
| Output | Any JSON | HTML inside JSON with 4 format variants |
| Invocation | Any LLM call | `invokeWithValidationAndRetry` |
| Wiring | None | Register in `stateGraphWorkflow.ts` |
| Checklist | §8 | §8 + §18 |

---

## Phase 1 — Research Existing Patterns

**Step 1 — Match by domain:**

| Domain | Reference prompts |
|--------|-------------------|
| Clinical section (HPI, Exam, ROS, History) | `hpiChiefAllergiesPrompt.ts`, `examPrompt.ts`, `reviewOfSystemPrompt.ts` |
| Coding / billing | `cptCodePrompt.ts`, `billingTablePrompt.ts`, `icdCodePrompt.ts` |
| Chatbot / edit journey | `chatBotPrompt.ts`, `preValidationPrompt.ts`, `editJourmeyPrompt.ts` |
| Validation / post-processing | `postValidationPrompt.ts`, `preValidationPrompt.ts` |
| Template processing | `suggestedTemplatesPrompt.ts`, `generateTemplatePrompt.ts` |

**Step 2 — Match by output shape:**

| Output shape | Pattern reference |
|--------------|-------------------|
| `{ summary, html }` | `cancerHistoryPrompt.ts`, `socialHistoryPrompt.ts` |
| `{ singleField }` | `examPrompt.ts`, `reviewOfSystemPrompt.ts` |
| `{ field_a, field_b }` (multi-field) | `medicationPrompt.ts` |
| `[ { title, content, ... } ]` (array) | `impression&planPrompt.ts`, `billingTablePrompt.ts` |
| Classification / routing object | `preValidationPrompt.ts`, `processAndLabelUserInputPrompt.ts` |

**Step 3 —** Read the agent node + validation schema for matched prompt:
`agent_nodes/<matched>.ts` + `validation_schema/<matched>.schema.ts`

**Step 4 —** Check `outputStructures.ts` for existing entries.

---

## Phase 2 — Write the Prompt File

Create at: `src/common/latest-agents/prompts/<sectionName>Prompt.ts`

### Mandatory Section Order (Skill 19)

```
1. Role & Persona          — "You are a [role] specializing in [domain]."
2. Input Declaration       — List every data source with XML tag names.
3. Extraction / Task Rules — What to extract/exclude, perspective rules.
4. Anti-Hallucination      — Exclusion list + omission-over-default minimum.
5. Format & Personalization— formatRules object + PERSONALIZED SETTINGS block.
6. Instruction Priority    — Pre-Processing > Personalized > Template > Transcript.
7. JSON Output Schema      — Exact schema from user's Item 2.
8. Strict Rules Footer     — Final "Do NOT" constraints.
```

### Token Budget (Skill 19 §12)

- < 16KB → ✅ | 16–32KB → ⚠️ review | > 32KB → 🔴 split via `sections_list` pattern

### Checklist

- [ ] Factory function: `export const get<Name>Prompt = (formatSettings, extraUserInstruction) => { ... }`
- [ ] `formatRules` with 4 variants: `paragraph`, `bullet points`, `extended paragraph`, `short bullet points`
- [ ] Tone: `formatSettings?.tone ?? "Professional"`
- [ ] Format: `formatSettings?.<sectionKey>?.toLowerCase() ?? "bullet points"`
- [ ] Custom instructions sandboxed with ALLOWED/PROTECTED pattern
- [ ] "Do NOT infer, assume, or fabricate" stated
- [ ] "Must not include any placeholders" stated
- [ ] JSON schema matches user's schema exactly
- [ ] HTML tag closure rules documented
- [ ] Family history / patient-only exclusion (if applicable)
- [ ] PHI firewall (no static PHI in prompt text)
- [ ] Edge cases from Skill 19 §6 addressed

---

## Phase 3 — Write / Update Agent Node

Create at: `src/common/latest-agents/agent_nodes/<sectionName>.ts`

### Checklist

- [ ] Import prompt factory from prompts directory
- [ ] Dual-mode validation schema in `validation_schema/<section>.schema.ts`:
  ```typescript
  export function get<Section>Schema(options: { type: "validation" | "llm" }) {
    const coreSchema = { /* JSON Schema */ };
    if (options.type === "validation") return coreSchema;
    return { type: "json_schema", name: "<section>", schema: coreSchema };
  }
  ```
- [ ] Extract settings via `getSectionSettingsForName(sectionSettingsData, "Display Name")` from `@utils/common.utils`:
  ```typescript
  const setting = getSectionSettingsForName(sectionSettingsData, "Section Name");
  const formatSettings = { my_section: setting.format, tone: setting.tone };
  const extraUserInstruction = { my_section: setting.custom_instructions };
  ```
- [ ] Call `invokeWithValidationAndRetry` with: `prompt`, `transcriptionData`, `validationSchema` (validation mode), `responseFormat` (llm mode), `maxRetries: 2–3`, `modelName`, `isPayloadSet: true`
- [ ] `emitNodeStatus()` for socket: `STARTED`, `SUCCESS`, `FAILED`
- [ ] `handleNodeError()` for centralized error handling
- [ ] Return `{ <Key>Detail: [parsedResult] }` for state concatenation
- [ ] Handle `isContinueRecording` / `PastDataDetails` re-run pattern:
  ```typescript
  import { ReRunExtraInstructions } from "@common/latest-agents/prompts/reRunExtraInstructions";
  const prompt = PastDataDetails ? `${dynamicPrompt} /n ${ReRunExtraInstructions}` : dynamicPrompt;
  ```
  For re-runs: use `<OLD_GENERATED_DATA>`, `<OLD_TRANSCRIPT>`, `<EXTENDED_TRANSCRIPT>` XML instead of `<transcript>`

---

## Phase 4 — Update Supporting Files

### 4A — Pipeline Wiring

In `stateGraphWorkflow.ts`:
1. Import agent node
2. Add state annotation with `stateConcatenation` reducer
3. Register with `withFailureHandler`
4. Wire edges (usually `→ finalize`)
5. Connect to appropriate routing condition

**Placement:** Clinical nodes = **parallel** | Billing/coding = **sequential** after impression&plan

### 4B — Edit Journey Integration

1. **`preValidationPrompt.ts`** → Add to `MAIN SECTIONS AND THEIR SUB-SECTIONS`: `"MY_SECTION": ["field_1", "field_2"]`
2. **`outputStructures.ts`** → Add to `EditJsonStructures`: `MY_SECTION: '{ "field_1": "...", "field_2": "..." }'`
3. **`sectionInstructionPrompt.ts`** → Add to `SectionInstructions` map:
   ```typescript
   MY_SECTION: `## **HIGH PRIORITY EDIT INSTRUCTION** ${editInstruction}\n ${MySectionPrompt}`
   ```
   > Uses **legacy static** prompt export (e.g., `CancerHistoryPrompt`), not the factory function.

---

## Phase 5 — Verify

1. **Type-check:** `npx tsc --noEmit`
2. **Prompt review:** Re-read against Skill 19 checklist (§8).
3. **Schema cross-check:** Validation schema must exactly match prompt's JSON output schema.

---

## Phase 5A — Dry-Run Validation

> Catches clinical logic errors that type-checking cannot.

1. **Create 3 synthetic test transcripts** (NO real PHI):

| # | Scenario | Tests |
|---|----------|-------|
| 1 | **Happy path** | All fields present — correct extraction |
| 2 | **Sparse** | Most data missing — omission rules |
| 3 | **Adversarial** | Family history mixed in, colloquial terms, abbreviations — exclusion + translation |

2. Trace prompt rules → predict JSON output → verify against schema.
3. Gap found → return to Phase 2 and fix before proceeding.
4. Document scenarios as code comments in the agent node file.

---

## Quick Reference — What to Ask

**New prompt:** "I need 4 things: (1) **Input** — data/variables, (2) **Output Schema** — JSON structure, (3) **Behavior** — clinical logic, (4) **Edge Cases** — exclusions/special rules (or 'None')."

**Edit prompt:** "I need 3 things: (1) **Which prompt?** (2) **What to change?** (3) **Why?** (scenario/bug)"
