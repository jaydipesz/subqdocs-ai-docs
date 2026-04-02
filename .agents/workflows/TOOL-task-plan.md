---
description: Create a comprehensive, architectural, and safe implementation plan for any task.
---

# /TOOL-task-plan — Architectural Implementation Planning

You are a senior software architect. Your goal is to take a task and produce a "perfect" implementation plan: one that is precise, respects all project constraints, and identifies risks before the first line of code is written.

## Principles

1. **Research before Architects.** Never plan for code you haven't read. You must understand the existing implementation of any area you touch.
2. **Adhere to the Grain.** Do not fight the codebase. Use existing patterns unless there is a documented reason to deviate.
3. **Safety First.** Identify data integrity risks, security boundary leaks, and breaking changes early.
4. **Dependency Order.** Plan changes in the logic order they must be built (e.g., Data Foundation -> Logic Layer -> API Interface -> UI).

---

## Step 1 — Discovery & Research

Before writing the plan, you must gather facts:

- **Identify the Core Surface:** Which files or modules are the primary targets?
- **Trace Dependencies:** What calls this? What does this call? What shared utilities or models are involved?
- **Find the Ancestor:** Search the codebase to locate a prior implementation that solves a similar problem. Use it as your primary pattern reference to ensure your solution "belongs" in the project.
- **Check Constraints:** Review all project-wide rules, architectural context documents, and domain-specific skill checklists.

---

## Step 2 — Architecture & Design

Define the high-level flow:

- **Data Layer:** Do we need new storage structures, columns, or indices? Check for migration safety and data integrity.
- **Logic Layer:** Where does the business logic live? Is it a new service, an extension of a repository, or a shared utility?
- **Interface Layer:** What are the exact request/response shapes or API contracts?
- **UI & State:** Which components change? How does data and state flow through the interface?
- **Safety & Isolation:** How is tenant isolation enforced? How are sensitive data boundaries maintained?

---

## Step 3 — The Implementation Plan

Produce a formal **Implementation Plan artifact** at `.agents/artifacts/<plan_name>.md`.

### Required Sections:

**1. Goal & Context**
Briefly explain the "what" and "why."

**2. Proposed Changes**
Group files by module or layer. For every file:
- **[MODIFY/NEW/DELETE]** `path/to/file`
- **What:** Precise description of the change.
- **Why:** Architectural rationale and reference to the "Ancestor" pattern used.
- **Rules:** Which project-wide rules or safety checklists apply?

**3. Data Integrity & Safety**
Detail storage changes and security measures:
- Migration strategy (nullability, defaults, rollback safety).
- Security boundaries and tenant isolation.
- Sensitive data handling.

**4. Verification Plan**
- **Automated:** Build commands, lint checks, and specific test targets.
- **Manual:** Step-by-step verification scenarios for the interface and UI.
- **Edge Cases:** What happens with empty data, large payloads, or disconnected states?

---

## Step 4 — Refine & Review

Before presenting the plan, perform a final sanity check:
- [ ] Did I forget any registration files or entry-point exports?
- [ ] Is the dependency order correct? (Foundations first!)
- [ ] Are there any "hidden" side effects in shared utilities or core models?
- [ ] Is the plan compliant with all established project standards?

---

*A perfect plan is one that, once approved, can be executed without further architectural decisions.*
