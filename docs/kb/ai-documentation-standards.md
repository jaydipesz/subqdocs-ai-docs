# AI-oriented documentation standards (SubQDocs)

**Last reviewed:** 2026-04-01 · **Doc set:** 1

Single place for **how** we structure docs for humans, **Cursor**, and **Antigravity**: principles, **decision/bootstrap order**, **stable `@` paths**, and tool roles.  
**Hard rules** are not duplicated here — use [`.agents/rules/production-rules.md`](../../.agents/rules/production-rules.md) and [`project/overview.md`](./project/overview.md).

---

## Principles

1. **Single source of truth** — Canonical narrative and landmarks live under [`docs/kb/`](./README.md). Step-by-step procedures live under [`docs/skills/`](../skills/INDEX.md) with an index. Do not add parallel root-level `AI-CONTEXT` trees.
2. **Roles** — **KB** = *what/where* (domains, modules, conventions). **Skills** = *how* (checklists). **Workflows** ([`.agents/workflows/`](../../.agents/workflows/)) = *when* to run which phase (CW*, TOOL-*), not a second KB.
3. **Hard rules** — One authoritative [`production-rules.md`](../../.agents/rules/production-rules.md). Cursor rules link to it; they do not fork the full list.
4. **Thin tool layers** — [`.cursor/rules/`](../../.cursor/rules/) = IDE guardrails (globs, always-apply pointers). [`.agents/`](../../.agents/README.md) = workflows + same production rules.
5. **Structure** — Short sections, tables for “if X then open Y,” trigger phrases in [`docs/skills/INDEX.md`](../skills/INDEX.md), numbered skill files for unambiguous references.
6. **Maintenance** — After structural code changes, follow [`maintenance.md`](./maintenance.md). Out-of-scope cases are defined there.
7. **Scoped Cursor rules** — Area-specific `.mdc` files under [`.cursor/rules/`](../../.cursor/rules/) (e.g. `subqdocs-backend-modules.mdc`, `subqdocs-frontend-domains.mdc`) use **narrow globs**; always-apply files stay minimal. Point at INDEX + conventions; avoid one giant always-on blob of KB text.
8. **Session hygiene** — [`.gemini/`](../../.gemini/README.md) is draft/session artifacts only; promote validated content into `docs/kb/` or `docs/skills/`.
9. **Alignment** — Cursor and Antigravity use the same bootstrap/decision order (this document).
10. **LLM-friendly content** — Prefer explicit negatives (“never…”), concrete file paths and symbols, and version-sensitive stack facts in KB docs.
11. **Discoverability** — Workspace map: [`docs/README.md`](../README.md). Antigravity index: [`.agents/README.md`](../../.agents/README.md).
12. **Anti-patterns** — Do not paste full `production-rules` into `.cursor/rules/`. Do not let skills replace the KB wholesale (link + checklist). Avoid huge always-apply rules; prefer links + scoped rules.
13. **Optional polish** — Glossary or “last reviewed” stamps only if the team needs them; not required for day-to-day work.
14. **Both tools** — Respect PHI, tenancy, and never-do items in [`project/overview.md`](./project/overview.md) and [`production-rules.md`](../../.agents/rules/production-rules.md).

---

## Bootstrap and decision order

### Decision hierarchy

Apply in this order:

1. [`.agents/rules/production-rules.md`](../../.agents/rules/production-rules.md) — non-negotiable; always wins.
2. Industry best practice — TypeScript, React, Node, Sequelize, SQL.
3. Codebase patterns — naming, libraries, reuse.
4. [`docs/kb/`](./README.md) and [`docs/skills/INDEX.md`](../skills/INDEX.md) — project-specific *what/where* and *how*.
5. Child workflows — execution sequence when using [FEATURE-IMPLEMENTATION-MASTER](../../.agents/workflows/FEATURE-IMPLEMENTATION-MASTER.md) or a [TOOL-*](../../.agents/workflows/) playbook.

When (2) and (3) conflict, prefer best practice and state the conflict.

### Default bootstrap (before large feature work)

- Read [`production-rules.md`](../../.agents/rules/production-rules.md) and [`anti-patterns.md`](./anti-patterns.md).
- Read [`project/overview.md`](./project/overview.md), [`backend/conventions.md`](./backend/conventions.md), and [`frontend/conventions.md`](./frontend/conventions.md).
- Scan [`docs/skills/INDEX.md`](../skills/INDEX.md) for trigger phrases that match the task.
- **Full-stack / Figma features:** follow [FEATURE-IMPLEMENTATION-MASTER.md](../../.agents/workflows/FEATURE-IMPLEMENTATION-MASTER.md) (CW1–CW5A).
- **Isolated operations:** use the matching [TOOL-add-*.md](../../.agents/workflows/) workflow and the skill files it references.

