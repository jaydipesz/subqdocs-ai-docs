# Implementation Plan — iPad Chatbot Session Persistence via Redis

## Problem Summary

The chatbot session is lost when an iPad user backgrounds the app or locks the screen. Four root causes were confirmed by code analysis:

| # | Root Cause | Evidence in Code |
|---|-----------|-----------------|
| 1 | **Session keyed by `socket.id`** | `sessions.set(socketId, state)` in [chatbot.service.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-backend/src/modules/chatbot/chatbot.service.ts#L45) |
| 2 | **Instant deletion on disconnect** | `sessions.delete(socketId)` called inside the `disconnect` listener in [chatbot.handler.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-backend/src/modules/chatbot/chatbot.handler.ts#L177) |
| 3 | **No reconnection handshake** | Frontend `connect` handler in [chatbotSocketService.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-frontend/src/components/Chatbot/chatbotSocketService.ts#L35-L38) only logs, never tells server who it is |
| 4 | **Volatile in-memory `Map`** | `const sessions = new Map<string, SessionState>()` at [chatbot.service.ts:20](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-backend/src/modules/chatbot/chatbot.service.ts#L20) |

---

## Proposed Changes

### Backend — Redis Session Layer

#### [NEW] [redis.session.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-backend/src/modules/chatbot/utils/redis.session.ts)

Redis CRUD helpers following the existing pattern in [redisSessionManagement.helper.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-backend/src/modules/transcript/helpers/redisSessionManagement.helper.ts). Uses the shared `redisClient` singleton.

- `saveChatSession(sessionId, state)` — Serializes `SessionState` → JSON, stores with `setEx` and a **2-hour TTL** (configurable via constant).
- `getChatSession(sessionId)` — Retrieves JSON from Redis and hydrates it back into a full `SessionState` instance via `fromJSON()`.
- `deleteChatSession(sessionId)` — Explicit delete for resets.
- `refreshSessionTTL(sessionId)` — Bumps the TTL on every interaction to keep active sessions alive.
- Key prefix: `chatbot:session:` (e.g., `chatbot:session:<uuid>`).

---

#### [MODIFY] [session.state.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-backend/src/modules/chatbot/session/session.state.ts)

Add two serialization methods to the `SessionState` class:

- **`toJSON(): string`** — Returns `JSON.stringify(this)`. The class is already a plain object with serializable fields.
- **`static fromJSON(json: string): SessionState`** — Reconstructs a full `SessionState` instance from stored JSON. Critically:
  - Casts `last_interaction_timestamp` back to a `Date` object.
  - Re-links the `complaints[].hpi` sub-objects to maintain correct references for `this.hpiState`.
  - Returns an instance with all class methods (`updateHPI`, `isHPIComplete`, etc.) intact.

> This leverages the existing `clone()` method's approach (L357-371) of `JSON.parse` + `Object.assign`.

---

#### [MODIFY] [chatbot.service.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-backend/src/modules/chatbot/chatbot.service.ts)

This is the biggest change. The in-memory `Map<string, SessionState>` is replaced entirely.

| Current (socket.id → memory) | New (session.id → Redis) |
|------|------|
| `sessions.set(socketId, state)` | `await saveChatSession(state.session_id, state)` |
| `sessions.get(socketId)` | `await getChatSession(sessionId)` |
| `sessions.delete(socketId)` | `await deleteChatSession(sessionId)` |
| `cleanupIdleSessions()` manual loop | **Removed** — Redis TTL handles this automatically |

**Key design decisions:**
- **Socket→Session in-memory map**: A lightweight `Map<string, string>` (`socketId` → `sessionId`) is kept *only* for the disconnect handler to know which session a socket belonged to. This map holds no clinical data.
- All `public static` methods change from `socketId` parameter to `sessionId` parameter.
- After every AI turn completes, `saveChatSession` is called to persist the updated state (atomic persistence).
- `processMessage` and `resetSession` become **async** for the Redis calls.

---

#### [MODIFY] [chatbot.events.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-backend/src/modules/chatbot/socket/chatbot.events.ts)

Add one new event constant:
```typescript
RECONNECT_SESSION: 'chatbot:reconnect_session',
```

---

#### [MODIFY] [chatbot.handler.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-backend/src/modules/chatbot/chatbot.handler.ts)

Three changes:

1. **New `RECONNECT_SESSION` handler** — Client sends `{ sessionId, patientId, visitId }`. Server:
   - Fetches session from Redis via `getChatSession`.
   - Validates `patientId`/`visitId` match the stored record (identity guard).
   - Registers the new `socket.id` → `sessionId` mapping.
   - Emits `SESSION_STARTED` + `HPI_PROGRESS` + replay of the latest `messages` array so the client can rebuild the conversation UI.

2. **Modify `disconnect` handler** — Instead of deleting the session, it only removes the volatile `socketId → sessionId` mapping. The Redis session remains intact and untouched.

3. **Update all handlers** — `START_SESSION`, `SEND_MESSAGE`, `RESET_SESSION` are updated to use `sessionId` instead of `socket.id` for session lookups, and persist to Redis after mutations.

---

### Frontend — Reconnection Logic

#### [MODIFY] [chatbotSocketService.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-frontend/src/components/Chatbot/chatbotSocketService.ts)

- Store the current `sessionId` in the service instance.
- On the `connect` event: if a `sessionId` already exists, emit `RECONNECT_SESSION` with the stored `sessionId`, `patientId`, and `visitId`.
- Add a public `setSessionId(id)` method for the hook to call when a session starts.

#### [MODIFY] [useChatbot.ts](file:///home/sagar/Desktop/Project/SubQDocs/subqdocs-frontend/src/components/Chatbot/useChatbot.ts)

- **Persist `sessionId` to `localStorage`** alongside the existing `patientId` and `visitId` storage.
- **`visibilitychange` listener**: When `document.visibilityState === 'visible'`, check socket connection and force reconnect if needed.
- On `SESSION_STARTED` from a reconnect, restore the conversation messages from the server's response (the message history in the reconnect payload).
- On `handleSessionReset`, clear the stored `sessionId` from localStorage.

---

## Safety & Consistency Measures

| Measure | Detail |
|---------|--------|
| **Class Hydration** | `fromJSON()` explicitly reconstructs `Date` objects and `complaints` array structure. All methods (`isHPIComplete`, `updateHPI`, etc.) work on recovered instances. |
| **Identity Guard** | Every `RECONNECT_SESSION` validates `patientId` + `visitId` match the stored session. Prevents accidental or malicious hijacking. |
| **Atomic Persistence** | `saveChatSession()` called at the end of every turn — Redis always has the latest state. |
| **Graceful Degradation** | If Redis is unreachable, the service falls back to a "start new session" flow — no crash. |
| **TTL Auto-Cleanup** | 2-hour default TTL replaces the manual `cleanupIdleSessions()` loop. TTL is refreshed on every interaction. |
| **Volatile Socket Map** | `Map<socketId, sessionId>` is ephemeral and holds zero clinical data. Safe to lose on server restart. |

---

## Verification Plan

### Manual Testing

> [!IMPORTANT]
> Since there are no existing chatbot tests in the codebase and the module relies on live Socket.io + Redis + OpenAI orchestration, manual testing is the appropriate approach.

#### Test 1: Basic Reconnection (iPad Simulation)
1. Start the backend with `npm run dev`.
2. Open the chatbot in a browser and send 2-3 messages to establish context.
3. Open DevTools → Network → disconnect the WebSocket (or run `chatbotSocketService.disconnect()` in console).
4. Wait 5 seconds, then reconnect (run `chatbotSocketService.initializeSocket()` or refresh).
5. **Expected**: The chat history reappears and AI maintains context. No "start new session" error.

#### Test 2: Server Restart Persistence
1. Send a few messages so the session has HPI data.
2. Stop and restart the backend server (`Ctrl+C` → `npm run dev`).
3. Trigger a reconnect from the frontend.
4. **Expected**: The session is restored from Redis. Conversation continues.

#### Test 3: Session Expiry
1. Set TTL to a short value (e.g., 30 seconds) for testing.
2. Start a session, then wait >30 seconds without interaction.
3. Attempt reconnection.
4. **Expected**: Redis returns `null`, client receives a `NO_SESSION` error prompting a fresh start.

#### Test 4: Identity Mismatch Guard
1. Start a session with `patientId=A`.
2. Attempt `RECONNECT_SESSION` with the correct `sessionId` but `patientId=B`.
3. **Expected**: Server rejects with `AUTH_MISMATCH` error.
