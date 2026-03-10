# Background Jobs

## Generation Instructions

Background Jobs provides two things: durable asynchronous job processing, and the runtime implementation of Charter's capability event system. The `emits` and `subscribes-to` contracts declared in functional capability files are not in-process function calls — they are jobs. When a capability emits an event, it enqueues a job. When a capability subscribes to an event, it registers a job handler. This is the mechanism that makes capability integration work across process boundaries and survive process restarts.

Generate a jobs client module at `lib/jobs` that exposes `enqueue(jobName, payload, options?)` and `schedule(cronExpression, jobName, payload)`. All capabilities that emit events use `enqueue`. All capabilities that declare scheduled work (trial expiry, scheduled report delivery, email batching) use `schedule`.

Generate a worker entrypoint at `worker.ts` (or equivalent) that is separate from the main application entrypoint. The worker imports all job handlers and starts processing. The app and worker run the same codebase — they are the same Docker image started with different commands (`node dist/server.js` vs `node dist/worker.js`). Never merge the worker into the app process unless `config.transport` is `in-process`.

Resolve the transport provider from `config.transport`:

- `in-process`: Use Node.js `EventEmitter` (or language-equivalent). Do not generate a worker entrypoint — handlers run synchronously in the emitting process. Generate a startup warning log: `"Background jobs transport is in-process. This is not safe for multi-instance deployments or separate worker containers."` If `infrastructure.scalability.horizontal` is `true`, escalate to a startup error and refuse to start.
- `database`: Use pg-boss. Jobs are stored in a `pgboss` schema in the primary PostgreSQL database. No additional infrastructure required. Generate pg-boss client setup with connection string from `DATABASE_URL`.
- `redis`: Use BullMQ. Jobs are stored in Redis. Generate BullMQ client setup with connection from `REDIS_URL`. If Caching is also present and uses Redis, share the Redis connection.
- `broker`: Generate the appropriate client for `config.provider` (RabbitMQ → amqplib, SQS → aws-sdk, NATS → nats.js). Generate connection setup, channel/queue declaration, and dead-letter queue configuration.

For `database`, `redis`, and `broker` transports, generate retry logic per `config.retry`. Failed jobs that exhaust retries go to a dead-letter store. Generate a dead-letter viewer in the admin surface if `config.admin-dashboard` is `true`.

When `config.coordination.mutex` is `true`, generate deduplication logic for jobs that must not execute more than once for a given key within `config.coordination.deduplication-window`. For the `database` transport, use PostgreSQL advisory locks. For `redis` and `broker`, use job ID deduplication. Apply mutex enforcement to any job handler registered by Billing (payment processing), Payment (charge execution), and Mailer (notification dispatch) — these are the canonical cases where duplicate execution has financial or user-experience consequences.

Generate a scheduled job runner that evaluates cron expressions and enqueues the appropriate jobs. Scheduled jobs declared by other capabilities (Billing trial expiry, Reporting scheduled delivery) are registered here.

If `config.admin-dashboard` is `true`, generate a job monitoring UI at `/admin/jobs` showing queue depth, processing rate, failed jobs, and dead-letter contents. Protect this route with the admin role from the Permissions capability if present.

## Configuration

```yaml
capability: background-jobs
type: infrastructure
version: 1.0

exposes:
  - jobs-client: >
      lib/jobs — enqueue(jobName, payload, options?) and schedule(cronExpression, jobName, payload).
      Imported by any capability that emits events or declares scheduled work.
  - worker-entrypoint: >
      worker.ts — separate process entrypoint that registers all job handlers and starts processing.
      Not generated when transport is in-process.

config:
  transport: database          # in-process | database | redis | broker
  provider: pg-boss            # pg-boss (database) | bullmq (redis) | rabbitmq | sqs | nats (broker)
  # provider is resolved automatically from transport unless overridden

  retry:
    attempts: 3
    backoff: exponential       # exponential | fixed
    backoff-delay: 5s          # initial delay; doubles on each retry for exponential

  coordination:
    mutex: true                # enforce at-most-once delivery for sensitive operations
    deduplication-window: 60s  # jobs with same name+key within this window are deduplicated

  admin-dashboard: false       # generate job monitoring UI at /admin/jobs
```
