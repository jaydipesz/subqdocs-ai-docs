---
description: Comprehensive Code Review for Pull Requests in SubQDocs
---

# Comprehensive Code Review for Pull Requests in SubQDocs

This workflow performs a structured pull request review, ensuring adherence to SubQDocs' architectural patterns and coding standards.

---

## PR Review Trigger

Trigger this workflow by saying: `"Review PR #[number]"` or `"Review PR #[number] in SubQDocs"`.

---

## Workflow Steps

### 1. Fetch PR Details

- Use `mcp_github-mcp-server_get_pull_request` to get the PR title, description, base/head branches, and metadata.
- Use `mcp_github-mcp-server_get_pull_request_files` to get the full list of changed files.
- **If the PR has more than 50 changed files**, apply the triage strategy in the **Large PR Handling** section below before fetching any file contents. Do not attempt to load all files at once.
- For files selected for review, use `mcp_github-mcp-server_get_file_contents` to fetch their actual contents from GitHub. Use `view_file` only for files already present in the local workspace (e.g. context/config files that aren't part of the diff).

---

### 2. Large PR Handling (50+ Changed Files)

When a PR contains more than 50 changed files, context window limits make full review impossible. Apply this prioritization strategy:

**Priority tiers — fetch in this order, stop when context is near capacity:**

| Tier | What to fetch | Why |
|------|--------------|-----|
| 1 — Critical path | Controllers, services, middleware, auth, permission guards, route definitions | Highest blast radius if wrong |
| 2 — Data layer | Models, migrations, repository/query files | Schema and data-integrity risk |
| 3 — Shared utilities | Helper functions, typed wrappers (axiosGet etc.), shared validators | Cross-cutting impact |
| 4 — Tests | Test files for anything in Tiers 1–3 | Confirms coverage exists |
| 5 — Config & infra | Environment configs, CI files, Dockerfiles | Security and deployment risk |
| Skip unless flagged | Auto-generated files, lock files (package-lock.json, yarn.lock), assets, pure type-definition files with no logic | Low review value |

**After applying tiers**, open your review with a transparent note in the summary body:

> "This PR contains [N] changed files. Due to context limits, this review focused on [list of reviewed files/areas]. Files not reviewed: [brief list or pattern]. A follow-up review of the remaining files is recommended."

---

### 3. Project Context Review

- Read `AI-CONTEXT.md`, `AI-CONTEXT-FRONTEND.md`, and `AI-CONTEXT-BACKEND.md` for standards.
- Confirm production rules are active (auto-injected via system prompt — do not re-read the file).
- Load files from `docs/skills/` **only if the PR touches a matching domain**. Use the file list from step 1 to determine relevance before loading:

| If changed files include... | Load this skill |
|-----------------------------|----------------|
| S3 upload/download logic, storage paths | `docs/skills/04-s3-multipart-upload.md` |
| WebSocket, socket.io, real-time events | `docs/skills/06-realtime-socket-events.md` |
| Background jobs, queues, workers | `docs/skills/07-bullmq-background-jobs.md` |
| Any other domain skill | Match by filename pattern; load only on a clear match |

**Source of truth**: These files are the primary reference. If a pattern is not covered, default to the OWASP Top 10 for security concerns and the existing codebase's dominant conventions for style concerns — not general internet best practices.

---

### 4. Construct the Review

#### Certainty rule

Only raise an issue when you can satisfy **all three** of these:
1. You can point to a **specific line or block** in the diff.
2. You can name the **specific rule or standard** it violates (from context files, production rules, or the fallback references above).
3. The bad outcome is **inevitable from the code as written** — not just possible or theoretically risky.

If any of the three cannot be met, stay silent on that concern.

#### Tone

Write every comment as a senior developer would in a real code review — direct, specific, and human.
- **NEVER** mention internal rule names or numbers (e.g., "Rule 21", "production-rules.md") in the final comments.
- **NEVER** use robotic headers or bullet-pointed boilerplate.
- No meta-commentary about what kind of issue this is; just describe the problem clearly and show the fix.
- If a comment naturally fits in one sentence, keep it to one sentence.

#### Comment content

State the problem and provide a concrete fix based on architectural patterns, without revealing the names of the internal guidelines driving the feedback. Where a code snippet makes the fix unambiguous, include one. Keep comments focused on a single concern — don't bundle multiple issues into one comment.

#### Severity (internal logic only — never mention these labels in posted comments)

- 🔴 **CRITICAL** — The bad outcome is inevitable from the code as written: a proved bug, a security breach, or data corruption. No speculation.
- 🟡 **IMPORTANT** — A clear, direct violation of an explicit rule in `AI-CONTEXT*.md` or `production-rules.md`, where the rule and the violation are unambiguous.
- 🔵 **MINOR** — An unambiguous factual error (typo, dead code, wrong variable name) or a clear deviation from fallback best practices when no project rule applies.

#### Placement

Use line-specific review comments for every identified issue. Match the `path` and `line` to the exact location in the diff.

---

### 5. Production Checklist

Before finalising the review, actively scan the changed files for each item — do not just acknowledge them mentally:

- [ ] No breaking API changes or cross-tenant PHI leaks.
- [ ] No hard-deletes on models with `paranoid: true`.
- [ ] No direct `console.log` calls in production code paths.
- [ ] All HTTP calls use typed Axios wrappers (`axiosGet`, `axiosPost`, etc.) — not raw `axios`.
- [ ] All responses use `generalResponse` — no ad-hoc response shapes.
- [ ] Errors thrown via `HttpException` — not generic `Error` or silent catches.
- [ ] All input validated with `joi` — no unvalidated `req.body` access.
- [ ] No hardcoded secrets, tokens, or credentials anywhere in the diff.
- [ ] No PII/PHI fields exposed in response payloads or logged.

---

### 6. Verdict & Confirmation Draft

Present the review to the user before posting. Show it in this format:

---

**Draft Review — PR #[number]**

*[If large PR: brief note on which files were reviewed and which were skipped.]*

**Verdict**: ✅ Ready to merge / 🚫 Do not merge

*[One or two sentences summarising the overall state of the PR — what it does well and what the critical blockers are, if any.]*

---

**Summary of findings**

| # | File | Line | Severity | Issue (one line - **NO RULE NUMBERS**) | Include? |
|---|------|------|----------|-----------------|---------|
| 1 | `src/auth/guard.ts` | 42 | 🟡 Important | Missing `user_id` check on query | ✅ Yes |
| 2 | `src/utils/helper.ts` | 18 | 🔵 Minor | `console.log` left in production code | ✅ Yes |
| … | | | | | |

*Check or uncheck any row before confirming. Say "remove #2" or "post it" to proceed.*

---

> "Here is the draft review. Remove any items from the table you don't want posted, or say 'post it' to submit as-is."

---

**Verdict comment on the PR**

Post a verdict summary **only if** there are 🔴 CRITICAL issues. In that case, include one short paragraph in the general review body explaining what blocks the merge and why. If there are no CRITICAL issues, the general review body should be empty or contain only the large-PR coverage note — do not add a verdict comment for IMPORTANT or MINOR findings. Those speak for themselves in the line comments.

---

### 7. Post Review

Once the user explicitly confirms (or removes specific items):

- Use `mcp_github-mcp-server_create_pull_request_review` to post.
- Every confirmed finding goes in the `comments` array with its correct `path` and `line`. Do not collapse them into the general `body`.
- The general review `body` contains: the large-PR coverage note (if applicable) and the CRITICAL verdict paragraph (if applicable). Nothing else.
- **For subsequent reviews on the same PR**: line-specific placement via the `comments` array is mandatory even if only one item is being posted. Never fall back to a single block comment for a re-review.

---

## Quick Reference: What Goes Where

| Content | Where it goes |
|---------|--------------|
| Individual code issue | `comments` array — line-specific |
| CRITICAL blocker explanation | General review `body` |
| Large-PR coverage note | General review `body` |
| IMPORTANT / MINOR findings | `comments` array only — no body summary |
| Verdict emoji (✅ / 🚫) | General review `body` — only when CRITICAL issues exist |