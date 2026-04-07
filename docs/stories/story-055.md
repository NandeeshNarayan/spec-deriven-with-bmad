# story-055 — Admin Dashboard

| Field | Value |
|---|---|
| **Story ID** | story-055 |
| **Title** | Admin Dashboard |
| **Epic** | E13 — Infrastructure |
| **Phase** | 4 |
| **Sprint** | Sprint 7 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | N/A (Internal — LIPILY Operations Team) |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-046, story-047, story-048, story-050 |

---

## Problem Statement

As LIPILY scales, the operations team needs a secure internal dashboard to manage users, review applications (reviewer and producer), moderate reported content, monitor platform health, and take administrative actions. The Admin Dashboard is the single back-office tool for running LIPILY.

---

## User Story

As a LIPILY admin, I want a secure internal dashboard where I can manage users, approve applications, moderate content, and monitor platform health — so that I can keep LIPILY running safely and at scale.

---

## Acceptance Criteria

**GIVEN** an admin navigates to `/admin`,  
**WHEN** the page loads,  
**THEN** access is gated by `profiles.is_admin = true` (checked server-side via JWT claim); non-admins get a 403 page; admins see a sidebar with 6 sections: Overview, Users, Content, Applications, Financials, Audit Log.

**GIVEN** the admin is on the Overview tab,  
**WHEN** it loads,  
**THEN** the following platform metrics are shown (updated every 5 minutes from pre-computed views): DAU (Daily Active Users), MAU (Monthly Active Users), New sign-ups today, Active subscriptions by tier (Free / Pro / Studio), Total published scripts, Total Listen productions, Pending reports count, Pending applications count (reviewer + producer).

**GIVEN** the admin is on the Users tab,  
**WHEN** they search for a user by email or username,  
**THEN** they see the user's profile: join date, subscription tier, script count, warning count, is_suspended, is_banned; available actions: Change Tier (override subscription plan), Issue Warning, Suspend Account (with reason field), Ban Account (with reason field), Lift Suspension/Ban, Impersonate User (read-only view of their dashboard for debugging — no edits possible while impersonating).

**GIVEN** the admin changes a user's tier,  
**WHEN** the action is saved,  
**THEN** `subscriptions.plan` is updated; this overrides Stripe (for comped accounts, chargebacks, etc.); `admin_audit_log` records the action with `admin_id`, `action`, `target_id`, `reason`, and `timestamp`.

**GIVEN** the admin is on the Content tab,  
**WHEN** it loads,  
**THEN** they see a queue of pending reports (story-050) sorted by priority: DMCA reports first, then by report creation date; each report shows: reporter username, report reason, reported entity (script title/user), details text; available actions: Dismiss, Warn User, Remove Content, Suspend User, Ban User.

**GIVEN** the admin takes action on a report,  
**WHEN** the action is saved,  
**THEN** `reports.status = 'actioned'`, `reports.actioned_by = admin_id`, `reports.actioned_at = NOW()`; `admin_audit_log` records the action; the reported user is notified via email (for warnings, removals, suspensions, bans).

**GIVEN** the admin is on the Applications tab,  
**WHEN** it loads,  
**THEN** they see two sub-tabs: Reviewer Applications and Producer Applications; each lists pending applications with the applicant's details and sample submissions; available actions: Approve, Reject (with optional rejection reason sent via email).

**GIVEN** the admin approves a reviewer application,  
**WHEN** approval is saved,  
**THEN** `reviewer_profiles.status = 'approved'`; Stripe Connect onboarding email is sent (story-046 flow).

**GIVEN** the admin is on the Financials tab,  
**WHEN** it loads,  
**THEN** they see: MRR (Monthly Recurring Revenue from Stripe), revenue by tier, total payout errors (story-046 `payout_errors` table), and a list of failed payouts requiring manual resolution.

**GIVEN** the admin is on the Audit Log tab,  
**WHEN** they search or browse the log,  
**THEN** all admin actions are shown in reverse chronological order with: admin name, action type, target entity, reason, timestamp; the log is immutable (no delete or edit); it can be filtered by admin name, action type, and date range.

---

## Performance Criteria

- Admin Dashboard load: < 2 seconds (data from pre-computed views)
- User search: < 500ms
- Report queue load (top 100 reports): < 1 second
- Audit log query: < 1 second for recent 1000 entries; paginated beyond that

---

## Offline Behaviour

- Admin Dashboard is not available offline (it is network-dependent)

---

## Mobile Behaviour (375px)

