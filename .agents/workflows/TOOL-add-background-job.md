---
description: Add a background job with BullMQ queue + worker + server.ts registration
---

1. Read `docs/skills/07-bullmq-background-jobs.md` for the exact BullMQ pattern and `lockDuration` rules.

2. Ask the user for:
   - Job name (e.g. `generate-report`, `sync-ema-data`)
   - Trigger: event-driven (from a controller) or scheduled (cron expression)
   - Expected max runtime (determines `lockDuration`)
   - Concurrency: 1 (sequential, memory-heavy) or higher
   - Retry strategy: attempts and backoff

3. **If event-driven (BullMQ):**
   - Create the queue in `src/common/queue/queues/<name>Queue.ts`
   - Create the worker in `src/common/queue/workers/<name>Worker.ts`
     - Use `bullmqRedisConnection` and `bullmqPrefix` from `@config/bullmqRedisConnection`
     - Set `lockDuration` >= expected max runtime in ms
     - Re-throw errors in the catch block (never swallow)
   - Register the worker import in `src/common/queue/workers/index.ts` or directly in `server.ts` via `runWorkers()`
   - Add the `queue.add()` call in the triggering controller

4. **If scheduled (cron):**
   - Create the cron in `src/modules/<module>/jobs/<module>.cron.ts`
   - Export the cron job and call `.start()` in `src/server.ts` inside `main()`

5. **Security:** Verify the job payload does not contain `password`, `token`, `otp`, `pin`, `secret_2fa`, or raw Sequelize model instances.

6. Validate:
// turbo
```bash
cd subqdocs-backend && npx tsc --noEmit
```

7. Report the queue/cron name, trigger, and files created/modified.
