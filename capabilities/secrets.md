# Secrets / Environment Management

## Generation Instructions

Secrets / Environment Management does two things: it aggregates all environment variable declarations from every capability into a single `.env.example` file, and it generates startup validation that fails fast when required variables are missing.

Generate `.env.example` at the project root. Organize variables by capability, with a comment header for each section and an inline comment on every variable describing its purpose and acceptable values. Variables with no obvious default should have an empty value with a description. Variables with safe defaults should have those defaults populated. Never populate secrets (API keys, database passwords, signing secrets) with real values — use descriptive placeholders like `your-stripe-secret-key`.

Generate a validation module at `lib/env` that reads `process.env` (or language equivalent), validates presence and format of declared variables, and exports typed, validated configuration for use throughout the application. Do not access `process.env` directly anywhere else in the codebase — all environment access goes through this module. This prevents scattered, unchecked env reads and makes the full configuration surface visible in one place.

Classify each variable as `required` or `optional`. Required variables cause the process to exit with a clear error message on startup if absent. Optional variables have documented fallback behavior. The error message for a missing required variable must name the variable, explain what it's for, and tell the developer where to get it.

If `config.provider` is not `env-files`, generate the appropriate integration: AWS Secrets Manager pulls secrets at startup and injects them into the validation module; Vault generates a Vault agent sidecar configuration; Doppler generates a Doppler CLI configuration for local development and a Doppler service token setup for production. The validation module interface is identical regardless of provider — callers do not know or care where values came from.

If `config.validation` is `warn`, log a warning for missing required variables instead of exiting — useful during early development when the full configuration surface isn't yet populated. If `none`, skip validation entirely. `strict` is the correct default for any production-bound application.

Generate a separate `.env.test` example for test environment variables. Test variables should use in-process or local-only backing services where possible to avoid test suites depending on external infrastructure.

## Configuration

```yaml
capability: secrets
type: infrastructure
version: 1.0

# No exposes: block — validation runs at startup and is transparent thereafter.
# All capabilities access configuration through the generated lib/env module.

config:
  provider: env-files          # env-files | aws-secrets-manager | vault | doppler

  validation: strict           # strict (exit on missing required) | warn (log only) | none

  environments:
    - local
    - staging
    - production
```
