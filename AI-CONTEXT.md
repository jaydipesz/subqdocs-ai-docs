# SubQDocs — AI Context (Root)

This is the **shared bridge document** — what crosses both sides of the system.  
For side-specific details, go to the dedicated files:

| Context | File |
|---|---|
| 🖥 Frontend | [`AI-CONTEXT-FRONTEND.md`](AI-CONTEXT-FRONTEND.md) |
| ⚙️ Backend | [`AI-CONTEXT-BACKEND.md`](AI-CONTEXT-BACKEND.md) |
| 🔧 Skills | [`docs/skills/INDEX.md`](docs/skills/INDEX.md) |

> **Skills:** Before performing a specific operation, load the relevant skill from `docs/skills/`. Each skill covers one operation end-to-end.

---

# Project

SubQDocs is a **medical practice management platform** for doctors and clinical staff. It unifies:

- **AI-powered visit note generation** — voice → Deepgram transcription → LangChain/LangGraph SOAP note
- **Patient scheduling and management** — create, edit, track visits across statuses
- **Medical records** — notes, transcripts, summaries, pre-visit forms
- **Billing/Ledger** — charges, payments, batches, insurance
- **Prescriptions, Lab Orders, Pathology**
- **E-Fax** — Westfax integration
- **Digital Consent + Intake Forms** — patient-facing with token auth
- **Third-party EHR sync** — EMA and Optum integrations
- **Stripe subscriptions** — organization billing
- **Calendar** — scheduling, office hours, unavailability with recurrence
- **Electronic Health Record (EHR) sync** — EMA and Optum integrations

---

# Environment Context

The project operates in three primary environments defined by `NODE_ENV`:

1.  **`local` / `development`**: Used for active coding. Combined via `isLocal` check in many modules.
2.  **`production`**: The live clinical environment.
3.  **`test`**: Used for Vitest and CI pipelines.

**Enforcement:**
- Sensitive sync operations (EMA, Optum) must have production guards: `if (NODE_ENV === "production") { ... }`.
- In `isLocal`, focus on fast feedback and mock data where external costs are involved (e.g., Westfax).

---

# Architecture Overview

1. **Frontend** triggers action → API call via typed Axios wrappers (see `AI-CONTEXT-FRONTEND.md`)
2. **Backend** receives via Express → middleware chain → controller → PostgreSQL via Sequelize (see `AI-CONTEXT-BACKEND.md`)
3. **Response** via `generalResponse()` (see Rule 6). Axios interceptor handles 401/404 globally.

### Real-Time Sync

Socket.io server in `subqdocs-backend/src/socket.ts`, attached to the same HTTP server as Express.  
Auth: token from handshake → decoded → `user.token === dbUser.token` (single-session enforcement).  
Public namespace `/chatbot` (no auth) — patient-facing chatbot.  
→ See [`docs/skills/06-socket-events.md`](docs/skills/06-socket-events.md) for room names, handlers, emit patterns.

### File Uploads

→ See [`docs/skills/04-s3-file-upload.md`](docs/skills/04-s3-file-upload.md) for S3, multipart, eFax, Multer bypass.

### PDF Generation

Server-side: PDFKit streams to HTTP response or S3.  
Frontend: PDF.js worker configured in `main.tsx` (see `AI-CONTEXT-FRONTEND.md`).

---

# Domain Rules

## PHI Fields — Never Log or Expose

**Patient:** `first_name`, `last_name`, `date_of_birth`, `email`, `contact_no`, `cellphone`, `home_phone`, `address`, `street_address`, `zipcode`, `gender`, `race`, `ethnicity`, `gender_identity`, `sexual_orientation`, `profile_image`, `sticky_note`

**Insurance:** all fields in `patient_insurance`

**Medical History:** all fields in `patient_medical_history`

**User:** `password`, `secret_2fa`, `otp`, `pin`, `token`, `optum_password`, `optum_username`

## Auth Rules

- **JWT stored in DB:** Login generates a new token written to `users.token`. Socket validation checks `user.token === incomingToken` — enforces single active session.
- **Dual-auth routes:** `authOrIntakeSessionMiddleware` allows either staff JWT or patient intake session token.

---

# Never Do

> These are the highest-severity rules. Violations cause data leaks, security breaches, or data loss.  
> Enforcement rules are in `.agents/rules/production-rules.md`. Detailed patterns are in `docs/skills/`.

## Security
- **Check Rule 1** before querying clinical tables (PHI protection).
- **Never bypass `authMiddleware`** on patient data endpoints.

## Data Integrity
- **Check Rule 3** before deleting records (Soft Delete).
- **Never upload large files via `PutObject`** — use multipart abstraction in `src/common/s3/`.
