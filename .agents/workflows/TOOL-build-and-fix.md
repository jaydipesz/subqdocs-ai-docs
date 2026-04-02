---
description: Build the specified module and automatically fix common build/lint errors upon approval.
---

This workflow ensures a clean, error-free build for either the backend or frontend.

1. **Select and Build Target**
   - Identify the user's target: `subqdocs-backend` or `subqdocs-frontend`.
   - Run `npm run build` in the target directory.
   - Wait for the build to complete and capture all stdout/stderr logs.

2. **Error Capture and Reporting**
   - If the build succeeds, report success and stop.
   - If the build fails:
     - Extract all files and line numbers with actual build errors (e.g., from `tsc` or `vite build` output).
     - Summarize the types of errors found (e.g., Missing Imports, Type Mismatches).
     - **Report to the user:** Present the error summary and ask: *"I found these build errors. Should I attempt to fix them automatically?"*

3. **Autonomous Repair (Requires Approval)**
   - Once the user says "fix it" or "yes":
     - Analyze each build error sequentially.
     - For **missing imports**: Search the codebase for the missing symbol and add the import.
     - For **type errors**: Fix simple mismatches or correctly type the variable/prop.
     - (Exclude any general lint or formatting fixes unless they cause a build failure.)

4. **Verification Run**
   - Re-run `npm run build` in the same target directory.
   - Verify that the specific errors fixed no longer appear.
   - If new errors arise, repeat the report/fix loop once; otherwise, present the final build status.

5. **Commit Fixes (Optional)**
   - If the build is fixed successfully, **ask the user:** *"Build errors are fixed! Should I commit these changes now?"*
   - **Upon approval:**
     - Perform `git add .`
     - Commit with the message `[s] {appropriate short description of fixes}` (e.g., `[s] resolve build and lint issues`).
     - Do NOT `git push`.
     - Confirm the commit has been made to the user.