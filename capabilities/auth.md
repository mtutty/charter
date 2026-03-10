# Auth

## Generation Instructions
Auth establishes and maintains user identity. Generate registration, login, 
logout, and password reset flows. Generate MFA flows if 
`security.mfa.enforcement` is not `off`. Generate social login flows for 
each enabled provider in `config.providers`.

Do not generate authorization logic — role and permission checks belong to 
the Permissions capability.

Generate admin surfaces for each enabled entry in `admin-surface`.

capability: auth
type: singleton
version: 1.0

depends-on: []

emits:
  - event: auth.registered
    payload:
      userId: string
      email: string
      provider: email-password | google | github | saml
  - event: auth.login
    payload:
      userId: string
      sessionId: string
  - event: auth.logout
    payload:
      userId: string
      sessionId: string
  - event: auth.password-reset-completed
    payload:
      userId: string
  - event: auth.mfa-enabled
    payload:
      userId: string

config:
  providers:
    email-password: true        # always available if any provider is enabled
    google: false
    github: false
    saml: false                 # enterprise SSO; mutually exclusive with others if true

  session:
    mechanism: jwt              # jwt | server-side
    duration: 30d
    sliding: true               # reset expiry on activity

  security:
    mfa:
      enforcement: optional     # off | optional | required
      methods: [ totp ]         # totp | sms
    password-policy: standard   # standard | strict | none
    rate-limiting: true

  registration:
    mode: open                  # open | invite-only | closed
    email-verification: true

  admin-surface:
    user-search: true           # find users by email, id
    suspend-user: true          # disable login without deleting account
    force-password-reset: true
    view-sessions: true         # see active sessions for a user
    revoke-session: true
