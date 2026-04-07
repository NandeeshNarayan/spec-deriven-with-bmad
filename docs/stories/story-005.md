# story-005 — Scene Navigator

| Field | Value |
|---|---|
| **Story ID** | story-005 |
| **Title** | Scene Navigator |
| **Epic** | E01 — Core Editor |
| **Phase** | 1 |
| **Sprint** | Sprint 1 |
| **Priority** | P0 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Feature film scripts average 120 pages with 80–120 scenes. Writers need to jump between scenes instantly, reorder them without copy-pasting, and see structural context (scene number, location, time of day) at a glance. Without a scene navigator, writers lose hours scrolling linearly through long scripts.

---

## User Story

As a screenwriter, I want a scene navigator panel that shows all my scenes as draggable cards so that I can jump to any scene instantly, reorder scenes by dragging, and understand my script's structure at a glance.

---

## Acceptance Criteria

**GIVEN** the editor is open and the user taps the scene navigator toggle (panel icon in the top-left),  
**WHEN** the panel opens,  
**THEN** a left sidebar (desktop: 280px fixed width; mobile: bottom sheet at 70% height) shows a vertically scrollable list of all scene cards ordered by `position`, each showing: scene number, scene heading text, scene type badge (INT/EXT), time of day, and AI-generated one-line summary (from `scene_intelligence.scene_summary` if available, else "—").

**GIVEN** the scene navigator is open and the writer clicks or taps a scene card,  
**WHEN** the tap is registered,  
**THEN** the editor scrolls smoothly to that scene's first block; the scene card in the navigator highlights with a primary-colour left border; the transition animation is < 300ms.

**GIVEN** the user drags a scene card up or down in the navigator,  
**WHEN** the drag is released,  
**THEN** the scene's `position` field is updated via `PATCH /scenes/:id` with the new position value using gap strategy (existing positions maintain their gap); all blocks within the scene maintain their relative order; other writers in the same script see the reorder reflected within 2 seconds via Yjs sync.

**GIVEN** the writer right-clicks (desktop) or long-presses (mobile) a scene card,  
**WHEN** the context menu opens,  
**THEN** it shows: "Add Scene Above", "Add Scene Below", "Duplicate Scene", "Delete Scene", "Collapse Scene" (hides blocks in editor view, keeps scene card visible).

**GIVEN** the user selects "Add Scene Below",  
**WHEN** the new scene is created,  
**THEN** a new `scene_heading` block is inserted below the last block of the selected scene with an empty scene heading; the editor scrolls to and focuses the new block; the scene navigator adds a new card in the correct position.

**GIVEN** the user selects "Delete Scene",  
**WHEN** they confirm in the modal ("Delete this scene? All blocks will be moved to Recently Deleted."),  
**THEN** all blocks within the scene are soft-deleted; the scene row is soft-deleted; the navigator card disappears; an undo toast appears for 5 seconds.

**GIVEN** the script has a scene with an AI-generated summary in `scene_intelligence`,  
**WHEN** the scene card is rendered in the navigator,  
**THEN** the one-line summary appears below the scene heading in 11px grey text; a ✦ icon precedes it to indicate AI origin; the summary truncates at 80 characters with an ellipsis.

**GIVEN** the user types in the scene heading block in the editor,  
**WHEN** the heading text changes,  
**THEN** the corresponding scene card in the navigator updates in real time (within 1 frame) without requiring a save or page refresh.

**GIVEN** the script has scenes with act structure labels (from story-017 Beat Board),  
**WHEN** the scene navigator is open,  
**THEN** act dividers ("ACT ONE", "ACT TWO", "ACT THREE") appear as non-draggable section headers between the relevant scene cards.

**GIVEN** the scene navigator is open on a 375px mobile screen,  
**WHEN** the bottom sheet is visible,  
**THEN** each scene card is 72px tall; the scene number is left-aligned in 12px bold; the heading text is truncated to one line; the drag handle icon is on the right side of each card.

---

## Performance Criteria

