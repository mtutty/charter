# Containerization

## Generation Instructions

Containerization generates Dockerfile(s), docker-compose configuration, and operational scripts. The outputs are deployment artifacts — this capability does not generate runtime application code.

**Every service runs in Docker, in both development and production.** Do not generate instructions or scripts that run any part of the application natively on the host. Dev/prod parity is not optional.

---

### Step 0 — Determine Architecture Topology

Before generating any files, read `stack` in `project-brief.yaml` to determine topology. This governs service naming and compose structure throughout.

**Monolithic** — a single process serves both frontend and API:
- `stack.frontend.meta-framework` is `next`, `remix`, or `nuxt`
- Services: `app`, `db`, `redis` (if needed), `worker` (if needed), `mailer` (if needed)

**Separated** — frontend and backend are distinct processes:
- `stack.frontend.meta-framework` is `vite`, or there is no frontend (`none`)
- Services: `api`, `web` (if frontend present), `db`, `redis` (if needed), `worker` (if needed), `mailer` (if needed)

Use `app` / `api` / `web` exactly as specified above. Do not use other names. Future charter-iterate instructions and operational scripts depend on these names being consistent.

---

### Dockerfile

**Monolithic or API service** — generate a multi-stage `Dockerfile`:

1. `deps` stage: copy lockfile, install all dependencies (including devDependencies). Cache aggressively — reruns only when lockfile changes.
2. `build` stage: copy source, run the build (TypeScript compilation, asset bundling). Produces `dist/` or equivalent.
3. `runtime` stage: start from the minimal base image (`node:lts-alpine`, `python:3.12-slim`, etc.). Copy only built artifacts and production dependencies from prior stages. Create and switch to a non-root user (`USER node` or equivalent). Expose the application port. Set `CMD` to the production entrypoint.

**Separated — frontend `web` service** — generate `Dockerfile.web` using the same multi-stage pattern:

1. `deps` stage: install frontend dependencies.
2. `build` stage: run `npm run build` to produce static assets (`dist/`).
3. `runtime` stage: serve built assets with `nginx:alpine`. Copy a minimal `nginx.conf` that serves the SPA and proxies `/api` requests to the `api` service. This stage is the production web image.

Generate `.dockerignore` excluding: `node_modules`, `.env*`, `*.test.*`, `coverage/`, `.git/`, and any secrets or credential files. If the project has both `Dockerfile` and `Dockerfile.web`, a single `.dockerignore` at the root covers both.

If Background Jobs declares a separate worker entrypoint, the API `Dockerfile` supports both app and worker from the same image — the entrypoint is specified via the compose service `command`, not baked into the image.

---

### Development Compose

Generate `docker-compose.yml` for local development. All application services run from source with hot reload. Infrastructure services (db, redis) run from official images.

#### Node.js volume pattern — applies to every Node.js service

Every Node.js service (app, api, web, worker) **must** include both of these volume entries:

```yaml
volumes:
  - .:/app                  # mounts source for hot reload
  - /app/node_modules       # anonymous volume — prevents host node_modules from overriding container's
```

The second volume is critical. Without it, the host `node_modules` (absent, wrong OS, or wrong Node version) overlays the container installation and the service fails to start. Never omit it.

#### Hot reload commands by framework

Use the correct development command for the `command:` field. Do not use `npm start` or the production entrypoint in development:

| Framework | Dev command |
|---|---|
| Next.js | `npm run dev` |
| Remix | `npm run dev` |
| NestJS | `npm run start:dev` (uses `nest start --watch`) |
| Hono (Node) | `npm run dev` (uses `@hono/node-server` with `--watch`) |
| Fastify | `npm run dev` (uses `fastify start -w` or nodemon) |
| Vite (React/Vue/etc.) | `npm run dev` |
| Django | `python manage.py runserver 0.0.0.0:8000` |
| FastAPI | `uvicorn main:app --host 0.0.0.0 --reload` |
| Laravel | `php artisan serve --host=0.0.0.0` |

**Bind to `0.0.0.0`**, not `localhost` or `127.0.0.1`. A service bound to `localhost` inside a container is unreachable from the host and from other containers.

#### Service definitions

**`app` service (monolithic)**

```yaml
app:
  build:
    context: .
    target: deps          # dev builds stop at deps stage — no production build needed
  volumes:
    - .:/app
    - /app/node_modules
  ports:
    - "${APP_PORT:-3000}:3000"
  environment:
    - NODE_ENV=development
    # ... all capability env vars
  depends_on:
    db:
      condition: service_healthy
  command: npm run dev
```

**`api` service (separated)**

```yaml
api:
  build:
    context: .
    target: deps
  volumes:
    - .:/app
    - /app/node_modules
  ports:
    - "${API_PORT:-3000}:3000"
  environment:
    - NODE_ENV=development
  depends_on:
    db:
      condition: service_healthy
  command: npm run dev
```

**`web` service (separated — frontend)**

