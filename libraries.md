# Charter Libraries Registry

Maps Charter capabilities to recommended libraries per stack family. Used by charter-init to select what to install rather than generate from scratch.

**How charter-init uses this file:**
1. For each selected capability, look up this file by capability name
2. Match the stack family to the project's declared stack
3. Install `recommended` (or `hosted-alternative` if appropriate)
4. Generation instructions configure the library — they do not reimplement it

**`when:` fields** reference config keys from the capability's project-brief.yaml configuration.

*Last updated: 2026-03-10*

---

## auth

> Auth establishes user identity; a library dramatically reduces the surface area Charter must generate for registration, sessions, password reset, OAuth, and MFA flows.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `better-auth` | Full-featured, TypeScript-first; covers email/password, OAuth, sessions, MFA, organizations |
| alternative | `passport` + strategy plugins | when: `config.session.mechanism` is `server-side` and only one provider needed; lower-level, requires more Charter generation |
| hosted-alternative | `Clerk` | when: team prefers zero-auth-infrastructure; hosted UI components, strong Next.js/React integration |
| hosted-alternative | `Auth0` | when: enterprise SSO (`config.providers.saml: true`) is the primary driver |

**What the library covers:**
- Email/password registration, login, logout, password reset
- OAuth provider flows (Google, GitHub, and others) via adapter plugins
- Session management (JWT or server-side) with configurable duration and sliding expiry
- MFA (TOTP) when `config.security.mfa.enforcement` is not `off`
- Email verification on registration when `config.registration.email-verification: true`
- Rate limiting on auth endpoints when `config.security.rate-limiting: true`

**What Charter generates on top:**
- Login, registration, password-reset, and MFA UI pages and forms
- Admin surfaces declared in `config.admin-surface` (user search, suspend, force-reset, session views)
- SAML SSO integration when `config.providers.saml: true` (requires enterprise better-auth plugin or Auth0)
- Invite-only registration gate when `config.registration.mode: invite-only`

### nextjs

Same as node-typescript. `better-auth` has first-class Next.js App Router support and is the recommended choice. `Clerk` is the most common hosted alternative specifically for Next.js.

### remix

Same as node-typescript. `better-auth` integrates with Remix via the standard fetch-based adapter. Epic Stack uses a hand-rolled auth layer on top of Remix — if starting from Epic Stack, Charter extends the existing auth implementation rather than installing better-auth.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `django-allauth` | Covers email/password, OAuth, email verification, 2FA via plugin |
| recommended (JWT) | `djangorestframework-simplejwt` | when: `config.session.mechanism: jwt`; pairs with django-allauth for token issuance |
| alternative | `dj-rest-auth` | Thin REST endpoints layer on top of django-allauth; include when building API-only Django |
| addon | `django-two-factor-auth` | when: `config.security.mfa.enforcement` is not `off`; TOTP support on top of django-allauth |
| hosted-alternative | `Clerk` | when: team prefers zero-auth-infrastructure; Python SDK available |

**What the library covers:**
- Email/password registration, login, logout, password reset (django-allauth)
- OAuth provider flows (Google, GitHub, and others) via allauth social adapters
- Email verification on registration
- TOTP-based MFA via django-two-factor-auth
- JWT token issuance and refresh via simplejwt when mechanism is jwt

**What Charter generates on top:**
- REST API auth endpoints (login, logout, register, refresh) via dj-rest-auth or custom views
- Admin surfaces: user search, suspend (status field), force-password-reset action
- SAML SSO integration when `config.providers.saml: true` (requires python3-saml or similar)
- Invite-only registration middleware when `config.registration.mode: invite-only`

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `fastapi-users` | Purpose-built for FastAPI; covers user model, registration, JWT sessions, OAuth, password reset |
| alternative | `authlib` | when: OAuth2/OIDC server (not just client) is required; lower-level |
| hosted-alternative | `Clerk` | when: team prefers zero-auth-infrastructure |

**What the library covers:**
- User model and database integration (SQLAlchemy or Beanie)
- Registration, login, logout, password reset flows
- JWT-based session management
- OAuth2 social login via fastapi-users oauth router
- Email verification on registration

**What Charter generates on top:**
- MFA (TOTP) — fastapi-users does not include MFA; Charter generates TOTP enrollment and verification routes
- Admin surfaces for user search, suspension, session management
- SAML SSO integration when `config.providers.saml: true`
- Invite-only registration gate when `config.registration.mode: invite-only`
- Rate limiting middleware on auth endpoints

---

## billing

> Billing manages subscription plan state, trial lifecycle, and feature gating; no single library owns this fully — Charter generates the state machine but should reuse plan/feature abstractions where they exist.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `stripe` (npm) | Stripe's SDK covers plan/product/price metadata; Charter maps `config.plans` to Stripe Products |
| alternative | none | Billing state machine (trial, active, past-due, cancelled) is Charter-generated regardless of provider |
| hosted-alternative | `Stripe Billing` | The hosted billing portal handles upgrade/downgrade UI when `config.plan-change-timing: end-of-period` |

**What the library covers:**
- Stripe Product and Price object management (plan definitions)
- Customer and Subscription object lifecycle
- Stripe Billing Portal (hosted) for plan changes when delegating UI to Stripe

**What Charter generates on top:**
- Local `Plan` and `Subscription` database models tracking `config.subscription-entity` (user or org)
- Trial state machine: start trial, detect expiry via scheduled job, display countdown
- Feature gate helpers `onPlan(user, planName)` and `hasFeature(user, featureName)`
- Plan comparison table UI and upgrade/downgrade flows
- Admin surfaces: subscription viewer, override-plan, trial-extension
- `payment.failed` → `past-due` and `payment.succeeded` → `active` state transitions

### nextjs

Same as node-typescript. Stripe's hosted Billing Portal integrates cleanly with Next.js App Router via server actions.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `stripe` (PyPI) | Stripe's Python SDK for plan/subscription management |
| alternative | none | Local billing state machine is Charter-generated regardless |
| hosted-alternative | `Stripe Billing` | Hosted portal for plan self-management |

**What the library covers:** Same Stripe API surface as node-typescript entry.

**What Charter generates on top:** Same as node-typescript but as Django models, views, and management commands.

### python-fastapi

Same as python-django. Stripe Python SDK is framework-agnostic.

---

## payment

> Payment handles money movement via provider SDK; Stripe is the only currently supported provider so library selection is deterministic.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `stripe` (npm) | Official Node SDK; covers Stripe Elements, PaymentIntents, webhooks, refunds |

**What the library covers:**
- Stripe Elements (payment method collection) — never touches raw card data
- PaymentIntent creation and confirmation
- Webhook event verification (`stripe.webhooks.constructEvent`)
- Customer and PaymentMethod management (list, attach, detach, set default)
- Refund initiation
- Invoice and charge retrieval for history view

