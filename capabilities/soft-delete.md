# Soft Delete + Trash

## Generation Instructions
Soft delete adds a non-destructive deletion layer to each entity listed in `config.attaches-to`. Generate a `deleted_at` timestamp column on each attached entity's database table. All default query scopes must exclude records where `deleted_at` is not null — soft-deleted records should be invisible to normal application queries without explicit opt-in.

Generate a trash view per attached entity: a filtered list showing only soft-deleted records for that entity, accessible to users who have permission to delete records of that type. Generate restore and permanent-delete operations on each trash view. Permanent delete removes the database row; it is irreversible and must require confirmation.

If `config.retention.auto-purge-enabled` is true, generate a scheduled job that permanently deletes records where `deleted_at` is older than `config.retention.auto-purge-after-days` days. Log purge activity.

Generate admin surfaces for each enabled entry in `config.admin-surface`. The bulk-empty-trash operation permanently deletes all soft-deleted records across all attached entities and must require confirmation before executing.

Do not generate UI for soft-deleted records in normal entity list views — they should be entirely absent. Access to the trash view follows the same permission rules as the entity itself.

capability: soft-delete
type: attachable
version: 1.0

depends-on: []

emits:
  - event: soft-delete.record-deleted
    payload:
      entityType: string
      entityId: string
      userId: string
  - event: soft-delete.record-restored
    payload:
      entityType: string
      entityId: string
      userId: string
  - event: soft-delete.record-purged
    payload:
      entityType: string
      entityId: string
      purgedBy: user | system   # user = manual permanent delete, system = auto-purge job

config:
  attaches-to:
    - entity: example-entity    # replace with actual entity names

  retention:
    auto-purge-enabled: false
    auto-purge-after-days: 30   # only relevant if auto-purge-enabled is true

  admin-surface:
    bulk-empty-trash: true      # permanently delete all trashed records across entities
    cross-entity-trash-view: true  # view all deleted records across entities in one place
