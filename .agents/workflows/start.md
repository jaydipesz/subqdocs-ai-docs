---
description: Start the full SubQDocs development environment (Backend & Frontend)
---

This workflow automates the startup of both the backend and frontend development servers.

// turbo-all
1. Start the Backend Server (if not already running)
   - Conditional Check: `lsof -i :8000` (Skip if output exists)
   - Commands: `npm run dev`
   - Directory: `subqdocs-backend`
   - Wait: 3000ms (Wait for initial startup logs)

2. Start the Frontend Server (if not already running)
   - Conditional Check: `lsof -i :5173` (Skip if output exists)
   - Commands: `npm run dev`
   - Directory: `subqdocs-frontend`
   - Wait: 3000ms (Wait for initial startup logs)

3. Final Verification
   - Verify that both servers are running and summarize the ports.