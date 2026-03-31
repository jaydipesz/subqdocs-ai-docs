# `.gemini/` — local Gemini session artifacts

This folder holds **point-in-time notes** from Gemini-assisted sessions. It is **not** the team knowledge base. Treat anything here as **draft or exploratory** until it is reflected in `docs/kb/` (or promoted into a skill under `docs/skills/`).

---

## Artifacts (`artifacts/`)

| File | Topic |
|------|--------|
| [`artifacts/calendar_controller_timezone_plan.md`](./artifacts/calendar_controller_timezone_plan.md) | Plan for calendar controller timezone behavior. |
| [`artifacts/ipad-chatbot-session-persistence-plan.md`](./artifacts/ipad-chatbot-session-persistence-plan.md) | Plan for iPad chatbot session persistence. |

If this directory grows, consider adding `artifacts/README.md` with a simple file list only.

---

## Source of truth

| Need | Location |
|------|----------|
| Architecture, product rules, PHI, conventions | [`docs/kb/`](../docs/kb/) |
| Step-by-step procedures (routes, uploads, sockets, …) | [`docs/skills/`](../docs/skills/) |

---

## When to add files here

- **Optional.** Prefer updating [`docs/kb/`](../docs/kb/) for anything that should stay true for the team.
- Use **`artifacts/`** for session-specific or experimental plans that might later be **merged into** the KB or discarded after validation.

---

## Workspace index

- [`docs/README.md`](../docs/README.md) — full map of `docs/`, `.cursor/`, `.agents/`, `.gemini/`, and read order.
- [`docs/kb/ai-documentation-standards.md`](../docs/kb/ai-documentation-standards.md) — canonical bootstrap order and stable `@` paths (this folder is not source of truth).
