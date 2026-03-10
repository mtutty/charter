# Announcements

## Generation Instructions
Announcements delivers broadcast communications from the application to all users or targeted user segments. It is distinct from Inbox (directed at individual users by system events) and Mailer (transactional email triggered by events). An announcement originates from staff and is pushed to users, not triggered by user actions.

Generate an admin-only announcement creation UI: title, body (rich text), display format selection, optional publish datetime for scheduled announcements, and optional audience targeting if `config.targeting-enabled` is true.

Generate the user-facing display surface for each format enabled in `config.display-formats`. Banner: a persistent or dismissible strip at the top of the application layout. Modal: an overlay shown on next page load after publish. Notification-center: an entry in the notification center feed (requires Inbox if generating this format). Render only the formats declared in `config.display-formats`.

If `config.track-dismissal` is true, record per-user dismiss state and do not show an already-dismissed announcement again. If false, re-show the announcement on each page load until its expiry.

If `config.targeting-enabled` is true, generate audience targeting controls on the creation form. Targeting matches users by attribute (plan, role, or custom property). Only matching users see the announcement.

capability: announcements
type: singleton
version: 1.0

depends-on: []

emits:
  - event: announcements.published
    payload:
      announcementId: string
      title: string
      targetedSegment: string | null
  - event: announcements.dismissed
    payload:
      announcementId: string
      userId: string

config:
  display-formats:
    - banner                    # persistent/dismissible strip in app layout
    - notification-center       # entry in notification center (requires Inbox)

  track-dismissal: true         # record per-user dismiss state; re-show if false
  targeting-enabled: false      # allow targeting announcements to user segments

  admin-surface:
    create-announcement: true
    schedule-announcement: true   # set future publish datetime
    announcement-list: true       # manage, edit, and unpublish announcements
    dismissal-stats: true         # view dismissal/read rates per announcement
