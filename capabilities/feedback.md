# Feedback Collection

## Generation Instructions
Feedback Collection generates in-product mechanisms for gathering structured user feedback. Generate only the feedback types listed in `config.types`. Each enabled type has its own survey/prompt UI, trigger logic, and storage model.

NPS (Net Promoter Score): 0–10 rating scale with an optional follow-up text field. CSAT (Customer Satisfaction): 1–5 rating with optional text. Feature request: structured form with title, description, and category. Bug report: structured form with title, description, severity, and optional reproduction steps.

For each type, generate trigger logic according to `config.types[name].trigger`: `on-event` subscribes to the named application event and shows the prompt after it fires; `scheduled` shows the prompt after a configured interval since last response; `manual` exposes an API the application can call to trigger the prompt programmatically.

If `config.attribution` is `attributed`, link each response to the authenticated user. If `anonymous`, store responses without user linkage.

Generate admin surfaces for each enabled entry in `config.admin-surface`. The response viewer must support filtering by type, date range, and rating value. Export produces CSV.

capability: feedback
type: singleton
version: 1.0

depends-on: []

emits:
  - event: feedback.submitted
    payload:
      type: nps | csat | feature-request | bug-report
      userId: string | null     # null if attribution is anonymous
      rating: number | null     # present for nps and csat types
      responseId: string

config:
  attribution: attributed       # attributed | anonymous

  types:
    nps:
      enabled: true
      trigger: scheduled        # on-event | scheduled | manual
      trigger-interval-days: 90  # days between prompts per user (scheduled only)
      trigger-event: null       # event name (on-event only)
    csat:
      enabled: false
      trigger: on-event
      trigger-event: null
    feature-request:
      enabled: true
      trigger: manual
    bug-report:
      enabled: true
      trigger: manual

  admin-surface:
    response-viewer: true       # filterable table of responses with ratings and text
    export: true                # export responses to CSV
