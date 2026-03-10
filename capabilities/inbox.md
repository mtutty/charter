# Inbox

## Generation Instructions
Inbox is the in-application notification surface directed at individual users. It is distinct from Announcements (broadcast from staff to all users) and Activity Feed (shared record of system activity). Inbox delivers messages that the system addresses specifically to one user.

Inbox does not contain business logic about when to notify. Other capabilities direct notifications to users by emitting events with a target userId and a notification type. Inbox subscribes to those events, receives the payload, and delivers a notification to the target user. The set of notification types Inbox accepts is declared in `config.notification-types`.

Generate a notification bell component with an unread count badge. Generate a notification feed UI showing the user's notifications in reverse chronological order with read/unread state. Generate mark-read (single) and mark-all-read operations.

Generate per-type notification preference controls: for each type where `opt-outable` is true, allow the user to disable that notification type. Notifications of a disabled type are not delivered to that user.

If `config.delivery.realtime` is `websocket`, establish a WebSocket connection on the authenticated session and push notifications in real time. If `polling`, poll the notifications endpoint at `config.delivery.polling-interval-seconds` intervals.

Notifications older than `config.retention-days` are automatically deleted.

capability: inbox
type: singleton
version: 1.0

depends-on:
  - auth

emits:
  - event: inbox.notification-delivered
    payload:
      userId: string
      notificationId: string
      type: string
  - event: inbox.notification-read
    payload:
      userId: string
      notificationId: string

config:
  notification-types:           # registry of notification types this application sends
    - name: comment-mention
      label: "Someone mentioned you in a comment"
      opt-outable: true
    - name: share-received
      label: "Someone shared a resource with you"
      opt-outable: true

  delivery:
    realtime: websocket         # websocket | polling
    polling-interval-seconds: 30  # only used if realtime is polling

  retention-days: 90            # notifications older than this are deleted
