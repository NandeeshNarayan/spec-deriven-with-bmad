# story-046 — Reviewer Program & Payouts

| Field | Value |
|---|---|
| **Story ID** | story-046 |
| **Title** | Reviewer Program & Payouts |
| **Epic** | E12 — Review Marketplace |
| **Phase** | 3 |
| **Sprint** | Sprint 6 |
| **Priority** | P2 |
| **Status** | DRAFT |
| **Persona** | N/A (Internal — Reviewer-facing) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-045, story-048 |

---

## Problem Statement

The Review Marketplace depends on a supply of qualified, reliable reviewers. The Reviewer Program is the application and vetting system that brings reviewers onto the platform, tracks their performance, and ensures they are paid fairly and promptly via Stripe Connect for the reviews they complete.

---

## User Story

As a professional script reader or industry professional, I want to apply to become a LIPILY reviewer, accept paid review requests, and receive my earnings via Stripe — so that I can build a side income doing work I'm qualified for.

---

## Acceptance Criteria

**GIVEN** a LIPILY user navigates to Settings → Become a Reviewer,  
**WHEN** the page loads,  
**THEN** they see an application form with: full name, professional background (free text, min 100 chars), IMDb or portfolio URL (optional), three sample analyses submitted as text (one per genre: Drama, Thriller, Comedy) each min 200 words, and agreement to the Reviewer Program Terms.

**GIVEN** the application is submitted,  
**WHEN** it reaches the admin queue (story-055),  
**THEN** `reviewer_profiles.status = 'pending'`; the applicant receives an email: "Your application is under review. You'll hear back within 5 business days."

**GIVEN** an admin approves the application (story-055 admin panel),  
**WHEN** approval is saved,  
**THEN** `reviewer_profiles.status = 'approved'`; Stripe Connect onboarding link is sent to the reviewer via email; the reviewer must complete Stripe identity verification before appearing in the marketplace.

**GIVEN** the reviewer completes Stripe Connect onboarding,  
**WHEN** the Stripe `account.updated` webhook fires with `charges_enabled = true`,  
**THEN** `reviewer_profiles.stripe_connect_enabled = true` and `reviewer_profiles.visible_in_marketplace = true`; the reviewer now appears in the Review Marketplace.

**GIVEN** a reviewer completes a review (story-045),  
**WHEN** the writer confirms receipt (or 7 days pass after completion without dispute),  
**THEN** a Stripe Connect transfer is initiated: 70% of the review fee goes to the reviewer; 30% is LIPILY's platform fee; the reviewer's `reviewer_profiles.total_earned_cents` is updated.

**GIVEN** a reviewer logs into their Reviewer Dashboard,  
**WHEN** the dashboard loads,  
**THEN** they see: Pending reviews (with deadlines), Completed reviews (with earnings), Total earned (lifetime + current month), On-time rate (%), Helpful rate (%), Average rating received from writers, and a "Withdraw Earnings" button that opens the Stripe Connect Express dashboard.

**GIVEN** a reviewer consistently misses deadlines (on-time rate < 80%),  
**WHEN** the daily scheduler runs,  
**THEN** `reviewer_profiles.visible_in_marketplace = false`; the reviewer is emailed: "Your reviewer status has been paused due to a below-threshold on-time rate (current: {n}%). Improve your rate to be reinstated. Contact support if you have questions."

**GIVEN** a reviewer wants to set their review pricing,  
**WHEN** they update their pricing from the Reviewer Dashboard,  
**THEN** they can set a price between $25 and $150 (in $5 increments); price updates take effect immediately for new requests; existing in-progress reviews retain their original price.

---

## Performance Criteria

