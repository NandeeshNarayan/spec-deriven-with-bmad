# story-013 — Branch Editing & Merge Requests

| Field | Value |
|---|---|
| **Story ID** | story-013 |
| **Title** | Branch Editing & Merge Requests |
| **Epic** | E05 — Version Control & Branching |
| **Phase** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Marcus (the Hollywood rewrite specialist) |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-012 |

---

## Problem Statement

A showrunner needs to have multiple writers pitch alternate versions of an act without disturbing the working script. A rewrite specialist needs to explore a structural change without committing it to the main draft. Git-style branches solve this: isolated copies for experimental work, with a formal merge request process that the owner can approve or reject.

---

## User Story

As a script owner, I want to create branches of my script where collaborators can make experimental changes — and then decide whether to merge those changes into the main script via a merge request review.

---

## Acceptance Criteria

**GIVEN** the owner or an editor-role collaborator is in the editor and selects "Create Branch" (from the Version menu),  
**WHEN** the branch dialog opens,  
**THEN** it shows: a branch name input, an optional description field, and a "Base off" selector (defaults to current main script state; can select any draft as base); clicking "Create Branch" calls `POST /scripts/:id/branches`.

**GIVEN** the branch is created,  
**WHEN** the API responds with the new branch ID,  
**THEN** a new entry in the `branches` table is created with: `script_id`, `name`, `created_by`, `base_draft_id`, `status = 'open'`; the user is navigated to the branch editor at `/editor/:script_id/branches/:branch_id`; a branch indicator banner at the top of the editor shows "Editing branch: [name]".

**GIVEN** the user is in the branch editor,  
**WHEN** they make edits,  
**THEN** all edits are saved to the branch's isolated Y.Doc (keyed by `branch_id` on the Rust WebSocket server); the main script's Y.Doc is completely unaffected.

**GIVEN** the branch editor shows the branch indicator banner,  
**WHEN** the user clicks "Submit Merge Request" (from the banner or Version menu),  
**THEN** a merge request dialog opens with: a title input (pre-filled with branch name), a description text area, a reviewer selector (all editors/owners of the script); clicking "Submit" calls `POST /branches/:id/merge-requests`.

**GIVEN** the merge request is submitted,  
**WHEN** the owner opens the merge request (visible in the "Merge Requests" panel in the Versions sidebar),  
**THEN** they see a diff view of the branch vs. the current main script (same format as story-012 diff: green/red block highlights); they can add comments on specific blocks in the diff view; there are "Accept" and "Reject" buttons.

**GIVEN** the owner clicks "Accept",  
**WHEN** the acceptance is confirmed,  
**THEN** `POST /branches/:id/merge` is called; the Rust API merges the branch's Y.Doc into the main Y.Doc using Yjs merge semantics; the main script is updated; a draft is auto-saved before the merge; all collaborators in the main editor receive the merged updates; the branch status is set to `merged`; the author receives a notification.

**GIVEN** the owner clicks "Reject",  
**WHEN** the rejection is confirmed (optional rejection note),  
**THEN** `POST /branches/:id/reject` is called with optional `rejection_note`; the branch status is set to `rejected`; the branch author receives a notification with the rejection note; the branch remains readable (not deleted) so the author can revisit the work.

**GIVEN** the script owner has a merge request pending review,  
**WHEN** they open the Merge Requests panel in the Versions sidebar,  
**THEN** open MRs are shown at the top with status badges (Open / Under Review / Merged / Rejected); each MR shows: title, author avatar, branch name, submission date, and change summary (X blocks added, Y blocks changed, Z blocks removed).

**GIVEN** a branch is rejected,  
**WHEN** the branch author wants to continue working on it,  
**THEN** they can reopen the branch editor from the Branches section of the Versions panel; all their edits remain intact; they can submit a new merge request after revisions.

**GIVEN** a Free or Pro user attempts to create a branch,  
**WHEN** the branch creation is attempted,  
**THEN** a paywall modal appears: "Branch editing is available on Studio plan. Upgrade to unlock experimental editing for your writing team."

---

## Performance Criteria

- Branch creation: < 1 second
- Branch editor load: < 2 seconds (fetches branch Y.Doc state)
- Merge request diff render: < 3 seconds
- Merge execution (Y.Doc merge): < 2 seconds
- Merge broadcast to collaborators: < 2 seconds

---

## Offline Behaviour

- Branch editing works offline (same as main editor — Drift + Y.Doc cache)
- Merge request submission requires internet
- Merge/reject actions require internet

---

## Mobile Behaviour (375px)

- Branch banner at top of editor: compressed to a coloured bar with branch name; tap to expand branch options
- "Create Branch" and "Submit MR" available in the "..." menu in the editor toolbar
- Diff view not available on mobile; shows "Review merge requests on desktop"

---

## Error States

**E1 — Merge Conflict**  
If the main script has diverged significantly from the branch base (> 50% of blocks changed): the merge may produce a result that the Yjs algorithm cannot perfectly reconcile; the Rust API flags this: show warning "Merge conflict detected. Review the merged result carefully." with amber highlights on conflicting blocks.

**E2 — Branch Creator Loses Editor Access**  
If the branch creator's collaborator access is revoked while their branch is open: the branch is closed (status = `closed`); the merge request is automatically rejected with note: "Branch closed — author's access was revoked."

**E3 — Simultaneous Merge**  
If two merge requests are submitted and both accepted concurrently: the second merge sees the result of the first merge (CRDT merge is associative); both are applied; no data loss.

---

## Security Considerations

- Only Owner and Studio-plan Editor can create branches
- Only the script Owner can accept or reject merge requests
- Branch Y.Doc is stored under the `branch_id` key; it is not accessible via the main script ID
- RLS on `branches` and `merge_requests` tables restricts access to script collaborators

---

## Human Authorship Considerations

All branch content is human-written. Merge operations do not involve AI.

---

## DB Tables Touched

- `branches` (INSERT, SELECT, UPDATE status)
- `merge_requests` (INSERT, SELECT, UPDATE status/rejection_note)
- `scripts` (UPDATE `ydoc_state` on merge)
- `drafts` (INSERT auto-save before merge)
- `notifications` (INSERT — for merge accepted/rejected)

---

## API Endpoints Used

- `POST /scripts/:id/branches` — create branch
- `GET /scripts/:id/branches` — list all branches
- `GET /scripts/:id/branches/:branch_id` — get branch editor (WebSocket upgrade for branch Y.Doc)
- `POST /branches/:id/merge-requests` — submit merge request
- `GET /scripts/:id/merge-requests` — list merge requests
- `POST /branches/:id/merge` — accept and execute merge
- `POST /branches/:id/reject` — reject merge request

---

## Test Cases

**TC-001:** Create branch → assert `branches` row created; navigate to branch editor with banner shown.  
**TC-002:** Edit in branch → assert main script Y.Doc unchanged.  
**TC-003:** Submit MR → assert owner sees diff view with correct added/removed blocks.  
**TC-004:** Owner accepts MR → assert main script updated; pre-merge draft created; branch status = `merged`.  
**TC-005:** Owner rejects MR → assert branch author notified with rejection note; branch remains readable.  
**TC-006:** Free user tries to create branch → assert Studio paywall shown.  
**TC-007:** Branch editor load → assert < 2 seconds for 120-page branch.  
**TC-008:** Merge broadcast → assert all main editor collaborators receive updates within 2s.

---

## Out of Scope

- Cherry-pick individual blocks from a branch (Phase 2)
- Multiple active merge requests from the same branch simultaneously (only one open MR per branch)
