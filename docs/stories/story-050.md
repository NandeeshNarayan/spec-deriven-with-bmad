# story-050 — Content Moderation & Reporting

| Field | Value |
|---|---|
| **Story ID** | story-050 |
| **Title** | Content Moderation & Reporting |
| **Epic** | E09 — Polish |
| **Phase** | 4 |
| **Sprint** | Sprint 7 |
| **Priority** | P2 |
| **Status** | DRAFT |
| **Persona** | N/A (Safety system) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-041, story-043, story-055 |

---

## Problem Statement

As LIPILY becomes a public community with published scripts, user profiles, and reviews, it needs a content moderation system to handle abuse, DMCA claims, and inappropriate content. Users must be able to report content; admins must be able to review and act on reports; and the system must comply with legal obligations.

---

## User Story

As a community member, I want to report inappropriate content, plagiarism, or copyright violations — and as an admin, I want to review and take action on those reports — so that LIPILY remains a safe, trustworthy platform.

---

## Acceptance Criteria

**GIVEN** a user views any published script card or detail page,  
**WHEN** they click the "•••" (more options) menu,  
**THEN** a "Report" option is shown; tapping it opens a Report Modal.

**GIVEN** the Report Modal is open,  
**WHEN** the user selects a report reason,  
**THEN** the available reasons are: Inappropriate Content, Spam or Misleading, Copyright / DMCA Violation, Plagiarism, Harassment or Hate Speech, Other; a free-text "Additional details" field is shown (optional for all except DMCA, where a copyright claim statement is required); submitting creates a `reports` row.

**GIVEN** a report is submitted,  
**WHEN** the submission is saved,  
**THEN** the user sees a confirmation: "Thank you for your report. Our team will review it within 72 hours."; the user cannot report the same content twice with the same reason (duplicate check).

**GIVEN** a DMCA violation report is submitted,  
**WHEN** the report is created,  
**THEN** the report is flagged with `is_dmca = true`; an automatic email is sent to `legal@lipily.com` with the report details; the `priority = 'high'`; admin queue (story-055) shows DMCA reports at the top.

**GIVEN** an admin reviews a report in the Admin Dashboard (story-055),  
**WHEN** they take action,  
**THEN** available actions are: Dismiss (no violation found), Warn User (send a warning email to the content owner), Remove Content (set `scripts.is_published = false` or delete the profile photo), Suspend Account (set `profiles.is_suspended = true`; user gets an email with appeal instructions), Ban Account (set `profiles.is_banned = true`; permanent).

**GIVEN** an admin removes a piece of content,  
**WHEN** the action is saved,  
**THEN** `reports.status = 'actioned'`; `admin_audit_log` (story-055) records the action; the content owner receives an email: "Your content '[Title]' has been removed for violating our Community Guidelines. If you believe this is a mistake, you can appeal within 14 days."

**GIVEN** a user is warned,  
**WHEN** the warning is issued,  
**THEN** `profiles.warning_count` increments; at 3 warnings, an admin review is automatically flagged for potential suspension.

**GIVEN** a suspended user tries to log in,  
**WHEN** the auth session is checked,  
**THEN** the login succeeds but the app shows a suspension notice: "Your account has been temporarily suspended. Reason: {reason}. Appeal by emailing support@lipily.com."; all write operations return 403.

---

## Performance Criteria

- Report submission: < 500ms
- Admin moderation queue load: < 1 second
- Content removal (unpublish): < 5 seconds (same as story-041 unpublish flow)

---

## Offline Behaviour

- Reporting is not available offline (requires network)

---

## Mobile Behaviour (375px)

- Report modal: bottom sheet with radio buttons for reason selection
- "Additional details" text field: expandable

---

## Error States

**E1 — Duplicate Report**  
If the user has already reported the same content with the same reason: "You've already reported this content. Our team is reviewing it."

**E2 — Report Submission Failure**  
Toast: "Report could not be submitted. Please try again."

---

## Security Considerations

- Reporter identity is never revealed to the reported user
- `reports` table is RLS-restricted: reporters can only read their own reports; admins can read all
- Suspension and ban logic is enforced server-side (Rust middleware checks `is_suspended` and `is_banned` on every authenticated request)
- DMCA notices are handled in compliance with the Digital Millennium Copyright Act; legal team is CC'd

---

## Human Authorship Considerations

- No AI involvement in moderation decisions — all decisions are made by human admins

---

## DB Tables Touched

- `reports` (INSERT/UPDATE)
- `profiles` (UPDATE `is_suspended`, `is_banned`, `warning_count`)
- `scripts` (UPDATE `is_published`)
- `admin_audit_log` (INSERT on every admin action)

```sql
CREATE TABLE reports (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  reporter_id UUID NOT NULL REFERENCES profiles(id),
  entity_type TEXT NOT NULL,
  entity_id UUID NOT NULL,
  reason TEXT NOT NULL,
  details TEXT,
  is_dmca BOOLEAN NOT NULL DEFAULT FALSE,
  status TEXT NOT NULL DEFAULT 'pending',
  priority TEXT NOT NULL DEFAULT 'normal',
  actioned_by UUID REFERENCES profiles(id),
  actioned_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(reporter_id, entity_type, entity_id, reason)
);
CREATE INDEX idx_reports_status ON reports(status, priority, created_at DESC);
```

---

## API Endpoints Used

- `POST /reports` — submit a report
- `GET /admin/reports?status=pending&priority=` — admin: list reports
- `PATCH /admin/reports/:id/action` — admin: take action on a report

---

## Test Cases

**TC-001:** Report script for "Inappropriate Content" → `reports` row created; confirmation shown.  
**TC-002:** Same user reports same script for same reason → duplicate error; no new row.  
**TC-003:** DMCA report → `is_dmca = true`; email sent to legal@lipily.com.  
**TC-004:** Admin removes content → script unpublished; `admin_audit_log` row created; content owner emailed.  
**TC-005:** 3 warnings → automatic admin flag for suspension review.  
**TC-006:** Suspended user → logs in; sees suspension notice; all write operations return 403.

---

## Out of Scope

- Automated AI content screening (Phase 4)
- In-platform appeals system UI (Phase 4; Phase 3 appeals are via email)
- Government content removal requests (handled by legal team outside the platform)
