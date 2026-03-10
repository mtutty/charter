# Onboarding

## Generation Instructions
Onboarding defines a structured flow that new users complete before reaching full product access. Subscribe to `auth.registered` — onboarding begins automatically when a new user registers. On receipt of `auth.registered`, create an onboarding record for the user with all steps initialized to incomplete.

Generate a step-by-step onboarding UI that renders each step declared in `config.steps` in order. Display a progress indicator showing completed and remaining steps. Each step has a name, a completion condition, and an optional required flag.

For each step, generate the UI surface and completion detection logic based on the step's declared completion-condition. A step is marked complete when its condition is met. Required steps must be completed before the user can proceed past the onboarding flow; optional steps can be skipped.

If `config.access-gating` is true, block access to the main application until all required steps are complete. Present the onboarding flow as a gate — the user cannot navigate away to the rest of the app. If false, allow the user to access the app while onboarding is in progress and surface onboarding as a persistent prompt or sidebar widget.

If `config.retriggerable` is true, generate an admin action to reset onboarding for a user, returning all steps to incomplete and re-initiating the flow on next login.

capability: onboarding
type: singleton
version: 1.0

depends-on:
  - auth

emits:
  - event: onboarding.started
    payload:
      userId: string
  - event: onboarding.step-completed
    payload:
      userId: string
      step: string
  - event: onboarding.completed
    payload:
      userId: string

config:
  access-gating: true           # block app access until required steps are complete

  retriggerable: false          # allow admin to reset onboarding for a user

  steps:
    - name: profile-setup
      required: true
      completion-condition: user-management.profile-updated  # event that marks step complete
    - name: verify-email
      required: true
      completion-condition: auth.email-verified

  admin-surface:
    view-completion-status: true  # see which users have completed onboarding
    retrigger: true               # reset onboarding for a specific user