**What Charter generates on top:**
- Payment method management UI (list, add, remove, set default)
- Invoice history view (when `config.invoice-history: user-facing`)
- Refund initiation UI (when `config.refunds: self-serve`) or admin-only refund surface
- Webhook handler registering for: `payment_intent.succeeded`, `payment_intent.payment_failed`, `invoice.payment_succeeded`, `invoice.payment_failed`, `charge.refunded`
- Admin surfaces: payment history, issue-refund, manage-subscriptions

### nextjs

Same as node-typescript. Stripe's `@stripe/stripe-js` is used for client-side Elements; `stripe` npm package for server-side API calls. Both work with App Router server actions.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `stripe` (PyPI) + `dj-stripe` | `dj-stripe` syncs Stripe objects to local Django models; reduces webhook boilerplate |
| alternative | `stripe` (PyPI) only | when: `dj-stripe`'s model opinions conflict with the app's data model |

**What the library covers (dj-stripe):**
- Automatic sync of Stripe Customer, Subscription, Invoice, Charge, PaymentMethod objects to Django models
- Webhook handler registration and event routing
- Django admin integration for Stripe objects

**What Charter generates on top:**
- Payment method management UI
- Invoice history and refund surfaces per `config.invoice-history` and `config.refunds`
- Custom webhook handler for any events dj-stripe doesn't route by default

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `stripe` (PyPI) | dj-stripe is Django-specific; use raw Stripe SDK for FastAPI |

**What the library covers:** Same Stripe Python SDK surface as node-typescript entry.

**What Charter generates on top:** Full webhook handler, payment method management API routes and UI, invoice and refund surfaces. More generation required than python-django since dj-stripe is not available.

---

## mailer

> Mailer sends transactional email; a provider SDK plus a template library eliminates SMTP plumbing and gives Charter a clean surface to generate templates on top of.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `resend` + `react-email` | Resend SDK for delivery; react-email for JSX-based templates; clean DX, modern API |
| alternative | `nodemailer` | when: `config.provider: smtp`; direct SMTP, requires transport configuration |
| alternative | `@sendgrid/mail` | when: `config.provider: sendgrid` |
| alternative | `postmark` | when: `config.provider: postmark` |
| alternative | `@aws-sdk/client-ses` | when: `config.provider: ses` |

**What the library covers:**
- Provider-authenticated email delivery
- react-email: JSX components compiled to email-safe HTML with inline CSS
- Bounce and delivery webhook parsing (provider-specific)

**What Charter generates on top:**
- Email template component for each type in `config.email-types`
- Delivery job handler (enqueued via background-jobs, not synchronous)
- Bounce tracking: mark user email undeliverable on hard bounce
- Unsubscribe link generation and unsubscribe landing page
- Email preference management page when `config.preference-center: true`
- Opt-out state check before delivery for types where `opt-outable: true`

### nextjs

Same as node-typescript. react-email components render well in Next.js. Resend has first-class Next.js documentation.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `django-anymail` | Unified backend for SendGrid, Postmark, Resend, SES, Mailgun, and others; single API regardless of provider |

**What the library covers:**
- Swappable email backend: change `EMAIL_BACKEND` setting to switch providers without code changes
- Provider-specific extras (tags, metadata, tracking) via AnymailMessage
- Webhook receiver views for bounce and delivery events (provider-specific)

**What Charter generates on top:**
- Django email templates (HTML + plain text) for each type in `config.email-types`
- Async delivery via Celery task (not synchronous in request cycle)
- Bounce tracking signal handler
- Unsubscribe link and landing page
- Email preference model and management page when `config.preference-center: true`

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `resend` (PyPI) | Clean Python SDK; works with any async framework |
| alternative | `sendgrid` (PyPI) | when: `config.provider: sendgrid` |
| alternative | `postmarker` | when: `config.provider: postmark` |
| alternative | `boto3` (`ses` client) | when: `config.provider: ses` |

**What the library covers:** Provider-authenticated email delivery via HTTP API.

**What Charter generates on top:** Same as python-django but as async FastAPI background tasks / ARQ jobs, Jinja2 HTML templates, and FastAPI route handlers for unsubscribe and preference management.

---

## background-jobs

> Background jobs require a transport library; Charter's capability spec explicitly maps transport config values to specific libraries, so selection is deterministic.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended (redis) | `bullmq` | when: `config.transport: redis`; production-grade, Redis-backed, widely deployed |
| recommended (database) | `pg-boss` | when: `config.transport: database`; PostgreSQL-backed, no extra infrastructure |
| alternative | `inngest` | when: team prefers managed job infrastructure; hosted service, no self-hosted broker |
| in-process | Node.js `EventEmitter` | when: `config.transport: in-process`; generated directly, no library needed |

**What the library covers (bullmq):**
- Queue creation, job enqueue with delay/priority/repeat options
- Worker process with concurrency control, retry with exponential backoff
- Dead-letter queue (failed jobs) with configurable max attempts
- Job deduplication via job ID when `config.coordination.mutex: true`
- Scheduled/repeatable jobs via cron expressions
- BullMQ Board or Bull Board for admin dashboard when `config.admin-dashboard: true`

**What the library covers (pg-boss):**
- Job storage in PostgreSQL `pgboss` schema (no extra infrastructure)
- Queue, schedule, worker, retry — same conceptual surface as BullMQ
- Advisory lock-based deduplication for mutex jobs

**What Charter generates on top:**
- `lib/jobs` module exposing `enqueue(jobName, payload, options?)` and `schedule(cronExpression, jobName, payload)`
- Worker entrypoint at `worker.ts` registering all capability job handlers
- Dead-letter viewer at `/admin/jobs` when `config.admin-dashboard: true`
- Startup warning/error when `config.transport: in-process` and horizontal scaling is enabled

### nextjs

Same as node-typescript. Note: Next.js serverless/edge deployments are incompatible with BullMQ workers (long-running process required). Use `pg-boss` with a separate worker process, or use `inngest` (serverless-compatible managed alternative).

### remix

Same as node-typescript. Remix on traditional Node.js server supports BullMQ and pg-boss. If deploying to Cloudflare Workers/edge, use `inngest`.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `celery` + `django-celery-beat` | Industry standard; `celery-beat` for cron scheduling; pairs with Redis or RabbitMQ broker |
| alternative | `dramatiq` + `django-dramatiq` | when: team wants a simpler, more modern alternative to Celery; lighter configuration |
| alternative | `django-rq` | when: `config.transport: redis` and team wants minimal setup; simpler than Celery, less feature-rich |

