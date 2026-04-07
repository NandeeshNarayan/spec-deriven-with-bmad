# story-004 — Core Editor Engine

| Field | Value |
|---|---|
| **Story ID** | story-004 |
| **Title** | Core Editor Engine |
| **Epic** | E01 — Core Editor |
| **Phase** | 1 |
| **Sprint** | Sprint 1 |
| **Priority** | P0 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 8 days |
| **Dependencies** | story-003 |

---

## Problem Statement

The screenwriting editor is the heart of LIPILY. Writers spend 90% of their time in it. It must produce WGA-compliant formatting automatically, respond to keyboard input at 60fps with < 16ms latency, support all 7 block types (Scene Heading, Action, Character, Parenthetical, Dialogue, Transition, Text Note), implement the Smart Flow Logic 7×2 transition matrix, persist every keystroke to Supabase via the Rust API, and co-exist with the offline Drift cache and Yjs CRDT layer. This is the most performance-critical story in the project.

---

## User Story

As a screenwriter, I want a professional editor that handles WGA-standard formatting automatically — so that I can focus entirely on storytelling without thinking about margins, caps, or tab stops.

---

## Acceptance Criteria

**GIVEN** a user navigates to `/editor/:script_id`,  
**WHEN** the editor loads,  
**THEN** all scene blocks from `blocks` table are fetched via `GET /scripts/:id/blocks`, rendered in a vertically stacked list ordered by `position`, and the cursor is placed at the last edited position.

**GIVEN** a writer is in a "Scene Heading" block and presses Enter,  
**WHEN** Smart Flow Logic runs,  
**THEN** a new "Action" block is inserted below; the cursor moves to it; the Smart Flow state machine transitions to `action` state.

**GIVEN** a writer is in an "Action" block and presses Tab,  
**WHEN** Smart Flow Logic runs,  
**THEN** the block type transitions to "Character" and the text is converted to ALL CAPS automatically; the cursor remains in the same position within the text.

**GIVEN** a writer is in a "Character" block and presses Enter,  
**WHEN** Smart Flow Logic runs,  
**THEN** a new "Dialogue" block is inserted; the cursor moves to it; the block type is auto-typed as "Dialogue".

**GIVEN** a writer is in a "Dialogue" block and presses Enter,  
**WHEN** Smart Flow Logic runs,  
**THEN** a new "Character" block is inserted (ready for next speaker); the cursor moves to it.

**GIVEN** a writer is in any block and presses Shift+Enter,  
**WHEN** Smart Flow Logic runs,  
**THEN** a "Parenthetical" block is inserted below the current block, pre-filled with `( )` and the cursor placed between the parentheses.

**GIVEN** a writer types text in any block,  
**WHEN** they pause for 800ms (debounce),  
**THEN** the updated block content is saved via `PATCH /blocks/:id` with the new `ydoc_delta`; the block's `content_source` is set to `human_written`; `updated_at` is updated.

**GIVEN** a writer types in a "Scene Heading" block,  
**WHEN** they begin typing "INT." or "EXT.",  
**THEN** an autocomplete dropdown shows all previously used scene headings matching the typed prefix, sorted by frequency; pressing Tab or Enter selects the top suggestion.

**GIVEN** a writer presses Cmd+Z (undo),  
**WHEN** the undo fires,  
**THEN** the Y.Doc's built-in Yjs undo manager reverses the last operation; the visual block content updates instantly; the undo stack supports at least 100 operations.

**GIVEN** a block exists with `content_source = 'ai_generated'` or `'ai_edited'`,  
**WHEN** the block is rendered,  
**THEN** a subtle sparkle icon (✦) appears in the left gutter of the block; tapping it shows the AI badge tooltip: "AI suggested — click to confirm" with a ✓ confirm button.

---

## Smart Flow Logic Matrix (Complete)

| Current Block | Enter | Tab | Shift+Enter |
|---|---|---|---|
| Scene Heading | Action | — | Parenthetical |
| Action | Action | Character | Parenthetical |
| Character | Dialogue | — | Parenthetical |
| Parenthetical | Dialogue | — | — |
| Dialogue | Character | Action | Parenthetical |
| Transition | Scene Heading | — | — |
| Text Note | Text Note | — | — |

---

## Block Type Formatting (WGA Standard)

| Block Type | Left Margin | Width | Case | Font |
|---|---|---|---|---|
| Scene Heading | 1.5" | 6.0" | ALL CAPS, bolded | Courier Prime 12pt |
| Action | 1.5" | 6.0" | Normal | Courier Prime 12pt |
| Character | 3.5" | 3.5" | ALL CAPS | Courier Prime 12pt |
| Parenthetical | 2.9" | 2.0" | Normal | Courier Prime 12pt |
| Dialogue | 2.5" | 3.5" | Normal | Courier Prime 12pt |
| Transition | 4.0" | 2.0" | ALL CAPS | Courier Prime 12pt |
| Text Note | 1.5" | 6.0" | Normal, italic | Courier Prime 12pt |

---

## Performance Criteria

- Keystroke-to-render latency: < 16ms (60fps target, measured with Flutter DevTools)
- Editor cold load for a 120-page (≈ 12,000 words) script: < 3 seconds
- Block save debounce: 800ms; save API call completes in < 500ms
- Undo/redo: < 8ms per operation
- Scene heading autocomplete dropdown appears within 100ms of typing
- Memory: editor with 120-page script consumes < 150MB RAM on mid-range Android

---

## Yjs Integration

