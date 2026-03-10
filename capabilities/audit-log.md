# Audit Log

## Generation Instructions
Audit Log maintains an immutable record of significant actions taken within the application, attributed to users and timestamps. The storage model is append-only: no updates, no deletes. Rows are never modified after insertion.

Audit Log is passive — it subscribes to events emitted by other capabilities and records them. It does not contain business logic and does not emit events of its own.

For each event type in `config.subscribed-events`, subscribe to that event and write an audit record containing: event type, actor userId (from event payload where available), target entity type and id (where applicable), event payload (sanitized — strip credentials, tokens, and payment card data), and timestamp. Do not store sensitive values in audit records.

Generate the admin audit log viewer: a searchable, filterable table of audit records. Filters must include: actor user, event type, target entity, and date range. Generate CSV export of filtered results.

If `config.user-facing` is true, generate a user-facing audit log view scoped to the authenticated user's own actions — they can see what was done under their account but cannot see other users' records.

Audit records are retained for `config.retention-days` days. After retention expiry, records are deleted by a scheduled job. If `config.retention-days` is null, records are retained indefinitely.

capability: audit-log
type: singleton
version: 1.0

depends-on:
  - auth

config:
  subscribed-events:            # events to record — add any capability event by name
    - auth.login
    - auth.logout
    - auth.password-reset-completed
    - user-management.deleted
    - impersonation.started
    - impersonation.ended

  user-facing: false            # if true, users can view their own audit trail

  retention-days: 365           # null for indefinite retention

  admin-surface:
    log-viewer: true            # searchable, filterable audit log table
    export: true                # export filtered results to CSV
