# story-053 — Writers' Room Mode

| Field | Value |
|---|---|
| **Story ID** | story-053 |
| **Title** | Writers' Room Mode |
| **Epic** | E04 — Collaboration |
| **Phase** | 4 |
| **Sprint** | Sprint 7 |
| **Priority** | P2 |
| **Status** | DRAFT |
| **Persona** | Priya (the TV writer — Showrunner) |
| **Estimated Effort** | 5 days |
| **Dependencies** | story-009, story-010, story-017 |

---

## Problem Statement

Professional TV writers rooms involve a Showrunner managing a team of writers working simultaneously on a single script or series bible. LIPILY's Writers' Room Mode is the Studio-tier feature that enables this — with a Showrunner superuser role, room controls (locking sections, muting edits), and a Writers' Room dashboard view that gives the Showrunner an overview of all activity.

---

## User Story

As a Showrunner on a TV series, I want to manage a Writers' Room of up to 20 writers in a shared script — with the ability to lock sections, mute writers, and see a real-time overview of who is writing what — so that my room runs like a professional production.

---

## Acceptance Criteria

**GIVEN** a Studio-tier user enables Writers' Room Mode for a script (from Editor → Settings → Enable Writers' Room),  
**WHEN** the mode is enabled,  
**THEN** the script's `writers_room_enabled = true`; the current owner's role is elevated to `showrunner`; up to 20 collaborators can be added (the Collaboration Invite system, story-009, accepts up to 20 for this script); a Writers' Room Dashboard panel becomes available.

**GIVEN** a user with the Showrunner role opens the Writers' Room Dashboard,  
**WHEN** the dashboard loads,  
**THEN** they see: a list of all active collaborators with their real-time cursor position (scene name + block type), a presence panel (online/idle/offline with last-seen time), total words written in this session per writer, and room controls.

**GIVEN** the Showrunner uses Room Controls,  
**WHEN** they interact with controls,  
**THEN** the available controls are: Lock Script (sets `writers_room_session.script_locked = true`; all non-Showrunner edits are blocked; a "Script is locked by the Showrunner" banner shows in the editor for all writers), Unlock Script (reverses lock), Mute Writer (prevents a specific collaborator from making edits for the current session; their cursor is visible but grayed out), Unmute Writer, Kick from Room (removes the collaborator from the active WebSocket session; they are not removed from the `collaborators` table — they can re-join).

**GIVEN** the script is locked,  
**WHEN** a non-Showrunner writer types in the editor,  
**THEN** keystrokes are rejected at the Flutter layer; a toast appears once: "The Showrunner has locked the script. Edits are paused."; the editor shows a subtle lock overlay at the top of the page.

**GIVEN** the Showrunner activates "Act Focus Mode" from the Writers' Room Dashboard,  
**WHEN** they select Act 1, 2, or 3 from the Beat Board (story-017),  
**THEN** all writers' editors automatically scroll to the selected Act's first scene; a banner appears: "Showrunner has focused the room on Act {n}."; individual writers can scroll away after dismissing the banner.

**GIVEN** a writer is in the Writers' Room and opens the Beat Board (story-017),  
**WHEN** they view the board,  
**THEN** they see which writers are assigned to each beat (if assigned by the Showrunner); Showrunner can drag-and-drop beat card assignments to writer names; `beat_assignments` table updated.

**GIVEN** the Writers' Room session ends (all writers disconnect for > 15 minutes),  
**WHEN** the session ends,  
**THEN** `writers_room_sessions` row is marked as ended; total words written per writer in the session are persisted to `writing_sessions` (story-028 stats); a session summary is available in the Writers' Room Dashboard history.

---

## Performance Criteria

- Writers' Room Dashboard real-time updates: < 500ms (same Yjs/WebSocket pipeline as story-010)
- Script lock propagation to all connected writers: < 200ms
- Room controls (lock/mute/kick): < 100ms round-trip
- Dashboard handles 20 concurrent writers without additional latency beyond the story-010 SLA

