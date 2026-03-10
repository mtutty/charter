# Payment

## Generation Instructions
Payment handles transaction execution: collecting payment details, processing charges, handling refunds, and managing payment methods. Billing owns plan state; Payment owns money movement. They are coupled: Payment subscribes to Billing events to execute charges, and Billing subscribes to Payment events to update subscription state.

Never collect or store raw card data. Generate payment method collection UI using provider-hosted elements (Stripe Elements or equivalent) so that payment data never touches the application server. The application receives only a payment method token.

Generate payment method management: list saved payment methods (type, last four, expiry), add a new method, remove a method, and set a default method. Generate invoice history view. If `config.invoice-history` is `user-facing`, expose the invoice list to authenticated users. If `admin-only`, expose only in admin surfaces.

Generate refund initiation. If `config.refunds` is `self-serve`, allow users to initiate refunds for eligible transactions from the invoice history view. If `admin-only`, restrict refund initiation to admin surfaces.

Generate a webhook handler for provider events: payment succeeded, payment failed, subscription renewal, and refund events. Process these events to update Payment and Billing state accordingly.

Subscribe to `billing.plan-upgraded` to execute the plan change charge. Subscribe to `billing.subscription-cancelled` to cancel the active subscription with the provider at period end.

capability: payment
type: singleton
version: 1.0

depends-on:
  - auth
  - billing

emits:
  - event: payment.succeeded
    payload:
      userId: string
      amount: number
      currency: string
      invoiceId: string
  - event: payment.failed
    payload:
      userId: string
      amount: number
      invoiceId: string
      reason: string
  - event: payment.refund-issued
    payload:
      userId: string
      amount: number
      invoiceId: string
  - event: payment.method-added
    payload:
      userId: string
      methodId: string
      type: card | ach | sepa
  - event: payment.method-removed
    payload:
      userId: string
      methodId: string

config:
  provider: stripe              # stripe (only supported provider currently)

  supported-payment-methods:
    - card
    # - ach
    # - sepa

  invoice-history: user-facing  # user-facing | admin-only

  refunds: admin-only           # admin-only | self-serve

  admin-surface:
    payment-history: true       # view transaction history for any user
    issue-refund: true          # initiate refunds from admin
    manage-subscriptions: true  # view and cancel active subscriptions