**What the library covers (celery):**
- Task definition, enqueue, retry with exponential backoff
- Cron scheduling via django-celery-beat
- Multiple broker backends (Redis, RabbitMQ, SQS)
- Flower for admin job monitoring dashboard when `config.admin-dashboard: true`

**What Charter generates on top:**
- `lib/jobs.py` module with `enqueue(task_name, payload, options)` and `schedule(cron, task_name, payload)`
- Worker entrypoint and Celery app configuration
- All capability-specific task functions (mailer delivery, billing trial expiry, etc.)
- Flower configuration when `config.admin-dashboard: true`

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `arq` | when: `config.transport: redis`; async-native, designed for asyncio frameworks like FastAPI |
| alternative | `celery` | when: team already uses Celery across services; sync-first but can work with FastAPI via sync task wrappers |
| alternative | `dramatiq` | when: team prefers Dramatiq's reliability model; middleware-based |

**What the library covers (arq):**
- Async task enqueue, worker, retry, cron scheduling
- Redis-backed job storage
- Designed for asyncio — no sync/async impedance mismatch with FastAPI route handlers

**What Charter generates on top:** Same as python-django pattern but as async functions and ARQ worker configuration.

---

## caching

> Caching requires a client library for the configured provider; the capability spec maps provider values directly to specific libraries.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended (redis) | `ioredis` | when: `config.provider: redis` and BullMQ is also present (BullMQ requires ioredis); share the connection |
| alternative (redis) | `node-redis` | when: `config.provider: redis` and BullMQ is not present; officially maintained by Redis |
| in-memory | `lru-cache` | when: `config.provider: in-memory`; zero infrastructure, single-instance only |
| memcached | `memjs` | when: `config.provider: memcached` |

**What the library covers:**
- Redis connection with pooling, automatic reconnection, Sentinel/Cluster support (ioredis)
- get/set/del/exists/expire operations
- Key pattern scanning for invalidation (Redis SCAN)
- Pub/Sub support (required by real-time-transport when horizontal scaling is enabled)

**What Charter generates on top:**
- `lib/cache` module exposing `get`, `set`, `del`, `exists`, `invalidate(pattern)`
- `cacheKey(capability, ...parts)` helper enforcing namespace conventions
- `DEFAULT_TTL_SECONDS` constant and per-capability TTL declarations
- Startup error when `config.provider: in-memory` and horizontal scaling is enabled

### nextjs

Same as node-typescript. Note: Next.js has its own built-in `unstable_cache` and `revalidateTag` for data layer caching — Charter's `lib/cache` is for application-layer caching (sessions, rate limiting, presence), not Next.js data caching. Both can coexist.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended (redis) | `django-redis` | when: `config.provider: redis`; integrates with Django's cache framework; uses redis-py under the hood |
| in-memory | Django's `LocMemCache` | when: `config.provider: in-memory`; built-in, no library needed |
| memcached | `pymemcache` | when: `config.provider: memcached`; Django ships a Memcached backend using this |

**What the library covers:**
- Django cache backend API (cache.get, cache.set, cache.delete, cache.get_many)
- Redis connection management via redis-py
- Cache versioning and key prefix support

**What Charter generates on top:**
- `lib/cache.py` wrapper with `cacheKey(capability, *parts)` helper
- Per-capability TTL declarations and invalidation signal handlers

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended (redis) | `redis-py` (async) | when: `config.provider: redis`; use `redis.asyncio` client for async FastAPI compatibility |
| alternative | `aiocache` | when: team wants a unified async cache interface supporting multiple backends |
| in-memory | `cachetools` | when: `config.provider: in-memory`; LRU/TTL cache primitives for async use |

**What Charter generates on top:** Same as python-django pattern but as async Python functions using `redis.asyncio`.

---

## search

> Search requires a client library for the configured external provider; the database provider uses native SQL full-text features and requires no extra library.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| database | none | when: `config.provider: database`; uses PostgreSQL `tsvector`/`tsquery` via the project's existing ORM |
| recommended (external) | `typesense-js` | when: `config.provider: typesense`; strongly typed, fast, self-hostable |
| alternative | `meilisearch` (npm) | when: `config.provider: meilisearch`; Rust-based, excellent DX, self-hostable |
| alternative | `@elastic/elasticsearch` | when: `config.provider: elasticsearch`; required for large-scale or log-heavy workloads |
| hosted-alternative | `Algolia` | when: team prefers fully managed search-as-a-service |

**What the library covers:**
- Index schema definition and management
- Document index/update/delete operations
- Search query execution with faceting and filtering
- Typo tolerance (Typesense, Meilisearch out of the box)

**What Charter generates on top:**
- Index configuration for each entity in `config.attaches-to`
- Entity create/update/delete hooks to keep index current
- Search API endpoint with query and filter parameters
- Search UI: input, results list, faceted filter controls
- Permission-scoped result filtering when `config.permission-scoped: true`
- `search.performed` event emission when `config.emit-analytics: true`

### nextjs

Same as node-typescript.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| database | `django.contrib.postgres` | when: `config.provider: database`; `SearchVector`, `SearchQuery`, `SearchRank` built into Django |
| recommended (external) | `typesense-python` | when: `config.provider: typesense` |
| alternative | `meilisearch-python` | when: `config.provider: meilisearch` |
| alternative | `elasticsearch-dsl` | when: `config.provider: elasticsearch`; higher-level DSL over the raw client |
| alternative | `django-elasticsearch-dsl` | when: `config.provider: elasticsearch` and tight Django model integration is desired |

**What Charter generates on top:** Same as node-typescript pattern but as Django views, serializers, and model signal handlers.

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| database | `sqlalchemy` full-text | when: `config.provider: database`; PostgreSQL `to_tsvector`/`to_tsquery` via SQLAlchemy text expressions |
| recommended (external) | `typesense-python` | when: `config.provider: typesense` |
| alternative | `meilisearch-python` | when: `config.provider: meilisearch` |
| alternative | `elasticsearch-py` | when: `config.provider: elasticsearch` |

**What Charter generates on top:** Same as python-django pattern but as FastAPI route handlers and SQLAlchemy event listeners.

---

## file-management

> File management requires a storage client and optionally an image processing library; Charter generates upload flow, metadata model, and admin surface on top.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `@aws-sdk/client-s3` + `multer` | when: `config.storage-provider: s3`; AWS SDK v3 modular client; multer handles multipart upload parsing |
| alternative (GCS) | `@google-cloud/storage` | when: `config.storage-provider: gcs` |
| alternative (Azure) | `@azure/storage-blob` | when: `config.storage-provider: azure` |
| local | `multer` (disk storage) | when: `config.storage-provider: local`; development/single-server only |
| image processing | `sharp` | when: `config.image-processing.enabled: true`; high-performance thumbnail generation |

