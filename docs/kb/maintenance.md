# Knowledge base maintenance

**Last reviewed:** 2026-04-02 · **Doc set:** 1

When you **create or materially change** application code, update the matching **knowledge base** under `docs/kb/` in the same change set so agents and humans stay aligned.

When you **ban** a pattern, **deprecate** a package, or add a repo-wide **never**, update [`anti-patterns.md`](./anti-patterns.md) in the same change set.

## Which KB to use

| Code area | Hub doc |
|-----------|---------|
| `subqdocs-backend/` | [`subqdocs-backend/INDEX.md`](./subqdocs-backend/INDEX.md) |
| `subqdocs-frontend/` | [`subqdocs-frontend/INDEX.md`](./subqdocs-frontend/INDEX.md) |
| `subqdocs-voice-to-voice/` | [`subqdocs-voice-to-voice/INDEX.md`](./subqdocs-voice-to-voice/INDEX.md) |
| Hooks (policy + global inventory) | [`hooks/README.md`](./hooks/README.md) |
| Cross-cutting rules | [`project/overview.md`](./project/overview.md), [`../../.agents/rules/production-rules.md`](../../.agents/rules/production-rules.md) |

## What to update (by change type)

| You added or changed | Documentation action |
|----------------------|-------------------------|
| **New** `src/modules/<name>/` (backend) | Add a row to the **Full module registry** in `subqdocs-backend/INDEX.md`. Add or extend `subqdocs-backend/modules/<name>.md` if the module is user-facing or complex. |
| **New** `src/domains/<name>/` (frontend) | Add a row to the **Domain registry** in `subqdocs-frontend/INDEX.md`. Add or extend `subqdocs-frontend/domains/<name>.md`. |
| **New or materially changed** `subqdocs-voice-to-voice/` routes, services, agent tools, or auth | Update `subqdocs-voice-to-voice/INDEX.md` (landmarks table or a new file under `docs/kb/subqdocs-voice-to-voice/` if the area is large). |
| **Hooks** under `**/hooks/*.{ts,tsx}` | Add or refresh a **Hooks** subsection in the domain doc (or [`hooks/global.md`](./hooks/global.md) for `src/hooks/`). See [`hooks/README.md`](./hooks/README.md). |
| **New routes** (`routePath.tsx`, guards) | Touch `subqdocs-frontend/routing-and-state.md` or the relevant **domain** doc so the route purpose is discoverable. |
| **Boot / providers** (`main.tsx`, `App.tsx`, `server.ts`, `app.ts`) | Update `subqdocs-frontend/entry-points.md` or `subqdocs-backend/entry-points.md`. |
| **Shared backend** (`src/common/`, `src/sequelize/`) | Update `subqdocs-backend/common.md` or `sequelize.md` when behavior or layout of those trees changes. |
| **PHI, auth, tenancy, or “never do”** | Update `project/overview.md` and/or `production-rules.md`; do not rely only on KB stubs. |
| **Task Planning Agent** (`.cursor/AGENT.md`, `.cursor/mcp.json` layout) | Update [`cursor-task-planning-agent.md`](./cursor-task-planning-agent.md) in the same change set so the KB hub stays the single index. |

## Hooks (concrete)

- **Domain hook** — e.g. `src/domains/patient/hooks/useArchivePatient.ts` → `subqdocs-frontend/domains/patient.md` **Hooks** list.
- **App-wide `src/hooks/`** — [`hooks/global.md`](./hooks/global.md) and the primary feature domain doc if applicable.

## If unsure

Prefer a **short addition** to the relevant INDEX or domain/module doc over a new file. Link to source paths in backticks for grep-friendly maintenance.
