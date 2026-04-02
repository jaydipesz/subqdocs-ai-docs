---
name: Pipeline Prompt Integration
description: >
  Mandatory skill for writing or modifying prompts that are part of the SubQDocs
  transcript-processing pipeline. Covers factory function signatures, format/tone
  personalization, HTML output rules, validation & retry integration, token budget,
  and a quick-start template. For universal medical prompt rules, see Skill 19.
  For agent node wiring, see Skill 20.
---

# Skill 21 — Pipeline Prompt Integration

> **When to load:** Before creating, editing, or reviewing ANY prompt file inside
> `src/common/latest-agents/prompts/` that is wired into the SubQDocs pipeline
> (i.e., invoked through `invokeWithValidationAndRetry` and registered in `stateGraphWorkflow.ts`).
>
> **Skip this skill** if you're building a standalone medical utility prompt
> (e.g., drug interaction checker, risk scoring, report analysis) that is NOT
> part of the documentation pipeline. Use Skill 19 alone for those.
>
> **Depends on:** Skill 19 (Universal Medical Prompt Rules) — always load Skill 19 first.

---

## 1. Format & Personalization Settings

Pipeline prompts support user-configurable formatting and tone. The prompt must include:

```
## PERSONALIZED SETTINGS - MANDATORY COMPLIANCE

  ### Tone Configuration
  **REQUIRED TONE: ${formatSettings?.tone ?? "Professional"}**
  - All responses MUST strictly follow this tone.

  ### Format Configuration
  **REQUIRED FORMAT: ${formatSettings?.[sectionKey] ?? "bullet points"}**
  - Output MUST follow ONLY the specified format within JSON field values.
```

**Rules:**
- Always provide a `formatRules` object mapping format names to HTML examples.
- The four standard format variants are: `"paragraph"`, `"bullet points"`, `"extended paragraph"`, `"short bullet points"`.
- Personalized settings override templates. Templates are content guides only.
- Custom instructions must use the standard override block:
  ```
  ## CUSTOM INSTRUCTIONS - HIGHEST PRIORITY
  **[AUTHORITY LEVEL: OVERRIDE]**
    - **ALLOWED:** Content, style, wording, additional info, presentation
    - **PROTECTED:** JSON structure, field names, data types, core rules
  ```

---

## 2. Function Signature Pattern

All pipeline prompts MUST be exported as **factory functions**, not raw string constants.

```typescript
// ✅ CORRECT — configurable factory
export const getMyPrompt = (formatSettings, extraUserInstruction) => {
  const formatRules = {
    "paragraph": `...`,
    "bullet points": `...`,
    "extended paragraph": `...`,
    "short bullet points": `...`,
  };

  return `You are a ...
  ...
  ${formatRules[formatSettings?.mySection?.toLowerCase() ?? "bullet points"]}
  `;
};

// ❌ WRONG — static string (legacy pattern, do not use for new prompts)
export const MyPrompt = `...`;
```

**Rules:**
- The function takes `formatSettings` (object) and `extraUserInstruction` (object) as parameters.
- Additional parameters (e.g., `sections_list`) are allowed when the prompt supports selective section generation.
- Legacy static `export const` prompts are kept for backward compatibility but must be marked with `// NOT IN USE NOW`.
- The function name convention is `get<SectionName>Prompt`.

---

## 3. Instruction Priority Hierarchy

Every pipeline prompt MUST declare this priority chain:

```
## [**IMPORTANT**] Instruction Priority
1. **Pre-Processing Requirements** (if applicable) — highest priority
2. **Personalized Settings** — tone, format, custom instructions
3. **Template** — content guide only, not formatting authority
4. **Provided Data** — sole source of factual truth
```

---

## 4. HTML Output Rules

Pipeline prompts produce HTML content within JSON fields. These rules must be followed:

1. **All HTML tags must be properly opened and closed.** Unclosed tags break downstream rendering.
2. **No raw placeholders in final output.** `{location}`, `{finding}`, `{{item}}` must never appear in the response.
3. **No logic/template syntax in output.** Strip `<< row >>`, `<< if >>`, `{{ }}`, `{% %}` constructs.
4. **Empty sections must be completely suppressed.** Do not output section headers (e.g., `<p><strong>Procedure:</strong></p>`) with no content below them.
5. **Inline styling** should use the established patterns:
   - Found value: `<span style="color: #37B0F6; font-weight: bold;">value</span>` (Blue)
   - Missing value: `<span style="color: Crimson; font-weight: bold;">placeholder_name</span>` (Red)

---

## 5. Validation & Retry Integration

Pipeline prompts are invoked through `invokeWithValidationAndRetry`, which supports:

| Parameter | Purpose |
|-----------|---------|
| `validationSchema` | JSON Schema that validates the LLM's response |
| `maxRetries` | Number of retry attempts if validation fails (default: 3) |
| `responseFormat` | OpenAI structured output format specification |
| `modelName` | Model to use (default: `gpt-5-mini-2025-08-07`) |
| `isPayloadSet` | When `true`, prompt and data are sent as separate system/user messages |

**When writing a pipeline prompt, always define a corresponding validation schema** in `src/common/latest-agents/validation_schema/<section>.schema.ts`. Use the dual-mode pattern:

```typescript
export function get<Section>Schema(options: { type: "validation" | "llm" }) {
  const coreSchema = { /* JSON Schema */ };
  if (options.type === "validation") return coreSchema;
  return { type: "json_schema", name: "<section>", schema: coreSchema };
}
```

---

## 6. Token Budget Awareness

### Size Thresholds