**What the library covers:**
- Multipart upload parsing (multer)
- Authenticated upload to cloud storage bucket with generated key
- Presigned URL generation for direct client-side uploads (large files)
- Image resize and format conversion (sharp)

**What Charter generates on top:**
- Upload API endpoint with client-side type/size validation against `config.accepted-types` and `config.max-file-size-mb`
- File metadata model (id, original filename, mime type, size, storage key, uploader userId, entity reference)
- Drag-and-drop upload UI with progress indication
- Thumbnail generation on upload when `config.image-processing.enabled: true`
- Download and inline preview surfaces
- Deletion (soft or hard depending on soft-delete capability presence)
- Admin surfaces: file library, bulk-delete

### nextjs

Same as node-typescript. Next.js App Router server actions can handle upload; use presigned URLs for large files to avoid routing through the Next.js server.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `django-storages` + `boto3` | when: `config.storage-provider: s3`; django-storages provides the Django storage backend; boto3 is the AWS client |
| alternative (GCS) | `django-storages` (GCS backend) | when: `config.storage-provider: gcs` |
| alternative (Azure) | `django-storages` (Azure backend) | when: `config.storage-provider: azure` |
| image processing | `Pillow` | when: `config.image-processing.enabled: true`; standard Python imaging library |

**What the library covers:**
- Django `DEFAULT_FILE_STORAGE` backend routing uploads to S3/GCS/Azure
- `boto3` for direct S3 operations (presigned URLs, delete)
- `Pillow` for thumbnail generation (note: CPU-bound; run in Celery task, not request cycle)

**What Charter generates on top:** Same surfaces as node-typescript entry but as Django models, views, serializers, and admin classes.

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `boto3` + `python-multipart` | when: `config.storage-provider: s3`; `python-multipart` for FastAPI `UploadFile` support; boto3 for S3 |
| alternative (GCS) | `google-cloud-storage` | when: `config.storage-provider: gcs` |
| image processing | `Pillow` | when: `config.image-processing.enabled: true` |

**What Charter generates on top:** Same surfaces as python-django entry but as async FastAPI route handlers and SQLAlchemy models.

---

## feature-flags

> Feature flags benefit from a self-hosted management backend and SDK so flag state can be updated without deployments; Charter generates the evaluation logic and admin UI on top.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `@openfeature/server-sdk` + in-database provider | OpenFeature is the CNCF standard API; Charter generates a database-backed provider storing flags in the app's own DB |
| alternative | `unleash-client` | when: team runs a self-hosted Unleash server; Unleash is fully compatible with OpenFeature |
| alternative | `flagsmith` (npm) | when: team runs a self-hosted Flagsmith server |
| hosted-alternative | `LaunchDarkly` | when: team prefers managed feature flag service with advanced targeting |
| hosted-alternative | `PostHog` | when: team already uses PostHog for analytics and wants feature flags in the same platform |

**What the library covers (OpenFeature + database provider):**
- Standardized `isEnabled(flagKey, user)` API
- Provider interface allowing backend swap (database → Unleash → LaunchDarkly) without application code changes
- Context propagation (userId, plan, role) for targeting evaluation

**What Charter generates on top:**
- Database-backed OpenFeature provider storing flag definitions and targeting rules
- Flag evaluation for all strategies in `config.targeting-strategies`: boolean, user-list, percentage-rollout (deterministic hash), attribute-rule
- Client-side flag state serialized into session payload when `config.evaluation-mode` includes `client-side`
- Flag management admin UI: create, enable/disable, configure targeting, debug evaluation for a specific user
- `feature-flags.flag-evaluated` event emission when `config.emit-evaluations: true`

### nextjs

Same as node-typescript. Next.js App Router Server Components can evaluate flags server-side; Charter serializes enabled flags into the initial page payload for client components.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `django-flags` | Lightweight flag evaluation with database or settings storage; clean Django integration |
| alternative | `openfeature-sdk` (PyPI) + database provider | when: team wants OpenFeature standard API for portability |
| hosted-alternative | `LaunchDarkly` Python SDK | when: managed service preferred |

**What the library covers (django-flags):**
- `flag_enabled(flag_name, request)` evaluation helper
- Database and settings-based flag storage
- Django admin integration for flag management

**What Charter generates on top:**
- Extended targeting strategies (percentage-rollout, attribute-rule) beyond django-flags defaults
- Client-side flag state in API responses when `config.evaluation-mode` includes `client-side`
- Full admin UI for flag management and user-level debug evaluation
- `feature-flags.flag-evaluated` event emission when `config.emit-evaluations: true`

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `openfeature-sdk` (PyPI) + database provider | OpenFeature Python SDK with Charter-generated SQLAlchemy-backed provider |
| hosted-alternative | `LaunchDarkly` Python SDK | when: managed service preferred |

**What Charter generates on top:** Same as python-django but as FastAPI dependency-injected evaluation helpers and SQLAlchemy flag models.

---

## permissions

> Permissions implements RBAC; Django and DRF have strong built-in permission primitives — Charter layers Charter-specific role definitions on top of what each stack already provides.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `casl` | Isomorphic authorization library; define role/action rules once, evaluate on server and client |
| alternative | none (hand-rolled) | when: permission model is simple (2–3 roles, no resource-scoping); Charter generates `can(user, action)` directly |

**What the library covers:**
- Ability definition: map roles to permitted actions
- `ability.can(action, subject)` evaluation
- Condition-based rules for resource-scoped access when `config.resource-scoped: true`
- CASL React helpers for UI conditional rendering

**What Charter generates on top:**
- Role definitions from `config.roles` as CASL ability rules
- Role assignment model and migration
- `can(user, action)` and `canAccess(user, resource)` helper wrappers
- Role assignment middleware for route protection
- Admin surfaces: role-list, assign-role, view-user-roles
- `auth.registered` subscription to assign `config.default-role` to new users

### nextjs

Same as node-typescript. CASL's isomorphic design works in both Server Components and client components.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | Django built-in permissions + DRF permission classes | `django.contrib.auth` groups and permissions; DRF `IsAuthenticated`, `DjangoModelPermissions` |
| addon | `django-guardian` | when: `config.resource-scoped: true`; adds per-object permission checks |
| alternative | `django-rules` | when: team prefers predicate-based permission rules over Django's model-permission system |

**What the library covers (built-in + guardian):**
- Group-based role assignment via `django.contrib.auth.Group`
- DRF permission classes enforcing role checks on API views
- Per-object permissions via django-guardian when resource-scoped

