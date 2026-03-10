# In-App Messaging

## Generation Instructions
In-App Messaging enables direct and group conversations between users within the application. It is distinct from Inbox (system-generated notifications directed at a specific user) and Announcements (broadcast from the application to users). Messaging is user-to-user communication: a human sending a message to another human or a group.

Real-time message delivery is provided by the Real-time Transport infrastructure capability — use `realtime-server.publish()` to push new messages to conversation participants and `realtime-client` to subscribe on the frontend. Do not generate a separate WebSocket server.

Generate a conversation list UI: all conversations the authenticated user is a participant of, ordered by most recent activity, with unread message count badges. Generate a message thread UI: the full message history for a selected conversation with pagination (load older messages on scroll), a message input, and send action.

If `config.group-conversations` is true, generate the ability to create group conversations with multiple participants and add/remove participants from existing conversations.

If `config.read-receipts` is true, track when each participant reads each message and display read state in the thread UI.

If `config.file-attachments` is true and the File Management capability is present, allow file attachment to messages using the File Management upload flow.

If `config.message-search` is true, generate a search interface scoped to the authenticated user's conversation history.

If `config.typing-indicators` is true, broadcast typing state via WebSocket and display "user is typing…" in the thread UI.

Retain message history per `config.history-retention`. If `null`, retain indefinitely. Delete messages older than the retention period via a scheduled job.

capability: messaging
type: singleton
version: 1.0

depends-on:
  - auth
  - user-management
  - real-time-transport

emits:
  - event: messaging.message-sent
    payload:
      conversationId: string
      senderId: string
      messageId: string
  - event: messaging.conversation-created
    payload:
      conversationId: string
      createdBy: string
      participantIds: string[]
  - event: messaging.participant-added
    payload:
      conversationId: string
      userId: string
      addedBy: string

config:
  group-conversations: false    # enable conversations with more than two participants
  read-receipts: true
  file-attachments: false       # requires File Management capability
  message-search: false
  typing-indicators: true

  history-retention: null       # null = indefinite; set a duration (e.g. 365d) to limit
