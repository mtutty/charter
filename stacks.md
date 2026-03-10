# Charter Stacks Registry

Reference document for charter-init. Lists known base stacks, what Charter capabilities each covers, and where to fetch current state at project init time.

**How charter-init uses this file:**
1. Propose an appropriate base stack based on the app description
2. Fetch the URL in `fetch-for-current-state` to verify current coverage
3. Cross-reference selected capabilities against `capabilities-covered` and `capabilities-partial` to identify what Charter needs to add
4. For gaps, consult `libraries.md` for recommended library per capability

*Last updated: 2026-03-10*

---

## Epic Stack

```yaml
name: Epic Stack
url: https://github.com/epicweb-dev/epic-stack
last-verified: 2026-03-10
fetch-for-current-state: https://raw.githubusercontent.com/epicweb-dev/epic-stack/main/docs/features.md

runtime: node
framework: remix
language: typescript
database: sqlite (LiteFS, distributed)

capabilities-covered:
  - auth            # email/password + cookie sessions + 2FA (TOTP) + password reset
  - mailer          # transactional email via Resend; password reset flow included
  - migrations      # Prisma Migrate; schema in prisma/schema.prisma
  - permissions     # role-based permissions via Prisma; helpers generated in codebase
  - caching         # cachified library; in-memory + SQLite-based caching
  - observability   # Sentry error monitoring + Grafana dashboards via Fly Metrics + healthcheck endpoint
  - containerization # Docker + Fly.io deployment; multi-region via LiteFS
  - ci-cd           # GitHub Actions pipelines for test, build, deploy to staging and production
  - secrets         # environment variable management; .env files with typed validation pattern
  - user-management # user profile management; admin surfaces for user search and suspension

capabilities-partial:
  - capability: file-management
    what-is-included: Tigris object storage integration for image storage and serving
    what-is-missing: General file upload/download flows, file metadata model, user-facing file management UI

  - capability: real-time-transport
    what-is-included: Remix's progressive enhancement model supports streaming; no dedicated WebSocket or SSE layer
    what-is-missing: WebSocket or SSE transport, presence, push subscriptions

notes: >
  The Epic Stack is the most production-complete Node.js/TypeScript starter available, built on Remix with
  opinionated choices that reflect real production experience (SQLite+LiteFS for distributed reads, Fly.io for
  deployment, Resend for email). It is best suited for teams that want a solid, deployable foundation with auth,
  observability, and CI/CD pre-wired. Its SQLite-first approach is a deliberate trade-off: excellent for most
  SaaS workloads but requires migration to PostgreSQL for applications needing advanced DB features or heavy
  write concurrency. Feature flags, billing, background jobs, and search are explicitly listed as planned but
  not yet included — Charter must generate all of these for any app that needs them.
```

---

## create-t3-app

```yaml
name: create-t3-app
url: https://create.t3.gg
last-verified: 2026-03-10
fetch-for-current-state: https://raw.githubusercontent.com/t3-oss/create-t3-app/main/README.md

runtime: node
framework: next.js
language: typescript
database: postgresql (Prisma) or sqlite/postgresql (Drizzle)

capabilities-covered:
  - auth            # NextAuth.js; email/password + OAuth providers; session management
  - migrations      # Prisma Migrate or Drizzle Kit depending on ORM selection at init

capabilities-partial:
  - capability: api-access
    what-is-included: tRPC end-to-end typesafe RPC layer; REST endpoints possible via Next.js API routes
    what-is-missing: Public API key management, rate limiting, versioning, developer-facing API docs

  - capability: secrets
    what-is-included: .env file convention with Next.js environment variable support; T3 Env package for typed validation
    what-is-missing: Charter's aggregated .env.example across all capabilities; startup exit-on-missing validation wiring

notes: >
  create-t3-app is a modular scaffold — each technology (tRPC, NextAuth, Prisma/Drizzle, Tailwind) is opt-in
  at generation time, not all included by default. It is an excellent foundation for TypeScript-first teams
  building tightly-typed full-stack apps, but it is deliberately thin: there is no auth UI beyond basic
  configuration, no email, no background jobs, no observability, and no CI/CD out of the box. Charter must
  generate the majority of functional capabilities for any non-trivial application. Best for teams who want
  type safety scaffolding and will add their own infrastructure choices.
```