**What Charter generates on top:**
- Django Group and Permission objects from `config.roles`
- `can(user, action)` helper wrapping DRF/Guardian checks
- Role assignment on `auth.registered` signal
- Admin surfaces via Django admin + DRF browsable API

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `casbin` + `pycasbin` | Flexible RBAC/ABAC policy engine; policy stored in database |
| alternative | none (hand-rolled) | when: permission model is simple (2–3 roles); Charter generates FastAPI dependency `require_role(roles)` directly |

**What Charter generates on top:**
- Casbin policy model from `config.roles`
- FastAPI `Depends(require_permission(action))` middleware wrappers
- Role assignment on registration
- `canAccess(user, resource)` resource-scoped check when `config.resource-scoped: true`

---

## api-access

> API access requires middleware for key validation and rate limiting; Charter generates key management UI and scoping logic on top.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `express-rate-limit` | when: `config.rate-limiting-per-key` is non-null; middleware for per-key request throttling |
| alternative (Redis-backed) | `rate-limiter-flexible` | when: horizontal scaling is enabled; distributed rate limiting with Redis backend |

**What the library covers:**
- Per-key request counting and limit enforcement
- 429 response with `Retry-After` header on limit exceeded
- Redis-backed counters shared across instances (rate-limiter-flexible)

**What Charter generates on top:**
- API key creation, storage (hashed), copy-once UI
- Key rotation (invalidate old, issue new) and revocation
- `Authorization: Bearer` header extraction and validation middleware
- OAuth2 client credentials flow when `oauth2` is in `config.mechanism`
- Scope enforcement middleware when `config.scopes` is non-empty
- Scope selection UI in key creation flow
- Developer portal with endpoint documentation when `config.developer-portal: true`
- `api-access.key-created`, `api-access.key-revoked`, `api-access.rate-limit-exceeded` event emission

### nextjs

Same as node-typescript. Rate limiting middleware applies to Next.js Route Handlers via middleware.ts.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `djangorestframework` throttling | DRF's built-in `ScopedRateThrottle` for per-key rate limiting |
| alternative | `django-ratelimit` | when: more flexible throttling rules are needed beyond DRF defaults |

**What the library covers:**
- DRF throttle classes: `AnonRateThrottle`, `UserRateThrottle`, `ScopedRateThrottle`
- Cache-backed (Redis) request counting
- 429 response generation

**What Charter generates on top:** Same as node-typescript surfaces but as DRF views, serializers, and middleware.

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `slowapi` | FastAPI-native rate limiting (port of Flask-Limiter); Redis-backed when horizontal scaling is enabled |

**What the library covers:**
- Per-route and per-key rate limiting via `@limiter.limit()` decorator
- Redis backend for distributed rate limiting
- 429 response with standard headers

**What Charter generates on top:** Same as python-django surfaces but as FastAPI route handlers and SQLAlchemy models.

---

## webhooks

> Webhooks require reliable async delivery with retry logic; Charter generates registration UI, signing, and delivery log on top of the job infrastructure.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `svix` | Managed webhook delivery service; handles retries, signatures, delivery log, and endpoint management UI |
| alternative | none (Charter-generated) | when: team requires fully self-hosted; Charter generates delivery via background-jobs with HMAC-SHA256 signing |
| signature verification | `standardwebhooks` | when: self-hosted; standardized HMAC signature verification for incoming webhook validation |

**What the library covers (svix):**
- Endpoint registration and management
- Delivery with exponential backoff retries up to configured attempts
- HMAC-SHA256 signing on all deliveries
- Per-endpoint delivery log with response status and body
- Webhook testing tool (send sample payload on demand)
- Automatic endpoint disable after consecutive failures

**What Charter generates on top (svix path):**
- Endpoint registration UI (URL, event type selection, secret display)
- Svix Application and Endpoint creation wired to Charter's subscribable event system
- Event fanout: on each emitted event, call `svix.message.create()` to fan out to all registered endpoints
- Svix Portal embed for user-facing delivery log and testing UI

**What Charter generates on top (self-hosted path):**
- Everything from the svix path but hand-generated: delivery job, signing, log model, retry logic, testing UI, auto-disable logic

### nextjs

Same as node-typescript.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `svix` (PyPI) | Same managed delivery service; Python SDK available |
| alternative | none (Charter-generated) | when: self-hosted required; delivery via Celery tasks with HMAC signing |

**What Charter generates on top:** Same as node-typescript but as Django models, views, and Celery delivery tasks.

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `svix` (PyPI) | Same managed delivery service |
| alternative | none (Charter-generated) | when: self-hosted; delivery via ARQ/Celery async tasks |

**What Charter generates on top:** Same as python-django but as FastAPI route handlers and async delivery tasks.

---

## observability

> Observability requires a structured logging library, optionally an error tracking SDK, and optionally an APM/tracing SDK; Charter generates the unified logger module and health endpoints on top.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended (logging) | `pino` | Fastest structured JSON logger for Node.js; request-id injection via `pino-http` |
| recommended (error tracking) | `@sentry/node` | when: `config.error-tracking: sentry`; captures unhandled exceptions and manual captureError calls |
| alternative (error tracking) | `rollbar` | when: `config.error-tracking: rollbar` |
| recommended (APM) | `@opentelemetry/sdk-node` | when: `config.apm: opentelemetry`; vendor-neutral; exporters for Datadog, New Relic, Jaeger |
| alternative (APM) | `dd-trace` | when: `config.apm: datadog`; Datadog-native APM agent |

**What the library covers:**
- Structured JSON log output to stdout (pino)
- Request logging middleware with method, path, status, response time, request ID (pino-http)
- Automatic unhandled exception capture (Sentry)
- Distributed tracing at I/O boundaries — HTTP, DB, Redis (OpenTelemetry)

**What Charter generates on top:**
- `lib/logger` module with automatic sensitive field redaction (passwords, tokens, API keys, credentials)
- `captureError(error, context?)` export routing to configured provider or log-level fallback
- `lib/health` router with `/health/live` and `/health/ready` endpoints (readiness checks database, Redis)
- SIGTERM/SIGINT graceful shutdown handlers with configurable timeout

### nextjs

Same as node-typescript. Sentry has a dedicated `@sentry/nextjs` package with App Router support and automatic error boundary integration.

### remix

Same as node-typescript. Sentry has a dedicated `@sentry/remix` package.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended (logging) | `structlog` + `django-structlog` | `structlog` for structured JSON output; `django-structlog` wires request context automatically |
| recommended (error tracking) | `sentry-sdk` (Django integration) | when: `config.error-tracking: sentry`; `sentry_sdk.init()` with `DjangoIntegration()` |
| recommended (APM) | `opentelemetry-sdk` + `opentelemetry-instrumentation-django` | when: `config.apm: opentelemetry` |