---

## Offline Behaviour

- Writers' Room Mode requires a network connection at all times (collaborative feature)
- If a writer loses connection during a locked session, they see the standard offline indicator; on reconnect, they re-join the room and see the current lock state

---

## Mobile Behaviour (375px)

- Writers' Room Dashboard is Desktop/Tablet-first (20-writer management is not practical on 375px)
- On mobile: a simplified "Room Status" panel shows (lock state, number of active writers, your role); full dashboard is accessible in landscape mode only

---

## Error States

**E1 — Non-Studio Tier Tries to Enable Writers' Room**  
Button is disabled with tooltip: "Writers' Room Mode requires a Studio plan."

**E2 — Over 20 Collaborators**  
The 21st invite attempt: "Writers' Room supports up to 20 writers. Remove an existing collaborator to add a new one."

**E3 — Showrunner Disconnects Mid-Session**  
If the Showrunner's WebSocket disconnects, the script remains in its current lock state for up to 5 minutes; after 5 minutes, the script automatically unlocks; co-owners can assume Showrunner controls if the Showrunner does not reconnect.

---

## Security Considerations

- Only the Showrunner role can lock the script, mute writers, or kick participants (enforced in Rust WebSocket handler)
- Script lock state is stored in `writers_room_sessions` (not just in-memory) so that reconnecting writers immediately receive the current state
- Writers' Room Mode is Studio-only; Rust subscription check fires on every room-control action

---

## Human Authorship Considerations

- No AI involvement in Writers' Room Mode
- Each writer's content_source attribution is preserved (if Writer A writes dialogue, it is attributed to their session even in a shared script)

---

## DB Tables Touched

- `scripts` (UPDATE `writers_room_enabled`)
- `writers_room_sessions` (INSERT/UPDATE)
- `beat_assignments` (INSERT/UPDATE)

```sql
CREATE TABLE writers_room_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id),
  showrunner_id UUID NOT NULL REFERENCES profiles(id),
  script_locked BOOLEAN NOT NULL DEFAULT FALSE,
  started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  ended_at TIMESTAMPTZ,
  muted_user_ids UUID[] NOT NULL DEFAULT '{}'
);

CREATE TABLE beat_assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id),
  beat_id UUID NOT NULL REFERENCES beat_board_cards(id),
  assigned_to UUID REFERENCES profiles(id),
  assigned_by UUID NOT NULL REFERENCES profiles(id),
  assigned_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(beat_id)
);
```

---

## API Endpoints Used

- `PATCH /scripts/:id/writers-room` — enable/disable Writers' Room Mode
- `GET /scripts/:id/writers-room/dashboard` — live dashboard data (SSE or WebSocket feed)
- `POST /scripts/:id/writers-room/lock` — lock script
- `DELETE /scripts/:id/writers-room/lock` — unlock script
- `POST /scripts/:id/writers-room/mute/:user_id` — mute a writer
- `DELETE /scripts/:id/writers-room/mute/:user_id` — unmute
- `POST /scripts/:id/writers-room/kick/:user_id` — kick from session

---

## Test Cases

**TC-001:** Enable Writers' Room → `writers_room_enabled = true`; owner's role becomes 'showrunner'.  
**TC-002:** Showrunner locks script → all non-Showrunner writers see lock banner; edits rejected.  
**TC-003:** Script remains locked for 5 min after Showrunner disconnects → auto-unlock fires.  
**TC-004:** Mute writer → muted writer's keystrokes rejected; cursor visible but grayed out.  
**TC-005:** Kick writer → writer's WebSocket session closed; they can re-join.  
**TC-006:** 21st collaborator invite → error shown; no invite sent.  
**TC-007:** Non-Studio user → Writers' Room button disabled.

---

## Out of Scope

- Video/audio conferencing within the Writers' Room — Phase 4
- Showrunner assigning individual scenes (not beats) to writers — Phase 4
- Writers' Room session recording/playback — Phase 4
