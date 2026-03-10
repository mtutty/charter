# Presence

## Generation Instructions
Presence tracks which users are currently viewing or active in the same entity context (document, record, page) and makes that awareness visible to all active viewers. Real-time delivery is provided by the Real-time Transport infrastructure capability — use `realtime-server.publish()` to broadcast presence events and `realtime-client` to subscribe on the frontend. Do not generate a separate WebSocket server.

For each entity declared in `config.attaches-to`, generate a presence tracking model: when a user loads an entity detail view, register their presence for that entity context. Maintain presence by sending a heartbeat from the client every `config.heartbeat-interval-seconds` seconds. If no heartbeat is received for `config.idle-timeout-seconds`, remove the user from the presence list for that context and emit `presence.user-absent`.

Generate a presence display UI component showing avatars of currently active users (up to `config.max-users-displayed`). If more users are active than the display cap, show a count of the overflow.

If `config.cursor-sharing` is true, broadcast cursor position updates from each active user in real time and render other users' cursor positions on the shared view. Cursor sharing is significantly more complex than presence tracking — only generate it when explicitly enabled.

Presence data is ephemeral — it is not persisted to the database. Store current presence state in the cache using `cache-client` with the key convention `presence:{entityType}:{entityId}`. TTL should be slightly longer than `config.idle-timeout-seconds` so expired presence entries are cleaned up automatically.

capability: presence
type: attachable
version: 1.0

depends-on:
  - auth
  - user-management
  - real-time-transport
  - caching

emits:
  - event: presence.user-present
    payload:
      entityType: string
      entityId: string
      userId: string
  - event: presence.user-absent
    payload:
      entityType: string
      entityId: string
      userId: string

config:
  attaches-to:
    - entity: example-entity    # replace with actual entity names

  heartbeat-interval-seconds: 15
  idle-timeout-seconds: 30      # seconds after last heartbeat before user is removed

  max-users-displayed: 5        # cap on simultaneous avatars shown in the UI

  cursor-sharing: false         # broadcast cursor position — significantly more complex
