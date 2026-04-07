# story-014 — Smart Deletion & Recovery

| Field | Value |
|---|---|
| **Story ID** | story-014 |
| **Title** | Smart Deletion & Recovery |
| **Epic** | E01 — Core Editor |
| **Phase** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 2 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Writers sometimes delete scenes or entire scripts by accident — a mis-swipe, a keyboard shortcut gone wrong — and discover the loss only after closing the app. Without a robust recovery system, months of creative work can vanish permanently. LIPILY's Smart Deletion uses soft deletes with a 30-day grace period and a "Recently Deleted" view that mirrors the iOS Photos experience.

---

## User Story

As a writer, I want all deleted scripts and scenes to be recoverable for 30 days after deletion — so that I can undo accidental deletions without losing work.

---

## Acceptance Criteria

**GIVEN** a user deletes a script from the dashboard (story-003),  
**WHEN** `DELETE /scripts/:id` is called,  
**THEN** the script's `deleted_at` is set to `NOW()`; the script disappears from the main dashboard grid; it appears in the "Recently Deleted" view at `GET /scripts?deleted=true`; a cron job (Cloud Scheduler) permanently deletes scripts where `deleted_at < NOW() - INTERVAL '30 days'`.

**GIVEN** a user navigates to "Recently Deleted" (accessible from the dashboard sidebar or the user profile menu),  
**WHEN** the page loads,  
**THEN** all soft-deleted scripts are shown with: title, deletion date, and a countdown ("Permanently deleted in 24 days"); they are sorted by `deleted_at DESC`; expired scripts (> 30 days) do not appear (already purged).

**GIVEN** a user selects a script in "Recently Deleted" and clicks "Restore",  
**WHEN** `POST /scripts/:id/restore` is called,  
**THEN** `deleted_at` is set to `null`; `script_count` in `profiles` is re-incremented; the script reappears in the main dashboard; if the Free plan limit was already at 3 scripts, the restore is blocked with a message: "You've reached your script limit. Delete another script or upgrade to Pro to restore this one."

**GIVEN** a user deletes a scene from the scene navigator (story-005),  
**WHEN** `DELETE /scenes/:id` is called,  
**THEN** the scene and all its blocks are soft-deleted; they appear in a "Deleted Scenes" section accessible from the scene navigator's overflow menu; the scene can be individually restored.

**GIVEN** a user deletes multiple blocks by selecting them and pressing Delete/Backspace,  
**WHEN** the deletion fires,  
**THEN** each block is soft-deleted; the undo toast ("Undo — [N] blocks deleted") appears for 5 seconds; pressing Undo within 5 seconds restores all deleted blocks to their original positions.

**GIVEN** a user's script has been in "Recently Deleted" for exactly 30 days,  
**WHEN** the Cloud Scheduler purge job runs (daily at UTC 03:00),  
**THEN** the script's `deleted_at` is >= the 30-day threshold; the Cloud Scheduler calls `DELETE /scripts/:id/purge` (internal endpoint); all associated data (blocks, scenes, drafts, comments, branches, story bible entries, scene intelligence) is permanently deleted from Supabase; the `recently_deleted` table row (if any) is also removed.

**GIVEN** a user on the Free plan is at their 3-script limit,  
**WHEN** they attempt to restore a deleted script,  
**THEN** they see: "You currently have 3 scripts. Delete one of your existing scripts or upgrade to Pro before restoring." with two CTAs: "Delete a Script" and "Upgrade to Pro".

**GIVEN** a script in "Recently Deleted" belongs to a collaboration,  
**WHEN** only the owner deleted the script,  
**THEN** collaborators who had access see the script as "Deleted" in their dashboard (greyed out, not interactive); a tooltip: "This script was deleted by the owner. It can be restored within 30 days."; collaborators cannot restore the script — only the owner can.

