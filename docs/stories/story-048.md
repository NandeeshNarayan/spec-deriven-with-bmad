# story-048 — Subscription (Stripe)

| Field | Value |
|---|---|
| **Story ID** | story-048 |
| **Title** | Subscription (Stripe) |
| **Epic** | E13 — Infrastructure |
| **Phase** | All |
| **Sprint** | Sprint 6 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | All personas |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-002 |

---

## Problem Statement

LIPILY is a SaaS product with tiered pricing. The subscription system must gate features by plan, handle upgrades/downgrades cleanly, process payments reliably via Stripe, and handle edge cases like failed payments and cancellations without data loss. It is the revenue backbone of the product.

---

## User Story

As a user, I want to upgrade my plan to Pro or Studio, manage my subscription from within the app, and have my feature access updated immediately — so that I can unlock professional features without friction.

---

## Acceptance Criteria

**GIVEN** a user on the Free plan clicks any paywall CTA ("Upgrade to Pro" or "Upgrade to Studio"),  
**WHEN** the CTA is triggered,  
**THEN** a Pricing Modal opens showing the Pro and Studio plans side-by-side with: pricing ($12/mo Pro, $39/mo Studio), annual discount option (20% off annual = $115.20/yr and $374.40/yr respectively), and a feature comparison table matching the constitution §0.

**GIVEN** the user selects a plan and clicks "Upgrade",  
**WHEN** Stripe Checkout opens,  
**THEN** the Stripe Checkout Session is created server-side via `POST /subscriptions/checkout` with the correct `price_id` for the chosen plan; the session has `allow_promotion_codes: true`; the Checkout page opens in the system browser (for native) or embedded (for web).

**GIVEN** the Stripe Checkout is completed successfully,  
**WHEN** the `checkout.session.completed` webhook fires,  
**THEN** the Rust webhook handler inserts or updates the `subscriptions` row: `status = 'active'`, `plan = 'pro'` (or `'studio'`), `stripe_subscription_id`, `current_period_end`; the feature tier is immediately reflected in the app (Riverpod `subscriptionProvider` refreshes from the server).

**GIVEN** a user's subscription payment fails,  
**WHEN** the `invoice.payment_failed` webhook fires,  
**THEN** `subscriptions.status = 'past_due'`; the user is shown a persistent in-app banner: "Your payment failed. Update your payment method to keep Pro access."; they retain feature access for a 7-day grace period; after 7 days with no payment: `status = 'canceled'` and tier reverts to Free.

**GIVEN** a user wants to manage their subscription (upgrade, downgrade, cancel),  
**WHEN** they navigate to Settings → Subscription,  
**THEN** they see their current plan, next billing date, and amount; a "Manage Subscription" button opens the Stripe Customer Portal (billing.stripe.com) for self-service management.

**GIVEN** a Pro user upgrades to Studio mid-cycle,  
**WHEN** the upgrade is processed via Stripe,  
**THEN** Stripe prorates the remaining days; the `subscriptions.plan` updates to `'studio'` immediately on the `customer.subscription.updated` webhook; Writers' Room and Storyboard features unlock within 10 seconds.

**GIVEN** a Studio user downgrades to Pro,  
**WHEN** the downgrade is processed,  
**THEN** Studio features remain active until the end of the current billing period (`current_period_end`); after that, `plan` updates to `'pro'`; if the user has more than 2 editor collaborators (the Pro limit), existing collaborators are preserved but no new ones can be added beyond the limit.

**GIVEN** a user cancels their subscription,  
**WHEN** the cancellation is processed,  
**THEN** access is maintained until `current_period_end`; after that, `plan = 'free'`; scripts beyond the Free limit (3) become read-only (not deleted); the user is shown a retention offer at cancellation: 1-month free extension offer via Stripe coupon.

**GIVEN** the app loads for any user,  
**WHEN** the auth state is confirmed,  
**THEN** the subscription tier is fetched and cached in Riverpod; all feature gate checks use `ref.watch(subscriptionProvider).plan`; tier checks NEVER happen client-side only — they are always re-validated server-side for any protected action.

---

## Tier Feature Gate Table

