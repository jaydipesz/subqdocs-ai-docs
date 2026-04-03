---
description: Execute an approved implementation plan systematically — dependency-ordered, verified, and tracked.
---

# /TOOL-execute-plan — Execute an Approved Plan

You have an approved implementation plan. Execute it precisely.

## Pre-Execution Context

Before writing any code:
1. Confirm production rules are active (auto-injected via system prompt — do not re-read the file).
2. Read `docs/skills/INDEX.md` — identify which skills the plan's tasks trigger.

## Steps

1. **Read the full plan** — understand every file, change, and dependency before writing code.
2. **Load skill files** — if the plan references any skill from `docs/skills/`, or if INDEX.md triggers match planned operations, read those skills first.
3. **Create `task.md`** — break the plan into an ordered checklist of atomic tasks. Track progress as you go (`[ ]` → `[/]` → `[x]`).
4. **Execute in dependency order** — build foundations before consumers (migration → model → repo → validation → controller → route → frontend). Read each file before editing it.
// turbo
5. **Verify after each major layer** — run `npx tsc --noEmit` in the relevant repo. Fix errors before moving forward.
6. **Run the plan's verification steps** — execute every check listed in the plan's verification section.
7. **Summarize** what was built, what was tested, and any deviations from the plan.

## Error Recovery

If a build or type-check fails mid-layer:
1. Stop execution immediately.
2. Log the error in `task.md` under the failing task.
3. Attempt a fix (max 2 tries).
4. If still failing after 2 attempts, **stop and ask the user** — do not guess or work around it.
5. Never silently skip a failing step.

## Commit Policy

- Do NOT commit or push unless the user explicitly requests it.
- If the plan specifies a commit strategy, follow it.
- If it doesn't, leave all changes uncommitted and report what was changed.

---

*Follow the plan. Verify as you go. No surprises at the end.*