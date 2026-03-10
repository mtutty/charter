# Charter: Capability Development Plan

## How To Use This Document

This plan sequences the development of all Charter capability definition files. Each capability produces two artifacts: a **Generation Instructions** section (directive prose for the LLM generating code) and a **Configuration block** (YAML defining the capability's config surface, event emissions, and dependency declarations).

Each work item is designed to be completed in an independent working context. Before starting any item, the working context must read:

1. `charter-conversation-summary.md` — conceptual foundation and design principles
2. `capabilities/auth.md` — the reference implementation; all capability files should follow its tone and structure
3. This document — for orientation on where the item fits and what depends on it

After completing an item, update its status and record any design decisions or deviations in the Outcome Notes field.

---

## Capability File Format

Every capability file follows this structure:

```markdown
# {Capability Name}

## Generation Instructions
{Directive prose. Every sentence is an instruction or a constraint.
References config keys directly. No human-orientation content.}

## Configuration
\`\`\`yaml
capability: {name}
type: singleton | attachable
version: 1.0

depends-on:
  - {capability-name}

emits:
  - event: {capability}.{event-name}
    payload:
      {key}: {type}

config:
  {layered configuration blocks}
\`\`\`
```

The `type` field distinguishes:
- **singleton** — one instance per application (Auth, Billing, Multi-tenancy)
- **attachable** — applied to one or more entities (Comments, Versioning, Soft Delete)

Attachable capabilities include an `attaches-to` config key declaring which entities they are applied to.

---

## Status Key

- ⬜ Not started
- 🟡 In progress
- ✅ Complete
- ⏸ Blocked (see notes)

---

## Wave 1 — No Dependencies

These capabilities have no `depends-on` entries and can be developed in parallel.

---

### Auth ✅

**Type:** Singleton
**Depends on:** —
**Reference file:** `capabilities/auth.md`

This capability is complete and serves as the reference implementation for all others.

**Outcome Notes:** Established the capability file format, narrative tone, and YAML config structure. The `admin-surface` block pattern — declaring staff-facing operations within the primary capability — should be carried forward where applicable rather than treating admin surfaces as separate capabilities.

---

### Soft Delete + Trash ✅

**Type:** Attachable
**Depends on:** —

**Prior Artifacts:**
- `capabilities/auth.md` (reference format)

**Deliverables:**
- `capabilities/soft-delete.md` with Generation Instructions and Configuration block

**Design guidance:**
- Attachable to any entity. Config declares which entities have soft delete enabled.
- Generates: deleted_at timestamp field on affected entities, scoped queries that exclude soft-deleted records by default, a trash view per entity, restore and permanent-delete operations.
- Admin surface: bulk empty trash, view all deleted records across entities.
- Consider: should trash have a retention policy (auto-purge after N days)? Include as an optional config key.

**Verification:**
- Config block includes `attaches-to` key
- Event surface covers at minimum: record soft-deleted, record restored, record permanently deleted
- Generation Instructions reference config keys directly

**Outcome Notes:** _(fill in after completion)_

---

### Archiving ✅

**Type:** Attachable
**Depends on:** —

**Prior Artifacts:**
- `capabilities/auth.md` (reference format)
- `capabilities/soft-delete.md` if complete (related pattern — distinguish archiving from deletion clearly)

**Deliverables:**
- `capabilities/archiving.md`

**Design guidance:**
- Archiving is distinct from soft delete: archived records are intentionally preserved long-term, not pending deletion. They are removed from active workflows but remain accessible.
- Generates: archived_at field, archived_by field (user reference), archive/unarchive operations, archived records view separate from active records view.
- Consider: should archiving be reversible? (Yes, by default.) Should archived records be excluded from search? (Configurable.)

**Verification:**
- Narrative clearly distinguishes archiving from soft delete
- Config includes reversibility and search-exclusion options

**Outcome Notes:** _(fill in after completion)_

---

### Waitlist ✅

**Type:** Singleton
**Depends on:** —

**Prior Artifacts:**
- `capabilities/auth.md` (reference format)

**Deliverables:**
- `capabilities/waitlist.md`

**Design guidance:**
- Pre-registration surface: collect email addresses before the product is available or before a user can access a feature.
- Generates: waitlist signup form, position display, admin surface for viewing and managing waitlist entries, invite flow that converts waitlist entries to registrations.
- Events: user joined waitlist, user invited off waitlist, invitation accepted.
- Config: whether position is shown to the user, whether referral bumps position (connect to Referral capability if present).

**Verification:**
- Event surface covers the join → invite → accept lifecycle
- Admin surface declarations are present

**Outcome Notes:** _(fill in after completion)_

---

### Feedback Collection ✅

**Type:** Singleton
**Depends on:** —

**Prior Artifacts:**
- `capabilities/auth.md` (reference format)

**Deliverables:**
- `capabilities/feedback.md`

**Design guidance:**
- In-product mechanisms for collecting structured user feedback: NPS surveys, CSAT prompts, feature request forms, bug reports.
- Generates: survey/prompt trigger logic (on event, on schedule, or manual), response collection and storage, admin surface for viewing and exporting responses.
- Config: which feedback types are enabled, trigger conditions per type, whether responses are anonymous or attributed.
- Events: feedback submitted (with type and attributed user if not anonymous).

**Verification:**
- Config covers at minimum: feedback types, trigger mechanism, attribution model
- Admin surface includes response viewing and export

**Outcome Notes:** _(fill in after completion)_

---

### Announcements ✅

**Type:** Singleton
**Depends on:** —

**Prior Artifacts:**
- `capabilities/auth.md` (reference format)

**Deliverables:**
- `capabilities/announcements.md`

**Design guidance:**
- Broadcast communications from the application to all or segmented users: product updates, maintenance notices, new feature announcements.
- Distinct from Inbox (which is directed at individual users) and Mailer (which is transactional).
- Generates: announcement creation and scheduling UI (admin), announcement display surface (user-facing — banner, modal, or notification center entry), read/dismiss tracking per user.
- Config: display formats available (banner, modal, notification center), whether announcements can be targeted by user segment, whether dismiss state is tracked.

**Verification:**
- Narrative clearly distinguishes Announcements from Inbox and Mailer
- Config covers display format and targeting options

**Outcome Notes:** _(fill in after completion)_

---

### File / Media Management ✅

**Type:** Attachable
**Depends on:** —

**Prior Artifacts:**
- `capabilities/auth.md` (reference format)

**Deliverables:**
- `capabilities/file-management.md`

**Design guidance:**
- Upload, storage, retrieval, and management of files and media assets attached to entities or standalone.
- Generates: upload UI (drag-and-drop, progress), storage integration (S3-compatible by default), file metadata model, download and preview surfaces, deletion with optional soft-delete integration.
- Config: accepted file types, size limits, storage provider, whether files attach to specific entities or are standalone, image processing options (resize, thumbnail generation).
- Events: file uploaded, file deleted.

**Verification:**
- Config covers storage provider, file type constraints, and attachment model
- Attachable pattern correctly implemented if files attach to entities

**Outcome Notes:** _(fill in after completion)_

---

## Wave 2 — Depends on Auth

These capabilities require Auth to be complete. They can be developed in parallel with each other.

---

### User Management ✅

**Type:** Singleton
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`

**Deliverables:**
- `capabilities/user-management.md`

**Design guidance:**
- The user-facing and admin-facing surfaces for managing user accounts after identity is established.
- User-facing: profile editing, avatar upload, account deletion request, notification preferences.
- Admin-facing: user search, view account details, edit profile fields, manage account status. (Auth owns suspend/force-reset; User Management owns everything else.)
- Generate a User entity with standard fields (id, email, name, avatar, created_at, status). Other capabilities extend this entity — User Management defines the base.
- Events: user profile updated, user deleted.

**Verification:**
- Clear boundary stated between Auth (identity) and User Management (profile)
- User entity definition is present in the config
- Admin surface declarations present

**Outcome Notes:** _(fill in after completion)_

---

### Onboarding ✅

**Type:** Singleton
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`

**Deliverables:**
- `capabilities/onboarding.md`

**Design guidance:**
- The structured flow a new user completes before reaching full product access. Subscribes to `auth.registered`.
- Generates: step-by-step onboarding UI, progress tracking, completion gating (optionally block access until complete), skip logic per step.
- Config: steps (ordered list with name, required/optional, completion condition), whether incomplete onboarding gates product access, whether onboarding can be re-triggered.
- Events: onboarding started, step completed, onboarding completed.

**Verification:**
- Subscribes to `auth.registered` explicitly noted
- Config allows arbitrary step definition
- Completion gating is a configurable option not a hard requirement

**Outcome Notes:** _(fill in after completion)_

---

### Inbox ✅

**Type:** Singleton
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`

**Deliverables:**
- `capabilities/inbox.md`

**Design guidance:**
- In-application notification surface directed at individual users. The notification bell, unread count, and feed of activity directed at a specific user.
- Distinct from Announcements (broadcast) and Activity Feed (system-wide activity log).
- Generates: notification bell with unread count, notification feed UI, mark-read and mark-all-read operations, notification preference controls (which types the user wants to receive).
- Config: notification types the application will send (defined as an enum the rest of the application emits against), retention period, whether real-time delivery is enabled (websocket) or polling.
- Other capabilities direct notifications to users by emitting events that Inbox subscribes to. Inbox does not need to know the business logic — it receives a notification payload and delivers it.
- Events: notification delivered, notification read.

**Verification:**
- Narrative distinguishes Inbox from Announcements and Activity Feed
- Config includes notification type registry
- Real-time vs. polling is a config option

**Outcome Notes:** _(fill in after completion)_

---

### Mailer ✅

**Type:** Singleton
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/inbox.md` if complete (contrast in-app vs. email delivery)

**Deliverables:**
- `capabilities/mailer.md`

**Design guidance:**
- Transactional email delivery: sending templated emails triggered by application events.
- Generates: email template system, delivery integration (SendGrid/Postmark/SES configurable), email preference management (user opt-out per email type), unsubscribe handling, bounce and delivery tracking.
- Config: email provider, from address and name, reply-to, email types defined (name, template, trigger event, opt-out-able flag), whether a preference center UI is generated.
- Other capabilities trigger emails by emitting events. Mailer subscribes and handles delivery.
- Events: email sent, email bounced, user unsubscribed from email type.

**Verification:**
- Config includes email type registry (parallel to Inbox notification type registry)
- Provider is configurable
- Unsubscribe and preference management are addressed

**Outcome Notes:** _(fill in after completion)_

---

### Audit Log ✅

**Type:** Singleton
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`

**Deliverables:**
- `capabilities/audit-log.md`

**Design guidance:**
- Immutable record of significant actions taken within the application, attributed to users and timestamps.
- Generates: audit event storage model (append-only), audit log viewer (admin), filtering by user/entity/action/date range, export.
- Config: which event types are logged (can subscribe to any emitted event across capabilities), retention period, whether log is exposed to end users (e.g., for compliance-sensitive apps) or admin-only.
- The audit log is passive — it subscribes to events emitted by other capabilities and records them. It does not emit events of its own.
- Admin surface: searchable, filterable log viewer with export.

**Verification:**
- Passive subscription model clearly stated — Audit Log does not emit events
- Config includes event type subscription list
- Admin surface requirements stated

**Outcome Notes:** _(fill in after completion)_

---

### API Access ✅

**Type:** Singleton
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`

**Deliverables:**
- `capabilities/api-access.md`

**Design guidance:**
- Programmatic access to application functionality via API keys or OAuth tokens, for third-party integrations or developer use.
- Generates: API key generation and management UI, key scoping model (which operations a key can perform), key rotation and revocation, API authentication middleware, developer documentation surface (optional).
- Config: authentication mechanism (API key, OAuth2, or both), whether keys are scoped by permission, rate limiting per key, whether a developer portal UI is generated.
- Events: API key created, API key revoked, rate limit exceeded.

**Verification:**
- Boundary between Auth (user sessions) and API Access (programmatic access) is clearly stated
- Scoping model is addressed in config

**Outcome Notes:** _(fill in after completion)_

---

### Webhooks ✅

**Type:** Singleton
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`

**Deliverables:**
- `capabilities/webhooks.md`

**Design guidance:**
- Outbound HTTP delivery of application events to user-configured endpoints.
- Generates: webhook endpoint registration UI, event type selection per endpoint, delivery with retry logic, delivery log (success/failure history), signature verification for receivers.
- Config: which application events can be subscribed to via webhook, retry policy, signature method (HMAC-SHA256 standard), whether a webhook testing tool is generated.
- Events: webhook delivery succeeded, webhook delivery failed (after retries exhausted).

**Verification:**
- Config includes subscribable event list
- Retry and signature verification are addressed
- Delivery log is included in generation scope

**Outcome Notes:** _(fill in after completion)_

---

### Feature Flags ✅

**Type:** Singleton
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`

**Deliverables:**
- `capabilities/feature-flags.md`

**Design guidance:**
- Runtime control over feature availability without code deployment. Flags can be boolean (on/off) or target specific user segments.
- Generates: flag evaluation logic (client and server), flag management admin UI, user/segment targeting configuration, flag state storage.
- Config: targeting strategies available (boolean, user list, percentage rollout, user attribute rule), whether flag state is evaluated server-side, client-side, or both, default state for undefined flags.
- Events: flag state changed, flag evaluated (optional, for analytics — likely high volume, consider carefully).

**Verification:**
- Targeting strategies are enumerated in config
- Default state for undefined/unknown flags is addressed
- Admin surface for flag management is declared

**Outcome Notes:** _(fill in after completion)_

---

### Search ✅

**Type:** Attachable
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`

**Deliverables:**
- `capabilities/search.md`

**Design guidance:**
- Full-text and filtered search across application entities.
- Generates: search index configuration per attached entity (which fields are indexed), search UI (input, results, facets), search API endpoint, index update triggers on entity create/update/delete.
- Config: search provider (database full-text, Elasticsearch, Typesense, Meilisearch), attached entities and indexed fields per entity, faceted filtering options, whether results respect permission boundaries.
- Events: search performed (for analytics, optional).

**Verification:**
- Provider is configurable
- Permission-aware results are addressed
- Attachable pattern correctly implemented

**Outcome Notes:** _(fill in after completion)_

---

### Import / Export ✅

**Type:** Attachable
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`

**Deliverables:**
- `capabilities/import-export.md`

**Design guidance:**
- Bulk data movement into and out of the application for attached entities.
- Generates: import UI (file upload, field mapping, validation preview, confirmation), export UI (format selection, filter options, download), background job handling for large datasets, error reporting for failed import rows.
- Config: attached entities, supported formats per direction (CSV, Excel, JSON), whether import is admin-only or user-facing, whether export is admin-only or user-facing, row limit before async processing is used.
- Events: import completed (with success/error counts), export completed.

**Verification:**
- Async processing for large datasets is addressed
- Format support is configurable per direction
- Error handling for partial import failures is addressed

**Outcome Notes:** _(fill in after completion)_

---

### Versioning / History ✅

**Type:** Attachable
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/soft-delete.md` if complete (related data lifecycle pattern)

**Deliverables:**
- `capabilities/versioning.md`

**Design guidance:**
- Immutable history of changes to entity records, with the ability to view and restore prior versions.
- Generates: version snapshot storage (on each update, store previous state), version history UI (timeline of changes with diff view), restore-to-version operation, attribution (who made each change).
- Config: attached entities, which fields are versioned (all or subset), retention (keep last N versions or all), whether restore creates a new version or overwrites current.
- Events: version created, version restored.

**Verification:**
- Field-level versioning selectivity is addressed in config
- Retention policy is configurable
- Restore behavior (new version vs. overwrite) is a config option

**Outcome Notes:** _(fill in after completion)_

---

### Reporting / Analytics ✅

**Type:** Singleton
**Depends on:** Auth

**Prior Artifacts:**
- `capabilities/auth.md`

**Deliverables:**
- `capabilities/reporting.md`

**Design guidance:**
- Internal reporting and analytics surfaces: dashboards, charts, and tabular reports over application data for admin or user consumption.
- Generates: report definition model (what data, what aggregation, what visualization), dashboard layout, scheduled report delivery (via Mailer if present), export to CSV/Excel.
- Config: report types enabled (predefined list or custom), audience (admin-only or user-facing), whether scheduled delivery is enabled, refresh interval for live data.
- This is internal analytics over the application's own data — not integration with external analytics platforms.

**Verification:**
- Boundary stated between internal reporting and external analytics integrations
- Audience (admin vs. user-facing) is a config option
- Scheduled delivery connects to Mailer if present

**Outcome Notes:** _(fill in after completion)_

---

## Wave 3 — Depends on Wave 2

These capabilities require one or more Wave 2 capabilities. Develop after their dependencies are complete.

---

### Permissions / RBAC ✅

**Type:** Singleton
**Depends on:** Auth, User Management

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/user-management.md`

**Deliverables:**
- `capabilities/permissions.md`

**Design guidance:**
- Role-based access control: what authenticated users are allowed to do.
- Generates: role definition model, role assignment to users, permission check middleware and helpers, role management admin UI.
- Config: roles (named list with descriptions), permissions per role (against a defined action list), whether roles are hierarchical, whether resource-level permissions are needed (i.e., user can edit their own records but not others'), default role for new users.
- Events: role assigned to user, role removed from user.
- Note: Permissions evaluates rules against an authenticated user. Auth establishes identity; Permissions determines capability.

**Verification:**
- Boundary between Auth and Permissions is clearly restated
- Resource-level permissions are addressed as a config option
- Default role for new registrations is in config

**Outcome Notes:** _(fill in after completion)_

---

### Billing ✅

**Type:** Singleton
**Depends on:** Auth, User Management

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/user-management.md`

**Deliverables:**
- `capabilities/billing.md`

**Design guidance:**
- Subscription and plan management: what plan a user or organization is on, trial state, upgrade/downgrade flows, and plan-based feature gating.
- Generates: plan definition model, subscription state tracking, upgrade/downgrade UI, trial management (start, expiry, conversion prompt), plan-based feature gate helpers, billing portal (typically delegated to payment provider).
- Config: plans (name, features included, trial eligible), billing interval (monthly/annual or both), trial duration, whether plan changes take effect immediately or at period end, which entity holds the subscription (user or org).
- Events: trial started, trial expired, plan upgraded, plan downgraded, subscription cancelled, subscription reactivated.
- Billing owns plan state. Payment owns transaction execution. They are coupled but separate.

**Verification:**
- Boundary between Billing (plan state) and Payment (transactions) clearly stated
- Trial lifecycle events are complete
- Feature gating helpers are in generation scope

**Outcome Notes:** _(fill in after completion)_

---

### Payment ✅

**Type:** Singleton
**Depends on:** Auth, Billing

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/billing.md`

**Deliverables:**
- `capabilities/payment.md`

**Design guidance:**
- Transaction execution: collecting payment details, processing charges, handling refunds, and managing payment methods.
- Generates: payment method collection UI (delegated to provider-hosted elements), payment method management (view, add, remove, set default), invoice history, refund initiation (admin), webhook handling for provider events.
- Config: payment provider (Stripe default), whether invoice history is user-facing or admin-only, whether refunds are admin-initiated or self-serve, supported payment methods (card, ACH, etc.).
- Events: payment succeeded, payment failed, refund issued, payment method added, payment method removed.
- Payment subscribes to Billing events (e.g., plan upgraded) to execute charges. Billing subscribes to Payment events (e.g., payment failed) to update subscription state.

**Verification:**
- Bidirectional relationship with Billing is clearly stated
- Provider webhook handling is in generation scope
- Refund initiation model (admin vs. self-serve) is a config option

**Outcome Notes:** _(fill in after completion)_

---

### Comments / Annotations ✅

**Type:** Attachable
**Depends on:** Auth, User Management

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/user-management.md`

**Deliverables:**
- `capabilities/comments.md`

**Design guidance:**
- Threaded comments and annotations attached to entities, attributed to users.
- Generates: comment thread UI, reply threading, edit and delete (own comments), moderation (admin delete any), reaction support (optional), mention support (optional, connects to Inbox if present).
- Config: attached entities, whether threading is enabled, whether reactions are enabled, whether @mentions are enabled, whether edit history is tracked, who can delete (own only, moderators, admins).
- Events: comment posted, comment edited, comment deleted, user mentioned (if mentions enabled).

**Verification:**
- Mention event connects to Inbox if present
- Deletion permission model is addressed in config
- Edit history is a config option

**Outcome Notes:** _(fill in after completion)_

---

### Activity Feed ✅

**Type:** Singleton
**Depends on:** Auth, User Management

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/user-management.md`
- `capabilities/inbox.md` if complete (contrast system activity vs. directed notifications)

**Deliverables:**
- `capabilities/activity-feed.md`

**Design guidance:**
- A chronological log of significant actions taken in the system, visible to users as a shared record of what happened.
- Distinct from Inbox (directed at a specific user) and Audit Log (admin-only, compliance-focused).
- Generates: activity event storage, activity feed UI, filtering by entity or user, pagination.
- Config: which events appear in the feed (subscribes to application events), whether the feed is global (all users see all activity) or scoped (users see activity in their context), retention period.
- Activity Feed is passive — it subscribes to events and renders them. It does not emit events.

**Verification:**
- Distinction from Inbox and Audit Log is clearly stated in narrative
- Passive subscription model stated — no events emitted
- Scope (global vs. contextual) is a config option

**Outcome Notes:** _(fill in after completion)_

---

### Sharing / Access Control ✅

**Type:** Attachable
**Depends on:** Auth, Permissions

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/permissions.md`

**Deliverables:**
- `capabilities/sharing.md`

**Design guidance:**
- Per-resource access grants: sharing specific records with specific users or groups, beyond the role-based rules defined in Permissions.
- Generates: share dialog UI, access level selection per share (view, comment, edit), share management (view who has access, revoke), link-based sharing (optional), notification to invitee (connects to Inbox/Mailer if present).
- Config: attached entities, access levels available (view/comment/edit or custom), whether link sharing is enabled, whether shares expire, whether public sharing is enabled.
- Events: resource shared, share revoked, share link created.

**Verification:**
- Relationship to Permissions clearly stated (Sharing extends Permissions to the resource level)
- Link sharing and expiry are config options
- Notification on share connects to Inbox/Mailer if present

**Outcome Notes:** _(fill in after completion)_

---

### Presence ✅

**Type:** Attachable
**Depends on:** Auth, User Management

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/user-management.md`

**Deliverables:**
- `capabilities/presence.md`

**Design guidance:**
- Real-time awareness of which users are viewing or active in the same context (document, page, record).
- Generates: presence tracking (heartbeat on active view), presence display UI (avatars of active users), cursor or selection sharing (optional, more complex), idle detection.
- Config: attached entities, heartbeat interval, idle timeout, whether cursor/selection sharing is enabled, max simultaneous users shown.
- Events: user became present, user became absent.
- Requires a real-time transport — websocket or SSE. Note this as a technical dependency in the narrative.

**Verification:**
- Real-time transport requirement is stated
- Idle detection and timeout are in config
- Cursor sharing is an optional config flag, not a default

**Outcome Notes:** _(fill in after completion)_

---

### In-app Messaging ✅

**Type:** Singleton
**Depends on:** Auth, User Management

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/user-management.md`
- `capabilities/inbox.md` if complete (contrast directed messaging vs. notifications)

**Deliverables:**
- `capabilities/messaging.md`

**Design guidance:**
- Direct and group messaging between users within the application: conversation threads, message history, read receipts.
- Distinct from Inbox (system-generated notifications directed at a user) and Announcements (broadcast from the application).
- Generates: conversation list UI, message thread UI, real-time message delivery (websocket), read receipt tracking, message search (optional), file attachment support (optional, connects to File Management if present).
- Config: whether group conversations are enabled, whether message history is retained (and for how long), whether read receipts are shown, whether file attachments are enabled, whether message search is enabled.
- Events: message sent, conversation created, user added to conversation.

**Verification:**
- Distinction from Inbox clearly stated
- Real-time delivery requirement noted
- File attachment connects to File Management if present

**Outcome Notes:** _(fill in after completion)_

---

## Wave 4 — Depends on Wave 3

---

### Multi-tenancy ✅

**Type:** Singleton
**Depends on:** Auth, User Management, Permissions

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/user-management.md`
- `capabilities/permissions.md`

**Deliverables:**
- `capabilities/multi-tenancy.md`

**Design guidance:**
- Organization-level data isolation: users belong to one or more organizations (teams, workspaces, accounts — the term is configurable), and data is scoped to the organization.
- Generates: organization entity, org creation and setup flow, member invitation and management, org-scoped data access middleware, org switcher UI (if users can belong to multiple orgs), org settings page.
- Config: the domain term for an organization (org, team, workspace, company, account), whether users can belong to multiple orgs, whether org creation is self-serve or admin-provisioned, member roles within an org (distinct from app-level roles in Permissions), data isolation strategy (row-level or schema-level).
- Events: org created, member invited, member joined, member removed, org deleted.

**Verification:**
- Domain term is configurable
- Multi-org membership is a config option
- Data isolation strategy is addressed
- Member roles within org vs. app-level roles in Permissions are distinguished

**Outcome Notes:** _(fill in after completion)_

---

### Impersonation ✅

**Type:** Singleton
**Depends on:** Auth, User Management, Permissions

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/user-management.md`
- `capabilities/permissions.md`

**Deliverables:**
- `capabilities/impersonation.md`

**Design guidance:**
- Allows privileged staff users to assume the identity of another user for support and debugging purposes.
- Generates: impersonation initiation UI (admin only), session-within-session model (original identity is preserved and recoverable), visible indicator in UI when impersonating, audit log entry on every impersonation start and end.
- Config: which roles can impersonate (admin only by default), whether impersonation requires a stated reason, whether impersonated sessions have reduced capabilities (e.g., cannot change password, cannot initiate payments), session duration limit for impersonated sessions.
- Events: impersonation started (with actor and target user), impersonation ended.
- Impersonation should always emit to Audit Log if present.

**Verification:**
- Audit log integration is stated as non-optional when Audit Log capability is present
- Reduced capabilities during impersonation is addressed
- Reason requirement is a config option

**Outcome Notes:** _(fill in after completion)_

---

### Referral / Affiliate ✅

**Type:** Singleton
**Depends on:** Auth, Billing

**Prior Artifacts:**
- `capabilities/auth.md`
- `capabilities/billing.md`

**Deliverables:**
- `capabilities/referral.md`

**Design guidance:**
- User referral program: unique referral links, tracking of referred signups, reward issuance on conversion.
- Generates: referral link generation, referral tracking (cookie-based attribution), referred signup detection, reward trigger on qualifying conversion, referral dashboard for users (link, stats, reward status), admin surface for program configuration and reward management.
- Config: reward type (account credit, plan extension, cash — credit is default), qualifying conversion event (registration, first payment, plan upgrade), reward amount, whether referee also receives a reward, referral link expiry.
- Events: referral link created, referred user registered, referral converted (qualifying event occurred), reward issued.

**Verification:**
- Qualifying conversion event is configurable
- Both referrer and referee rewards are addressed
- Attribution model (cookie-based) is stated

**Outcome Notes:** _(fill in after completion)_

---

## Dependency Map

```
Infrastructure Layer (no functional dependencies — declare before capabilities):
  Secrets / Environment Management
  Migrations
  Caching
  Background Jobs
  Real-time Transport
  Observability
  Containerization
  CI/CD

Wave 1 (no functional dependencies):
  Auth ✅
  Soft Delete + Trash ✅
  Archiving ✅
  Waitlist ✅
  Feedback Collection ✅
  Announcements ✅
  File / Media Management ✅

Wave 2 (depends on Auth):
  User Management ✅
  Onboarding ✅
  Inbox ✅
  Mailer ✅                        ← also depends on: background-jobs
  Audit Log ✅
  API Access ✅
  Webhooks ✅
  Feature Flags ✅
  Search ✅
  Import / Export ✅               ← also depends on: background-jobs
  Versioning / History ✅
  Reporting / Analytics ✅         ← also depends on: background-jobs

Wave 3 (depends on Wave 2):
  Permissions ✅ ← Auth, User Management
  Billing ✅ ← Auth, User Management, background-jobs
  Payment ✅ ← Auth, Billing
  Comments ✅ ← Auth, User Management
  Activity Feed ✅ ← Auth, User Management
  Sharing ✅ ← Auth, Permissions
  Presence ✅ ← Auth, User Management, real-time-transport, caching
  In-app Messaging ✅ ← Auth, User Management, real-time-transport

Wave 4 (depends on Wave 3):
  Multi-tenancy ✅ ← Auth, User Management, Permissions
  Impersonation ✅ ← Auth, User Management, Permissions
  Referral ✅ ← Auth, Billing
```

---

## Infrastructure Layer

Infrastructure capabilities have no functional dependencies and should be configured before functional capabilities are declared. They generate middleware, provider integrations, deployment artifacts, and operational scaffolding rather than user-facing features.

### Infrastructure Capability File Format

Infrastructure capabilities follow a modified version of the standard capability format:

```markdown
# {Capability Name}

## Generation Instructions
{Directive prose. Every sentence is an instruction or constraint.
References config keys directly. No human-orientation content.}

## Configuration
\`\`\`yaml
capability: {name}
type: infrastructure
version: 1.0

exposes:
  - {identifier}: {description of what is made available to other capabilities}

config:
  {layered configuration blocks}
\`\`\`
```

Key differences from functional capabilities:
- **`type: infrastructure`** instead of `singleton` or `attachable`
- **`exposes:`** instead of `emits:` — declares runtime clients and utilities made available to other capabilities (e.g., a cache client, a job queue client, a WebSocket hub). Not all infrastructure capabilities expose runtime APIs; deployment-only capabilities (Containerization, CI/CD) omit this block.
- **No `depends-on:`** — infrastructure capabilities have no functional capability dependencies
- **No `attaches-to:`** — infrastructure capabilities are not attachable
- Files live in `capabilities/` alongside functional capabilities — the `type: infrastructure` field is the distinguishing marker, not folder separation

---

### Secrets / Environment Management ⬜

**Type:** Infrastructure
**Depends on:** —

**Deliverables:**
- `capabilities/secrets.md`

**Design guidance:**
- Aggregates all `.env` variables declared across all capabilities into a single `.env.example` file, grouped by capability.
- Generates startup validation: on process start, check that all required variables are present and fail fast with a clear error message if not. Distinguish required (app cannot start without them) from optional (feature degrades gracefully).
- Config: secret provider (env-files, AWS Secrets Manager, Vault, Doppler), validation mode (strict = fail on missing required, warn = log warning, none = skip validation), declared environments (local, staging, production).
- Does not expose a runtime client — validation runs at startup and is transparent thereafter.

**Verification:**
- Required vs. optional variable distinction is implemented
- Startup validation generates a clear, actionable error message
- Provider is configurable without code changes

**Outcome Notes:** _(fill in after completion)_

---

### Migrations ⬜

**Type:** Infrastructure
**Depends on:** —

**Deliverables:**
- `capabilities/migrations.md`

**Design guidance:**
- Schema migration tooling: generates migration files on schema change, applies them in order, tracks applied migrations.
- Strategy: forward-only (default — no down migrations, safer in production) vs. up/down (rollback support, more complex).
- Development seeding: generates a seed script that populates the database with representative development data. Seed is idempotent (safe to run multiple times).
- Production deployment hook: configures whether migrations run automatically on startup or require a manual `migrate` command before deployment. Note the tradeoffs: auto-run is convenient but can fail a deployment mid-start; manual gives control but requires a deployment step.
- Tool is resolved from `stack.database.orm`: Prisma Migrate if ORM is Prisma, Drizzle Kit if Drizzle, raw SQL files with a custom runner otherwise.

**Verification:**
- Forward-only vs. up/down is a config choice with tradeoffs stated
- Seed script is idempotent
- Production run strategy (auto vs. manual) is a config option

**Outcome Notes:** _(fill in after completion)_

---

### Caching ⬜

**Type:** Infrastructure
**Depends on:** —

**Deliverables:**
- `capabilities/caching.md`

**Design guidance:**
- Provides a cache client used by other capabilities for read-through caching, session storage, ephemeral state, and rate limiting counters.
- Provider options: Redis (default for multi-instance deployments), Memcached, in-memory (single-process only — not safe for multi-instance).
- Generates: provider connection setup, cache client wrapper with `get`, `set`, `del`, `invalidate` operations, TTL helpers, key namespacing convention to prevent collisions across capabilities.
- In-memory provider should emit a warning at startup if the application is configured for horizontal scaling (statelessness posture = horizontal).
- Exposes `cache-client` to all other capabilities.

**Verification:**
- In-memory provider incompatibility with horizontal scaling is stated
- Key namespacing convention is specified
- TTL defaults are configurable

**Outcome Notes:** _(fill in after completion)_

---

### Background Jobs ⬜

**Type:** Infrastructure
**Depends on:** —

**Deliverables:**
- `capabilities/background-jobs.md`

**Design guidance:**
- Provides durable async job processing and the runtime implementation of Charter's capability event system (emits/subscribes contracts between capabilities).
- Transport options on a sliding scale of robustness:
  - `in-process`: Node.js EventEmitter / Python signals. Zero infrastructure. Single-process, single-instance only. Not safe for multi-instance or separate worker containers. Generate a startup warning if horizontal scaling is declared.
  - `database`: pg-boss (PostgreSQL) or equivalent. Uses the existing primary database. Durable, multi-container safe, supports advisory locks for mutex semantics. Default when PostgreSQL is the primary database.
  - `redis`: BullMQ or Celery with Redis broker. Uses the existing Redis instance if Caching is present; otherwise adds Redis. Higher throughput than database-backed.
  - `broker`: RabbitMQ, SQS, or NATS. Dedicated infrastructure. Adds routing, topic fanout, consumer groups, dead-letter queues. Use when volume or routing complexity exceeds queue semantics.
- Event delivery model: when `transport` is not `in-process`, Charter's capability emits/subscribes contracts are implemented as durable jobs. A subscribed handler receives the event as a job payload rather than a synchronous function call.
- Worker process: generates a separate worker entry point for transports that benefit from dedicated worker containers (database, redis, broker). The worker runs the same codebase with a different entrypoint — not a separate application.
- Coordination: when `coordination.mutex` is true, at-most-once delivery is enforced for sensitive operations (payments, notification dispatch). Uses advisory locks (database transport) or job deduplication (redis/broker transports).
- Generates: job queue setup, job registration helpers, worker entrypoint, scheduled/cron job declarations, dead-letter handling, optional job monitoring dashboard.
- Exposes `jobs-client` (enqueue, schedule) and `worker-entrypoint`.

**Verification:**
- Transport options are clearly differentiated with tradeoffs stated
- In-process incompatibility with multi-instance is enforced via startup warning
- Event delivery model (how emits/subscribes are wired at runtime) is clearly stated
- Worker entrypoint is distinct from app entrypoint

**Outcome Notes:** _(fill in after completion)_

---

### Real-time Transport ⬜

**Type:** Infrastructure
**Depends on:** —

**Deliverables:**
- `capabilities/real-time-transport.md`

**Design guidance:**
- Provides a WebSocket or SSE server and client utilities used by Presence and Messaging (and any other capability requiring real-time push).
- Declaring this capability as a singleton ensures Presence and Messaging share one transport rather than each generating their own.
- Protocol options: WebSocket (default — bidirectional, lower latency) or SSE (server-to-client only, simpler, works through HTTP/2 proxies).
- Authentication: connections are authenticated against the Auth session on handshake. Unauthenticated connections are rejected.
- Multi-instance scaling: a single WebSocket server cannot broadcast to clients connected to a different instance. When horizontal scaling is declared, generate Redis pub/sub as the coordination layer — each instance publishes to Redis, each instance subscribes and forwards to its own connected clients.
- Generates: transport server setup, authenticated connection handler, channel/room abstraction, client-side connection utility.
- Exposes `realtime-server` (server-side publish API) and `realtime-client-config` (connection URL and auth token delivery for the frontend).

**Verification:**
- Single shared transport for all capabilities is stated
- Multi-instance pub/sub is automatic when horizontal scaling is declared
- Authentication on connection is required, not optional

**Outcome Notes:** _(fill in after completion)_

---

### Observability ⬜

**Type:** Infrastructure
**Depends on:** —

**Deliverables:**
- `capabilities/observability.md`

**Design guidance:**
- Generates structured logging, error tracking, health check endpoints, and optional APM integration. Distinct from the Audit Log functional capability, which is user/admin-facing and compliance-focused. Observability is the operational layer for engineers and systems.
- Logging: structured JSON to stdout (12-factor compliant — log aggregation is the platform's job). Log level configurable via environment variable. Request logging middleware (method, path, status, duration). No sensitive fields (passwords, tokens, PII) in log output — generation instructions must enforce this.
- Error tracking: Sentry integration (default), Rollbar, or none. Captures unhandled exceptions and rejected promises with stack traces and request context.
- Health checks: generate `/health/live` (liveness — is the process running) and `/health/ready` (readiness — is the process ready to serve traffic, i.e., database connected, required services reachable). Readiness endpoint is used by container orchestrators to gate traffic.
- APM: OpenTelemetry (default if enabled), Datadog, New Relic, or none. Traces request lifecycle, database query timing, external HTTP calls.
- Graceful shutdown: generate SIGTERM and SIGINT handlers that stop accepting new connections, drain in-flight requests (configurable timeout), and close database connections cleanly before exiting. Workers drain current job before exiting. This is a generation requirement, not optional config.
- Exposes `logger` (structured log client), `captureError` (manual error capture), `healthRouter` (mountable health check routes).

**Verification:**
- Logging never includes sensitive fields — this must be stated as a generation constraint
- Liveness vs. readiness distinction is implemented correctly
- Graceful shutdown is generated unconditionally, not gated by config
- Observability vs. Audit Log distinction is clearly stated

**Outcome Notes:** _(fill in after completion)_

---

### Containerization ⬜

**Type:** Infrastructure
**Depends on:** —

**Deliverables:**
- `capabilities/containerization.md`

**Design guidance:**
- Generates Dockerfile and docker-compose configuration for development and production environments.
- Dockerfile: multi-stage build (deps → build → runtime). Runtime stage uses a minimal base image. Non-root user for the runtime process. `.dockerignore` to exclude dev dependencies, test files, and secrets.
- Development compose: includes all backing services declared in other infrastructure capabilities — PostgreSQL (always), Redis (if Caching or Background Jobs with redis/broker transport is present), worker service (if Background Jobs declares a separate worker entrypoint). Dev compose mounts source code as a volume for hot reload.
- Production compose (or Kubernetes manifests if `orchestration: kubernetes`): separate services for app and worker. Environment variable injection via `.env` file or secrets provider.
- Dev/prod parity: the development compose uses the same database engine and backing service versions as production. In-memory substitutes (SQLite for PostgreSQL, in-memory cache for Redis) are not generated — they undermine the parity principle.
- Generates `Makefile` or `package.json` scripts for common operations: `dev`, `build`, `migrate`, `seed`, `worker`.
- Does not expose a runtime API — deployment artifact only.

**Verification:**
- Multi-stage build with minimal runtime stage is required
- Dev compose automatically includes backing services declared by other infrastructure capabilities
- Dev/prod parity is stated as a generation constraint — no in-memory substitutes

**Outcome Notes:** _(fill in after completion)_

---

### CI/CD ⬜

**Type:** Infrastructure
**Depends on:** —

**Deliverables:**
- `capabilities/ci-cd.md`

**Design guidance:**
- Generates CI/CD pipeline configuration for automated testing, building, and deployment.
- Provider options: GitHub Actions (default), GitLab CI, CircleCI, none.
- Pipeline stages: lint → type-check → test → build → deploy. Each stage is a separate job that must pass before the next runs. Test stage runs against a real database (spun up as a service in CI) — not in-memory substitutes.
- Environments: staging (auto-deploy on merge to main) and production (manual approval gate). Environment-specific secrets are declared but not populated — the pipeline references them as secrets variables.
- Deploy strategy: push-to-deploy to a container registry + deployment target. Deployment target is configurable (Fly.io, Railway, AWS ECS, Kubernetes, generic SSH). Migration step runs before new containers are started.
- Does not expose a runtime API — CI/CD artifact only.

**Verification:**
- CI test stage uses real backing services, not in-memory substitutes
- Production deployment requires manual approval gate
- Migration step is included in deploy sequence before container rollout

**Outcome Notes:** _(fill in after completion)_

---

## Dependency Map

```
Infrastructure Layer (no functional dependencies — declare before capabilities):
  Secrets / Environment Management
  Migrations
  Caching
  Background Jobs
  Real-time Transport
  Observability
  Containerization
  CI/CD

Wave 1 (no functional dependencies):
  Auth ✅
  Soft Delete + Trash ✅
  Archiving ✅
  Waitlist ✅
  Feedback Collection ✅
  Announcements ✅
  File / Media Management ✅

Wave 2 (depends on Auth):
  User Management ✅
  Onboarding ✅
  Inbox ✅
  Mailer ✅                        ← also depends on: background-jobs
  Audit Log ✅
  API Access ✅
  Webhooks ✅
  Feature Flags ✅
  Search ✅
  Import / Export ✅               ← also depends on: background-jobs
  Versioning / History ✅
  Reporting / Analytics ✅         ← also depends on: background-jobs

Wave 3 (depends on Wave 2):
  Permissions ✅ ← Auth, User Management
  Billing ✅ ← Auth, User Management, background-jobs
  Payment ✅ ← Auth, Billing
  Comments ✅ ← Auth, User Management
  Activity Feed ✅ ← Auth, User Management
  Sharing ✅ ← Auth, Permissions
  Presence ✅ ← Auth, User Management, real-time-transport, caching
  In-app Messaging ✅ ← Auth, User Management, real-time-transport

Wave 4 (depends on Wave 3):
  Multi-tenancy ✅ ← Auth, User Management, Permissions
  Impersonation ✅ ← Auth, User Management, Permissions
  Referral ✅ ← Auth, Billing
```

---

*Document version: 1.1.0*
*Reference capability: capabilities/auth.md*
*Reference infrastructure capability: capabilities/background-jobs.md*
*Total functional capabilities: 28 (28 complete, 0 remaining)*
*Total infrastructure capabilities: 8 (0 complete, 8 remaining)*
