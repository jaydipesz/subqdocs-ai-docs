---
description: Diagnose and resolve bugs with evidence-based analysis, safe fix plans, and regression checks
---

# /TOOL-bug-solve — Evidence-Based Bug Resolution

Take full ownership of the bug. Deliver a precise, verified, and safe resolution.

## Principles

1. **Evidence over intuition.** Every conclusion must trace back to a line, a log, or a behavior. No assumptions.
2. **Read before reasoning.** Open every file in the execution chain before forming a theory. Symptom location ≠ root cause.
3. **One fix, one reason.** Every change must explain what it fixes and why. No bundled refactors.
4. **Do no harm.** Think through shared state, dependent modules, and edge cases before changing anything.
5. **Verify every symbol.** Before writing any fix, confirm that every variable, function, type, and component you will reference actually exists, is in scope, and has the shape you expect. Never assume — check the declaration.

---

## Step 1 — Gather Evidence

Restate the bug: expected vs. actual behavior. Classify the domain. Then investigate:

- **Read every file in the execution path** — entry point through to the leaf.
- **Trace the data flow** — follow data from entry to where the bug manifests. Where does it transform?
- **Check recent changes** — `git log -n 10 --oneline -- <file>`. What changed?
- **Search for related patterns** — `grep_search` to find how the same function is used elsewhere. Does it work there?
- **Inspect schema/state** — use MCP tools to verify DB schema, migration history, or API routes as needed.
- **Audit fix dependencies** — before forming a fix, verify every symbol your fix will touch: check declarations (not just usage), destructuring patterns, imports, component prop contracts, and third-party component child constraints. If you plan to inject JSX into a component tree, confirm the parent accepts arbitrary children.
- **Load relevant skill files** from `docs/skills/` (check `INDEX.md`) if the bug touches a listed domain.

### Runtime Data Gate

If the bug involves intermittent failures, race conditions, async ordering, state corruption, or data that's correct at one layer but wrong at another:

1. Say: **"This bug needs runtime data before I can diagnose it confidently."**
2. Specify exact diagnostic logs to add (file + line). Ensure they follow project rules — no PHI, use the project logger.
3. Ask the user to reproduce and share output.
4. Only then proceed to Step 2.

**Never guess on bugs that require live data.**

---

## Step 2 — Diagnose

1. Form hypotheses from the evidence.
2. Test each against ALL evidence — a hypothesis that explains one symptom but contradicts another is wrong.
3. Rank candidates by evidence strength, not by what "usually" causes this.
4. Verify the cause explains both why it happens here AND why it doesn't happen in similar scenarios.
5. Check if project conventions or production rules apply to the affected code.

---

## Step 3 — Plan, Fix, Verify

Present the full diagnosis as a structured artifact at `.agents/artifacts/bug-fix-plan.md`, and auto-proceed with execution (do not ask for user approval).

### Output Structure:

**1. Bug Interpretation** — What the bug is. Expected vs. actual.

**2. Root Cause** — Exact cause with evidence. Multiple candidates ranked if needed. Link to files and lines.

**3. Fix Plan** — Step-by-step, file-level changes. Each step: what file, what changes, why, and risk flag if it touches shared logic. Must follow project conventions.

**3a. Symbol & Structure Checklist** — For each file being modified, list: (1) every new variable/function/component your fix references and confirm its declaration exists, (2) every insertion point and confirm the parent element accepts the injected content, (3) every import your fix requires and confirm it exists. This checklist must be present in the plan.

**4. Side Effects & Regression** — Every module/flow sharing the modified code. Why each will or won't be affected.

**5. Test Checklist** — Specific scenarios with expected results. Cover: happy path (bug is fixed), edge cases (boundary conditions), regression (existing flows still work).

**6. Root Cause Insight** — Why this bug existed. What gap allowed it. Written so any developer fully understands the origin.

### Execution & Verification Gate:

1. Make all edits before testing — partial edits produce misleading errors.
2. **Run the project build command.** This is mandatory, not optional. Use the project's actual build command (e.g., `npm run build`, `tsc`, `go build`). Do NOT skip this step.
3. If the build fails, fix the errors as part of this resolution. Return to step 2. Repeat until the build passes.
4. Only after a clean build passes: update task.md and walkthrough.md artifacts.
5. **Never declare a fix complete without a passing build.**

---

*Sharp. Evidence-based. Safe. Complete. No guesses. No gaps.*