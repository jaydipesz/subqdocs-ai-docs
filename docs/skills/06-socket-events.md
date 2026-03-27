# Skill: Emit and Receive Socket Events

**When NOT to use this:** Polling for updates (use TanStack Query `refetchInterval` instead). Sending data that is only needed on the next page load (REST is fine). One-time request/response patterns (REST is correct — sockets are for push).

---

## Files involved
```
subqdocs-backend/src/socket.ts                              ← ALL handlers go here
subqdocs-backend/src/common/latest-agents/utils/centralizedSocket.ts  ← AI node events
subqdocs-frontend/src/services/socket.ts                   ← singleton socket instance
subqdocs-frontend/src/components/layout/hooks/useStatusSocket.ts
```

---

## Room name reference (exact strings — must match)

| Room format | Joined via client event |
|---|---|
| `doctor_{doctorId}_{visitId}` | `joinRoom` |
| `recording_{visitId}_{deviceId}` | `joinRecordingRoom` |
| `EMA_{userId}` | `EMA_user_joined` |
| `optum_user_{userId}` | `optumJoinRoom` |
| `inquireInfo_{userId}_{patientId}_{screenName}` | `inquireInfoJoinRoom` |
| `inquireInfo_{userId}_{patientId}_{screenName}_{visitId}` | `inquireInfoJoinRoom` |

Underscores only — never hyphens. A mismatch silently drops the event.

---

## Backend: add a handler in socket.ts

All handlers go **inside** the `io.on('connection', socket => { ... })` block:

```ts
// src/socket.ts
io.on('connection', socket => {
    // ... existing handlers

    socket.on('myEvent', async (data) => {
        const authUser = socket.data?.user;  // set by socket auth middleware
        if (!authUser) {
            socket.emit('myEventError', { message: 'Unauthorized' });
            return;
        }

        // business logic — can use repositories here
        const result = await someRepo.get({ where: { id: data.id } });

        // respond to sender
        socket.emit('myEventResponse', result);

        // OR broadcast to a room
        io.to(`doctor_${authUser.id}_${data.visitId}`).emit('myEventBroadcast', result);
    });
});
```

Exact codebase reference — `joinRoom` handler in `src/socket.ts`:
```ts
socket.on('joinRoom', async ({ doctor_id, visit_id }) => {
    const roomName = `doctor_${doctor_id}_${visit_id}`;
    socket.join(roomName);
    const patientVisit = await getPatientVisitRepo({ where: { id: visit_id } });
    socket.emit('joinedRoom', { room: roomName, visitStatus: patientVisit?.visit_status });
});
```

---

## Backend: emit from a controller or BullMQ worker

```ts
import { getIO } from '@/socket';

// Must be called INSIDE the function body — not at module level
const io = getIO();
io.to(`doctor_${doctorId}_${visitId}`).emit('workflowExecutionCompleted', {
    visitId,
    doctor_id: doctorId,
    visitStatus,
    workflowName: '',
});
```

From `chunkProcessWorker.ts` (exact):
```ts
const io = getIO();
io.to(`doctor_${doctor_id}_${visitId}`).emit('workflowExecutionCompleted', {
    visitId,
    doctor_id,
    visitStatus: visitStatus || '',
    workflowName: '',
});
```

`getIO()` throws if called before the server initializes — never call it at module scope.

---

## Backend: AI workflow node events

```ts
import { emitNodeStatus } from '@common/latest-agents/utils/centralizedSocket';

// state must contain { doctor_id, visit_id, transcriptId }
emitNodeStatus('CHIEF_COMPLAINT_HPI_ALLERGY', state, 'STARTED', {});
emitNodeStatus('CHIEF_COMPLAINT_HPI_ALLERGY', state, 'COMPLETED', { data: result });
emitNodeStatus('ALL_TAB_STATUS', state, 'FAILED', {});
```

Valid node names are the section constants in `chunkProcessWorker.ts` (e.g., `SKIN_HISTORY`, `MEDICATIONS`, `EXAMINATION`, etc.).

---

## Frontend: listen in a component

```ts
import { socket } from '../../services/socket';

useEffect(() => {
    socket.emit('joinRoom', { doctor_id: user.id, visit_id: visitId });

    socket.on('myEventBroadcast', (data) => {
        // handle data
    });

    return () => {
        socket.off('myEventBroadcast');  // ← cleanup prevents listener stacking
    };
}, [visitId]);
```

---

## Checklist
- [ ] Handler is inside `io.on('connection', socket => {...})` in `socket.ts`
- [ ] Handler checks `socket.data?.user` before any logic
- [ ] Room name uses underscore format matching the table above
- [ ] `getIO()` called inside function body, never at module level
- [ ] Frontend `useEffect` cleanup calls `socket.off('eventName')`
- [ ] AI workflow events use `emitNodeStatus()`, not raw `io.to(...).emit(...)`
