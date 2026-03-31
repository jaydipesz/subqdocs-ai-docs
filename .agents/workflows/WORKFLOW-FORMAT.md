# Workflow formatting (Antigravity)

**Last reviewed:** 2026-03-31

All playbooks under `.agents/workflows/` follow these conventions so steps are machine- and human-scannable.

## Numbering

- Use a single top-level ordered list: **1., 2., 3.,** … for every executable step.
- Avoid mixing `### Step N` headings as the primary step list; use **bold** labels inside numbered items if you need substeps (e.g. **Verification:**).

## Verification gates

- After any step that can fail silently (typecheck, build, migration, lint), add an explicit **Verification** line in the **same** numbered step or the next one.
- Wording pattern: **Verification:** Run `…` in `…`. Do **not** proceed to the next step until this passes (exit code 0).

## Turbo annotations

- **`// turbo`** on a line: safe to run without user approval (read-only grep, `ls`, creating an empty directory when idempotent).
- **`// turbo-all`** at the top of a file: entire workflow may be treated as low-risk automation (use sparingly).
- Do **not** mark destructive steps, production deploys, or migrations that touch shared DBs as turbo without human review.

## Links

- Prefer links to [`docs/kb/`](../../docs/kb/README.md), [`docs/skills/`](../../docs/skills/INDEX.md), and [`.agents/rules/production-rules.md`](../rules/production-rules.md) instead of pasting long rules.

## When editing a workflow

- Keep steps numbered sequentially.
- Add verification before “report done” or “hand off to next phase.”
