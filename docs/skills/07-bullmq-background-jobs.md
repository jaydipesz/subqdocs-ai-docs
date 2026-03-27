# Skill: Add a BullMQ Background Job or Cron

**When NOT to use this:** Fast operations (< 1 second) that don't risk timing out the HTTP response. One-off tasks that only run on demand — just run them synchronously in the controller. Sending a single email — `sendEmail` is already fire-and-forget (see skill 08).

---

## Files involved
```
subqdocs-backend/src/
  common/queue/queues/<name>Queue.ts        ← queue definition
  common/queue/workers/<name>Worker.ts      ← worker definition
  common/queue/workers/index.ts             ← may need to register worker here
  modules/<module>/jobs/<name>.cron.ts      ← cron jobs (not queue jobs)
  server.ts                                 ← start workers and crons
  config/bullmqRedisConnection.ts           ← import from here
```

---

## Type A — Event-driven queue job

### 1. Define the queue

```ts
// src/common/queue/queues/myFeatureQueue.ts
import { Queue } from 'bullmq';
import { bullmqRedisConnection, bullmqPrefix } from '@config/bullmqRedisConnection';

export const myFeatureQueue = new Queue('my-feature-queue', {
    connection: bullmqRedisConnection,
    prefix: bullmqPrefix,
    defaultJobOptions: {
        attempts: 3,
        backoff: { type: 'exponential', delay: 2000 },
        removeOnComplete: { age: 3600, count: 100 },
        removeOnFail: { age: 86400 },
    },
});
```

### 2. Define the worker

```ts
// src/common/queue/workers/myFeatureWorker.ts
import { Job, Worker } from 'bullmq';
import { bullmqRedisConnection, bullmqPrefix } from '@config/bullmqRedisConnection';
import { logger } from '@utils/logger';

export const myFeatureWorker = new Worker(
    'my-feature-queue',         // must match queue name exactly
    async (job: Job) => {
        try {
            logger.info(`[MyFeatureWorker] job ${job.id}`);
            // business logic
        } catch (error: any) {
            logger.error(`[MyFeatureWorker] failed:`, error);
            throw error;            // ← MUST re-throw — swallowed errors mark the job as succeeded
        }
    },
    {
        concurrency: 1,             // set to 1 for memory-intensive or order-sensitive work
        connection: bullmqRedisConnection,
        prefix: bullmqPrefix,
        lockDuration: 300000,       // milliseconds — must be >= expected max job runtime
    }
);
```

From `chunkProcessWorker.ts` (exact worker options):
```ts
{
    concurrency: 1,
    connection: bullmqRedisConnection,
    prefix: bullmqPrefix,
    lockDuration: 1800000,   // 30 minutes — matches workflow timeout
    stalledInterval: 3000,
    maxStalledCount: 1,
}
```

If `lockDuration` is shorter than the job runtime, another worker instance will steal the job mid-execution, causing duplicate processing.

### 3. Start the worker in server.ts

Importing the worker file is sufficient to start it (the `Worker` constructor auto-starts). The existing pattern in `server.ts` imports workers via `runWorkers()` which is defined in `src/common/queue/workers/index.ts`. Add your worker import there if that file exists, or directly in `server.ts`.

```ts
// src/server.ts
runWorkers();   // already called — registers all workers
```

### 4. Enqueue from a controller

```ts
import { myFeatureQueue } from '@common/queue/queues/myFeatureQueue';

await myFeatureQueue.add('my-feature-job', {
    visitId,
    organizationId: loggedInUser.organization_id,
    loggedInUser: { id: loggedInUser.id },  // ← do NOT pass full Sequelize model (not serializable)
});

return generalResponse(res, null, 'Processing started', 'success', false, 200);
// toast param is false — user doesn't need a toast for async work
```

---

## Type B — Scheduled cron

The real pattern in this codebase: cron is defined with `cron.schedule(...)` directly (no `{ scheduled: false }`), exported, and `.start()` is called in `server.ts`:

```ts
// src/modules/my-module/jobs/my-module.cron.ts
import cron from 'node-cron';
import { logger } from '@utils/logger';

export const myModuleCronJob = cron.schedule('0 0 * * *', async () => {
    try {
        logger.info('[MyModuleCron] Starting');
        // business logic
        logger.info('[MyModuleCron] Done');
    } catch (error) {
        logger.error('[MyModuleCron] Failed:', error);
    }
});
```

```ts
// src/server.ts — inside main()
import { myModuleCronJob } from '@modules/my-module/jobs/my-module.cron';

myModuleCronJob.start();
```

From `server.ts` (exact):
```ts
hourlyCronJob.start();
optumLabsCronJob.start();
subscriptionCronJob.start();
autoDeleteTranscriptsCronJob.start();
consecutiveUsageCheckCronJob.start();
inactivityCheckCronJob.start();
```

---

## PHI / security rules for job payloads

Job data is written to Redis. Never include in the payload: `password`, `token`, `otp`, `pin`, `secret_2fa`, `optum_password`. Pass `{ id: loggedInUser.id }` rather than the full user object.

---

## Checklist
- [ ] Queue name string matches between `new Queue(...)` and `new Worker(...)`
- [ ] Queue and worker both use `bullmqRedisConnection` and `bullmqPrefix`
- [ ] Worker re-throws errors (no swallowed catch)
- [ ] `lockDuration` ≥ expected max job runtime in milliseconds
- [ ] Worker registered/imported so it starts with `runWorkers()`
- [ ] Cron exported and `.start()` called in `server.ts` `main()`
- [ ] Job payload contains no raw Sequelize model instances or sensitive fields
