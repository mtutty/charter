# Sharing / Access Control

## Generation Instructions
Sharing provides per-resource access grants: the ability to share specific entity records with specific users, beyond the role-based rules defined in the Permissions capability. Sharing extends Permissions to the resource level — a share grants a specific user access to a specific record regardless of their global role.

Generate a share dialog UI on each entity declared in `config.attaches-to`: a user search input to find the person to share with, an access level selector from `config.access-levels`, and a confirm action. Generate a share management view on each entity showing all current grants (user, access level, who granted it, when), with the ability to revoke individual grants.

If `config.link-sharing.enabled` is true, generate link-based sharing: create a URL that grants access to the record without requiring the recipient to be an existing user. If `config.link-sharing.expiry` is non-null, the link expires after the declared duration. If `config.public-sharing` is true, unauthenticated users can access the record via a share link.

When a share is granted, emit `sharing.resource-shared`. If the Inbox or Mailer capability is present, they subscribe to this event and notify the recipient.

Generate permission check integration: when evaluating access to a shared entity, check both the user's role (via Permissions) and any active share grants. Access is granted if either check passes.

capability: sharing
type: attachable
version: 1.0

depends-on:
  - auth
  - permissions

emits:
  - event: sharing.resource-shared
    payload:
      entityType: string
      entityId: string
      sharedWithUserId: string
      accessLevel: string
      sharedByUserId: string
  - event: sharing.share-revoked
    payload:
      entityType: string
      entityId: string
      userId: string
      revokedByUserId: string
  - event: sharing.link-created
    payload:
      entityType: string
      entityId: string
      linkId: string
      accessLevel: string
      expiresAt: string | null

config:
  attaches-to:
    - entity: example-entity    # replace with actual entity names

  access-levels:
    - view
    - edit
    # - comment               # add if Comments capability is present

  link-sharing:
    enabled: false              # generate shareable link for a record
    expiry: null                # duration string (e.g. 7d, 30d) or null for no expiry

  public-sharing: false         # allow unauthenticated access via share link