- Reviewer Dashboard load: < 1 second
- Stripe Connect transfer: initiated within 60 seconds of review completion confirmation
- Application form: auto-saves every 60 seconds (so long applications aren't lost)

---

## Offline Behaviour

- Reviewer Dashboard requires network
- Review form auto-save uses local browser storage as backup (syncs on reconnect)

---

## Mobile Behaviour (375px)

- Reviewer Dashboard: card-based layout; one card per pending review with countdown timer showing days remaining
- Earnings section: summary card at top, history list below

---

## Error States

**E1 — Stripe Connect Onboarding Incomplete**  
If reviewer has been approved but hasn't completed Stripe Connect after 7 days: reminder email is sent; they cannot appear in the marketplace.

**E2 — Transfer Failure**  
If Stripe Connect transfer fails: logged to `payout_errors` table; admin notified (story-055); manual resolution required within 48 hours.

**E3 — Application Rejected**  
Applicant emailed: "Thank you for applying to the LIPILY Reviewer Program. After careful review, we're unable to approve your application at this time. You're welcome to reapply in 90 days."

---

## Security Considerations

- Stripe Connect identity verification is handled entirely by Stripe; LIPILY never holds reviewer personal identity data
- Reviewer's Stripe Connect account ID is stored in `reviewer_profiles.stripe_connect_account_id` (encrypted at rest via Supabase column encryption)
- Reviewer profile and earnings data is RLS-restricted to the reviewer and admins only
- The platform fee split is enforced server-side in Rust; it cannot be overridden by clients

---

## Human Authorship Considerations

- No AI involvement in the reviewer program

---

## DB Tables Touched

- `reviewer_profiles` (INSERT/UPDATE)
- `payout_errors` (INSERT on transfer failure)

```sql
CREATE TABLE reviewer_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES profiles(id),
  status TEXT NOT NULL DEFAULT 'pending',
  bio TEXT NOT NULL,
  portfolio_url TEXT,
  sample_analyses JSONB,
  price_cents INT NOT NULL DEFAULT 5000,
  stripe_connect_account_id TEXT,
  stripe_connect_enabled BOOLEAN NOT NULL DEFAULT FALSE,
  visible_in_marketplace BOOLEAN NOT NULL DEFAULT FALSE,
  total_earned_cents BIGINT NOT NULL DEFAULT 0,
  reviews_completed INT NOT NULL DEFAULT 0,
  on_time_rate FLOAT NOT NULL DEFAULT 1.0,
  helpful_rate FLOAT NOT NULL DEFAULT 0.0,
  average_rating FLOAT,
  applied_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  approved_at TIMESTAMPTZ,
  paused_at TIMESTAMPTZ
);

CREATE TABLE payout_errors (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  review_id UUID NOT NULL REFERENCES reviews(id),
  reviewer_id UUID NOT NULL REFERENCES profiles(id),
  amount_cents INT NOT NULL,
  error_message TEXT NOT NULL,
  resolved BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## API Endpoints Used

- `POST /reviewer-applications` — submit reviewer application
- `PATCH /reviewer-profiles/:id` — admin approve/reject
- `PATCH /reviewer-profiles/:id/pricing` — reviewer updates pricing
- `GET /reviewer-profiles/me` — reviewer's own dashboard data
- `POST /internal/reviewer-payout` — Rust internal endpoint called after review completion

---

## Test Cases

**TC-001:** Submit reviewer application → `reviewer_profiles` row created with status 'pending'.  
**TC-002:** Admin approves → status 'approved'; Stripe Connect email sent.  
**TC-003:** Stripe Connect onboarding complete (`charges_enabled = true`) → `visible_in_marketplace = true`.  
**TC-004:** Review completed → Stripe transfer: reviewer gets 70%, LIPILY 30%.  
**TC-005:** On-time rate drops below 80% → `visible_in_marketplace = false`; email sent.  
**TC-006:** Reviewer updates price to $75 → new review requests show $75; old in-progress at original price.

---

## Out of Scope

- Reviewer tiers / premium reviewer designation (Phase 4)
- Group payouts or invoice-based payment (Phase 4)
- Reviewer content insurance / dispute escrow (Phase 4)
