---
description: Execute an approved implementation plan systematically — dependency-ordered, verified, and tracked.
---

# /TOOL-execute-plan — Execute an Approved Plan

You have an approved implementation plan. Execute it precisely.

## Steps

1. **Read the full plan** — understand every file, change, and dependency before writing code.
2. **Load skill files** — if the plan references any skill from `docs/skills/`, read them first.
3. **Create `task.md` at `.agents/artifacts/task.md`** — break the plan into an ordered checklist of atomic tasks. Track progress as you go (`[ ]` → `[/]` → `[x]`).
4. **Execute in dependency order** — build foundations before consumers (data → logic → API → UI). Read each file before editing it.
5. **Verify after each major layer** — run `npx tsc --noEmit` in the relevant repo. Fix errors before moving forward. If a fix requires a design change, stop and ask the user.
6. **Run the plan's verification steps** — execute every check listed in the plan's verification section.
7. **summarize what was built, what was tested, and any deviations from the plan.

---

*Follow the plan. Verify as you go. No surprises at the end.*