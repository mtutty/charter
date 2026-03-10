# Webhooks

## Generation Instructions
Webhooks delivers application events to user-configured external HTTP endpoints. Generate endpoint registration UI: URL input, event type multi-select from `config.subscribable-events`, and automatic secret generation (displayed once at creation for the user to copy — used to verify webhook signatures).

Generate delivery logic: on each subscribable event, find all registered endpoints subscribed to that event type and deliver the event payload via HTTP POST. Include an `X-Webhook-Signature` header containing an HMAC-SHA256 signature of the request body using the endpoint's secret. Always use `config.signature-method`.

Generate retry logic per `config.retry-policy`: on non-2xx response or connection failure, retry with exponential backoff up to `config.retry-policy.max-attempts`. After all retries exhausted, mark the delivery as failed.

Generate a delivery log per endpoint: each delivery attempt recorded with event type, attempt number, response status, response body (truncated), and timestamp. The delivery log is user-facing — endpoint owners can inspect it.

If `config.testing-tool` is true, generate a webhook testing UI that sends a sample payload for a selected event type to the registered endpoint on demand.

Automatically disable endpoints that have exceeded a configurable consecutive failure threshold. Notify the endpoint owner via Inbox or Mailer if those capabilities are present.

capability: webhooks
type: singleton
version: 1.0

depends-on:
  - auth

emits:
  - event: webhooks.delivery-succeeded
    payload:
      endpointId: string
      eventType: string
      deliveryId: string
      responseStatus: number
  - event: webhooks.delivery-failed
    payload:
      endpointId: string
      eventType: string
      deliveryId: string
      attempts: number

config:
  subscribable-events:          # application events that can be subscribed via webhook
    - auth.registered
    - user-management.deleted
    - billing.plan-upgraded
    - billing.subscription-cancelled

  retry-policy:
    max-attempts: 5
    backoff: exponential        # exponential | linear | fixed

  signature-method: hmac-sha256

  auto-disable-after-failures: 10   # consecutive failures before endpoint is disabled

  testing-tool: true            # generate webhook testing UI
