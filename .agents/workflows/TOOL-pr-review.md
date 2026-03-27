---
description: Comprehensive Code Review for Pull Requests in Antigravity
---

This workflow performs a structured pull request review, ensuring adherence to Antigravity's architectural patterns and coding standards.

### PR Review Trigger
Trigger this workflow by saying: "Review PR #[number]" or "Review PR #[number] in antigravity".

### Workflow Steps:

1. **Fetch PR details**:
   - Use `mcp_github-mcp-server_get_pull_request` and `mcp_github-mcp-server_get_pull_request_files`.
   - Use `mcp_github-mcp-server_get_file_contents` to fetch actual contents. Use `view_file` for local versions.

2. **Project Context Review**:
   - Read `AI-CONTEXT.md`, `AI-CONTEXT-FRONTEND.md`, and `AI-CONTEXT-BACKEND.md` for standards.
   - Read `.agents/rules/production-rules.md` for mandatory constraints.
   - Load relevant `docs/skills/` only if PR touches specific domains (e.g., S3, Sockets).
   - **Source of Truth**: These files are the primary reference. If a pattern is not covered here, it must follow industry-standard best practices for that specific configuration.

3. **Construct Structured Review**:
   - **Raise issues ONLY IF 100% certain.** Uncertainty = silence.
   - **Human-Generated Tone**: Write comments as a professional developer would. Avoid robotic headers or repetitive boilerplate.
   - **Comment Content**: Briefly state the problem and provide a concrete fix in a natural, technical tone. **Do NOT mention or include the bug category/severity labels (e.g., 🔴 CRITICAL, 🟡 IMPORTANT) in the actual posted comments.**
   - **Severity Internal Logic**:
     - 🔴 **CRITICAL** — Proved bug, security breach, or data corruption.
     - 🟡 **IMPORTANT** — Clear violation of an explicit rule in `AI-CONTEXT*.md` or `production-rules.md`.
     - 🔵 **MINOR** — Unambiguous factual error (typo, dead code) or clear violation of fallback best practices.
   - **Placement**: Use line-specific reviews for every identified issue.

4. **Production Checklist**:
   - [ ] No breaking API changes or cross-tenant PHI leaks (`organization_id`).
   - [ ] No hard-deletes (`paranoid: true`) or direct `console.log`.
   - [ ] Typed Axios wrappers (`axiosGet`, etc.) and `generalResponse` used.
   - [ ] Follows `HttpException` and `joi` validation patterns.
   - [ ] No hardcoded secrets or exposed PII/PHI fields.

5. **Verdict & Confirmation**:
   - ✅ **Ready to merge** — No 🔴 CRITICAL issues. IMPORTANT and MINOR items are noted for the author but do not block merge.
   - 🚫 **Do not merge** — One or more 🔴 CRITICAL issues present.
   - Show draft to user: *"Here is the review. Modify anything before I post?"*

6. **Post Review**:
   - Once explicitly confirmed, use `mcp_github-mcp-server_create_pull_request_review`.
   - Post to the correct file and line range using the `comments` array (with `path` and `line` properties).
   - **IMPORTANT**: If this is a subsequent review on the PR, ensure that new line-specific feedback is still accurately placed on the precise lines via the `comments` array. Do NOT fall back to posting all feedback as a single block in the general review `body`.
   - The overall verdict and summary go in the general review body.