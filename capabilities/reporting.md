# Reporting / Analytics

## Generation Instructions
Reporting generates internal analytics surfaces — dashboards, charts, and tabular reports — over the application's own data. This is not integration with external analytics platforms (Mixpanel, Amplitude, etc.). Generate only internal reports over data the application owns.

Generate a report definition model: each report has a data source (entity or event type), an aggregation function (count, sum, average, by-period), a time dimension, and a visualization type (line chart, bar chart, table, stat card). Generate only the report types listed in `config.reports`.

Generate a dashboard layout composed of configurable panels, each bound to a report definition. Allow admin users to add, remove, and reorder panels.

If `config.audience` includes `user-facing`, generate user-facing report views alongside admin reports. User-facing reports are scoped to the authenticated user's own data — they cannot see aggregate data across users unless explicitly designed that way.

If `config.scheduled-delivery.enabled` is true and the Mailer capability is present, generate scheduled report delivery: register a scheduled job via `jobs-client` (`jobs.schedule`) for each configured report. The job generates the CSV export and triggers delivery via Mailer.

Refresh live data on dashboard load. If `config.refresh-interval-seconds` is non-null, also refresh on interval while the dashboard is open.

capability: reporting
type: singleton
version: 1.0

depends-on:
  - auth
  - background-jobs

emits:
  - event: reporting.report-exported
    payload:
      reportId: string
      format: string
      userId: string

config:
  audience:
    - admin                     # admin | user-facing

  refresh-interval-seconds: null  # null = only refresh on load; set number for live refresh

  scheduled-delivery:
    enabled: false              # requires Mailer capability
    schedules:
      - daily
      - weekly
      - monthly

  reports:                      # predefined report types to generate
    - name: user-signups
      data-source: auth.registered
      aggregation: count-by-period
      visualization: line-chart
    - name: active-users
      data-source: auth.login
      aggregation: unique-count-by-period
      visualization: line-chart
