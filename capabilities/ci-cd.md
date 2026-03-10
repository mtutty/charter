# CI/CD

## Generation Instructions

CI/CD generates pipeline configuration for automated testing, building, and deployment. The outputs are CI configuration files — this capability does not generate runtime application code.

Generate pipeline configuration for `config.provider`. The pipeline runs on every push to any branch, with deployment stages gated to specific branches or requiring manual approval.

**Pipeline Stages**

Generate the following stages in order, each as a separate job that must pass before the next runs:

1. **Lint**: run the project's linter (ESLint, Ruff, golangci-lint, etc.) and formatter check. Fail on any lint errors or formatting violations. This stage has no service dependencies and should be fast.

2. **Type Check**: run the type checker (tsc --noEmit, mypy, etc.). Fail on any type errors. No service dependencies.

3. **Test**: run the test suite. This stage spins up real backing services as CI service containers — PostgreSQL at the same major version as production, Redis if Caching or Background Jobs (redis transport) is present. Do not use in-memory substitutes. Run migrations against the test database before running tests. Set `DATABASE_URL` and other service URLs to the CI service container addresses.

4. **Build**: build the Docker image using the project's Dockerfile. Tag with the commit SHA and `latest` (on main branch). Push to the container registry declared in `config.registry`.

5. **Deploy to Staging**: trigger on merge to the main branch only. Pull the image tagged with the commit SHA. Run database migrations as a one-off task before updating the running containers. Deploy to the staging environment via `config.deploy-target`. Staging deployments are automatic — no approval required.

6. **Deploy to Production**: trigger on merge to main after staging deploy succeeds, but require manual approval. The approval gate prevents automated production deployments while still keeping the path short. Generate a clear approval step in the pipeline UI. Production deploy follows the same migration-then-rollout sequence as staging.

**Migration Safety**

The deploy sequence is always: migrate → rollout. Never rollout → migrate. New code must be compatible with both the old and new schema during the rollout window (zero-downtime deployments require backwards-compatible migrations — note this in the generated pipeline README if `config.deploy-target` implies zero-downtime).

**Secrets**

Pipeline secrets (database URLs, API keys, deploy tokens) are referenced as environment variables from the CI provider's secret store — not committed to the repository. Generate a `CI_SECRETS.md` (not committed) listing every secret the pipeline needs, where to get each one, and which pipeline stage uses it.

**Provider Generation**

Resolve the output format from `config.provider`:
- `github-actions`: generate `.github/workflows/ci.yml`. Use official actions for checkout, Docker buildx, and registry push.
- `gitlab-ci`: generate `.gitlab-ci.yml`. Use GitLab's built-in Docker registry.
- `circleci`: generate `.circleci/config.yml`. Use CircleCI orbs for Docker.
- `none`: generate a `CI_SETUP.md` documenting what a CI pipeline needs to do (the stages above), without generating provider-specific config.

**Deploy Target**

Resolve deployment commands from `config.deploy-target`:
- `fly`: `flyctl deploy --image {registry}/{image}:{sha}` with `flyctl postgres connect` for migration step.
- `railway`: `railway up` with Railway's deploy hooks for migration.
- `aws-ecs`: `aws ecs update-service` with an ECS task definition update. Generate task definition template.
- `kubernetes`: `kubectl set image` for rolling update. Migration runs as a Kubernetes Job before the rollout.
- `ssh`: `ssh deploy@host 'cd /app && docker-compose pull && docker-compose up -d'`. Migration runs via SSH before container restart.

## Configuration

```yaml
capability: ci-cd
type: infrastructure
version: 1.0

# No exposes: block — CI/CD artifact only, no runtime API.

config:
  provider: github-actions     # github-actions | gitlab-ci | circleci | none

  registry: ghcr.io            # ghcr.io | docker.io | ecr | gcr | registry.gitlab.com

  environments:
    - staging                  # auto-deploy on merge to main
    - production               # manual approval required

  deploy-target: fly           # fly | railway | aws-ecs | kubernetes | ssh
```
