# story-012 — Git-Style Versioning & Drafts

| Field | Value |
|---|---|
| **Story ID** | story-012 |
| **Title** | Git-Style Versioning & Drafts |
| **Epic** | E05 — Version Control & Branching |
| **Phase** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Marcus (the Hollywood rewrite specialist) |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Professional screenwriters submit multiple draft versions — "First Draft", "Revised Draft", "Production Draft" — and must be able to return to any previous version if a new direction fails. Unlike binary file backups (Final Draft .bak files), LIPILY must provide named, timestamped snapshots of the full script state that can be restored or compared at any time.

---

## User Story

As a professional screenwriter, I want to save named draft checkpoints of my script — like "First Draft", "Network Notes Draft", "Production Draft" — so that I can return to any version of my work without fear of losing progress.

---

## Acceptance Criteria

**GIVEN** a user is in the editor and clicks "Save Draft" (in the Version menu, Cmd+Shift+D),  
**WHEN** the draft dialog opens,  
**THEN** it shows a text input pre-filled with "Draft — [current date]", a dropdown for draft type (Working Draft / First Draft / Revised Draft / Production Draft / WGA Registration Draft), and a "Notes" optional text area; clicking "Save Draft" calls `POST /scripts/:id/drafts`.

**GIVEN** the draft is saved,  
**WHEN** the API call succeeds,  
**THEN** a snapshot of the current Y.Doc encoded state is stored in `drafts.ydoc_snapshot`; the `drafts` table row includes: `script_id`, `name`, `type`, `notes`, `created_by`, `created_at`, `page_count`, `word_count`; a success toast: "Draft saved: [name]".

**GIVEN** the user opens the "Versions" panel (from the editor sidebar or Cmd+Shift+V),  
**WHEN** the panel loads,  
**THEN** all drafts for the current script are listed in reverse chronological order; each entry shows: draft name, type badge, created by (avatar + name), date, page count; there is a "Restore" and "Preview" button on each entry.

**GIVEN** the user clicks "Preview" on a draft,  
**WHEN** the preview opens,  
**THEN** a read-only modal/panel shows the full script content at that draft's snapshot (rendered from the stored Y.Doc state) with a "Restore to This Draft" button at the top.

**GIVEN** the user clicks "Restore to This Draft" from the preview,  
**WHEN** they confirm in the dialog ("Restore to [draft name]? This will replace the current script content. Your current state will be saved as a draft automatically."),  
**THEN** the current state is auto-saved as a draft named "Auto-save before restore [timestamp]"; the Y.Doc is replaced with the snapshot's `ydoc_snapshot`; the Rust API updates `scripts.ydoc_state`; all collaborators' editors sync to the restored state within 2 seconds via Y.Doc broadcast.

**GIVEN** the script is in an auto-save mode (every 15 minutes while the editor is active),  
**WHEN** 15 minutes elapses since the last edit,  
**THEN** an auto-draft is saved silently with name "Auto-save [timestamp]" and type `auto`; auto-drafts are shown in the Versions panel under a collapsible "Auto-saves" section; maximum 20 auto-saves are retained (oldest is deleted when 21st is created).

**GIVEN** a user is on the Free plan and has 3 manual drafts,  
**WHEN** they try to save a 4th manual draft,  
**THEN** an upgrade prompt appears: "Free plan includes 3 manual drafts. Upgrade to Pro for unlimited draft history."

**GIVEN** a user wants to compare two drafts,  
**WHEN** they select two drafts with the checkboxes and click "Compare",  
**THEN** a side-by-side diff view opens showing: blocks that are unchanged (white), blocks added in the newer draft (green), blocks removed in the newer draft (red); the diff is computed by the Rust API by decoding both Y.Doc snapshots.

**GIVEN** a draft has been restored and the script was previously shared with collaborators,  
**WHEN** the restore completes,  
**THEN** all currently connected collaborators receive a WebSocket broadcast: `{"type": "script_restored", "draftName": "...", "restoredBy": "..."}` and see a banner: "[Name] restored script to [draft name]. Your editor has been updated."