- Each `script_id` has one `Y.Doc` in memory
- Each block is a `Y.Map` keyed by `block_id` within `Y.Array blocks`
- Block `content` is a `Y.Text` — all edits are Yjs delta operations
- Undo manager: `Y.UndoManager` wraps the `blocks` Y.Array
- On editor open: fetch current Y.Doc state vector from Rust WebSocket endpoint via `GET /scripts/:id/ydoc`
- If WebSocket is available: sync state vector and receive missing updates
- If offline: use local Drift `CachedBlocks` as source of truth; Y.Doc reconstructed from serialised state stored in Drift

---

## Offline Behaviour

- All keystroke edits write to both Y.Doc in memory AND to `CachedBlocks` in Drift (serialised Y.Doc state)
- When the device comes back online, the offline write queue (`OfflineWriteQueue` in Drift) replays pending block updates to the Rust API in order
- If a conflict is detected (another user has edited the same block offline), Yjs CRDT merge resolves it automatically; no user intervention required
- The editor remains fully functional with no degraded experience during offline mode; the only indicator is the offline banner at the top

---

## Mobile Behaviour (375px)

- The formatting toolbar (7 block type buttons) is a horizontally scrollable strip above the keyboard, height 44px
- Scene navigator is accessed via the hamburger icon in the top-left — slides in as a bottom sheet (70% of screen height)
- AI sidebar collapses to a bottom sheet triggered by the AI icon in the toolbar
- Font size is 14px on mobile (from 12pt Courier Prime on desktop) to improve readability on small screens; layout margins are proportionally reduced
- The cursor is always kept visible above the keyboard using `MediaQuery.viewInsetsBottom`
- Long-press on selected text opens context menu: Bold, Italic, AI Suggest, Comment

---

## Error States

**E1 — Block Save Failure**  
`PATCH /blocks/:id` returns 4xx/5xx: the change is queued in `OfflineWriteQueue`; a subtle red dot appears on the editor status bar; no toast interrupts writing; retry happens automatically every 5 seconds.

**E2 — Script Not Found**  
`GET /scripts/:id/blocks` returns 404: full-screen error "This script no longer exists." with a "Back to Dashboard" button.

**E3 — Concurrent Edit Conflict**  
Yjs merge produces a visible conflict in rendered text (detected as identical position overlap): highlight the conflicting block in amber; show inline: "Merged conflict — review this block." The writer can accept or undo.

**E4 — Y.Doc Desync**  
The Rust server reports a state vector mismatch that cannot be resolved by incremental sync: `DELETE /scripts/:id/ydoc_reset` is called; the full Y.Doc is re-fetched and all in-memory edits since last server sync are replayed.

**E5 — Undo Stack Overflow**  
If the undo stack exceeds 200 operations: silently trim the oldest 50 operations from the bottom of the stack; no user notification.

---

## Security Considerations

- `GET /scripts/:id/blocks` verifies that the requesting user is the owner OR an accepted collaborator via JWT + RLS
- All block content is stored encrypted-at-rest in Supabase (column-level encryption for `content` column — AES-256)
- The editor never sends `ydoc_delta` to the MCP service — AI features receive a sanitised plain-text context only
- Rate limiting: max 600 block save requests per minute per user (10/second)
- Collaborator role enforcement: `viewer` role cannot call `PATCH /blocks/:id` — the Rust API enforces this regardless of client-side guards

---

## Human Authorship Considerations

- All text typed by the user has `content_source = 'human_written'`
- If an AI suggestion is accepted without editing, `content_source = 'ai_generated'`
- If a user edits an AI suggestion, `content_source = 'human_edited'`
- If a user types on top of a `human_confirmed` block, it reverts to `human_written`
- The AI badge (✦) is only shown for `ai_generated` and `ai_edited` blocks; it disappears on confirmation

---

## DB Tables Touched

- `scripts` (SELECT — load script metadata)
- `blocks` (SELECT all, INSERT new block, UPDATE content/position/type, DELETE)
- `scenes` (SELECT — for scene navigator; scene metadata)

---

## API Endpoints Used

- `GET /scripts/:id/blocks` — load all blocks
- `POST /blocks` — create a new block
- `PATCH /blocks/:id` — update block content or type
- `DELETE /blocks/:id` — delete a block
- `PATCH /blocks/reorder` — update positions after reorder
- `GET /scripts/:id/ydoc` — fetch current Y.Doc state (WebSocket upgrade)

---

## Test Cases

**TC-001:** Type "INT. LIVING ROOM - DAY" in a new scene heading → assert block saved with type `scene_heading` and text in ALL CAPS.  
**TC-002:** Enter from scene heading → assert new action block created and cursor placed in it.  
**TC-003:** Tab from action block → assert block type changes to `character` and text converts to ALL CAPS.  
**TC-004:** Type in dialogue block → pause 800ms → assert `PATCH /blocks/:id` called exactly once.  
**TC-005:** Cmd+Z after 3 edits → assert 3 undo operations revert correctly.  
**TC-006:** Load a 120-page script → measure cold load time < 3 seconds.  
**TC-007:** Keystroke in editor → measure render time < 16ms with Flutter DevTools.  
**TC-008:** Disable network → type 5 blocks → reconnect → assert all 5 blocks appear in server state.  
**TC-009:** Viewer-role collaborator tries to edit → assert API returns 403.  
**TC-010:** AI-generated block rendered → assert ✦ icon visible; confirm click → assert `content_source` changes to `human_confirmed`.

---

## Out of Scope

- Scene navigator panel (story-005)
- Focus Mode (story-006)
- AI sidebar (story-007)
- Collaboration cursors (story-010)
- Comment threads (story-011)
- Versioning (story-012)
- Find and Replace (story-026)
- Formatting preferences (story-027)
- Revision mode asterisks (story-029)
- Export (story-019, story-020)
