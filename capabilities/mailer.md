# Mailer

## Generation Instructions
Mailer handles transactional email delivery: sending templated emails triggered by application events. It is distinct from Inbox (in-app delivery to individual users) and Announcements (broadcast to all users). Mailer sends email to individuals in response to events.

Mailer does not contain business logic about when to send. Other capabilities emit events; Mailer subscribes to those events and sends the corresponding email via Background Jobs — each inbound event is enqueued as a delivery job rather than sent synchronously. This ensures delivery survives process restarts and can be retried on provider failure. The set of email types, their templates, and their trigger events are declared in `config.email-types`.

Generate an email template for each type in `config.email-types`. Templates use the declared trigger event's payload for variable substitution. Generate the delivery integration for `config.provider`. All outbound emails use `config.from-address` and `config.from-name`. Set `config.reply-to` on all messages if provided.

Every email must include an unsubscribe link. For types where `opt-outable` is true, honor user unsubscribe state: if the user has unsubscribed from that type, do not send. For types where `opt-outable` is false (transactional critical — e.g., password reset), always send regardless of preferences.

Generate user-facing email preference management if `config.preference-center` is true: a page listing all opt-outable email types with toggle controls. Generate the unsubscribe landing page that handles the link in emails.

Track bounce events from the provider webhook. On hard bounce, mark the user's email address as undeliverable and suppress future sends.

capability: mailer
type: singleton
version: 1.0

depends-on:
  - auth
  - background-jobs

emits:
  - event: mailer.sent
    payload:
      userId: string
      emailType: string
      messageId: string
  - event: mailer.bounced
    payload:
      userId: string
      emailType: string
      bounceType: hard | soft
  - event: mailer.unsubscribed
    payload:
      userId: string
      emailType: string

config:
  provider: postmark            # sendgrid | postmark | ses | smtp
  from-address: hello@example.com
  from-name: Example App
  reply-to: null                # optional reply-to address

  preference-center: true       # generate user-facing email preference management page

  email-types:
    - name: welcome
      label: "Welcome email"
      trigger-event: auth.registered
      opt-outable: false        # critical transactional — always send
    - name: password-reset
      label: "Password reset"
      trigger-event: auth.password-reset-completed
      opt-outable: false
    - name: weekly-digest
      label: "Weekly digest"
      trigger-event: null       # manually triggered
      opt-outable: true
