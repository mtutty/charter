# API Access

## Generation Instructions
API Access enables programmatic access to application functionality via long-lived credentials. It is distinct from Auth, which handles interactive user session authentication. Auth establishes who a person is; API Access establishes which machine or integration is acting, and what it is permitted to do.

Generate an API key creation and management UI for authenticated users: create a named key, copy the key value (shown only once at creation — never again), rotate a key (invalidate old, issue new), and revoke a key. Store only the hashed key value; never store or display the plaintext after creation.

Generate authentication middleware that validates API keys on inbound requests. If `config.mechanism` includes `api-key`, accept keys in the `Authorization: Bearer` header. If `config.mechanism` includes `oauth2`, generate the OAuth2 client credentials flow in addition.

If `config.scopes` is non-empty, each key is issued with a declared scope subset. Generate scope enforcement middleware that rejects requests where the required scope is not present on the key. Generate the scope selection UI in the key creation flow.

If `config.rate-limiting-per-key` is non-null, enforce the declared requests-per-minute limit per key. Return 429 with a `Retry-After` header on limit exceeded.

If `config.developer-portal` is true, generate a documentation surface listing available API endpoints, authentication instructions, and scope descriptions.

capability: api-access
type: singleton
version: 1.0

depends-on:
  - auth

emits:
  - event: api-access.key-created
    payload:
      userId: string
      keyId: string
      name: string
      scopes: string[]
  - event: api-access.key-revoked
    payload:
      userId: string
      keyId: string
  - event: api-access.rate-limit-exceeded
    payload:
      keyId: string
      endpoint: string

config:
  mechanism:
    - api-key                   # api-key | oauth2

  scopes:                       # available permission scopes; empty means no scope enforcement
    - read
    - write
    - admin

  rate-limiting-per-key:
    requests-per-minute: 60     # null to disable per-key rate limiting

  developer-portal: false       # generate API documentation surface