---

## create-next-app (bare Next.js)

```yaml
name: create-next-app
url: https://nextjs.org/docs/getting-started/installation
last-verified: 2026-03-10
fetch-for-current-state: https://nextjs.org/docs/app/getting-started/installation

runtime: node
framework: next.js
language: typescript
database: none (no database included)

capabilities-covered: []

capabilities-partial:
  - capability: api-access
    what-is-included: Next.js App Router Route Handlers and Server Actions provide a backend API surface
    what-is-missing: Auth, API key management, rate limiting, versioning — all must be added

  - capability: secrets
    what-is-included: Next.js environment variable conventions (.env.local, .env); NEXT_PUBLIC_ prefix for client exposure
    what-is-missing: Typed validation module, startup fast-fail on missing vars, aggregated .env.example

notes: >
  Bare create-next-app (with defaults: TypeScript, Tailwind, ESLint, App Router, Turbopack) is a framework
  skeleton, not an application starter. It provides zero capabilities in the Charter sense — no auth, no
  database, no email, no observability. Its value is maximum flexibility and the largest ecosystem in the
  Node.js space. Charter must generate every capability from scratch. Use this stack when the application
  requires a custom infrastructure combination not served by Epic Stack or T3, or when deploying to Vercel
  and want full Next.js platform integration.
```

---

## OpenSaaS (Wasp)

```yaml
name: OpenSaaS
url: https://github.com/wasp-lang/open-saas
last-verified: 2026-03-10
fetch-for-current-state: https://raw.githubusercontent.com/wasp-lang/open-saas/main/README.md

runtime: node
framework: wasp (react + node.js)
language: typescript
database: postgresql (Prisma)

capabilities-covered:
  - auth            # email/password + social (Google, GitHub, Slack, Microsoft); email verification
  - mailer          # SendGrid, Mailgun, or SMTP; transactional email integrated
  - background-jobs # Wasp Jobs: cron and queue-based background job processing built into framework
  - payment         # Stripe and Lemon Squeezy (Polar.sh planned); subscription and one-time payment flows
  - file-management # AWS S3 integration for file uploads; upload flows included
  - migrations      # Prisma Migrate; managed by Wasp framework

capabilities-partial:
  - capability: billing
    what-is-included: Stripe subscription management, product/plan configuration, payment status tracking
    what-is-missing: Invoice history UI, dunning logic, proration, Charter's billing config schema wiring

  - capability: observability
    what-is-included: Plausible or Google Analytics integration for usage analytics
    what-is-missing: Structured server-side logging, error tracking (Sentry), health check endpoints, APM

  - capability: user-management
    what-is-included: Admin dashboard with user overview; basic user management surfaces
    what-is-missing: Full Charter admin surface (suspend, force password reset, session revocation, impersonation hooks)

  - capability: api-access
    what-is-included: Wasp's typesafe RPC (Operations) layer connects client and server
    what-is-missing: Public REST/GraphQL API with key management, rate limiting, versioning

notes: >
  OpenSaaS is the most feature-rich free SaaS boilerplate in the Node.js ecosystem (~13k GitHub stars as of
  early 2026). Built on Wasp, which handles much of the full-stack wiring declaratively, it covers more
  Charter capabilities out of the box than any other Node.js starter. The trade-off is Wasp's DSL: teams
  adopt a framework-level abstraction that is not standard Next.js or Remix, which can constrain hiring and
  library compatibility. Multi-tenancy is not included (tracked as an open issue). Best for solo founders
  or small teams building B2C or B2B SaaS who want auth, payments, and background jobs wired up on day one.
```

---