Do not invent new skills during feature work; skills are authored under `docs/skills/` and indexed in [`INDEX.md`](../skills/INDEX.md).

**Optional — pre-code planning only:** If the user is driving the **Task Planning Agent** (structured `plans/*.md` with approval gates before coding), read [`cursor-task-planning-agent.md`](./cursor-task-planning-agent.md) for artifact paths, MCP notes, and handoff to [FEATURE-IMPLEMENTATION-MASTER](../../.agents/workflows/FEATURE-IMPLEMENTATION-MASTER.md). This does not replace the default bootstrap above for implementation work.

### Master workflow bootstrap

_Applies to [FEATURE-IMPLEMENTATION-MASTER](../../.agents/workflows/FEATURE-IMPLEMENTATION-MASTER.md)._

Use this **after** intake is complete for the master workflow. The **Child Workflows** table lives only in [FEATURE-IMPLEMENTATION-MASTER.md](../../.agents/workflows/FEATURE-IMPLEMENTATION-MASTER.md).

1. Read [`.agents/rules/production-rules.md`](../../.agents/rules/production-rules.md).
2. Read `docs/kb/project/overview.md`, `docs/kb/anti-patterns.md`, `docs/kb/backend/conventions.md`, `docs/kb/frontend/conventions.md`.
3. Read [`docs/skills/INDEX.md`](../skills/INDEX.md).
4. Read every child workflow file listed in the **Child Workflows** table in [FEATURE-IMPLEMENTATION-MASTER.md](../../.agents/workflows/FEATURE-IMPLEMENTATION-MASTER.md).
5. Check INDEX files against what the feature requires. If a skill is missing, **stop and report**: “Missing skill for [operation]. Cannot proceed without: [specific skill requirement].” Do not create skills during the master workflow — skills must be authored and reviewed separately.
6. Confirm: **“Context loaded. Skills verified. Inputs collected. Missing skills: [list or none].”**

Do not write any code or produce any artifact until intake and this bootstrap are both complete.

---

## Stable paths (repo root)

Use these in Cursor chat or Composer for consistent context (copy paths relative to repo root):

| Path |
|------|
| `docs/README.md` |
| `docs/kb/README.md` |
| `docs/kb/ai-documentation-standards.md` (this file) |
| `docs/kb/cursor-task-planning-agent.md` |
| `docs/kb/project/overview.md` |
| `docs/kb/project/architecture.md` |
| `docs/kb/frontend/conventions.md` |
| `docs/kb/backend/conventions.md` |
| `docs/kb/anti-patterns.md` |
| `docs/kb/subqdocs-frontend/INDEX.md` |
| `docs/kb/subqdocs-backend/INDEX.md` |
| `docs/kb/maintenance.md` |
| `docs/kb/hooks/README.md` |
| `docs/skills/INDEX.md` |
| `.cursor/AGENT.md` |
| `.cursor/rules/project-workflow-alignment.mdc` |
| `.cursor/rules/documentation-maintenance.mdc` |
| `.cursor/rules/subqdocs-backend-modules.mdc` |
| `.cursor/rules/subqdocs-backend-sequelize.mdc` |
| `.cursor/rules/subqdocs-backend-core.mdc` |
| `.cursor/rules/subqdocs-frontend-domains.mdc` |
| `.cursor/rules/subqdocs-frontend-ui.mdc` |
| `.cursor/rules/subqdocs-frontend-core.mdc` |
| `.agents/README.md` |
| `.agents/workflows/FEATURE-IMPLEMENTATION-MASTER.md` |
| `.agents/workflows/WORKFLOW-FORMAT.md` |
| `.agents/rules/production-rules.md` |
| `docs/kb/subqdocs-voice-to-voice/INDEX.md` |

---

## Cursor vs Antigravity

| Tool | Loads / reads | Use for |
|------|----------------|---------|
| **Cursor** | [`.cursor/rules/*.mdc`](../../.cursor/rules/); you can `@` files under `docs/` | Day-to-day editing with project rules and KB in context. |
| **Antigravity** | [`.agents/workflows/`](../../.agents/workflows/), [`.agents/rules/production-rules.md`](../../.agents/rules/production-rules.md); follow bootstrap above | Orchestrated flows: full feature delivery, PR review, scaffolding TOOLs. |

The knowledge base lives only under **`docs/kb/`** (not under `.cursor/`). Other tools do not load `.cursor/rules` unless configured — treat them as **Cursor guardrails**.

---

## Related

- [`docs/README.md`](../README.md) — folder map and read order.
- [`docs/kb/maintenance.md`](./maintenance.md) — what to update when code changes.
