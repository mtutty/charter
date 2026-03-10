# Import / Export

## Generation Instructions
Import/Export handles bulk data movement into and out of the application for entities declared in `config.attaches-to`. Generate import and export surfaces independently — an entity may have import enabled, export enabled, or both.

Generate import UI if import is enabled for an entity: file upload (accepting `config.import.formats`), field mapping interface where the user maps uploaded columns to entity fields, a validation preview step that shows errors (invalid values, missing required fields) before any data is committed, a confirmation step, and a results view after completion showing success count and per-row errors. For files exceeding `config.async-threshold-rows`, enqueue the import as a job via `jobs-client` and show a progress indicator. Notify the user via Inbox or Mailer when the job completes if those capabilities are present.

Generate export UI if export is enabled for an entity: format selection from `config.export.formats`, optional filter and date-range controls, and a download trigger. For exports exceeding `config.async-threshold-rows`, enqueue an export job via `jobs-client` that emails the download link when ready.

Respect `config.import.who-can-import` and `config.export.who-can-export` access controls: `admin` restricts to admin users, `any-user` allows all authenticated users.

Generate a partial import failure model: if a row fails validation or insertion, skip that row and continue processing remaining rows. Report the failed rows with error messages in the results view.

capability: import-export
type: attachable
version: 1.0

depends-on:
  - auth
  - background-jobs

emits:
  - event: import-export.import-completed
    payload:
      entityType: string
      userId: string
      successCount: number
      errorCount: number
  - event: import-export.export-completed
    payload:
      entityType: string
      userId: string
      format: string
      rowCount: number

config:
  async-threshold-rows: 1000    # row count above which processing is async

  attaches-to:
    - entity: example-entity
      import:
        enabled: true
        formats:
          - csv
          - excel
        who-can-import: admin   # admin | any-user
      export:
        enabled: true
        formats:
          - csv
          - excel
          - json
        who-can-export: any-user  # admin | any-user
