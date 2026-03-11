# Charter Stacks Registry

Reference document for charter-init. Lists known base stacks, what Charter capabilities each covers, and where to fetch current state at project init time.

**How charter-init uses this file:**
1. Propose an appropriate base stack based on the app description
2. Fetch the URL in `fetch-for-current-state` to verify current coverage
3. Cross-reference selected capabilities against `capabilities-covered` and `capabilities-partial` to identify what Charter needs to add
4. For gaps, consult `libraries.md` for recommended library per capability

*Last updated: 2026-03-11*

---

## create-next-app

```yaml
name: create-next-app
url: https://nextjs.org/docs/app/getting-started/installation
last-verified: 2026-03-11
fetch-for-current-state: https://nextjs.org/docs/app/api-reference/cli/create-next-app

runtime: node
framework: next.js
language: typescript
database: none (developer's choice; Prisma or Drizzle added separately)
architecture: monolithic  # single process serves both frontend and API routes

scaffold:
  command: npx create-next-app@latest {name} --yes --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --no-git
  non-interactive: true  # --yes accepts all defaults

capabilities-covered: []

capabilities-partial:
  - capability: api-access
    what-is-included: App Router Route Handlers (REST endpoints) and Server Actions provide a full backend API surface from the same process
    what-is-missing: Auth, API key management, rate limiting, versioning, public API docs — all must be added

  - capability: secrets
    what-is-included: Next.js environment variable conventions (.env.local, NEXT_PUBLIC_ prefix for client exposure)
    what-is-missing: Typed validation module, startup fast-fail on missing vars, aggregated .env.example

notes: >
  Bare create-next-app is a framework skeleton, not an application starter. It provides no Charter
  capabilities — no auth, no database, no email, no observability. Its value is the largest Node.js
  ecosystem, Vercel-native deployment, and the flexibility to choose every infrastructure layer.
  Charter must generate every capability from scratch. The monolithic architecture means a single
  docker-compose app service handles both the React frontend and the Next.js API routes. Use this
  stack when maximum flexibility is needed or when deploying to Vercel with full platform integration.
  138K GitHub stars, 242K weekly downloads, actively maintained by Vercel.
```

---

## create-t3-app

```yaml
name: create-t3-app
url: https://create.t3.gg
last-verified: 2026-03-11
fetch-for-current-state: https://raw.githubusercontent.com/t3-oss/create-t3-app/main/README.md

runtime: node
framework: next.js + trpc
language: typescript
database: postgresql (Prisma) or sqlite/postgresql (Drizzle) — selected at init
architecture: monolithic  # Next.js serves both frontend and tRPC API

scaffold:
  command: npm create t3-app@latest {name} -- --CI --trpc --prisma --nextAuth --tailwind --dbProvider postgres --noGit
  non-interactive: true  # --CI flag enables programmatic mode; individual packages selected via flags
  flags:
    - --trpc          # include tRPC end-to-end type-safe API layer
    - --prisma        # include Prisma ORM (alternative: --drizzle)
    - --nextAuth      # include NextAuth.js (omit if using different auth)
    - --tailwind      # include Tailwind CSS
    - --dbProvider    # postgres | mysql | sqlite | planetscale

capabilities-covered:
  - migrations      # Prisma schema pre-generated with NextAuth models; prisma migrate dev ready to run

capabilities-partial:
  - capability: auth
    what-is-included: NextAuth.js configured with session provider and database adapter wired to Prisma; basic session management
    what-is-missing: Registration UI, social provider callback pages, MFA, password reset flow, email verification — Charter generates these

  - capability: api-access
    what-is-included: tRPC router scaffold with end-to-end TypeScript types; procedure definitions; React Query integration on client
    what-is-missing: Public REST API, API key management, rate limiting, versioning, developer-facing docs

  - capability: secrets
    what-is-included: T3 Env package for typed environment variable validation with startup fast-fail
    what-is-missing: Charter's aggregated .env.example across all capabilities; validation wiring for Charter-added capabilities

notes: >
  create-t3-app is the most Charter-compatible Node.js monolithic starter. The --CI flag enables
  fully non-interactive scaffolding. It is the only Node.js starter that ships with a database
  schema, ORM, and typed API layer pre-wired — meaning Charter's migration, auth, and API access
  capabilities start from a real foundation rather than from nothing. The trade-off is strong
  opinions: Next.js, tRPC, and Prisma are not configurable at Charter-init time within this stack.
  Teams that need a different ORM or API style should use create-next-app instead. The monolithic
  architecture means a single docker-compose app service. 28K GitHub stars, actively maintained
  by the t3-oss organization.
```

---

## NestJS

