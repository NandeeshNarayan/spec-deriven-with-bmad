# story-049 — Notifications

| Field | Value |
|---|---|
| **Story ID** | story-049 |
| **Title** | Notifications |
| **Epic** | E11 — Community |
| **Phase** | 3 |
| **Sprint** | Sprint 6 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | All personas |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-002, story-009, story-043, story-045 |

---

## Problem Statement

Writers need to be informed of meaningful events in real time: a collaborator joining their script, a producer expressing interest, a review being completed, their Listen being ready. Without a coherent notification system, users miss critical engagement and trust the platform less.

---

## User Story

As a user, I want to receive timely, relevant notifications — both in-app and as push notifications — so that I never miss important activity on my scripts or profile.

---

## Acceptance Criteria

**GIVEN** any system event occurs that warrants a notification,  
**WHEN** the Rust backend processes the event,  
**THEN** a `notifications` row is inserted; if the user is currently connected via WebSocket, the notification is delivered in real-time via the Supabase Realtime channel `notifications:{user_id}`; if the user is not connected, a push notification is dispatched via Supabase Push (FCM for Android, APNs for iOS, Web Push for PWA).

**GIVEN** the user opens the Notification Center (bell icon in the header),  
**WHEN** the panel opens,  
**THEN** notifications are shown in reverse chronological order; unread notifications are highlighted with a blue dot; the bell icon shows an unread count badge (max "99+"); the panel has a "Mark All as Read" button.

**GIVEN** the user taps a notification,  
**WHEN** the tap registers,  
**THEN** the notification's `read_at` is set to `NOW()`; the user is deep-linked to the relevant screen (e.g., a review notification deep-links to the review panel; a collaborator invitation notification deep-links to the script).

**GIVEN** the user wants to manage notification preferences,  
**WHEN** they navigate to Settings → Notifications,  
**THEN** they see a toggle for each notification type (grouped by category) to enable/disable in-app notifications and push notifications independently.

**GIVEN** a notification is generated for a push-disabled notification type,  
**WHEN** the event fires,  
**THEN** only the in-app `notifications` row is created (no push is sent); in-app notification still appears.

---

## Notification Types (16)

| ID | Event | Category | Default Push |
|---|---|---|---|
| N01 | New collaborator joined script | Collaboration | ✓ |
| N02 | Collaborator made an edit | Collaboration | ✗ |
| N03 | New comment on your script | Collaboration | ✓ |
| N04 | @mention in a comment | Collaboration | ✓ |
| N05 | Comment resolved | Collaboration | ✗ |
| N06 | MR opened on your script | Collaboration | ✓ |
| N07 | MR accepted/rejected | Collaboration | ✓ |
| N08 | Script was liked | Social | ✗ |
| N09 | New follower | Social | ✗ |
| N10 | Producer contact request | Producer | ✓ |
| N11 | Review submitted | Review | ✓ |
| N12 | Review overdue warning (reviewer) | Review | ✓ |
| N13 | Listen production ready | Listen | ✓ |
| N14 | Listen pipeline failed | Listen | ✓ |
| N15 | Payment failed (subscription) | Billing | ✓ |
| N16 | Script approaching deletion (30-day limit) | System | ✓ |

---

## Performance Criteria

- In-app notification delivery (WebSocket): < 500ms from event trigger
- Push notification delivery: < 10 seconds (dependent on FCM/APNs latency)
- Notification Center open: < 300ms
- Unread count badge: updated in real-time via Supabase Realtime

---

## Offline Behaviour

- Notifications received while offline are delivered as push notifications to the device
- On reconnect, the Notification Center fetches any new notifications not yet loaded
- Notifications are NOT cached in Drift (they are always fetched fresh)

---

## Mobile Behaviour (375px)

- Notification Center: full-screen bottom sheet on mobile
- Push notifications: standard OS notification format with LIPILY logo; tapping deep-links to the correct screen
- Unread badge: red dot on the bell icon in the bottom navigation bar

---

## Error States

**E1 — Push Permission Not Granted**  
If the user has not granted push notification permission: only in-app notifications are sent; a subtle banner in Settings: "Enable push notifications to stay updated." with a button to open system settings.

**E2 — Push Delivery Failure**  
If FCM/APNs returns a delivery error: log to Sentry; do not retry push (the in-app notification is the primary delivery mechanism); the notification remains in the Notification Center.

**E3 — Notification Center Load Failure**  
Toast: "Could not load notifications. Pull to refresh."

---

## Security Considerations

- Notifications are RLS-restricted: users can only read their own notifications
- Deep-links from push notifications re-authenticate via JWT before routing (standard go_router auth guard)
- Notification content never contains sensitive data (e.g., payment card numbers); only references to entity IDs

---

## Human Authorship Considerations

- No AI involvement in notifications

---

## DB Tables Touched

- `notifications` (INSERT/UPDATE)
- `notification_preferences` (per-user per-type toggle)

```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  type TEXT NOT NULL,
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  entity_type TEXT,
  entity_id UUID,
  read_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_notifications_user ON notifications(user_id, created_at DESC);
CREATE INDEX idx_notifications_unread ON notifications(user_id, read_at) WHERE read_at IS NULL;

CREATE TABLE notification_preferences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  notification_type TEXT NOT NULL,
  in_app_enabled BOOLEAN NOT NULL DEFAULT TRUE,
  push_enabled BOOLEAN NOT NULL DEFAULT TRUE,
  UNIQUE(user_id, notification_type)
);
```

---

## API Endpoints Used

- `GET /notifications?limit=50&cursor=` — paginated notification list
- `PATCH /notifications/read-all` — mark all as read
- `PATCH /notifications/:id/read` — mark single notification as read
- `GET /notification-preferences` — get user's preferences
- `PATCH /notification-preferences` — update preferences
- `POST /internal/notify` — internal endpoint called by other services (service-to-service key)

---

## Test Cases

**TC-001:** Collaborator joins script → N01 notification created; delivered via WebSocket in < 500ms.  
**TC-002:** User offline → push notification delivered via FCM; on reconnect, notification appears in Notification Center.  
**TC-003:** Tap notification N10 (producer contact) → deep-links to producer contact request screen.  
**TC-004:** "Mark All as Read" → all `read_at` set to NOW(); unread count badge disappears.  
**TC-005:** User disables push for N08 (Script liked) → 50 likes trigger 50 in-app notifications but 0 push notifications.  
**TC-006:** Notification Center load → 50 notifications paginated in < 300ms.

---

## Out of Scope

- Email digest notifications (weekly summary via Resend) — Phase 4
- Notification grouping (e.g., "5 new likes" grouped into one) — Phase 4
- In-app notification sound — Phase 4
