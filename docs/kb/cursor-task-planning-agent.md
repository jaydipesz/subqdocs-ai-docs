# Cursor Task Planning Agent (SubQDocs)

**Last reviewed:** 2026-04-02 · **Doc set:** 1

**Scope:** This workflow produces **pre-build planning documents** under `plans/` (master plan, FE plan, BE plan, QA plan, merged final doc). It does **not** replace product implementation rules or KB content in `docs/kb/subqdocs-*`.

**Authoritative narrative for this workflow is this file.** Rely on [AGENT.md](../../.cursor/AGENT.md) for exact steps and chat prompts; use this doc for discovery, MCP setup, and stable `@` paths.

---

## Artifact map

| Path | Role | When to read |
|------|------|----------------|
| [`../../.cursor/AGENT.md`](../../.cursor/AGENT.md) | Executable agent instructions: strict steps, approval gates, exact chat prompts | Before running the planning agent; follow verbatim in Agent mode |
| [`../../.cursor/mcp.json`](../../.cursor/mcp.json) | MCP servers (Figma, GitHub, filesystem) | Before relying on Figma/GitHub in planning; edit tokens and paths here |
| This file | Hub: workflow summary, MCP notes, smoke test, handoff | Bootstrap the planning workflow in chat |

Planning behavior is **not** duplicated in a root `.cursorrules` file; `@` [AGENT.md](../../.cursor/AGENT.md) in Agent mode (or follow it manually). There is no `plans/templates/` tree—create plan files directly under `plans/` per AGENT.md.

Generated outputs:

- `plans/00-master-plan.md`
- `plans/01-fe-plan.md`
- `plans/02-be-plan.md`
- `plans/03-qa-plan.md`
- `plans/04-final-task-doc.md`

---

## Workflow summary

1. **Read context first** — Figma, tickets/PRDs (MCPs), user-referenced files. See [AGENT.md](../../.cursor/AGENT.md) STEP 1.
2. **Master plan** — Write `plans/00-master-plan.md`. **Stop** until the user replies **approved** (exact approval line in AGENT.md).
3. **Three sub-plans** — After approval, write `01-fe`, `02-be`, `03-qa` under `plans/`. **Stop** until approved.
4. **Final doc** — After all three are approved, write `plans/04-final-task-doc.md`.

Never skip approval gates or assume approval. Always write files under `plans/`, not only chat text.

---

## MCP setup

- **Config file:** [`.cursor/mcp.json`](../../.cursor/mcp.json).
- **Figma:** In Figma, open Settings → Personal access tokens → generate a token; put it in place of `YOUR_FIGMA_TOKEN_HERE` in `mcp.json` (or in your team’s secret workflow).
- **GitHub:** Create a classic PAT with `repo` (or scopes your org allows); set `GITHUB_PERSONAL_ACCESS_TOKEN` in the `github` server `env` block.
- **Filesystem MCP** `args` path should be your monorepo root (currently `/home/jaydip-dhangru/projects`); change if your clone lives elsewhere.
- **Optional extra MCPs** (e.g. Notion, Jira): add new `mcpServers` entries per each provider’s MCP documentation; do not commit real secrets.
- **Secrets:** Do not commit tokens. Prefer local-only `mcp.json` edits or credentials your team already uses.

After editing MCP config, restart Cursor. Verify servers in the command palette (e.g. MCP: list / status, per your Cursor version).

---

## Handoff to implementation

When the user moves from **planning** to **coding**:

1. [AI documentation standards](./ai-documentation-standards.md) — bootstrap order, KB vs skills vs workflows.
2. [FEATURE-IMPLEMENTATION-MASTER.md](../../.agents/workflows/FEATURE-IMPLEMENTATION-MASTER.md) — full-stack / Figma implementation phases (CW1–CW5A).
3. [project-workflow-alignment.mdc](../../.cursor/rules/project-workflow-alignment.mdc) — Cursor alignment with `docs/kb` and `.agents`.

Implementation work also follows [`maintenance.md`](./maintenance.md) when you change app code.

---

## Stable `@` paths (Cursor)

From repo root in Cursor chat or Composer:

- `@docs/kb/cursor-task-planning-agent.md` — this hub
- `@.cursor/AGENT.md` — strict workflow

---

## Smoke test

1. Open Cursor chat in **Agent** mode.
2. Paste:

```
New task: [paste your requirement or Figma URL here]

Please follow the AGENT.md workflow — read all context first,
then generate the master plan and wait for my approval.
```

3. Expect: context read (including Figma if URL given), `plans/00-master-plan.md` created, and the agent **stops** for your approval before sub-plans.
