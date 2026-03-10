# Containerization

## Generation Instructions

Containerization generates Dockerfile, docker-compose configuration, and operational scripts. The outputs are deployment artifacts — this capability does not generate runtime application code.

**Dockerfile**

Generate a multi-stage Dockerfile:
1. `deps` stage: install all dependencies (including devDependencies) into a clean image. This stage is cached aggressively — it only reruns when the package lockfile changes.
2. `build` stage: copy source, run the build (TypeScript compilation, asset bundling). Produces a `dist/` or equivalent output directory.
3. `runtime` stage: start from the minimal base image (node:alpine, python:slim, etc.). Copy only the built artifacts and production dependencies from previous stages. Run as a non-root user — generate a `node` user (or equivalent) and switch to it with `USER`. Expose the application port. Set `CMD` to the application entrypoint.

Generate `.dockerignore` excluding: `node_modules`, `.env*`, `*.test.*`, `coverage/`, `.git/`, and any secrets or credential files.

If Background Jobs declares a separate worker entrypoint, the Dockerfile supports both app and worker from the same image — the entrypoint is specified via the compose service command, not baked into the image.

**Development Compose**

Generate `docker-compose.yml` for local development. Include:
- `app` service: mounts source code as a volume, runs with hot reload (nodemon, ts-node-dev, uvicorn --reload, air, etc.). Exposes application port to host.
- `db` service: PostgreSQL (or the primary database from `stack.database.primary`). Uses a named volume for data persistence across restarts. Exposes database port to host for direct access from database tools.
- `redis` service: include if Caching is present or if Background Jobs uses a redis transport. Uses a named volume. Exposes Redis port to host.
- `worker` service: include if Background Jobs declares a separate worker entrypoint. Runs the worker command against the same codebase volume as the app service. Shares environment with the app service.

**Dev/prod parity constraint**: the development compose uses the same database engine and major version as production. Do not substitute SQLite for PostgreSQL, in-memory cache for Redis, or synchronous job execution for a queue. These substitutions mask real bugs and undermine the value of local development. If a developer does not want to run Redis locally, they should use `transport: database` in Background Jobs rather than using an in-memory substitute.

**Production Compose**

If `config.orchestration: compose`, generate `docker-compose.prod.yml`. Unlike the dev compose:
- No source code volume mounts — uses the built image.
- No exposed database ports — database is accessible only within the compose network.
- Environment variables sourced from `.env.production` or the secrets provider.
- `restart: unless-stopped` on all services.
- App and worker are separate services from the same image with different commands.

If `config.orchestration: kubernetes`, generate Kubernetes manifests instead: `Deployment` for app, `Deployment` for worker (if present), `Service`, `ConfigMap` for non-secret config, and notes on where to inject secrets (Kubernetes Secrets or external secret operator). Do not generate Helm charts unless explicitly requested.

**Operational Scripts**

Generate a `Makefile` (or add to `package.json` scripts) with:
- `make dev` / `dev`: start development compose
- `make build`: build the Docker image
- `make migrate`: run database migrations as a one-off container (uses the same image and `DATABASE_URL`)
- `make seed`: run the development seed script
- `make logs`: tail logs from all compose services
- `make shell`: open a shell in the running app container

## Configuration

```yaml
capability: containerization
type: infrastructure
version: 1.0

# No exposes: block — deployment artifact only, no runtime API.

config:
  orchestration: compose       # compose | kubernetes
  # compose: generates docker-compose.yml (dev) and docker-compose.prod.yml (prod)
  # kubernetes: generates k8s manifests
```