**What the library covers:**
- Structured JSON request/response logging with automatic request ID propagation (django-structlog)
- Unhandled exception capture and user context attachment (Sentry Django integration)
- Django ORM query duration tracing (OpenTelemetry Django instrumentation)

**What Charter generates on top:** Same as node-typescript — `lib/logger.py`, `captureError`, health check views at `/health/live` and `/health/ready`, SIGTERM handler in WSGI/ASGI entrypoint.

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended (logging) | `structlog` | when: structured logging; configure in FastAPI lifespan; use middleware for request context |
| recommended (error tracking) | `sentry-sdk` (FastAPI integration) | when: `config.error-tracking: sentry`; `sentry_sdk.init()` with `FastApiIntegration()` |
| recommended (APM) | `opentelemetry-sdk` + `opentelemetry-instrumentation-fastapi` | when: `config.apm: opentelemetry` |

**What Charter generates on top:** Same as python-django but mounted as FastAPI middleware and lifespan handlers.

---

## migrations

> Migrations are tightly coupled to the ORM; library selection follows directly from `stack.database.orm` per the capability spec.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `prisma migrate` | when: ORM is Prisma; `prisma migrate dev` and `prisma migrate deploy` |
| alternative | `drizzle-kit` | when: ORM is Drizzle; `drizzle-kit generate` and `drizzle-kit migrate` |
| alternative | `typeorm` migrations | when: ORM is TypeORM; `typeorm migration:generate` and `typeorm migration:run` |
| fallback | Charter-generated runner | when: no ORM (raw SQL); numbered SQL files in `db/migrations/`, tracked in `_migrations` table |

**What the library covers:**
- Schema diffing and migration file generation
- Ordered migration application and tracking
- Rollback support (Prisma: not supported in forward-only mode; Drizzle/TypeORM: up-down when `config.strategy: up-down`)

**What Charter generates on top:**
- `package.json` scripts: `db:migrate`, `db:migrate:create`, `db:seed`, `db:reset`
- Seed script at `db/seed.ts` when `config.seed: true` (idempotent, with existence checks)
- Startup migration run when `config.run: on-startup` (with 60-second timeout guard)
- Deploy runbook note for manual migration when `config.run: manual`

### nextjs

Same as node-typescript.

### remix

Same as node-typescript. Epic Stack uses Prisma — Charter extends the existing Prisma setup rather than installing a new migration tool.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | Django Migrations (built-in) | `makemigrations` + `migrate`; no additional library required |

**What the library covers:**
- Schema diffing from model changes
- Ordered migration application with dependency tracking
- Forward and reverse (down) migrations when `config.strategy: up-down`

**What Charter generates on top:**
- Management command aliases: `db:migrate`, `db:migrate:create`, `db:seed`, `db:reset`
- Seed script (Django management command) when `config.seed: true`
- Startup migration execution in `wsgi.py`/`asgi.py` when `config.run: on-startup`

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `alembic` | Standard SQLAlchemy migration tool; `alembic revision --autogenerate` and `alembic upgrade head` |

**What the library covers:**
- SQLAlchemy model diffing and migration script generation
- Ordered migration application with history tracking
- Up and down migrations when `config.strategy: up-down`

**What Charter generates on top:** Same management commands and seed script as python-django, but implemented as Python scripts and Alembic environment configuration.

---

## real-time-transport

> Real-time transport requires a WebSocket or SSE server library; Charter generates the channel abstraction, auth enforcement, and client utility on top.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended (websocket) | `ws` | when: `config.protocol: websocket`; minimal, production-proven, attaches to existing HTTP server |
| alternative (websocket) | `socket.io` | when: room/namespace abstractions are explicitly needed; heavier than `ws` |
| sse | none (built-in) | when: `config.protocol: sse`; standard HTTP response streaming, no library required |
| scaling | `ioredis` pub/sub | when: horizontal scaling is enabled; share existing ioredis connection from Caching |

**What the library covers (ws):**
- WebSocket server attached to existing HTTP server (same port)
- Connection lifecycle: open, message, close, error events
- Per-connection state management

**What Charter generates on top:**
- `lib/realtime` server module: `publish(channel, event, payload)` for server-side broadcasting
- Connection registry mapping authenticated userId/sessionId to WebSocket connections
- Auth enforcement on handshake: validate session token, reject unauthenticated connections with 401
- Channel/room routing: messages delivered only to subscribers of a specific channel
- Redis pub/sub fan-out when horizontal scaling is enabled
- `lib/realtime-client` browser utility: `connect()`, `subscribe(channel, handler)`, `unsubscribe(channel)`, exponential backoff reconnection

### nextjs

Same as node-typescript. Note: WebSocket servers cannot run inside Next.js serverless functions. A separate Express/Node.js server (`server.ts`) is required. If deploying purely to Vercel serverless, use `sse` protocol or a managed service (Ably, Pusher) as hosted alternative.

### remix

Same as node-typescript. Remix on a traditional Node.js server supports `ws` attached to the same HTTP server.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `channels` (Django Channels) | when: `config.protocol: websocket`; ASGI WebSocket support for Django with channel layers |
| recommended (layer) | `channels-redis` | Redis channel layer for multi-instance pub/sub when horizontal scaling is enabled |
| sse | `django-eventstream` | when: `config.protocol: sse`; SSE support for Django via ASGI |

**What the library covers:**
- ASGI WebSocket consumers with authentication support
- Channel groups for broadcasting to multiple connections
- Redis channel layer for cross-process message fan-out

**What Charter generates on top:**
- Authenticated WebSocket consumer with session validation
- `publish(channel, event, payload)` utility wrapping channel layer sends
- `lib/realtime_client` JavaScript bundle (same as node-typescript pattern)

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended (websocket) | Built-in Starlette WebSocket | when: `config.protocol: websocket`; FastAPI/Starlette has native WebSocket route handlers |
| recommended (layer) | `redis-py` pub/sub | when: horizontal scaling is enabled; async Redis pub/sub for cross-instance fan-out |
| sse | `sse-starlette` | when: `config.protocol: sse`; SSE response streaming for FastAPI |

**What the library covers:**
- Native FastAPI `WebSocket` route handler with async send/receive
- Redis pub/sub for cross-instance broadcasting when horizontal scaling is enabled

**What Charter generates on top:** Same as python-django — authenticated connection handler, channel registry, pub/sub fan-out, client-side utility.

---

## secrets