```yaml
name: NestJS
url: https://nestjs.com
last-verified: 2026-03-11
fetch-for-current-state: https://docs.nestjs.com

runtime: node
framework: nestjs
language: typescript
database: none (developer's choice; TypeORM, Prisma, or Drizzle added separately)
architecture: separated  # API-only backend; pairs with a frontend service (typically create-vite)
frontend-pair: create-vite (react-ts template) — `npm create vite@latest web -- --template react-ts --no-interactive`

scaffold:
  command: npx @nestjs/cli new {name} --package-manager pnpm --skip-git --strict
  non-interactive: false  # one prompt for package manager; use --package-manager to reduce interaction
  note: nest new is fast; the package manager prompt is the only interactive step

capabilities-covered: []

capabilities-partial:
  - capability: api-access
    what-is-included: Modular controller/service/module architecture; decorator-based routing; dependency injection; CLI generators for adding controllers, services, and modules
    what-is-missing: Database integration, auth, API key management, rate limiting, versioning, OpenAPI docs (available via @nestjs/swagger but not scaffolded)

notes: >
  NestJS is an opinionated, Angular-inspired backend framework with strong TypeScript support,
  decorator-based architecture, and excellent documentation. It is the most structured of the
  Node.js backend options — appropriate for enterprise teams or anyone who wants explicit
  module boundaries enforced by the framework. Zero database, zero auth, zero Docker are
  included in the scaffold — Charter generates everything. The separated architecture means
  docker-compose has an api service (NestJS) and a web service (Vite/React). Hot reload uses
  nest start --watch inside the api container. 74K GitHub stars, Series A funded with support
  commitment through 2030, 7.7M weekly downloads.
```

---

## Hono

```yaml
name: Hono
url: https://hono.dev
last-verified: 2026-03-11
fetch-for-current-state: https://hono.dev/docs/getting-started/nodejs

runtime: node  # also runs on Cloudflare Workers, Bun, Deno — Charter targets node unless specified
framework: hono
language: typescript
database: none (developer's choice; Drizzle or Prisma are common pairings)
architecture: separated  # API-only backend; pairs with a frontend service (typically create-vite)
frontend-pair: create-vite (react-ts template) — `npm create vite@latest web -- --template react-ts --no-interactive`

scaffold:
  command: npm create hono@latest {name} -- --template nodejs --install --pm pnpm
  non-interactive: true  # --template suppresses runtime selection; --pm suppresses package manager selection

capabilities-covered: []

capabilities-partial:
  - capability: api-access
    what-is-included: Minimal but complete routing layer; typed RPC via hono/rpc (end-to-end type safety without tRPC); built-in middleware for CORS, logger, JWT; OpenAPI generation via @hono/zod-openapi
    what-is-missing: Auth implementation, API key management, rate limiting, versioning

  - capability: real-time-transport
    what-is-included: WebSocket support via upgradeWebSocket helper
    what-is-missing: Connection registry, rooms, presence, broadcast — Charter generates these

notes: >
  Hono is the fastest-growing Node.js backend framework (9.3M weekly downloads, +26%
  month-over-month as of early 2026). Its scaffolded output is intentionally minimal — a
  single entry point and package.json — which means there is nothing to undo or work around.
  hono/rpc provides end-to-end TypeScript types between server and client without requiring
  tRPC, making it a good match for teams that want type safety without the full T3 ecosystem.
  The nodejs template targets @hono/node-server; if the deployment target is Cloudflare Workers
  or another edge runtime, select that template instead and adjust the containerization
  accordingly. Fully non-interactive scaffold. 28K GitHub stars.
```

---

## Fastify

```yaml
name: Fastify
url: https://fastify.dev
last-verified: 2026-03-11
fetch-for-current-state: https://fastify.dev/docs/latest/

runtime: node
framework: fastify
language: typescript
database: none (developer's choice; Prisma and Drizzle both have documented Fastify integrations)
architecture: separated  # API-only backend; pairs with a frontend service (typically create-vite)
frontend-pair: create-vite (react-ts template) — `npm create vite@latest web -- --template react-ts --no-interactive`

scaffold:
  command: npx fastify-cli generate {name} --lang=ts
  non-interactive: false  # no CI flag; command is single-step with no interactive prompts in practice

capabilities-covered: []

capabilities-partial:
  - capability: api-access
    what-is-included: Route definitions with JSON schema validation; automatic OpenAPI schema generation via @fastify/swagger; Fastify plugin architecture for modular route registration
    what-is-missing: Auth, API key management, rate limiting, versioning — all require Charter generation or plugin installation

notes: >
  Fastify is the highest-performance Node.js HTTP framework, optimized for throughput with
  built-in JSON schema validation and serialization. Its plugin system enforces encapsulation
  and is well-suited to teams building large APIs that need explicit boundary enforcement.
  The scaffold generates a minimal TypeScript project with a plugin/route structure.
  Unlike Hono, Fastify targets Node.js exclusively — there is no edge runtime story.
  Appropriate for teams with performance-critical API requirements or existing Fastify
  expertise. 35K GitHub stars, 5M weekly downloads, actively maintained. Hot reload in
  development uses fastify start -w (via fastify-cli) or nodemon inside the api container.
```

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
