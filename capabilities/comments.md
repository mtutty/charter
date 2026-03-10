# Comments / Annotations

## Generation Instructions
Comments provides threaded commenting on entity records declared in `config.attaches-to`. Generate a comment thread UI per attached entity that renders inline on the entity detail view.

Generate comment submission with attribution to the authenticated user. Generate edit and delete operations for a user's own comments. If `config.deletion` is `moderators` or `admins`, also generate a moderation delete action visible to users with the appropriate role.

If `config.threading-enabled` is true, generate one level of reply threading: comments can have replies, but replies cannot have replies. Render as an indented sub-list beneath the parent comment.

If `config.reactions-enabled` is true, generate emoji reaction support on each comment: users can add and remove reactions, grouped counts displayed per reaction type.

If `config.mentions-enabled` is true, generate @mention autocomplete in the comment input. When a comment containing a mention is posted, emit `comments.mention` for each mentioned user. If the Inbox capability is present, it subscribes to this event and delivers a notification to the mentioned user.

If `config.edit-history-tracked` is true, record each edit and expose an edit history view per comment showing the previous versions.

capability: comments
type: attachable
version: 1.0

depends-on:
  - auth
  - user-management

emits:
  - event: comments.posted
    payload:
      entityType: string
      entityId: string
      commentId: string
      userId: string
  - event: comments.edited
    payload:
      commentId: string
      userId: string
  - event: comments.deleted
    payload:
      commentId: string
      userId: string
  - event: comments.mention
    payload:
      mentionedUserId: string
      commentId: string
      entityType: string
      entityId: string
    # only emitted if mentions-enabled is true

config:
  attaches-to:
    - entity: example-entity    # replace with actual entity names

  threading-enabled: true       # enable one level of reply threading

  reactions-enabled: false      # enable emoji reactions on comments

  mentions-enabled: false       # enable @mention autocomplete and notifications

  edit-history-tracked: false   # record and display edit history per comment

  deletion: own-only            # own-only | moderators | admins
