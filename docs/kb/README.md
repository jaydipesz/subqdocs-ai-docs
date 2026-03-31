# SubQDocs — knowledge base (`docs/kb/`)

**Last reviewed:** 2026-03-31 · **Doc set:** 1

**Parent workspace index:** [`docs/README.md`](../README.md) (how `docs/`, `.cursor/`, `.agents/`, `.gemini/` fit together).

Single home for **product**, **architecture**, **stack conventions**, and **per-app** deep docs. Update these files when behavior or structure changes — see [`maintenance.md`](./maintenance.md).

## Start here

| Doc | Contents |
|-----|----------|
| [`ai-documentation-standards.md`](./ai-documentation-standards.md) | AI/tooling: bootstrap order, stable `@` paths, Cursor vs Antigravity |
| [`project/overview.md`](./project/overview.md) | Product scope, PHI, auth, never-do |
| [`project/architecture.md`](./project/architecture.md) | Diagram, repo landmarks |
| [`frontend/conventions.md`](./frontend/conventions.md) | React/Vite, Axios, Redux, Query, routes, SVG, PDF.js |
| [`backend/conventions.md`](./backend/conventions.md) | Express, Sequelize, modules, validation |
| [`anti-patterns.md`](./anti-patterns.md) | Forbidden patterns, deprecations, task → doc routing |

## Per application

| Doc | Contents |
|-----|----------|
| [`subqdocs-frontend/INDEX.md`](./subqdocs-frontend/INDEX.md) | Frontend KB hub, domain registry |
| [`subqdocs-backend/INDEX.md`](./subqdocs-backend/INDEX.md) | Backend KB hub, module registry |
| [`subqdocs-voice-to-voice/INDEX.md`](./subqdocs-voice-to-voice/INDEX.md) | Voice agent KB hub (Python / Twilio / Deepgram) |

## Hooks and maintenance

| Doc | Contents |
|-----|----------|
| [`hooks/README.md`](./hooks/README.md) | Where hook docs live |
| [`hooks/global.md`](./hooks/global.md) | `src/hooks/*` inventory |
| [`maintenance.md`](./maintenance.md) | What to update when code changes |

## Procedural skills

Step-by-step checklists (scaffolding, S3, sockets, migrations, …): [`../skills/INDEX.md`](../skills/INDEX.md)

## AI context and stable `@` paths

Single list of bootstrap order and Cursor `@` paths: [`ai-documentation-standards.md`](./ai-documentation-standards.md).