## Django + Django REST Framework

```yaml
name: Django + DRF
url: https://www.django-rest-framework.org
last-verified: 2026-03-10
fetch-for-current-state: https://www.django-rest-framework.org/

runtime: python
framework: django + django-rest-framework
language: python
database: postgresql (conventional; sqlite for development)

capabilities-covered:
  - auth            # django.contrib.auth: registration, login, logout, password reset, session + token auth
  - permissions     # DRF permission classes (IsAuthenticated, DjangoModelPermissions, custom); object-level permissions via django-guardian
  - migrations      # Django Migrations: makemigrations + migrate; forward and rollback support
  - user-management # django.contrib.auth User model + admin site; user CRUD, group management, superuser
  - api-access      # DRF: serializers, viewsets, routers, browsable API, schema generation (drf-spectacular)
  - secrets         # django-environ or python-decouple conventional patterns; settings split by environment

capabilities-partial:
  - capability: background-jobs
    what-is-included: >
      Celery is the universal Django background job solution; django-celery-beat for cron scheduling.
      Not included in a base Django project — requires separate installation and Redis/RabbitMQ broker.
    what-is-missing: >
      Charter must generate Celery config, broker connection, worker entrypoint, and retry logic.
      No built-in admin dashboard for job monitoring without additional packages (flower).

  - capability: mailer
    what-is-included: django.core.mail with SMTP backend; email_backend settings; basic send_mail() utility
    what-is-missing: >
      Transactional provider integration (SendGrid, Postmark, SES), templated email system,
      bounce handling, preference center, unsubscribe links — all must be generated by Charter.

  - capability: observability
    what-is-included: Django's logging framework; request/response logging via middleware
    what-is-missing: Structured JSON logging, Sentry integration, health check endpoints, APM, graceful shutdown handling

  - capability: containerization
    what-is-included: No containerization included; Docker is a common addition but not part of Django itself
    what-is-missing: Charter must generate Dockerfile, docker-compose.yml, .dockerignore, and operational scripts

  - capability: ci-cd
    what-is-included: No CI/CD pipeline included
    what-is-missing: Charter must generate full GitHub Actions or equivalent pipeline

notes: >
  Django + DRF is the most battle-tested Python web stack, with exceptional built-in coverage of auth,
  permissions, migrations, and admin surfaces. Django 5.2 LTS (released April 2025, supported through April
  2028) is the recommended target. The conventional setup does not prescribe a single canonical repo —
  Charter should assume a standard project layout (settings/, apps/, requirements/). Django's sync-first
  architecture suits request/response APIs well but requires async configuration for WebSockets and
  high-concurrency workloads. Background jobs (Celery), containerization, and CI/CD are universal additions
  but always require Charter to generate. Best for data-heavy, admin-driven, or API-first applications
  where Python's data ecosystem (pandas, numpy, ML libraries) is an advantage.
```

---

## FastAPI

