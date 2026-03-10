# Activity Feed

## Generation Instructions
Activity Feed is a chronological log of significant actions taken in the system, visible to users as a shared record of what happened. It is distinct from Inbox (system messages directed at a specific individual user) and Audit Log (admin-only, compliance-focused, immutable). Activity Feed is a shared social record: users see what has happened in their context, not messages addressed to them.

Activity Feed is passive — it subscribes to events emitted by other capabilities and renders them as feed entries. It does not emit events of its own.

For each event type in `config.subscribed-events`, subscribe to that event and write a feed entry containing: event type, a human-readable description generated from the payload, actor user reference, target entity reference (where applicable), and timestamp.

Generate an activity feed UI: a reverse-chronological list of entries with pagination. Each entry displays the actor's avatar (from user-management), a description of the action, the target entity as a link, and the timestamp. Generate filtering by entity type and by actor user.

If `config.scope` is `global`, all authenticated users see the same feed containing all activity. If `contextual`, scope feed entries to the user's organizational context (requires Multi-tenancy capability) or to entities the user has access to.

Retain feed entries for `config.retention-days` days, then delete via scheduled job. If `config.retention-days` is null, retain indefinitely.

capability: activity-feed
type: singleton
version: 1.0

depends-on:
  - auth
  - user-management

config:
  scope: contextual             # global | contextual

  retention-days: 90            # null for indefinite retention

  subscribed-events:            # events to display in the feed
    - comments.posted
    - user-management.profile-updated
    - sharing.resource-shared
