# story-010 — Real-Time Collaboration (Yjs)

| Field | Value |
|---|---|
| **Story ID** | story-010 |
| **Title** | Real-Time Collaboration (Yjs) |
| **Epic** | E04 — Collaboration & CRDT |
| **Phase** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | P0 |
| **Status** | DRAFT |
| **Persona** | Priya (the TV staff writer) |
| **Estimated Effort** | 5 days |
| **Dependencies** | story-004, story-009 |

---

## Problem Statement

TV writers' rooms require real-time collaborative editing where multiple writers work on the same script simultaneously without conflicts or data loss. Current tools like Google Docs don't understand screenplay format; Final Draft's collaboration is fragile and file-based. LIPILY must provide Google Docs–quality real-time collaboration within a WGA-compliant screenplay editor using the Yjs CRDT engine.

---

## User Story

As a TV staff writer, I want to see my collaborators' cursors in real time, see their edits as they type, and never lose any of my own edits — so that I can work in a live writers' room session without coordination overhead.

---

## Acceptance Criteria

**GIVEN** two users are both in the editor for the same script,  
**WHEN** User A types in any block,  
**THEN** User B sees the text appear in their editor within 200ms (end-to-end latency target); User A's cursor is shown as a colour-coded vertical bar in User B's editor with User A's name in a label above it.

**GIVEN** two users simultaneously edit the same block,  
**WHEN** both edits are submitted at the same time,  
**THEN** the Yjs CRDT algorithm merges both edits without losing any content; the final state is consistent across both clients; no user sees a conflict dialog.

**GIVEN** a user opens the editor for a script,  
**WHEN** the WebSocket connection is established to the Rust API,  
**THEN** the client sends its local Y.Doc state vector; the server responds with missing updates (differential sync); the client's Y.Doc is brought up to date within 500ms.

**GIVEN** there are 3 active collaborators in the editor,  
**WHEN** any of them are typing,  
**THEN** the presence bar at the top of the editor shows up to 5 colour-coded avatar initials; a tooltip on each avatar shows the collaborator's name and which scene they are currently in; the presence bar updates within 500ms of cursor movement.

**GIVEN** a user closes their browser or the WebSocket disconnects,  
**WHEN** the disconnect is detected (WebSocket `onclose` event),  
**THEN** that user's cursor disappears from other users' editors within 5 seconds (Redis presence TTL expires); the user's avatar is removed from the presence bar.

**GIVEN** the WebSocket connection drops due to network instability,  
**WHEN** the connection is lost,  
**THEN** the Flutter client attempts automatic reconnection with exponential backoff (1s, 2s, 4s, 8s, max 30s); all edits made during the disconnection are queued in the Y.Doc's pending updates; on reconnect, the pending updates are synced to the server.

**GIVEN** a Studio user is collaborating in a script,  
**WHEN** they reach 20 simultaneous collaborators (Studio plan limit),  
**THEN** the 21st user attempting to join the editor sees a message: "This script has reached the maximum number of simultaneous collaborators (20) for the Studio plan." and is placed in a read-only viewer mode.

**GIVEN** the script owner locks a block (right-click → "Lock Block"),  
**WHEN** any non-owner collaborator tries to edit the locked block,  
**THEN** the block appears with a lock icon; clicking it shows a tooltip "This block is locked by [owner name]"; keystrokes in the block are ignored; the lock is stored as a Y.Map entry on the block's `Y.Map`.

**GIVEN** a collaborator is in the editor with active cursor presence,  
**WHEN** they switch to a different scene in the navigator,  
**THEN** the presence label on their cursor disappears from the previous scene; a new presence indicator appears in the scene navigator card for their new scene (coloured dot on the card).

**GIVEN** the Rust WebSocket server is the authoritative Y.Doc persistence layer,  
**WHEN** any Y.Doc update arrives from any client,  
**THEN** the server (using `yrs-tokio`) applies the update to the server-side Y.Doc, persists the new encoded state to `scripts.ydoc_state` in Supabase, and broadcasts the update to all other connected clients for the same `script_id` within 50ms.

---

## Cursor Presence Protocol

```
Client → Server (JOIN):
  { type: "join", scriptId: "...", userId: "...", userName: "...", color: "#6C63FF" }

Server → All Clients (PRESENCE_UPDATE):
  { type: "presence", users: [{ userId, userName, color, sceneId, blockId, cursorOffset }] }

Client → Server (CURSOR_MOVE):
  { type: "cursor", blockId: "...", offset: 42 }

Server → All Other Clients:
  { type: "cursor", userId: "...", blockId: "...", offset: 42 }

Client → Server (Y.Doc UPDATE):
  Binary Yjs update payload

Server → All Other Clients:
  Same binary Yjs update payload

Server → Disconnecting Client (on ACCESS_REVOKED):
  { type: "access_revoked", userId: "..." }
```

