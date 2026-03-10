# Referral / Affiliate

## Generation Instructions
Referral enables a user referral program: unique referral links per user, tracking of referred signups, and reward issuance on qualifying conversion. Generate referral link generation for each authenticated user — one unique link per user, persistent across sessions.

Generate cookie-based attribution: when a visitor arrives via a referral link, store the referrer's userId in a cookie. On registration, read the attribution cookie and associate the new user with the referrer. Subscribe to `auth.registered` and check for attribution cookie presence to detect referred signups.

Generate reward trigger logic: subscribe to the event declared in `config.qualifying-event`. When the event fires for a referred user (a user who was attributed to a referrer on registration), issue the reward to the referrer. If `config.referee-reward` is non-null, also issue a reward to the referred user.

Generate reward issuance for `config.reward-type`. For `account-credit`, create a credit record on the referrer's account that is applied to the next invoice (connects to Payment and Billing). For `plan-extension`, extend the referrer's current billing period. For `cash`, record a cash reward pending payout — generate an admin surface for processing payouts.

Generate a user-facing referral dashboard: the user's referral link with copy-to-clipboard, referral statistics (total referrals, converted, pending, total rewards earned), and reward status.

If `config.link-expiry` is non-null, expire referral links after the declared duration and generate new ones automatically.

capability: referral
type: singleton
version: 1.0

depends-on:
  - auth
  - billing

emits:
  - event: referral.link-created
    payload:
      userId: string
      linkId: string
  - event: referral.user-referred
    payload:
      referrerId: string
      referredUserId: string
  - event: referral.converted
    payload:
      referrerId: string
      referredUserId: string
      qualifyingEvent: string
  - event: referral.reward-issued
    payload:
      userId: string
      rewardType: account-credit | plan-extension | cash
      amount: number
      currency: string | null   # null for non-monetary rewards

config:
  reward-type: account-credit   # account-credit | plan-extension | cash

  qualifying-event: first-payment  # registration | first-payment | plan-upgrade
                                   # maps to: auth.registered | payment.succeeded | billing.plan-upgraded

  reward-amount: 10             # monetary amount or days (for plan-extension)
  reward-currency: USD          # only relevant for account-credit and cash

  referee-reward:               # reward for the referred user; null to disable
    type: account-credit
    amount: 5
    currency: USD

  link-expiry: null             # null for permanent links; set duration (e.g. 90d) to expire

  admin-surface:
    referral-overview: true     # total referrals, conversions, rewards issued
    reward-management: true     # view and process pending rewards (required for cash type)