- Navigator open animation: < 300ms
- Scene jump (click-to-scroll): < 100ms to start scroll, < 300ms to complete
- 120-scene list rendering: < 200ms initial render using ListView.builder (virtualised)
- Drag-and-drop reorder: 60fps during drag; position save completes within 500ms of release
- Real-time heading update (editor → navigator): < 16ms (same frame)

---

## Offline Behaviour

- Scene navigator reads from Drift `CachedScenes` table when offline
- Reorder operations are queued in `OfflineWriteQueue` and synced when online
- Add Scene and Delete Scene are also queued offline
- AI summaries that haven't been fetched yet show "—" in offline mode

---

## Mobile Behaviour (375px)

- Scene navigator opens as a bottom sheet triggered by the ☰ icon in the top-left of the editor toolbar
- Bottom sheet height: 70% of screen height; drag handle visible at top
- Scene cards: 72px tall, full width minus 16px each side
- Drag handles: ReorderableListView used; long-press to initiate drag on mobile
- "Add Scene" FAB at bottom of the sheet: adds a new scene at the bottom

---

## Error States

**E1 — Reorder Conflict**  
If two collaborators simultaneously drag the same scene: CRDT resolves by last-write-wins on position; a brief amber flash on the moved card indicates a remote update; no error shown.

**E2 — Scene Delete While Collaborator is In It**  
If a collaborator's cursor is inside a scene being deleted: their editor shows an alert banner "This scene was deleted by [name]. You've been moved to the next scene." and their cursor moves to the adjacent scene's first block.

**E3 — Navigator Load Failure**  
If `GET /scripts/:id/scenes` fails: the navigator shows "Couldn't load scenes." with a retry button; the editor itself remains fully functional.

**E4 — Script with Zero Scenes**  
If the script has no scene heading blocks yet: the navigator shows an empty state: "No scenes yet. Start by typing a scene heading (INT. or EXT.) in the editor." with a "Jump to Editor" button.

---

## Security Considerations

- Scene reorder calls `PATCH /scenes/:id` which verifies editor or owner role
- Viewer-role collaborators see the navigator but all context menu items (except "Jump to scene") are disabled and greyed out
- AI summary text in scene cards is rendered as plain text (no HTML injection possible)

---

## Human Authorship Considerations

- AI summaries shown in scene cards are clearly marked with ✦; they are read-only in the navigator
- Users can tap the ✦ icon on a summary to open the Scene Intelligence sidebar for that scene

---

## DB Tables Touched

- `scenes` (SELECT all by script_id, UPDATE position, INSERT new, soft DELETE)
- `blocks` (SELECT scene_heading blocks for heading text; soft DELETE on scene delete)
- `scene_intelligence` (SELECT scene_summary for display in cards)

---

## API Endpoints Used

- `GET /scripts/:id/scenes` — list all scenes with metadata and summaries
- `POST /scenes` — add new scene
- `PATCH /scenes/:id` — rename or reorder scene
- `DELETE /scenes/:id` — soft-delete scene and its blocks
- `POST /scenes/:id/restore` — undo scene delete

---

## Test Cases

**TC-001:** Open editor with 10 scenes → assert navigator shows 10 cards in correct order.  
**TC-002:** Tap scene card 7 → assert editor scrolls to scene 7 in < 300ms.  
**TC-003:** Drag scene 3 above scene 1 → assert database positions updated correctly.  
**TC-004:** Add Scene Below on scene 5 → assert new scene card appears at position 6 and editor focuses new block.  
**TC-005:** Delete scene with 3 blocks → assert scene and all 3 blocks soft-deleted; undo restores all 4.  
**TC-006:** Type in scene heading in editor → assert navigator card heading updates in real time.  
**TC-007:** Load 120-scene script → assert navigator initial render < 200ms.  
**TC-008:** Open navigator on 375px mobile → assert bottom sheet opens and cards are fully readable.

---

## Out of Scope

- Beat board structural overlays on scene cards (story-017)
- Scene colour coding by tag (story-024)
- Scene statistics (page count per scene, shooting time estimate) (story-031)
- Scene collapse in editor (stubs the action; full collapse deferred to a future sprint)