**GIVEN** the user wants to delete a manual draft,  
**WHEN** they click the "Delete" option on a draft entry and confirm,  
**THEN** `DELETE /scripts/:id/drafts/:draft_id` is called; the draft row is hard-deleted (not soft-deleted — draft data can be large); the Versions panel updates immediately.

---

## Performance Criteria

- Draft save (snapshot creation): < 2 seconds for a 120-page (≈ 80KB Y.Doc state) script
- Versions panel load (50 drafts): < 300ms
- Draft preview render: < 1 second
- Diff computation: < 3 seconds for two 120-page scripts
- Restore broadcast to all collaborators: < 2 seconds

---

## Offline Behaviour

- Drafts cannot be saved offline (requires API to store snapshot)
- The Versions panel shows cached draft list (read-only) with "Connect to the internet to save a new draft" message
- Auto-save is paused when offline; resumes when online

---

## Mobile Behaviour (375px)

- Versions panel accessed via the "Versions" icon in the mobile editor bottom toolbar
- Opens as a full-screen bottom sheet
- Each draft entry: name + type badge + date + page count + "..." menu (Preview / Restore / Delete)
- Draft comparison (diff view) is not available on mobile — "Available on desktop" placeholder

---

## Error States

**E1 — Snapshot Too Large**  
If the Y.Doc encoded state exceeds 5MB (very large script): show warning "This draft snapshot is large (5.2MB). It may take a moment to save." and proceed; no hard limit.

**E2 — Restore Failure**  
If the restore API call fails: the auto-save that was created before restore is preserved; the script remains at its pre-restore state; a toast: "Restore failed. Your script is unchanged. Try again."

**E3 — Diff Computation Timeout**  
If the diff takes > 10 seconds: cancel and show "Comparison timed out for very large scripts. Please try again." No partial diff is shown.

---

## Security Considerations

- Drafts are associated with `script_id`; RLS ensures only script owner and editor-role collaborators can create or delete drafts; all collaborators can preview
- Y.Doc snapshots are stored in Supabase and encrypted at rest
- Free plan draft limit (3) is enforced server-side, not just client-side

---

## Human Authorship Considerations

Not applicable — versioning stores human-written content; no AI is involved.

---

## DB Tables Touched

- `drafts` (SELECT all, INSERT snapshot, DELETE)
- `scripts` (UPDATE `ydoc_state` on restore)

---

## API Endpoints Used

- `GET /scripts/:id/drafts` — list all drafts
- `POST /scripts/:id/drafts` — save a named draft
- `GET /scripts/:id/drafts/:draft_id` — load draft for preview
- `POST /scripts/:id/drafts/:draft_id/restore` — restore to draft
- `DELETE /scripts/:id/drafts/:draft_id` — delete draft
- `GET /scripts/:id/drafts/diff` — compute diff between two drafts

---

## Test Cases

**TC-001:** Save draft with name "First Draft" → assert `drafts` row created with snapshot and correct metadata.  
**TC-002:** Open Versions panel → assert all drafts listed in reverse chronological order.  
**TC-003:** Preview draft → assert read-only view renders correctly.  
**TC-004:** Restore draft → assert auto-save created before restore; script content updated; collaborator editors sync.  
**TC-005:** 15-minute auto-save → assert auto-draft created silently; visible under "Auto-saves" section.  
**TC-006:** Free user saves 4th draft → assert upgrade prompt shown.  
**TC-007:** Compare two drafts → assert diff view shows added/removed blocks in correct colours.  
**TC-008:** Restore broadcast → assert all connected collaborators see banner within 2s.

---

## Out of Scope

- Branch editing and merge requests (story-013)
- Draft export as PDF (story-019)
- WGA Draft Registration integration (Phase 2)
