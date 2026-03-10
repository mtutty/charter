# Billing

## Generation Instructions
Billing owns plan state: what plan a user or organization is on, whether they are in a trial, and what features their plan includes. Billing does not execute payment transactions — that belongs to the Payment capability. Billing owns the subscription state machine; Payment owns the money movement. These capabilities are coupled but distinct.

Generate a plan definition model from `config.plans`. Each plan has a name, a list of included features, a trial eligibility flag, and optional pricing fields. Generate a subscription state model per `config.subscription-entity` (user or org): current plan, trial state, trial expiry, billing period, and subscription status (trialing, active, past-due, cancelled).

Generate upgrade and downgrade UI: plan comparison table showing features by plan, upgrade/downgrade action with plan change timing per `config.plan-change-timing` (immediate takes effect now; end-of-period schedules the change at the next billing renewal).

Generate trial management: start a trial on registration if the plan is trial-eligible, detect trial expiry via a scheduled job registered with the Background Jobs infrastructure capability (`jobs.schedule`), display trial countdown and conversion prompts within the application. Generate a trial-to-paid conversion flow that connects to the Payment capability.

Generate feature gate helpers available throughout the application: `onPlan(user, planName)` and `hasFeature(user, featureName)`. These helpers allow other capabilities and application code to conditionally enable features based on the user's current plan.

Subscribe to `payment.failed` to set subscription status to `past-due`. Subscribe to `payment.succeeded` to return status to `active`.

capability: billing
type: singleton
version: 1.0

depends-on:
  - auth
  - user-management
  - background-jobs

emits:
  - event: billing.trial-started
    payload:
      userId: string
      planName: string
      trialExpiresAt: string    # ISO 8601 datetime
  - event: billing.trial-expired
    payload:
      userId: string
      planName: string
  - event: billing.plan-upgraded
    payload:
      userId: string
      fromPlan: string
      toPlan: string
  - event: billing.plan-downgraded
    payload:
      userId: string
      fromPlan: string
      toPlan: string
  - event: billing.subscription-cancelled
    payload:
      userId: string
      planName: string
  - event: billing.subscription-reactivated
    payload:
      userId: string
      planName: string

config:
  subscription-entity: user     # user | org — which entity holds the subscription

  billing-interval:
    - monthly
    - annual                    # remove if annual billing not offered

  trial-duration-days: 14

  plan-change-timing: immediate # immediate | end-of-period

  plans:
    - name: free
      features:
        - basic-access
      trial-eligible: false
      price-monthly: 0
      price-annual: 0
    - name: pro
      features:
        - basic-access
        - advanced-features
        - api-access
      trial-eligible: true
      price-monthly: 29
      price-annual: 290
    - name: enterprise
      features:
        - basic-access
        - advanced-features
        - api-access
        - sso
        - custom-roles
      trial-eligible: false
      price-monthly: null       # custom pricing — contact sales
      price-annual: null

  admin-surface:
    subscription-viewer: true   # view and manage any user's subscription
    override-plan: true         # manually set a user's plan (for comped accounts)
    trial-extension: true       # extend a user's trial period