> Secrets management requires an environment validation library; provider integration (Vault, Doppler, AWS Secrets Manager) is generated on top.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `zod` (env schema) | Parse and validate `process.env` against a Zod schema; fast-fail on startup with descriptive errors |
| alternative | `envalid` | Lighter alternative to Zod for env-only validation; less flexible but simpler API |
| provider (AWS) | `@aws-sdk/client-secrets-manager` | when: `config.provider: aws-secrets-manager` |
| provider (Vault) | `node-vault` | when: `config.provider: vault` |

**What the library covers:**
- Schema-based env parsing with type coercion (string → number, boolean, URL)
- Descriptive error messages on missing required variables including variable name, purpose, and where to obtain
- TypeScript-typed config object exported for use throughout the codebase

**What Charter generates on top:**
- `lib/env.ts` module that validates all env vars declared by all capabilities in the project
- `.env.example` organized by capability with inline comments
- `.env.test` example for test environment
- AWS Secrets Manager pull-and-inject when `config.provider: aws-secrets-manager`
- Vault agent sidecar configuration when `config.provider: vault`
- Doppler CLI configuration when `config.provider: doppler`

### nextjs

Same as node-typescript. T3 Env (`@t3-oss/env-nextjs`) is an alternative that handles NEXT_PUBLIC_ prefix enforcement for client-exposed variables.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `django-environ` | `env()` calls with type coercion; `.env` file loading; fail-fast on missing required vars |
| alternative | `python-decouple` | when: team prefers decouple's `config()` API |
| provider (AWS) | `boto3` + `botocore` | when: `config.provider: aws-secrets-manager` |

**What the library covers:**
- `.env` file loading and `os.environ` override
- Typed env access: `env.str()`, `env.int()`, `env.bool()`, `env.url()`
- `default` and `cast` parameters for optional variables with fallbacks

**What Charter generates on top:**
- `config/env.py` aggregating all capability env declarations
- `.env.example` and `.env.test` files
- Provider-specific secret injection for AWS/Vault/Doppler

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `pydantic-settings` | `BaseSettings` class with automatic env var loading, type validation, and `.env` file support |

**What the library covers:**
- `BaseSettings` subclass as the single typed configuration object
- Automatic env var loading with optional `.env` file
- Nested settings models for per-capability configuration
- Startup `ValidationError` on missing required fields

**What Charter generates on top:** Same `.env.example`, `.env.test`, and provider integrations as python-django, but as a `pydantic-settings` `Settings` class exported from `lib/env.py`.

---

## multi-tenancy

> Multi-tenancy has no single dominant library for Node.js; row-level isolation is generated directly. Django has a useful library for schema-level isolation.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | none (Charter-generated) | Row-level isolation (`config.data-isolation: row-level`) is generated via ORM query scoping — no library covers this cleanly across all ORMs |
| alternative | `@clerk/clerk-sdk-node` org feature | when: Clerk is used for auth and `config.data-isolation: row-level`; Clerk Organizations maps to Charter's multi-tenancy model |

**What Charter generates:**
- Organization entity (id, name, slug, logo_url, created_at, created_by)
- Org-scoped middleware: extract current org from subdomain/path/session, inject into request context
- `org_id` foreign key on all scoped entities; ORM query filter enforcing current org
- Member invitation flow (email via Mailer), acceptance URL, member model
- Member management UI: list, view role, change role, remove
- Org switcher when `config.multi-org-membership: true`
- Separate schema per org when `config.data-isolation: schema-level` (Prisma schema-per-tenant pattern)
- Admin surfaces: org-list, org-impersonation

### nextjs

Same as node-typescript.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `django-tenants` | when: `config.data-isolation: schema-level`; PostgreSQL schema-per-tenant with automatic schema routing |
| alternative | none (Charter-generated) | when: `config.data-isolation: row-level`; Charter generates `org_id` scoping in managers and serializers |

**What the library covers (django-tenants):**
- Automatic PostgreSQL schema creation per tenant
- Schema-routing middleware extracting tenant from subdomain
- `TenantMixin` base model for tenant and domain management
- Separate INSTALLED_APPS lists for shared vs. tenant-specific apps

**What Charter generates on top:**
- Organization/tenant creation flow, invitation, and member management
- Row-level scoping managers when not using schema isolation
- Admin surfaces for org management

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | none (Charter-generated) | No mature FastAPI-specific multi-tenancy library; row-level isolation implemented via SQLAlchemy session scoping and FastAPI dependency injection |

**What Charter generates:** Same as node-typescript but as FastAPI middleware, SQLAlchemy session factories scoped to `org_id`, and Pydantic models.

---

## versioning

> No standard library dominates entity version history in any stack; Charter generates the append-only version model and diff UI from scratch.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | none (Charter-generated) | No widely adopted production-proven library covers the full versioning spec (snapshot, diff, restore, retention) |
| alternative | `@prisma-history` / custom Prisma middleware | when: ORM is Prisma; a Prisma middleware intercepts updates to generate version records |

**What Charter generates:** Full append-only version record model, snapshot capture on each entity update, diff view UI, restore-to-version operation, retention enforcement scheduled job.

### nextjs

Same as node-typescript.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `django-simple-history` | Actively maintained; appends history records on model save/delete; built-in diff and restore |

**What the library covers:**
- Automatic history record creation on `save()` and `delete()` with actor attribution
- `history` accessor on the model: `instance.history.all()`, `instance.history.as_of(date)`
- `HistoricalRecord.diff_against()` for field-level change diffing
- Restore to a previous version via `historical_record.instance.save()`
- `excluded_fields` to snapshot only declared `config.attaches-to[entity].versioned-fields`

**What Charter generates on top:**
- Version history UI component (timeline, diff view)
- Retention enforcement: delete oldest versions exceeding `max-versions` limit via scheduled job
- `versioning.version-created` and `versioning.version-restored` event emission
- Wiring of `config.restore-creates-new-version` behavior (new history record vs. direct save)

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | none (Charter-generated) | `django-simple-history` is Django-specific; no equivalent FastAPI/SQLAlchemy library at production maturity |

**What Charter generates:** Full custom version model, SQLAlchemy event listeners on update, and REST API endpoints for history access and restore.

---

## import-export

> Import/export requires CSV and Excel parsing/writing libraries; Charter generates the import validation flow, async job handling, and export UI on top.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended (CSV) | `papaparse` | when: CSV format enabled; zero-dependency, streaming-capable CSV parser |
| recommended (Excel) | `exceljs` | when: Excel format enabled; read and write `.xlsx`; supports formatting |
| alternative (Excel) | `xlsx` (SheetJS) | Larger ecosystem; community edition sufficient for basic read/write |

**What the library covers:**
- CSV parsing with header mapping and type coercion (papaparse)
- Excel workbook read and write with sheet and cell access (exceljs)
- Streaming for large files to avoid memory pressure

