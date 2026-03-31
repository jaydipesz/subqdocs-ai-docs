---
description: Add a real-time socket event end-to-end (backend handler + frontend listener)
---

1. Read `docs/skills/06-realtime-socket-events.md` for room name formats and handler placement rules.

2. Ask the user for:
   - Event name (e.g. `visitStatusUpdated`)
   - Direction: server→client, client→server, or bidirectional
   - Which room type it broadcasts to (see room table in skill 06)
   - Payload shape

3. **Backend — add the handler** in `subqdocs-backend/src/socket.ts` inside the `io.on('connection', socket => { ... })` block:
   - Check `socket.data?.user` before any logic
   - Join/emit to the correct room format (underscore-delimited)

4. **Backend — if emitting from a controller or worker**, use `getIO()` inside the function body (never at module level). Follow the exact pattern from `chunkProcessWorker.ts` in skill 06.

5. **Frontend — add the listener** in the target React component:
   - Import `socket` from `../../services/socket`
   - `socket.emit('joinRoom', ...)` to join the right room
   - `socket.on('<eventName>', handler)` in `useEffect`
   - `socket.off('<eventName>')` in the cleanup return

6. Validate:
// turbo
```bash
cd subqdocs-backend && npx tsc --noEmit
```

7. Report the event name, room format, and files modified.
