# Skill 23 — Autonomous E2E Engineering (Human-style Verification)

This skill provides the logic and terminal commands for a "Human-Out-The-Loop" verification phase. It enables the agent to start the full application stack, log in as a real user, and verify feature fulfillment in the browser.

## 1. Server Orchestration

To run E2E tests, the backend and frontend must be running in parallel background terminals.

### Step 1: Start Backend (Port 8000)
- **Cwd**: `subqdocs-backend/`
- **Command**: `PORT=8000 npm run dev`
- **Confirmation**: Use `command_status` to wait for `"Server is running on port 8000"`.

### Step 2: Start Frontend (Port 5173)
- **Cwd**: `subqdocs-frontend/`
- **Command**: `npm run dev`
- **Confirmation**: Use `command_status` to wait for `"VITE v[X] ready in [X] ms"`.

### Step 3: Heartbeat Verification
If terminal output is unclear, use a manual heartbeat check before proceeding to browser interactions:
```bash
# Check Backend
curl -s http://localhost:8000/api/v1/health # Or similar public endpoint
# Check Frontend
curl -s http://localhost:5173
```

## 2. Browser Identity Flow (test123@yopmail.com)

Every E2E test must begin by authenticating the session.

- **URL**: `http://localhost:5173/login` (or root if redirected)
- **Input (Email)**: `test123@yopmail.com`
- **Input (Password)**: `Dev@1234`
- **Action**: Click the Submit/Login button.
- **Verification**: Wait for navigation to the `/dashboard` or the appearance of a Logout button.

## 3. Feature Execution & Evidence

Once logged in, perform the following based on the **IMPLEMENTATION PLAN**:

1.  **Navigation**: Use the sidebar or URL to reach the feature under test.
2.  **Interaction**: Fill forms, click buttons, or trigger events as defined in the test scenario.
3.  **Success Detection**: 
    - Look for "Successfully created" or similar toast notifications.
    - Verify new data appears in lists.
4.  **Evidence Capture**: 
    - Use `browser_subagent`'s `capture_screenshot` at the moment of peak visibility (e.g. while the "Success" toast is still visible).
    - Save the screenshot as `e2e_evidence_[feature_name].png`.

## 4. Visual Sync Protocol (Pixel-Perfect Audit)

During the E2E session, the agent must perform a visual comparison between the browser and Figma.

### Step 1: Aspect Ratio Alignment
- Identify the width/height of the Figma frame.
- Resize the `browser_subagent` window to match the target resolution (e.g., `1440x900` for desktop).

### Step 2: Audit by Exception
Do NOT create a table for every element. Instead:
- Compare the screenshot to the Figma design.
- Identify only **Mismatches**:
  - **Spacing**: Is the padding/gap visibly different?
  - **Typography**: Are fonts too bold or small?
  - **Color**: Are we using the wrong design token?
  - **Interactions**: Do hover states or transitions feel clunky?

### Step 3: Immediate Remediation
- If a mismatch is found, fix the CSS/Tailwind classes immediately.
- Re-capture the screenshot to confirm the fix.

## 5. Multi-Viewport Verification (Responsiveness Audit)

Clinical users often use iPads and smartphones. Testing must be performed at three standard responsive breakpoints:

| Viewport | Resolution | Clinical Device Match |
|---|---|---|
| **Desktop** | `1440x900` | Office Monitor |
| **Tablet** | `768x1024` | iPad (Portrait) |
| **Mobile** | `375x667` | Staff Smartphone (iPhone SE sized) |

### Step 1: Resizing
Use the `browser_subagent` to resize the window for each breakpoint before taking evidence.

### Step 2: Critical Visibility Audit
In **Tablet** and **Mobile** viewports, the agent must verify:
- **Action Buttons**: Primary buttons (Save, Submit) are visible without horizontal scrolling.
- **Form Fields**: Input fields do not overflow the screen width.
- **Navigation**: Sidebar collapses into a "Hamburger" or "Bottom Bar" correctly and is interactable.
- **Typography**: Text remains readable (no tiny text or huge overlapping fonts).

## 6. Automated Cleanup

**CRITICAL**: Do NOT leave local servers running after the test.
- Use `send_command_input` with `terminate: true` for both backend and frontend Terminal IDs.
- Verify `localhost:8000` and `5173` are free before finishing the workflow.

## 7. Common Pitfalls
- **Port Conflict**: If 8000 or 5173 is already in use, find and kill the process first using `fuser -k [port]/tcp`.
- **Database Reset**: Ensure the test uses a clean "Test Organization" or a known test user to avoid state pollution.
