# Waitlist

## Generation Instructions
Waitlist captures email addresses from users who want access before the product is available or before a specific feature is accessible. Generate a waitlist signup form — email input with optional name field — and a submission confirmation page or inline confirmation state.

If `config.show-position` is true, display the user's current queue position after signup and in any follow-up communications.

Generate an invite flow: admin selects waitlist entries to invite, the system sends an invitation email with a registration link (connects to Mailer if present, otherwise generates a direct email send). Accepting the invitation converts the waitlist entry to a full registration and triggers `auth.registered` if Auth is present.

Generate admin surfaces for each enabled entry in `config.admin-surface`. The admin view must show each entry's email, join date, position, and invitation status.

If `config.referral-bumps-position` is true and the Referral capability is present, subscribe to `referral.user-referred` and move the referring user's waitlist position forward by one place per successful referral.

capability: waitlist
type: singleton
version: 1.0

depends-on: []

emits:
  - event: waitlist.joined
    payload:
      email: string
      name: string | null
      position: number
  - event: waitlist.invited
    payload:
      email: string
      invitedBy: string         # admin userId
  - event: waitlist.invitation-accepted
    payload:
      email: string
      userId: string            # userId created on acceptance

config:
  scope: product                # product | feature — what access the waitlist gates

  show-position: true           # display queue position to the user after signup
  referral-bumps-position: false  # move position forward when user refers others (requires Referral)

  admin-surface:
    waitlist-viewer: true       # paginated list with email, join date, position, status
    invite-selected: true       # invite one or more entries at once
    bulk-invite: true           # invite all entries above a position threshold
    export: true                # export waitlist to CSV