Cursor colours are assigned from a fixed palette of 8 colours, assigned in join order. The same user always gets the same colour for a given script session.

---

## Redis Presence Pattern

```
Key: presence:{script_id}:{user_id}
Value: { userName, color, sceneId, blockId, cursorOffset, joinedAt }
TTL: 10 seconds (refreshed every 5 seconds by client heartbeat)
```

---

## Performance Criteria

- Real-time edit propagation: < 200ms end-to-end (client write → server → other clients rendered)
- WebSocket connection establishment: < 500ms
- Y.Doc differential sync on reconnect: < 1 second for up to 10,000 pending updates
- Cursor position broadcast: < 100ms (cursor moves are throttled to 50ms on client)
- Presence update (user joins/leaves): < 500ms visible to all clients
- Server-side Y.Doc save to Supabase: < 200ms per update (async, non-blocking)

---

## Offline Behaviour

- When offline, the user continues editing; all updates are stored in the Y.Doc pending queue and in Drift `CachedBlocks`
- The offline banner shows: "Offline — your edits will sync when you reconnect"
- On reconnect, the client sends all pending Y.Doc updates; the server merges them using CRDT (no conflicts possible by design)
- The client receives all updates made by other collaborators during the offline period

---

## Mobile Behaviour (375px)

- Cursor presence labels are not shown on mobile (screen too small); instead, a "3 writers active" pill appears in the top-right of the editor toolbar
- Tapping the pill shows a bottom sheet listing all active collaborators, their current scene, and a "Jump to their cursor" button
- Block locking is not available on mobile (gesture conflict with drag-to-reorder)

---

## Error States

**E1 — Y.Doc Desync Detected**  
Server and client state vectors diverge irrecoverably: server sends `{ type: "full_sync_required" }`; client discards in-memory Y.Doc and fetches full encoded state from `GET /scripts/:id/ydoc`; local edits since last sync are replayed on top.

**E2 — WebSocket Server Overloaded**  
Rust API returns 503 on WebSocket upgrade: client falls back to polling `GET /scripts/:id/blocks` every 5 seconds; a banner shows "Live collaboration temporarily unavailable. Your edits are saved."; polling stops when WebSocket reconnects.

**E3 — Studio Plan Limit (21st User)**  
As described in acceptance criteria: 21st user placed in read-only mode with message.

**E4 — Invalid JWT on WebSocket Upgrade**  
The Rust API WebSocket handler validates the JWT on the HTTP upgrade request; if invalid/expired, the server rejects with 401 and the client is redirected to `/auth`.

---

## Security Considerations

- Every WebSocket message from a client is authenticated via the JWT passed in the `Authorization` header on the WebSocket upgrade request
- The server verifies the user is an owner or editor role collaborator before accepting any Y.Doc update messages; viewer/commenter connections receive broadcasts only
- Y.Doc state stored in Supabase `scripts.ydoc_state` is encrypted at rest
- The WebSocket server uses `sessionAffinity=true` on Cloud Run to ensure a user's WebSocket stays on the same instance; this is required for `yrs-tokio` in-memory Y.Doc management
- Maximum 20 concurrent WebSocket connections per `script_id` (Studio plan limit); enforced via Redis counter

---

## Human Authorship Considerations

- All edits made via real-time collaboration are `human_written` (regardless of which collaborator typed them)
- The Y.Doc tracks each update's origin user ID in the `clientID` field of the Yjs update

---

## DB Tables Touched

- `scripts` (UPDATE `ydoc_state` on every update)
- `blocks` (UPDATE via Y.Doc → block save debounce path, same as story-004)

---

## API Endpoints Used

- `GET /scripts/:id/ydoc` — WebSocket upgrade endpoint for Y.Doc sync
- WebSocket messages (binary Yjs protocol) handled within the same connection

---

## Test Cases

**TC-001:** Two users in same script → User A types → User B sees text within 200ms.  
**TC-002:** Simultaneous edit of same block → assert CRDT merge produces correct combined text with no data loss.  
**TC-003:** User opens editor → WebSocket connects → assert Y.Doc sync completes within 500ms.  
**TC-004:** 3 active users → assert presence bar shows 3 colour-coded avatars.  
**TC-005:** Close browser → assert cursor disappears from other clients within 5s.  
**TC-006:** Disconnect network → type 5 blocks → reconnect → assert all 5 appear in server state.  
**TC-007:** 21st user joins → assert they are placed in read-only with correct message.  
**TC-008:** Owner locks block → Editor tries to type in it → assert keystrokes ignored and lock icon shown.  
**TC-009:** Invalid JWT on WebSocket upgrade → assert server returns 401.  
**TC-010:** WebSocket drops → assert reconnection with backoff; pending updates sync on reconnect.

---

## Out of Scope

- Comment threads (story-011)
- Conflict UI for merge requests (story-013)
- Writers' Room showrunner controls (story-053)
- Block locking on mobile (deferred)
