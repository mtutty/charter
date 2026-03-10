# Archiving

## Generation Instructions
Archiving is semantically distinct from soft delete: archived records are intentionally preserved for long-term reference and removed from active workflows, but they are not pending deletion. Do not conflate archiving with trash. An archived record may never be deleted. A soft-deleted record is expected to eventually be permanently removed.

Generate an `archived_at` timestamp column and an `archived_by` user reference column on each entity listed in `config.attaches-to`. All default query scopes must exclude archived records — archived records are invisible to normal application queries and workflows without explicit opt-in.

Generate archive and unarchive operations on each attached entity. If `config.reversible` is false, generate only the archive operation and omit unarchive. Generate an archived records view per entity showing only archived records, with attribution (who archived it and when).

If `config.exclude-from-search` is true and the Search capability is present, exclude archived records from search index updates and search results.

Generate admin surfaces for each enabled entry in `config.admin-surface`.

capability: archiving
type: attachable
version: 1.0

depends-on: []

emits:
  - event: archiving.record-archived
    payload:
      entityType: string
      entityId: string
      userId: string
  - event: archiving.record-unarchived
    payload:
      entityType: string
      entityId: string
      userId: string

config:
  attaches-to:
    - entity: example-entity    # replace with actual entity names

  reversible: true              # if false, archived records cannot be unarchived
  exclude-from-search: true     # exclude archived records from search results if Search is present

  admin-surface:
    cross-entity-archive-view: true   # view all archived records across entities
