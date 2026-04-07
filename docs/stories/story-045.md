# story-045 — Review Submission

| Field | Value |
|---|---|
| **Story ID** | story-045 |
| **Title** | Review Submission |
| **Epic** | E12 — Review Marketplace |
| **Phase** | 3 |
| **Sprint** | Sprint 6 |
| **Priority** | P2 |
| **Status** | DRAFT |
| **Persona** | Priya (the professional TV writer) |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-041, story-046, story-048 |

---

## Problem Statement

Writers need professional feedback on their scripts before pitching to producers. LIPILY's Review Marketplace allows writers to pay for structured, professional script coverage from qualified reviewers. The submission flow must be clear, the payment must be secure (Stripe), and the 14-day SLA must be enforced to maintain trust.

---

## User Story

As a screenwriter, I want to purchase a professional script review from a qualified LIPILY reviewer — so that I receive actionable, industry-standard feedback within 14 days.

---

## Acceptance Criteria

**GIVEN** a writer views their published script detail page,  
**WHEN** they click "Get Professional Review",  
**THEN** they see the Review Marketplace panel: list of available reviewers (with their bios, rating, and number of reviews completed), review price ($25–$150 depending on reviewer tier), and estimated turnaround time; the writer selects a reviewer and clicks "Request Review."

**GIVEN** the writer selects a reviewer and clicks "Request Review",  
**WHEN** the payment flow initiates,  
**THEN** Stripe Checkout opens for the review fee; the writer completes payment; on `payment_intent.succeeded` webhook, a `reviews` row is inserted with `status = 'pending'` and `deadline_at = NOW() + INTERVAL '14 days'`.

**GIVEN** a review request is created,  
**WHEN** the reviewer logs in,  
**THEN** they see the review in their "Pending Reviews" dashboard; they can click "Accept" (changing status to `in_progress`) or "Decline" (triggering a full refund via Stripe and reassignment if available).

**GIVEN** a reviewer accepts and completes their review,  
**WHEN** they submit via the review form,  
**THEN** the form captures: Overall Rating (1–5 stars, required), and 6 structured sections with text fields: (1) Concept & Logline, (2) Structure, (3) Character, (4) Dialogue, (5) Visuals & Pacing, (6) Market Potential; each section has a rating (1–5) and text field (min 50 chars per section); the total review must be ≥ 500 words; submit sets `reviews.status = 'completed'` and `reviews.completed_at = NOW()`.

**GIVEN** a review is completed,  
**WHEN** the writer views the review,  
**THEN** the review is shown in a formatted panel: star ratings for each section displayed as a radar/spider chart using fl_chart, full text per section, overall rating, reviewer name and profile link; the writer can mark the review as "Helpful" (which increments the reviewer's helpful_count).

**GIVEN** the 14-day deadline passes without a review being submitted,  
**WHEN** the Cloud Scheduler checks deadlines at 08:00 UTC daily,  
**THEN** `reviews.status = 'overdue'`; the reviewer is sent an email warning via Resend: "You have an overdue review. Submit within 48 hours or the request will be automatically refunded."; after 48 more hours, status becomes `refunded`; full Stripe refund is issued automatically; the reviewer's on-time rate metric is decremented.

**GIVEN** a writer wants to see all their received reviews,  
**WHEN** they navigate to Profile → Reviews,  
**THEN** they see a list of all completed reviews for their scripts, sortable by date and rating.

---

## Performance Criteria

- Review Marketplace panel load: < 1 second (list of reviewers from DB)
- Stripe Checkout redirect: < 3 seconds
- Review form: auto-saves draft every 60 seconds to `reviews.draft_content` JSON so reviewers don't lose work
- Review spider chart render: < 100ms (fl_chart)

---

## Offline Behaviour

- The Review Marketplace requires network (payment processing)
- Reviewer draft auto-save uses local storage as a backup in case of network loss; syncs on reconnect

---

## Mobile Behaviour (375px)

- Reviewer list: scrollable cards with rating badge
- Review form: one section per screen in a step-flow (to avoid overwhelming the reviewer)
- Spider chart: 300×300dp, centred

---

## Error States

**E1 — Payment Failure**  
Stripe returns a failed payment: "Payment could not be processed. Please check your card details." No review row created.

**E2 — Reviewer Declines**  
Toast to writer: "Your selected reviewer is unavailable. Your payment has been fully refunded within 5 business days. Please choose another reviewer."

**E3 — Review Form Min Length Not Met**  
Inline validation: "Section 'Structure' must be at least 50 characters. You've written {n}."

**E4 — Reviewer Submission After Deadline**  
If reviewer attempts to submit after the `deadline_at + 48 hours` window: form is locked: "This review's deadline has passed and a refund has been issued."

---

## Security Considerations

- Payment is processed entirely by Stripe; no card data touches LIPILY servers
- Review can only be read by the script owner and the reviewer (RLS)
- Reviewer identity is verified through the Reviewer Program (story-046) before they appear in the marketplace
- Writers cannot review their own scripts (server-side check)

---

## Human Authorship Considerations

- Reviews are written entirely by human reviewers
- AI assistance in writing a review is explicitly prohibited in the Reviewer Program terms

---

## DB Tables Touched

- `reviews` (INSERT/UPDATE)
- `reviewer_profiles` (READ for reviewer list)

```sql
CREATE TABLE reviews (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id),
  publication_id UUID NOT NULL REFERENCES script_publications(id),
  writer_id UUID NOT NULL REFERENCES profiles(id),
  reviewer_id UUID NOT NULL REFERENCES profiles(id),
  status TEXT NOT NULL DEFAULT 'pending',
  price_cents INT NOT NULL,
  stripe_payment_intent_id TEXT NOT NULL,
  stripe_refund_id TEXT,
  overall_rating INT CHECK (overall_rating >= 1 AND overall_rating <= 5),
  section_ratings JSONB,
  section_text JSONB,
  draft_content JSONB,
  helpful_count INT NOT NULL DEFAULT 0,
  deadline_at TIMESTAMPTZ NOT NULL,
  completed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_reviews_reviewer ON reviews(reviewer_id, status);
CREATE INDEX idx_reviews_writer ON reviews(writer_id);
```

---

## API Endpoints Used

- `GET /reviewers` — list available reviewers with bios and pricing
- `POST /reviews` — create a review request (triggers Stripe Checkout session)
- `PATCH /reviews/:id/accept` — reviewer accepts
- `PATCH /reviews/:id/decline` — reviewer declines (triggers refund)
- `PATCH /reviews/:id/submit` — reviewer submits completed review
- `GET /reviews?script_id=` — writer views their reviews

---

## Test Cases

**TC-001:** Writer requests review → Stripe Checkout opens; on success, `reviews` row created with status 'pending'.  
**TC-002:** Reviewer accepts → status becomes 'in_progress'.  
**TC-003:** Reviewer submits valid review (500+ words, all sections filled) → status 'completed'; writer notified.  
**TC-004:** Review form section < 50 chars → validation error; submit blocked.  
**TC-005:** 14-day deadline passes → status 'overdue'; reviewer email sent.  
**TC-006:** 14+2 day deadline passes → status 'refunded'; Stripe refund issued automatically.  
**TC-007:** Writer marks review as "Helpful" → reviewer's helpful_count increments.

---

## Out of Scope

- Writer disputing a review (Phase 4 — moderation escalation)
- Group/bulk review packages (Phase 4)
- AI-assisted review suggestions for reviewers (explicitly prohibited)
