# Versioning / History

## Generation Instructions
Versioning maintains an immutable history of changes to entity records declared in `config.attaches-to`. On each update to an attached entity, capture a version snapshot before the change is applied and store it as a version record attributed to the user making the change. Version records are append-only — never modify or delete them except to enforce retention policy.

Each version record contains: version id, entity type, entity id, the snapshot data (either the full record or the declared field subset), userId of the actor, and created_at timestamp.

If `config.attaches-to[entity].versioned-fields` is specified, snapshot only those fields. If omitted, snapshot the entire record.

Generate a version history UI per attached entity: a timeline of versions showing actor, timestamp, and a diff view comparing each version to the previous one. Generate a restore-to-version operation. If `config.restore-creates-new-version` is true, restoring creates a new version record containing the restored state rather than overwriting the current record — this preserves full history. If false, the restore overwrites the current record in place.

Enforce `config.attaches-to[entity].retention.max-versions` if set: after each new version is created, delete the oldest version(s) exceeding the limit. If `max-versions` is null, retain all versions indefinitely.

capability: versioning
type: attachable
version: 1.0

depends-on:
  - auth

emits:
  - event: versioning.version-created
    payload:
      entityType: string
      entityId: string
      versionId: string
      userId: string
  - event: versioning.version-restored
    payload:
      entityType: string
      entityId: string
      restoredVersionId: string
      userId: string

config:
  restore-creates-new-version: true   # if false, restore overwrites current record

  attaches-to:
    - entity: example-entity
      versioned-fields: null          # null = version all fields; list field names to restrict
      retention:
        max-versions: null            # null = retain all; set a number to cap per record