- Admin Dashboard is explicitly Desktop-only; it is not optimised for mobile; a warning banner shows on screens < 768px: "The Admin Dashboard is designed for desktop use."

---

## Error States

**E1 — Non-Admin Access Attempt**  
HTTP 403; page shows: "Access Denied. This area is restricted to LIPILY administrators."

**E2 — Tier Override Conflict with Stripe**  
If an admin overrides a tier but the user has an active Stripe subscription at a different tier: a warning dialog: "This user has an active Stripe subscription at the {current_stripe_tier} tier. Overriding will diverge from Stripe billing. Confirm?" Admin must explicitly confirm.

**E3 — Impersonation Safety Check**  
If an admin tries to impersonate another admin: blocked with "You cannot impersonate another admin account."

---

## Security Considerations

- `is_admin = true` is set ONLY via the Supabase SQL console (never via any API endpoint); there is no in-app way to promote a user to admin
- Admin JWT claim `is_admin: true` is signed by Supabase; Rust verifies this claim on every `/admin/**` route
- All admin actions are logged to `admin_audit_log` (immutable)
- Impersonation is read-only and is logged; impersonation session expires after 30 minutes
- The Admin Dashboard is served from a separate subdomain `admin.lipily.com` with stricter CSP headers
- Two-factor authentication (TOTP) is required for admin accounts (enforced at Supabase auth level)

---

## Human Authorship Considerations

- No AI involvement in admin operations

---

## DB Tables Touched

- `admin_audit_log` (INSERT — immutable)
- `profiles` (UPDATE `is_suspended`, `is_banned`, `warning_count`)
- `subscriptions` (UPDATE `plan` for tier overrides)
- `reviewer_profiles` (UPDATE `status`)
- `producer_profiles` (UPDATE `status`)
- `reports` (UPDATE `status`, `actioned_by`, `actioned_at`)

```sql
CREATE TABLE admin_audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  admin_id UUID NOT NULL REFERENCES profiles(id),
  action TEXT NOT NULL,
  entity_type TEXT,
  entity_id UUID,
  reason TEXT,
  metadata JSONB,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_admin_audit_log_created ON admin_audit_log(created_at DESC);
CREATE INDEX idx_admin_audit_log_admin ON admin_audit_log(admin_id, created_at DESC);

-- RLS: no user can read or write admin_audit_log directly; only service role (Rust)
ALTER TABLE admin_audit_log ENABLE ROW LEVEL SECURITY;
CREATE POLICY "deny_all_admin_audit_log" ON admin_audit_log FOR ALL USING (false);
```

---

## API Endpoints Used

All `/admin/**` endpoints require `Authorization: Bearer {JWT}` with `is_admin = true` claim.

- `GET /admin/overview` — platform metrics
- `GET /admin/users?q=` — search users
- `PATCH /admin/users/:id` — update user (suspend, ban, tier override)
- `POST /admin/users/:id/impersonate` — start read-only impersonation session
- `GET /admin/reports?status=pending` — pending content reports
- `PATCH /admin/reports/:id/action` — action a report
- `GET /admin/applications?type={reviewer|producer}&status=pending` — applications queue
- `PATCH /admin/applications/:id/approve` — approve application
- `PATCH /admin/applications/:id/reject` — reject with reason
- `GET /admin/financials` — MRR and payout error summary
- `GET /admin/audit-log?admin_id=&action=&from=&to=&limit=&cursor=` — audit log

---

## Test Cases

**TC-001:** Non-admin user accesses `/admin` → 403 response; access denied page.  
**TC-002:** Admin searches user by email → user profile shown with all fields.  
**TC-003:** Admin suspends user → `profiles.is_suspended = true`; `admin_audit_log` row created; user emailed.  
**TC-004:** Admin approves reviewer application → `reviewer_profiles.status = 'approved'`; Stripe Connect email sent.  
**TC-005:** Admin actions a DMCA report (Remove Content) → script unpublished; `reports.actioned_at` set.  
**TC-006:** Admin tier override to Studio → `subscriptions.plan = 'studio'`; conflict warning shown if Stripe tier differs.  
**TC-007:** Admin attempts to impersonate another admin → blocked with error.  
**TC-008:** Audit log → all previous admin actions in correct reverse chronological order; no entries missing.

---

## Out of Scope

- Automated fraud detection / AI-assisted moderation — Phase 4
- Multi-admin roles (Super Admin vs. Support Admin) — Phase 4
- Revenue analytics dashboard beyond MRR — Phase 4