```yaml
web:
  build:
    context: ./web        # or the frontend directory
    target: deps
  volumes:
    - ./web:/app
    - /app/node_modules
  ports:
    - "${WEB_PORT:-5173}:5173"
  environment:
    - NODE_ENV=development
    - VITE_API_URL=http://localhost:${API_PORT:-3000}
  command: npm run dev -- --host 0.0.0.0
```

**Networking note for separated architecture**: The `web` container runs the Vite dev server. API calls from the browser originate on the host, not inside Docker — so `VITE_API_URL` must use `localhost` and the host-mapped API port, not the Docker service name. Server-side calls (SSR, Vite proxy) that originate inside the `web` container use the Docker service name: `http://api:3000`. If the frontend makes server-side API calls, configure Vite's dev server proxy:

```typescript
// vite.config.ts
server: {
  proxy: {
    '/api': 'http://api:3000'
  }
}
```

**`db` service**

```yaml
db:
  image: postgres:16-alpine      # use stack.database.primary to select image
  environment:
    POSTGRES_USER: ${POSTGRES_USER:-app}
    POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-password}
    POSTGRES_DB: ${POSTGRES_DB:-app_dev}
  volumes:
    - db_data:/var/lib/postgresql/data
  ports:
    - "${DB_PORT:-5432}:5432"    # exposed to host for database tools (TablePlus, etc.)
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-app}"]
    interval: 5s
    timeout: 5s
    retries: 10
```

The `healthcheck` is required. Every service that depends on the database must use `condition: service_healthy` in `depends_on`, not just `depends_on: db`. Without the health check condition, the app starts before PostgreSQL is ready to accept connections, and migrations fail.

**`redis` service** — include if Caching is present or Background Jobs uses redis transport:

```yaml
redis:
  image: redis:7-alpine
  volumes:
    - redis_data:/data
  ports:
    - "${REDIS_PORT:-6379}:6379"
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 5s
    timeout: 5s
    retries: 10
```

**`worker` service** — include if Background Jobs declares a separate worker entrypoint. Shares the same image and source volume as the api/app service:

```yaml
worker:
  build:
    context: .
    target: deps
  volumes:
    - .:/app
    - /app/node_modules
  environment:
    - NODE_ENV=development
  depends_on:
    db:
      condition: service_healthy
    redis:
      condition: service_healthy
  command: npm run worker:dev
```

**`mailer` service** — include if Mailer capability is present. Mailpit provides a local SMTP server and web UI for inspecting sent email:

```yaml
mailer:
  image: axllent/mailpit:latest
  ports:
    - "${MAILER_UI_PORT:-8025}:8025"   # web UI
    - "${MAILER_SMTP_PORT:-1025}:1025" # SMTP
```

Configure the Mailer capability to use `SMTP_HOST=mailer`, `SMTP_PORT=1025` in development.

#### Named volumes declaration

Declare all named volumes at the bottom of docker-compose.yml:

```yaml
volumes:
  db_data:
  redis_data:
```

---

### Dev/prod parity constraint

The development compose uses the same database engine and major version as production. Do not substitute SQLite for PostgreSQL, in-memory cache for Redis, or synchronous job execution for a queue. These substitutions mask real bugs and undermine the value of local development. If a developer does not want to run Redis locally, they should use `transport: database` in Background Jobs rather than an in-memory substitute.

---

### Production Compose

If `config.orchestration: compose`, generate `docker-compose.prod.yml`. Unlike the dev compose:

- No source volume mounts — services run from the built image.
- No exposed database or redis ports — infrastructure is accessible only within the compose network.
- Environment variables sourced from `.env.production` or the secrets provider.
- `restart: unless-stopped` on all services.
- `app`/`api` and `worker` are separate services from the same image with different `command` values.
- For separated architecture: `web` service uses `Dockerfile.web` runtime stage (nginx serving built assets).

If `config.orchestration: kubernetes`, generate Kubernetes manifests instead: `Deployment` for app/api, `Deployment` for worker (if present), `Deployment` for web (if present), `Service` for each, `ConfigMap` for non-secret config, and notes on where to inject secrets. Do not generate Helm charts unless explicitly requested.

---

### Operational Scripts

Generate a `Makefile` with:

```makefile
dev:          ## Start all services in development mode
	docker compose up

dev-build:    ## Rebuild images and start
	docker compose up --build

down:         ## Stop all services
	docker compose down

logs:         ## Tail logs from all services
	docker compose logs -f

shell:        ## Open a shell in the app/api container
	docker compose exec api sh     # or 'app' for monolithic

migrate:      ## Run database migrations
	docker compose exec api npm run migrate     # adjust command per ORM

seed:         ## Run database seed
	docker compose exec api npm run seed

test:         ## Run tests inside the container
	docker compose exec api npm test
```

Adjust service names (`api` vs `app`) and commands to match the project's topology and ORM.

---

## Configuration

```yaml
capability: containerization
type: infrastructure
version: 1.1

config:
  orchestration: compose       # compose | kubernetes
  topology: separated          # monolithic | separated — set by charter-init based on stack
  # monolithic: single app service (Next.js, Remix, Nuxt)
  # separated: api + web services (NestJS/Hono/Fastify + Vite)
```
