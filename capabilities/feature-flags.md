# Feature Flags

## Generation Instructions
Feature Flags provides runtime control over feature availability without code deployment. Generate flag evaluation logic and a flag management admin UI.

Generate flag evaluation as a function `isEnabled(flagKey, user)` available on both server and client according to `config.evaluation-mode`. On the server, evaluate flags synchronously against flag state in the database. On the client, provide flag state as part of the authenticated session payload so client components can evaluate without an extra round trip.

For each targeting strategy in `config.targeting-strategies`: boolean evaluates to on or off for all users; user-list returns true only for users in the flag's declared user list; percentage-rollout uses a deterministic hash of (flagKey + userId) to assign users consistently to a percentage bucket; attribute-rule evaluates a condition against a user attribute (plan, role, or custom property).

Generate the flag management admin UI: create, enable/disable, and configure flags. For each flag, show the current targeting configuration and allow editing. Display the evaluated state for any given user (useful for debugging).

For any flag key not found in flag storage, return `config.default-state`.

If `config.emit-evaluations` is true, emit `feature-flags.flag-evaluated` on every evaluation. This is disabled by default because evaluation volume can be extremely high — only enable when explicitly configured.

capability: feature-flags
type: singleton
version: 1.0

depends-on:
  - auth

emits:
  - event: feature-flags.flag-changed
    payload:
      flagKey: string
      previousState: boolean | string
      newState: boolean | string
      changedBy: string         # admin userId
  - event: feature-flags.flag-evaluated
    payload:
      flagKey: string
      userId: string
      result: boolean
    # high volume — only emitted if emit-evaluations is true

config:
  targeting-strategies:
    - boolean
    - user-list
    - percentage-rollout
    - attribute-rule

  evaluation-mode: both         # server-side | client-side | both

  default-state: off            # state returned for any flag not in storage (off | on)

  emit-evaluations: false       # emit flag-evaluated event on every evaluation