| Prompt Size | Action |
|-------------|--------|
| < 4,000 tokens (~16KB) | ✅ Normal — single prompt, no concerns |
| 4,000–8,000 tokens (~32KB) | ⚠️ Review — consider if all rules are necessary, look for deduplication |
| > 8,000 tokens (~32KB+) | 🔴 Must split into sub-prompts or use selective section generation |

### Estimation Rule of Thumb
- **1 token ≈ 4 characters** of English text.
- A 10KB prompt ≈ 2,500 tokens of instruction.
- The transcript/payload adds tokens **on top** of the prompt instruction tokens.
- The total (prompt + payload + expected response) must fit within the model's context window.

### When to Split a Prompt
- If the prompt handles **multiple independent sections** (e.g., HPI + Chief Complaint + Allergies), accept a `sections_list` parameter to generate only the requested sections.
- **Existing pattern:** See `hpiChiefAllergiesPrompt.ts` → `final_sections(sections_list)` function, which conditionally includes only the requested section instructions and output fields.
- **Never duplicate** the same rule in multiple prompt sections — factor shared rules into a top-level block.

---

## 7. Quick-Start Pipeline Template

For simple transcript-processing prompts, copy this skeleton and fill in the `[PLACEHOLDERS]`:

```typescript
export const get[Section]Prompt = (formatSettings, extraUserInstruction) => {
  const formatRules = {
    "paragraph": `<p>[paragraph HTML example]</p>`,
    "bullet points": `<ul><li>[bullet HTML example]</li></ul>`,
    "extended paragraph": `<p>[detailed paragraph HTML example]</p>`,
    "short bullet points": `<ul><li>[short bullet, max 3-5 words]</li></ul>`,
  };

  return `You are a clinical documentation assistant specializing in dermatology.
Your tone should be ${formatSettings?.tone ?? "Professional"}.

## INPUT:
- <transcript>: Doctor-patient conversation.
[Add other inputs as needed]

## TASK:
Extract [WHAT TO EXTRACT] from the transcript.

## RULES:
- Document ONLY information explicitly stated in the transcript.
- Do NOT infer, assume, or fabricate any data.
- If information is not present, OMIT it entirely.
- Use first-person perspective for physician actions.
- Use third-person for patient-reported information.
- Ensure all HTML tags are properly opened and closed.
- The final note MUST not include any placeholders.

## PERSONALIZED SETTINGS - MANDATORY COMPLIANCE
  ### Tone: ${formatSettings?.tone ?? "Professional"}
  ### Format: ${formatSettings?.[sectionKey]?.toLowerCase() ?? "bullet points"}

${extraUserInstruction?.[sectionKey] ? `
## CUSTOM INSTRUCTIONS - HIGHEST PRIORITY
  **[AUTHORITY LEVEL: OVERRIDE]**
  - **ALLOWED:** Content, style, wording, additional info, presentation
  - **PROTECTED:** JSON structure, field names, data types, core rules
  \`\`\`
  ${extraUserInstruction?.[sectionKey]}
  \`\`\`
` : ''}

## [**IMPORTANT**] Instruction Priority
1. **Personalized Settings** — tone, format, custom instructions
2. **Template** — content guide only
3. **Provided Data** — sole source of truth

## Return your output as valid JSON:
{
  "[field_1]": "<description of expected value or fallback HTML>",
  "[field_2]": "<description of expected value or fallback HTML>"
}

STRICT RULES:
- Do NOT infer or assume.
- Do NOT include [DOMAIN-SPECIFIC EXCLUSIONS].
- Ensure all HTML tags are valid and closed.
- Output only valid JSON — no extra keys, comments, or text.`;
};
```

> **Note:** This template covers the most common pipeline case. For complex prompts
> (multi-diagnosis, template-aware, or cascading), start from the full
> architecture in Skill 19 §2 + this skill's §1–§3 instead.

---

## 8. Pipeline Checklist — Additional Items for Pipeline Prompts

> Use this **in addition to** the Universal Checklist (Skill 19 §8) when building a pipeline prompt.

- [ ] **Format factory:** Exported as `get<Name>Prompt(formatSettings, extraUserInstruction)`.
- [ ] **Format rules object:** Contains all 4 standard format variants.
- [ ] **Tone configurable:** Uses `formatSettings?.tone ?? "Professional"`.
- [ ] **Custom instruction sandbox:** Uses ALLOWED/PROTECTED pattern.
- [ ] **Instruction priority:** Declared in order (Pre-Processing > Personalized > Template > Data).
- [ ] **HTML rules:** Closed tags, empty section suppression, no logic syntax.
- [ ] **Validation schema:** Dual-mode schema in `validation_schema/<section>.schema.ts`.
- [ ] **Token budget:** Prompt size reviewed against thresholds (§6). Split if > 32KB.
- [ ] **Pipeline wiring:** Node registered in `stateGraphWorkflow.ts` if applicable (Skill 20).

---

## 9. Pipeline-Specific Mistakes to Avoid

| Mistake | Correct Pattern |
|---------|-----------------|
| Using `export const Prompt = \`...\`` for new prompts | Use `export const getPrompt = (formatSettings, extraUserInstruction) => \`...\`` |
| Leaving empty section headers in output | Add "Empty Section Suppression" rule |
| Hardcoding tone as "Professional" | Use `formatSettings?.tone ?? "Professional"` |
| Forgetting to close HTML tags | Add "Ensure all HTML tags are properly opened and closed" |
| No validation schema for retry loop | Always define dual-mode schema in `validation_schema/` |