**GIVEN** the Cloud Scheduler purge deletes a script,  
**WHEN** the purge completes,  
**THEN** an email is sent to the script owner via Resend: "Your script '[title]' was permanently deleted after 30 days in the trash. This action cannot be undone."

**GIVEN** the user wants to permanently delete a script before the 30-day period,  
**WHEN** they click "Delete Permanently" in the Recently Deleted view and confirm (with an explicit "I understand this cannot be undone" checkbox),  
**THEN** `DELETE /scripts/:id/purge` is called immediately; the script and all data are permanently deleted; no undo is possible.

---

## Performance Criteria

- Recently Deleted page load: < 500ms (up to 30 deleted scripts)
- Restore API call: < 500ms
- Purge job execution: < 30 seconds for up to 100 expired scripts
- Undo toast response (block restore): < 100ms

---

## Offline Behaviour

- Recently Deleted page reads from a cached list in Drift; shows offline banner
- Restore and permanent delete actions require internet connection; disabled when offline

---

## Mobile Behaviour (375px)

- "Recently Deleted" accessible from the profile/settings bottom sheet on mobile dashboard
- Each deleted script: row tile with title + deletion date + countdown + "Restore" and "Delete Permanently" options in a "..." menu
- Permanent delete requires double confirmation on mobile (first tap shows confirmation modal; second tap in the modal confirms)

---

## Error States

**E1 — Restore Blocked by Plan Limit**  
As described in AC-7: show modal with two options.

**E2 — Script Already Purged**  
If a user somehow has a stale link to a deleted script and tries to restore it after purge: `POST /scripts/:id/restore` returns 404; show "This script no longer exists. It was permanently deleted."

**E3 — Purge Job Failure**  
If the purge Cloud Scheduler job fails (non-zero exit): Sentry captures the error; Cloud Scheduler retries up to 3 times; the script is NOT deleted until the job succeeds; a Sentry alert is sent.

---

## Security Considerations

- Soft-deleted scripts are excluded from RLS SELECT policy on `scripts` (the base policy filters `deleted_at IS NULL`); a separate policy allows owner to query `?deleted=true`
- Permanent delete is irreversible — enforced server-side, not client-side
- The purge endpoint is internal-only (not accessible from the Flutter client); it is called by Cloud Scheduler using a service-to-service API key

---

## Human Authorship Considerations

Not applicable.

---

## DB Tables Touched

- `scripts` (UPDATE deleted_at, UPDATE deleted_at = null on restore, hard DELETE on purge)
- `scenes` (UPDATE deleted_at, restore)
- `blocks` (UPDATE deleted_at, restore)
- `recently_deleted` (INSERT record on delete for 30-day countdown display)
- `profiles` (UPDATE script_count on restore/delete)

---

## API Endpoints Used

- `GET /scripts?deleted=true` — list soft-deleted scripts
- `POST /scripts/:id/restore` — restore from trash
- `DELETE /scripts/:id/purge` — permanent delete (internal endpoint)
- `GET /scenes?deleted=true&script_id=:id` — list deleted scenes
- `POST /scenes/:id/restore` — restore deleted scene

---

## Test Cases

**TC-001:** Delete script → assert it appears in Recently Deleted with 30-day countdown.  
**TC-002:** Restore script → assert it reappears in dashboard; `script_count` re-incremented.  
**TC-003:** Free user at limit tries to restore → assert blocked with correct upgrade prompt.  
**TC-004:** Purge job runs on script with `deleted_at` = 31 days ago → assert all data permanently deleted; purge email sent.  
**TC-005:** Delete scene → assert scene card disappears from navigator; restore from deleted scenes → reappears in correct position.  
**TC-006:** Delete 3 blocks → undo within 5s → assert all 3 blocks restored to original positions.  
**TC-007:** Permanent delete before 30 days → assert data immediately purged after double confirmation.

---

## Out of Scope

- Trash for collaborator-created content (only owner can access their own trash)
- Backup/export before purge (Phase 2)
