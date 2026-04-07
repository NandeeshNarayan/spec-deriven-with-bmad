# story-003 — Dashboard

| Field | Value |
|---|---|
| **Story ID** | story-003 |
| **Title** | Dashboard |
| **Epic** | E13 — System & Infrastructure |
| **Phase** | 1 |
| **Sprint** | Sprint 0 |
| **Priority** | P0 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-002 |

---

## Problem Statement

After sign-in, writers need a home base to manage their scripts. The dashboard must let writers create, rename, duplicate, and delete scripts; display metadata (title, last edited, page count, tier usage); and navigate into the editor. Without a working dashboard, no other feature can be reached from the authenticated state.

---

## User Story

As a writer, I want a dashboard that shows all my scripts, lets me create new ones, and gives me quick access to script management actions so that I can organise my work without friction.

---

## Acceptance Criteria

**GIVEN** an authenticated user navigates to `/dashboard`,  
**WHEN** the screen loads,  
**THEN** a grid (desktop: 3-column, tablet: 2-column, mobile: 1-column) of script cards displays all scripts owned by or shared with the user, sorted by `updated_at DESC`.

**GIVEN** a user taps "New Script",  
**WHEN** the creation modal opens,  
**THEN** it shows a title input field (placeholder: "Untitled Script"), a format selector (Feature Film / Short Film / TV Pilot / TV Episode / Web Series), and a "Create" button; on "Create", a new row is inserted into the `scripts` table via `POST /scripts` and the user is navigated to `/editor/:script_id`.

**GIVEN** a user with a Free tier has 3 scripts and taps "New Script",  
**WHEN** the creation attempt is made,  
**THEN** instead of the creation modal, a paywall modal appears showing: "You've reached your script limit (3/3 on Free). Upgrade to Pro for unlimited scripts." with a "Upgrade to Pro" CTA; no script is created.

**GIVEN** a script card is rendered,  
**WHEN** the user views it,  
**THEN** it displays: script title (truncated to 2 lines), format badge, last edited relative time ("2 hours ago"), page count estimate (word count ÷ 250), and a cover colour or the first 3 characters of the title as a monogram if no cover image exists.

**GIVEN** a user right-clicks (desktop) or long-presses (mobile) a script card,  
**WHEN** the context menu opens,  
**THEN** it shows: Rename, Duplicate, Move to Folder (stub — greyed out with "Coming Soon"), Export, Delete; each option is accessible via keyboard navigation.

**GIVEN** a user selects "Rename" from the context menu,  
**WHEN** they enter a new title and confirm,  
**THEN** the `PATCH /scripts/:id` endpoint is called with the new title; the card updates immediately (optimistic update); if the API call fails, the title reverts with an error toast.

**GIVEN** a user selects "Duplicate" from the context menu,  
**WHEN** the duplication completes,  
**THEN** a new script is created via `POST /scripts/:id/duplicate`; the new script appears in the grid with title "Copy of [original title]" and status "draft"; Free users who are already at their 3-script limit see the paywall modal instead.

**GIVEN** a user selects "Delete" from the context menu,  
**WHEN** they confirm in the confirmation dialog ("Delete '[title]'? This script will be moved to Recently Deleted and permanently erased after 30 days."),  
**THEN** `DELETE /scripts/:id` is called; the script is soft-deleted (`deleted_at = NOW()`); the card disappears from the grid; a "Undo" toast appears for 5 seconds.

**GIVEN** the user taps "Undo" in the delete toast within 5 seconds,  
**WHEN** the undo action fires,  
**THEN** `POST /scripts/:id/restore` is called; the script reappears in the grid in its original position; the soft-delete is reversed.

**GIVEN** the dashboard is loaded on a 375px mobile screen,  
**WHEN** a script card is rendered,  
**THEN** the card is full width with 16px horizontal padding; the three-dot menu icon is visible and tappable (minimum 44×44dp); the card height is 120px with title and metadata visible.

---

## Performance Criteria

- Dashboard initial load (scripts grid): < 1.5 seconds for up to 50 scripts
- Script card renders with no layout shift (CLS = 0)
- "New Script" → editor navigation: < 800ms
- Rename optimistic update: UI updates within 1 frame (< 16ms) before API response
- Grid scroll performance: maintained 60fps with 100 script cards (virtualised list)

