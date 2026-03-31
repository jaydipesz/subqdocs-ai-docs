# Hooks — knowledge base

React hooks are documented in two places:

1. **App-wide hooks** (`subqdocs-frontend/src/hooks/`) — inventory and one-line purpose in [`global.md`](./global.md).
2. **Domain hooks** (`subqdocs-frontend/src/domains/<domain>/hooks/`) — add or update a **Hooks** subsection in the matching file under [`../subqdocs-frontend/domains/`](../subqdocs-frontend/domains/) (e.g. `patient.md`, `messaging.md`).

## When to update docs

See [`../maintenance.md`](../maintenance.md). Any new hook file or behavior change should add or refresh documentation in the same change set.

## Backend

This repo’s backend is not React; “hooks” here mean **frontend** hooks only. Server middleware stays in backend conventions and module docs.
