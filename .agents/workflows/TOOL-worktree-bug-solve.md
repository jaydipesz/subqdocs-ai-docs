---
description: TOOL — End-to-end bug resolution using isolated git worktrees, dynamic ports, and automated PRs on a shared DB.
---

# /TOOL-worktree-bug-solve — Isolated Multi-Repo Bug Resolution

Take full ownership of diagnosing and fixing bugs across frontend and backend using completely isolated `git worktree` environments. Guarantee that the base branch is never modified locally.

## Prerequisite: The Strict Environment Guarantee
The user MUST provide the Target Environment (`dev`, `stage-ema`, or `production`). 
**MANDATORY CHECK:** 
1. `cd` into both `subqdocs-frontend` and `subqdocs-backend` bases.
2. Run `git branch --show-current`. If they do NOT exactly match the Target Environment, ABORT the workflow immediately.
3. Run `git status --porcelain`. If any uncommitted changes or stashes are altering the base state, ABORT immediately to prevent merge conflicts.

---

## Phase 1: Input & Context Gathering
Ask the user for the following inputs if not provided:
1. **Target Environment** (e.g., `dev`).
2. **Bug Description & Symptoms**.
3. **Bug Context** (e.g., reproduction steps, logs, specific files).
4. **Target Repo** (`frontend`, `backend`, `both`, or `unknown`).
**Auto-Load Context:** Note that default test credentials for E2E tests are: Email: `test123@yopmail.com` / Password: `Dev@1234`.

## Phase 2: Base Synchronization (Shared DB Sync)
In the *base* repositories:
1. `cd subqdocs-backend` -> `git pull origin <environment>`.
2. Run `npm run migrate` in the base backend to ensure optimal shared DB parity for this environment.
3. `cd ../subqdocs-frontend` -> `git pull origin <environment>`.

## Phase 3: The Investigation & Blast Radius (If Target Repo = unknown)
1. Use `grep_search` and `view_file` to trace the bug across the base `subqdocs-frontend` and `subqdocs-backend` repositories.
2. Determine exactly which repository (or both) actually requires code modifications. Do NOT modify any code in the base repo.

## Phase 4: Targeted Secure Worktree Generation (.env & Node Safe)
Based on Phase 3 (or user explicit instruction), create isolated physical folders ONLY for the repos that require fixes:

**If backend requires changes:**
1. Generate descriptive branch: e.g., `fix/backend-<bugname>`.
2. Set worktree path: `../subqdocs-backend-fix-<bugname>`.
3. `cd subqdocs-backend` -> `git worktree add <worktree-path> origin/<environment> -b <branch>`.
4. **CRITICAL OVERRIDE 1:** `cp .env <worktree-path>/.env`. 
5. **CRITICAL OVERRIDE 2:** `cd <worktree-path>` -> run `npm install`.

**If frontend requires changes:**
1. Generate descriptive branch: e.g., `fix/frontend-<bugname>`.
2. Set worktree path: `../subqdocs-frontend-fix-<bugname>`.
3. `cd subqdocs-frontend` -> `git worktree add <worktree-path> origin/<environment> -b <branch>`.
4. **CRITICAL OVERRIDE 1:** `cp .env <worktree-path>/.env`.
5. **CRITICAL OVERRIDE 2:** `cd <worktree-path>` -> run `npm install`.

## Phase 5: Execute Fix & Mandatory Build
1. Activating `@[/TOOL-bug-solve]` principles: Implement the code fix directly inside the targeted worktree directories.
2. **Mandatory Build:** Run `npm run build` / `tsc` inside the modified worktrees. You must fix any compilation errors before continuing.
*(Note: If the fix involves adding a new database migration, execute it here on the shared database using `npm run migrate` in the worktree).*

## Phase 6: Zero-Conflict E2E Server Boot
Spin up the application using persistent terminal commands (`WaitMsBeforeAsync > 500`).

1. **Dynamic Port Acquisition:** Find two unused local ports (e.g., `PORT_FE`=3042, `PORT_BE`=8042).
2. **Backend Boot:** 
   - Enter the backend directory (worktree or base).
   - Inject environment overrides: `PORT=<PORT_BE> FRONTEND_URL=http://localhost:<PORT_FE> API_URL=http://localhost:<PORT_BE> npm run dev`.
3. **Frontend Boot:**
   - Enter the frontend directory (worktree or base).
   - Inject environment overrides: `PORT=<PORT_FE> VITE_REACT_APP_API_URL=http://localhost:<PORT_BE> npm run dev`.

## Phase 7: Autonomous Browser Verification
1. Give the servers up to 10 seconds to fully boot.
2. Use the `browser_subagent` to navigate to `http://localhost:<PORT_FE>`.
3. Auto-login using `test123@yopmail.com` / `Dev@1234`.
4. **Screenshot Verification:** Instruct the `browser_subagent` to take a screenshot confirming the fix.
5. **MANDATORY RE-FIX:** If the verification fails, you MUST return to Phase 5 and fix the remaining issues before trying again.
6. **STRICT GATE:** You MUST NOT proceed to Phase 8 until the `browser_subagent` screenshot explicitly confirms the defect is eliminated and presented to the user.

## Phase 8: Pull Request, Report, & Cleanup
When E2E tests pass and the user has reviewed the fix in Phase 7, finalize the work for each modified worktree:

1. **Commit & Push:**
   - `git add .`
   - `git commit -m "fix(<scope>): <message>"`
   - `git push -u origin <branch>`

2. **MANDATORY PERMISSION:** Ask the user explicitly: "Changes have been committed and pushed. Shall I proceed to open the Pull Request to <environment>?"

3. **Automate PR:** If and ONLY IF the user provides explicit confirmation (e.g., "Yes", "Proceed"), use the `mcp_github-mcp-server_create_pull_request` tool to open a Pull Request. The `base` MUST exactly match the original Target Environment.

Provide the final summary to the user:
*   Root Cause Analysis & verified behaviors.
*   Confirmation of Build and E2E visual pass.
*   Links to the created GitHub PR(s).
*   **Zombie Worktree Cleanup:** Provide the exact bash commands (e.g., `git worktree remove ../subqdocs-backend-fix-<bugname> --force`) that the user should copy/paste to safely delete the physical folders locally once the PR is merged online.