---

## Offline Behaviour

- If the device is offline when the dashboard loads, the cached scripts from Drift (`CachedScripts` table) are displayed with an offline banner: "You're offline. Showing your last synced scripts."
- "New Script" is disabled when offline with tooltip: "Creating scripts requires an internet connection."
- Rename and Delete actions are queued to `OfflineWriteQueue` and applied when connectivity is restored.
- Duplicate is disabled when offline.

---

## Mobile Behaviour (375px)

- Top app bar: LIPILY logo (left), avatar icon (right — taps to profile menu with Sign Out)
- Search bar below app bar: full width, height 44px, "Search scripts..." placeholder
- Script cards: single column, full width minus 32px total padding, 120px height
- Floating action button (FAB): bottom-right, `+` icon, taps to "New Script" creation modal
- Bottom navigation bar: "Scripts" (active), "Discover" (navigates to feed), "Profile"

---

## Error States

**E1 — API Unavailable**  
If `GET /scripts` returns 5xx: show full-screen error state with illustration, message "Couldn't load your scripts. Pull down to retry.", and a retry button.

**E2 — Script Limit Reached (Free Tier)**  
Attempting to create a 4th script on Free tier: paywall modal (described in acceptance criteria AC-3).

**E3 — Delete Conflict**  
If a script being deleted has active collaborators currently in the editor: show warning modal "Other writers have this script open. Deleting will disconnect them. Continue?" before proceeding.

**E4 — Duplicate Timeout**  
If `POST /scripts/:id/duplicate` takes > 10 seconds: show toast "Duplication is taking longer than expected. It will appear in your dashboard when ready." and continue in background.

**E5 — Empty State**  
If the user has zero scripts: show an illustrated empty state with "Your first script starts here." headline, a brief subtext explaining LIPILY's format types, and a large "Create Your First Script" CTA button.

---

## Security Considerations

- `GET /scripts` uses RLS: only returns scripts where `user_id = auth.uid()` OR the user is in `collaborators` with an accepted invite
- The dashboard never exposes the `script_id` in a way that allows enumeration — UUIDs are used
- The soft-delete RLS policy prevents deleted scripts from appearing in the grid without a separate "Recently Deleted" query
- Duplication creates a new script owned by the duplicating user — the original owner's collaborators are NOT copied over

---

## Human Authorship Considerations

Not applicable — no AI-generated content is involved in this story.

---

## DB Tables Touched

- `scripts` (SELECT, INSERT, UPDATE title, soft DELETE)
- `collaborators` (SELECT — to show shared scripts in grid)
- `profiles` (SELECT — to check `script_count` for tier enforcement)

---

## API Endpoints Used

- `GET /scripts` — list all scripts for the current user
- `POST /scripts` — create a new script
- `PATCH /scripts/:id` — rename a script
- `DELETE /scripts/:id` — soft-delete a script
- `POST /scripts/:id/restore` — undo soft-delete
- `POST /scripts/:id/duplicate` — duplicate a script

---

## Test Cases

**TC-001:** Authenticated user with 2 scripts opens dashboard → assert 2 cards rendered.  
**TC-002:** Free user with 3 scripts taps "New Script" → assert paywall modal shown, no API call made.  
**TC-003:** Rename script → assert optimistic update happens before API response; API success → title persists; API failure → title reverts with toast.  
**TC-004:** Delete script → assert card disappears; tap undo within 5s → assert card reappears.  
**TC-005:** Load dashboard with no network → assert Drift cache is used and offline banner shown.  
**TC-006:** Empty state — new user with 0 scripts → assert empty state illustration and CTA shown.  
**TC-007:** Long-press on mobile → assert context menu appears with all options.  
**TC-008:** "Create" with empty title field → assert inline validation: "Please enter a title."  
**TC-009:** Create script → assert navigation to `/editor/:id` and `script_count` incremented in `profiles`.

---

## Out of Scope

- Folder organisation ("Move to Folder" is stubbed in this story)
- Script cover image upload (Phase 2)
- Recently Deleted view (story-014)
- Collaboration invite (story-009)
- Search within scripts (story-023)
- Script publishing toggle (story-041)
- Any dashboard analytics or statistics (story-028)
