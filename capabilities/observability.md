# Observability

## Generation Instructions

Observability generates structured logging, error tracking, health check endpoints, graceful shutdown handling, and optional APM integration. This is the operational layer for engineers and systems — it is distinct from the Audit Log functional capability, which is user/admin-facing and compliance-focused. Do not conflate them.

**Logging**

Generate a logger module at `lib/logger` using a structured logging library appropriate to the stack (pino for Node.js, structlog for Python, zap for Go). All log output is JSON to stdout — never to files, never to a remote logging service directly. Log aggregation is the platform's responsibility. This is 12-factor compliant and compatible with all major log aggregation platforms (Datadog, Splunk, CloudWatch, Loki).

Log level is configurable via `LOG_LEVEL` environment variable. Default to `info`. The logger module reads this at startup — changing log level requires a restart, not a code change.

Generate request logging middleware that records: HTTP method, path, status code, response time in milliseconds, and request ID. The request ID is a UUID generated per request and injected into all log lines emitted during that request's lifecycle via async context (AsyncLocalStorage in Node.js, context variables in Python). This makes it possible to trace all log lines for a single request.

**Critical constraint**: no sensitive fields may appear in log output. Generate explicit redaction for: passwords, tokens, API keys, credit card numbers, SSNs, and any field whose name contains `password`, `token`, `secret`, `key`, `authorization`, or `credential`. The logger module must apply this redaction automatically — callers should not need to remember to redact.

**Error Tracking**

If `config.error-tracking` is not `none`, generate the provider integration:
- `sentry`: initialize Sentry SDK with `SENTRY_DSN` env var. Capture unhandled exceptions and unhandled promise rejections automatically. Generate `captureError(error, context?)` for manual capture. Attach user ID and request ID to error reports when available.
- `rollbar`: equivalent setup with `ROLLBAR_TOKEN`.

Generate a `captureError(error, context?)` export from `lib/logger` regardless of provider — when `none`, it logs the error at `error` level. Callers always use `captureError`, never the provider SDK directly.

**Health Checks**

Generate a health router at `lib/health` with two endpoints:
- `GET /health/live`: liveness probe. Returns `200 OK` if the process is running. Always returns 200 — if the process can respond, it is alive. No database checks here.
- `GET /health/ready`: readiness probe. Returns `200 OK` if the process is ready to serve traffic. Checks: database connectivity (run a `SELECT 1`), Redis connectivity if Caching is present, any other backing service declared in the infrastructure. Returns `503 Service Unavailable` with a JSON body listing which checks failed if any check fails. Container orchestrators use this to gate traffic — a failed readiness probe removes the instance from the load balancer rotation without killing it.

Mount the health router before authentication middleware — health checks must be reachable without authentication.

**Graceful Shutdown**

Generate SIGTERM and SIGINT handlers unconditionally. This is not optional and is not gated by any config flag. On signal receipt:
1. Stop accepting new HTTP connections (call `server.close()`).
2. Wait for in-flight HTTP requests to complete, up to `config.shutdown-timeout-seconds`. After the timeout, force-close remaining connections.
3. If this is a worker process, finish processing the current job. Do not pick up new jobs after the signal.
4. Close database connections cleanly.
5. Close Redis connections if present.
6. Exit with code 0.

Log each step at `info` level so the shutdown sequence is visible in logs.

**APM**

If `config.apm` is not `none`, generate the provider integration at application startup. OpenTelemetry is preferred for vendor neutrality — it works with Datadog, New Relic, Jaeger, and others via exporters. APM instruments: HTTP request duration, database query duration, external HTTP call duration. Do not instrument every function — only I/O boundaries.

## Configuration

```yaml
capability: observability
type: infrastructure
version: 1.0

exposes:
  - logger: >
      lib/logger — structured JSON logger with automatic sensitive field redaction.
      request-logging middleware for HTTP servers.
      Imported by all capabilities that emit log output.
  - captureError: >
      lib/logger — captureError(error, context?) for manual error capture.
      Routes to error tracking provider or logs at error level if no provider configured.
  - healthRouter: >
      lib/health — mountable Express router (or equivalent) exposing /health/live and /health/ready.

config:
  logging:
    format: json               # always json — this is not configurable by design
    level: info                # debug | info | warn | error — overridable via LOG_LEVEL env var

  error-tracking: none         # none | sentry | rollbar

  apm: none                    # none | opentelemetry | datadog | newrelic

  health-checks: true          # always true — this is not configurable by design

  shutdown-timeout-seconds: 30 # time to wait for in-flight requests before force-closing
```