**What Charter generates on top:**
- Import UI: file upload, field mapping interface, validation preview, confirmation, results view
- Export UI: format selection, filter/date-range controls, download trigger
- Async job enqueue via `jobs-client` when rows exceed `config.async-threshold-rows`
- Partial failure model: skip invalid rows, report per-row errors in results view
- Access control enforcement per `config.import.who-can-import` and `config.export.who-can-export`
- Inbox/Mailer notification when async job completes

### nextjs

Same as node-typescript.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `tablib` | when: CSV and Excel formats enabled; unified API for CSV, Excel, JSON, and other formats |
| alternative (Excel) | `openpyxl` | when: Excel-only or advanced Excel formatting required |
| alternative (CSV) | Python built-in `csv` | when: CSV only and no extra dependency is preferred |

**What the library covers (tablib):**
- `Dataset` object with unified read/write for CSV, Excel, JSON
- Column header mapping and type handling
- Export to multiple formats from same Dataset

**What Charter generates on top:** Same import/export surfaces as node-typescript but as Django views, Celery tasks, and DRF serializers.

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended | `tablib` | Same as python-django |
| alternative (Excel) | `openpyxl` | when: Excel-only |

**What Charter generates on top:** Same surfaces as python-django but as async FastAPI route handlers and ARQ/Celery tasks.

---

## reporting

> Reporting generates internal analytics dashboards; charting libraries are client-side and stack-agnostic. Backend aggregation is generated directly against the app database.

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended (charts) | `recharts` | React-based; clean API, widely used, Recharts is maintained and has 1M+ weekly downloads |
| alternative (charts) | `chart.js` + `react-chartjs-2` | when: team prefers Canvas-based rendering; more chart types than recharts |
| alternative (charts) | `tremor` | when: using Tailwind CSS and want chart + dashboard component primitives together |

**What the library covers:**
- Client-side chart rendering (line, bar, stat card) from data arrays
- Responsive layout and tooltip/legend components

**What Charter generates on top:**
- Backend aggregation queries for each report in `config.reports` (count-by-period, unique-count-by-period)
- Dashboard layout with configurable panels
- Data refresh on page load; interval refresh when `config.refresh-interval-seconds` is non-null
- User-facing report views scoped to own data when `config.audience` includes `user-facing`
- Scheduled report delivery job when `config.scheduled-delivery.enabled: true` (requires Mailer)
- CSV export of report data

### nextjs

Same as node-typescript. recharts and tremor work as React Client Components in App Router.

### remix

Same as node-typescript.

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended (charts) | `recharts` (frontend) | Chart rendering is always client-side regardless of backend stack |
| backend aggregation | Django ORM + `django.db.models` | `annotate()`, `Count()`, `Sum()`, `TruncDay()`/`TruncWeek()` for period aggregation |

**What Charter generates on top:**
- Django views returning aggregated JSON data for each configured report
- DRF serializers for report data
- Celery tasks for scheduled report delivery when `config.scheduled-delivery.enabled: true`
- Same frontend dashboard components as node-typescript

### python-fastapi

| Role | Library | Notes |
|---|---|---|
| recommended (charts) | `recharts` (frontend) | Client-side rendering regardless of backend |
| backend aggregation | SQLAlchemy `func.count()`, `func.date_trunc()` | PostgreSQL window functions for period aggregation |

**What Charter generates on top:** Same as python-django but as FastAPI route handlers and SQLAlchemy aggregate queries.

---

## Capabilities without standard libraries

These capabilities are generated entirely from the Charter spec. No library covers enough of the required surface to justify a dependency.

| Capability | Reason |
|---|---|
| `comments` | Threaded comments require application-specific entity attachment, mention autocomplete against the app's user model, and reaction storage. Libraries like Commento or Isso are self-hosted products, not embeddable libraries. |
| `presence` | Presence is implemented directly on the real-time-transport and caching infrastructure already generated by Charter. No standalone library covers the heartbeat/TTL/avatar pattern without coupling to a specific framework. |
| `activity-feed` | Activity feeds are application-specific: the events subscribed to, the human-readable description format, and the entity link targets all vary per app. No general library maps Charter's event system to a feed model. |
| `inbox` | In-app notification delivery is tightly coupled to the app's user model and event system. Libraries like Novu offer managed notification infrastructure but are a hosted service, not an embeddable library. Novu can be used as a hosted alternative if the team prefers it. |
| `announcements` | Admin-to-user broadcast UI is application-specific in targeting, display format, and dismissal tracking. No library covers this without being an opinionated SaaS product. |
| `onboarding` | Onboarding step definitions, completion conditions, and access gating are entirely app-specific. Generic onboarding tour libraries (Shepherd.js, Intro.js) cover UI walkthroughs only, not Charter's step/state model. |
| `feedback` | NPS/CSAT/feature-request/bug-report forms are generated from `config.types`. Libraries like Hotjar/Typeform are hosted SaaS products. Charter generates the full collection and storage model. |
| `waitlist` | Waitlist position management, invite flows, and admin controls are simple enough to generate directly; no library covers the full spec. |
| `archiving` | Archiving adds `archived_at`/`archived_by` columns and query scope exclusion to existing ORM models. This is a few lines of generation per entity — no library required. |
| `soft-delete` | Soft delete adds `deleted_at` and default scope exclusion. `django-safedelete` is available for Django but its model opinions conflict with Charter's generated patterns. Charter generates this directly. |
| `sharing` | Per-resource access grants are tightly coupled to the app's entity model and Permissions capability. No general sharing library exists that does not also own the auth model. |
| `impersonation` | Session-within-session impersonation is a security-sensitive feature that must integrate deeply with the project's specific auth implementation. Generic libraries do not exist at production maturity. |
| `referral` | Referral link generation, cookie attribution, qualifying event detection, and reward issuance are all app-specific and depend on the Billing/Payment event system. No standalone library covers this. |
| `messaging` | Direct/group messaging requires the real-time-transport infrastructure and app-specific conversation/participant models. Libraries like Stream Chat and Sendbird are hosted SaaS products, not embeddable libraries. |
| `containerization` | Dockerfile and docker-compose generation are deployment artifacts — there is no library involved. Charter generates static configuration files. |
| `ci-cd` | CI/CD pipeline configuration is generated as YAML files for the configured provider. No library is involved. |
| `audit-log` | Audit records are append-only writes triggered by subscribed events. `django-auditlog` is available for Django but its storage model and event subscriptions differ from Charter's capability event system. Charter generates the audit model and event subscriptions directly. |
| `user-management` | User profile management builds on the app's existing auth library (better-auth, django-allauth, fastapi-users) user model. There is no separate "user management library" — Charter generates the profile views and admin surfaces on top of the auth library's user model. |
