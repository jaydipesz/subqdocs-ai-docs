---
description: CW5C — Final Reality Check (Visual & Functional E2E)
---

# CW5C — Final Reality Check

This workflow performs the final "Human-style" verification. It merges functional E2E testing with a visual fidelity audit against Figma. This is the single definitive source of truth before declaring a feature complete.

## 1. Prerequisites
- **CW1 to CW5B** must be complete.
- **Skill 23** (Autonomous E2E Engineering) must be loaded.
- **Port 8000** (Backend) and **Port 5173** (Frontend) must be available.

## 2. Server Bootstrap
// turbo-all
1. Start the **Backend** in a background terminal.
2. Start the **Frontend** in a background terminal.
3. Pulse-check both ports until they respond with a `200` or `OK`.

## 3. Autonomous Verification Session
Invoke the **browser_subagent** with the following combined task:

> **Phase A: Authentication**
> - Navigate to `http://localhost:5173/login`.
> - Log in with `test123@yopmail.com` / `Dev@1234`.
> 
> **Phase B: Visual Audit (UI Sync)**
> - Navigate to the feature and compare the real browser output to the Figma frame: [FIGMA_URL].
> - **Audit by Exception:** Only list mismatches (spacing, typography, colors, or hidden elements).
> - Fix any purely visual mismatches immediately in the code.
> 
> **Phase C: Functional Audit (E2E)**
> - Execute the primary feature flow: [Describe Flow from Plan].
> - Verify success toasts, DOM updates, and data persistence.
> 
> **Phase D: Visual Audit (Responsive Sync)**
> - Repeat the visual verification for **Tablet (768px)** and **Mobile (375px)** resolutions.
> - Verify that common mobile components (Hamburger Menu, Modal Overlays, Form Spacing) remain functional.
> - Capture a screenshot for each breakpoint.
> 
> **Phase E: Evidence**
> - Provide a **Responsive Screenshot Gallery** (Desktop, Tablet, Mobile).
> - Compare to Figma's specific "Mobile" designs if they exist.

## 4. Teardown
// turbo-all
1. Terminate both backend and frontend Terminal IDs.
2. Confirm cleanup.

## 5. Output Artifact: **FINAL REALITY CHECK REPORT**
- **Functional Status**: PASSED / FAILED
- **Visual Fidelity**: MATCH / EXCEPTION
- **Responsive Gallery**: 
  - [Desktop Screenshot]
  - [Tablet Screenshot]
  - [Mobile Screenshot]
- **Audit Log**: 
  - [List any fixes made for responsive behavior]
- **Traceability**: [Link to requirements verified]