| Feature | Free | Pro | Studio |
|---|---|---|---|
| Scripts | 3 | Unlimited | Unlimited |
| AI requests/day | 50 | 500 | Unlimited |
| Collaborators/script | 0 | 2 | 20 |
| Writers' Room | ✗ | ✗ | ✓ |
| Storyboard | ✗ | ✗ | ✓ |
| Revision Mode | ✗ | ✗ | ✓ |
| Branch Editing | ✗ | ✗ | ✓ |
| Export FDX | ✗ | ✓ | ✓ |
| Export PDF | ✗ | ✓ | ✓ |
| Listen the Script | 1/mo | 10/mo | Unlimited |
| Published scripts | 1 | Unlimited | Unlimited |
| Script drafts | 3 | Unlimited | Unlimited |

---

## Performance Criteria

- Webhook processing: < 2 seconds (Stripe has a 30-second timeout on webhooks; well within budget)
- Feature tier refresh after upgrade: < 10 seconds
- Subscription Settings page load: < 1 second

---

## Offline Behaviour

- Subscription tier is cached in Drift; if offline, the cached tier is used for feature gate checks
- Feature gates use the most recently confirmed server-side tier; cached value is used for UI rendering only; critical server actions re-validate tier

---

## Mobile Behaviour (375px)

- Pricing Modal: stacked plan cards (not side-by-side); toggle switch for annual/monthly pricing
- Manage Subscription: opens Stripe Customer Portal in in-app browser (SFSafariViewController / Chrome Custom Tab)

---

## Error States

**E1 — Checkout Creation Failure**  
Toast: "Could not start checkout. Please try again." Log error to Sentry.

**E2 — Webhook Signature Verification Failure**  
Rust webhook handler returns 400; logs to Sentry; does NOT update subscription.

**E3 — Subscription Not Found After Checkout**  
If the app polls for subscription update and doesn't see it within 30 seconds: show "Verifying your subscription… this may take a moment." Manual refresh option.

---

## Security Considerations

- Webhook endpoint verifies Stripe-Signature header with `stripe_webhook_secret` env var
- Subscription tier is re-validated on every protected server action (not just UI)
- `stripe_customer_id` and `stripe_subscription_id` are stored server-side only; never in the Flutter app's local state
- All Stripe API calls are made from Rust only; Flutter never calls Stripe directly

---

## Human Authorship Considerations

- No AI involvement

---

## DB Tables Touched

- `subscriptions` (INSERT/UPDATE)

```sql
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES profiles(id),
  plan TEXT NOT NULL DEFAULT 'free',
  status TEXT NOT NULL DEFAULT 'active',
  stripe_customer_id TEXT UNIQUE,
  stripe_subscription_id TEXT UNIQUE,
  current_period_end TIMESTAMPTZ,
  cancel_at_period_end BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## Stripe Price IDs (environment variables)

```
STRIPE_PRICE_PRO_MONTHLY=price_xxx
STRIPE_PRICE_PRO_ANNUAL=price_xxx
STRIPE_PRICE_STUDIO_MONTHLY=price_xxx
STRIPE_PRICE_STUDIO_ANNUAL=price_xxx
```

---

## API Endpoints Used

- `POST /subscriptions/checkout` — create Stripe Checkout Session
- `POST /subscriptions/webhook` — Stripe webhook handler (public, no auth, signature-verified)
- `GET /subscriptions/me` — get current user's subscription
- `POST /subscriptions/portal` — create Stripe Customer Portal session URL

---

## Test Cases

**TC-001:** Free user upgrades to Pro → Checkout Session created; on webhook, `subscriptions.plan = 'pro'`.  
**TC-002:** `invoice.payment_failed` → `status = 'past_due'`; banner shown; access maintained for 7 days.  
**TC-003:** 7 days after `past_due` with no payment → `status = 'canceled'`; tier reverts to Free.  
**TC-004:** Pro → Studio upgrade mid-cycle → Studio features unlock within 10s; proration applied.  
**TC-005:** Studio → Pro downgrade → Studio features remain until `current_period_end`.  
**TC-006:** User cancels → Stripe coupon offer shown; access until `current_period_end`.  
**TC-007:** Invalid Stripe webhook signature → 400 returned; no DB change.

---

## Out of Scope

- Team billing (one invoice for multiple Writers' Room members) — Phase 4
- Lifetime deal pricing — Phase 4
- Gift subscriptions — Phase 4
