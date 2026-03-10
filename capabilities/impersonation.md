# Impersonation

## Generation Instructions
Impersonation allows privileged staff users to assume the identity of another user for support and debugging purposes. Generate an impersonation initiation surface accessible only to users whose role is in `config.allowed-roles`. This surface must not be visible to non-privileged users.

Generate a session-within-session model: when impersonation is initiated, preserve the admin's original session identity and create a new session layer representing the target user. The impersonated session has access to the target user's data and views as if the admin were that user. At any time, the impersonator can end the session and return to their original identity.

Generate a persistent, prominent UI indicator that is visible at all times during an impersonated session — a banner or overlay clearly stating the impersonation is active and who is being impersonated. This indicator must not be dismissible.

If `config.require-reason` is true, present a reason input before initiating impersonation. The reason is stored with the audit log entry.

Generate session restriction enforcement: operations listed in `config.restricted-operations` are blocked during impersonated sessions and return a clear error if attempted. Defaults include password changes, payment initiation, and destructive account operations.

Always emit `impersonation.started` and `impersonation.ended`. If the Audit Log capability is present, these events are subscribed and recorded — this connection is non-optional.

Enforce `config.session-duration-limit`: automatically end impersonated sessions that exceed the limit and return the admin to their own session.

capability: impersonation
type: singleton
version: 1.0

depends-on:
  - auth
  - user-management
  - permissions

emits:
  - event: impersonation.started
    payload:
      actorUserId: string
      targetUserId: string
      reason: string | null     # present if require-reason is true
  - event: impersonation.ended
    payload:
      actorUserId: string
      targetUserId: string
      durationSeconds: number

config:
  allowed-roles:
    - admin                     # roles permitted to initiate impersonation

  require-reason: true          # prompt for a reason before initiating

  restricted-operations:        # operations blocked during impersonation
    - change-password
    - initiate-payment
    - delete-account
    - generate-api-key

  session-duration-limit: 60m  # auto-end impersonated session after this duration