```yaml
name: FastAPI
url: https://fastapi.tiangolo.com
last-verified: 2026-03-10
fetch-for-current-state: https://fastapi.tiangolo.com

runtime: python
framework: fastapi (starlette)
language: python
database: none (no ORM included by default)

capabilities-covered:
  - api-access      # OpenAPI schema auto-generation, Swagger UI (/docs), ReDoc (/redoc), Pydantic validation, dependency injection

capabilities-partial:
  - capability: auth
    what-is-included: >
      OAuth2 password flow, Bearer token, HTTP Basic, JWT support via python-jose (optional).
      Security dependency utilities (OAuth2PasswordBearer, HTTPBearer).
    what-is-missing: >
      No user model, registration, login UI, password hashing setup, session management, or email
      verification out of the box. Charter must generate the full auth implementation.

  - capability: observability
    what-is-included: Uvicorn access logging; middleware support for custom request logging
    what-is-missing: Structured JSON logging, Sentry integration, health check endpoints, graceful shutdown, APM

  - capability: secrets
    what-is-included: pydantic-settings for typed environment variable management (optional dependency)
    what-is-missing: Aggregated .env.example, startup fast-fail validation wired to all capabilities

  - capability: real-time-transport
    what-is-included: WebSocket support built into Starlette/FastAPI (websocket route handlers)
    what-is-missing: Connection registry, broadcast, rooms, presence — Charter must generate these

  - capability: background-jobs
    what-is-included: FastAPI BackgroundTasks for simple fire-and-forget tasks within a request lifecycle
    what-is-missing: >
      BackgroundTasks are in-process and non-durable. For production background jobs Charter must
      generate Celery or ARQ integration with a real broker.

notes: >
  FastAPI is a high-performance async API framework, not a full-stack application starter. It ships with
  exceptional API documentation tooling and type-driven validation via Pydantic, but provides almost nothing
  else — no database, no user model, no auth implementation, no email, no admin. Charter must generate nearly
  all capabilities from scratch. Best for microservices, ML inference APIs, and data-intensive backends where
  async performance and OpenAPI compliance matter. Teams building a full SaaS product should expect Charter
  to do substantially more work than with Django+DRF or any of the Node.js stacks.
```

---

## Laravel (Starter Kits)

```yaml
name: Laravel Starter Kits
url: https://laravel.com/docs/starter-kits
last-verified: 2026-03-10
fetch-for-current-state: https://laravel.com/docs/starter-kits

runtime: php
framework: laravel
language: php
database: mysql or postgresql (Eloquent ORM)

capabilities-covered:
  - auth            # Laravel Fortify: registration, login, password reset, email verification, 2FA (TOTP), rate limiting
  - migrations      # Laravel Migrations: artisan make:migration, migrate, rollback; full up/down support
  - user-management # Eloquent User model; Fortify action classes for registration and password management
  - permissions     # Laravel Gates and Policies for authorization; role assignment via middleware
  - mailer          # Laravel Mail + Mailables; SMTP/Mailgun/Postmark/SES driver support; queue-backed delivery

capabilities-partial:
  - capability: background-jobs
    what-is-included: >
      Laravel Queues: queue driver config (database, Redis, SQS), job classes, artisan queue:work.
      Laravel Scheduler for cron-based jobs. Laravel Horizon for Redis queue monitoring (optional).
    what-is-missing: >
      Queue infrastructure is present but no pre-built job workers or monitoring UI are included in
      starter kits by default. Charter must generate specific job handlers and configure the worker.

  - capability: containerization
    what-is-included: Laravel Sail provides Docker Compose for local development (PHP, MySQL/PostgreSQL, Redis, Mailpit)
    what-is-missing: >
      Production Dockerfile and multi-stage build not included. Charter must generate a production
      container setup suitable for deployment.

  - capability: observability
    what-is-included: Laravel Telescope for local debugging (optional dev package); basic application logging via Monolog
    what-is-missing: Structured JSON logging for production, Sentry integration, health check endpoints, APM

  - capability: ci-cd
    what-is-included: No CI/CD pipeline included in starter kits
    what-is-missing: Charter must generate full pipeline configuration

  - capability: api-access
    what-is-included: Laravel Sanctum for SPA/mobile API authentication (token-based); API route scaffolding
    what-is-missing: >
      Public developer API with key management, rate limiting tiers, versioning, and OpenAPI docs
      require additional generation. Sanctum covers authentication, not API product management.

notes: >
  Laravel starter kits (React, Vue, Svelte, and Livewire variants, all using Inertia.js or Blade) are the
  most complete PHP full-stack starters available, covering auth with 2FA, email verification, migrations,
  and a coherent queue system out of the box. Laravel is particularly strong for teams that want Rails-like
  productivity in PHP. The four frontend variants share identical backend capabilities — the choice is purely
  a frontend preference. Laravel Sail covers local development containerization, but Charter must generate
  production Docker and CI/CD. Best for PHP teams or projects where the existing infrastructure is
  PHP/MySQL-centric.
```

---
