# story-017 — Beat Board

| Field | Value |
|---|---|
| **Story ID** | story-017 |
| **Title** | Beat Board |
| **Epic** | E06 — Story Tools |
| **Phase** | 2 |
| **Sprint** | Sprint 3 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-005 |

---

## Problem Statement

Structural frameworks (3-Act Structure, Save the Cat, Hero's Journey) are the professional vocabulary of script development. Writers need to map their scenes to these frameworks to understand structural weaknesses before writing 120 pages of a mis-structured script. The Beat Board is a visual card-based view of all scenes mapped to beat frameworks, with drag-and-drop reordering and AI-assisted beat mapping.

---

## User Story

As a screenwriter, I want a Beat Board view that maps all my scenes to structural frameworks (3-Act, Save the Cat) — so that I can see structural weaknesses and reorder scenes at a high level before committing to page-level writing.

---

## Acceptance Criteria

**GIVEN** a writer opens the Beat Board (accessible from the top navigation or Cmd+B),  
**WHEN** the view loads,  
**THEN** all scenes are displayed as draggable cards in a horizontal Kanban-style layout; the layout is divided into structural columns based on the selected framework; the default framework is "3-Act Structure".

**GIVEN** the "3-Act Structure" framework is selected,  
**WHEN** the Beat Board renders,  
**THEN** three columns are shown: "Act One (Setup)", "Act Two (Confrontation)", "Act Three (Resolution)"; each column shows the scenes assigned to it; unassigned scenes appear in a grey "Unassigned" column on the left.

**GIVEN** the "Save the Cat" framework is selected,  
**WHEN** the Beat Board renders,  
**THEN** fifteen columns are shown corresponding to Blake Snyder's 15 beats: Opening Image, Theme Stated, Set-Up, Catalyst, Debate, Break into Two, B Story, Fun and Games, Midpoint, Bad Guys Close In, All Is Lost, Dark Night of the Soul, Break into Three, Finale, Final Image.

**GIVEN** a scene card is dragged from one column to another,  
**WHEN** the drop is confirmed,  
**THEN** the scene's `beat_board_assignment` field in `scenes` table is updated via `PATCH /scenes/:id`; if the framework is "3-Act", the scene's `act` is updated; the card animates to its new column; the scene navigator (story-005) shows the updated act dividers.

**GIVEN** the writer clicks on a scene card in the Beat Board,  
**WHEN** the card detail expands,  
**THEN** it shows: scene heading, AI scene summary (from `scene_intelligence`), emotional arc label (from Scene Intelligence `emotionalArc`), scene page count estimate, and a "Jump to Scene" button.

**GIVEN** the writer enables "AI Beat Mapping" (Pro/Studio feature),  
**WHEN** the Beat Board view opens with a script that has complete Scene Intelligence data for all scenes,  
**THEN** a `POST /ai/beat-mapping` job is enqueued; the MCP service analyses all scene summaries and emotional arcs and suggests assignments for unassigned scenes; suggestions appear as ghost card positions (dashed border); the writer can accept or dismiss each suggestion.

**GIVEN** the AI suggests a beat assignment for a scene,  
**WHEN** the writer taps "Accept" on the ghost card,  
**THEN** the assignment is confirmed; `content_source` changes from `ai_generated` to `human_confirmed`; the card becomes a solid card in the column; the `agent_approvals` table is updated.

**GIVEN** the Beat Board is open and the writer is in a framework column with more than 8 scene cards,  
**WHEN** the column renders,  
**THEN** the cards are displayed in a vertically scrollable column with a card height of 72px; the column header shows a count badge.

**GIVEN** the writer uses the Beat Board to reorder a scene,  
**WHEN** the scene is dropped to a new column position,  
**THEN** the corresponding scene in the scene navigator (story-005) also updates its position to match the beat board order; the editor's block order is updated accordingly.

**GIVEN** the writer is on a Free plan and opens the Beat Board,  
**WHEN** the Beat Board loads,  
**THEN** the AI Beat Mapping toggle is locked with an upgrade prompt; manual beat assignment (dragging cards) works for Free users.

---

## Performance Criteria

- Beat Board initial load (50 scenes): < 500ms
- Drag-and-drop animation: 60fps
- AI beat mapping job: < 15 seconds for 50 scenes
- Scene card click-to-detail expand: < 100ms

---

## Offline Behaviour

- Beat Board reads from Drift cache offline (shows last-synced assignments)
- Drag-and-drop reorder is queued in `OfflineWriteQueue` offline
- AI Beat Mapping not available offline

---

## Mobile Behaviour (375px)

- Beat Board on mobile: horizontal scroll (ScrollView) with columns as fixed-width cards (200px per column)
- Drag-and-drop uses a long-press gesture to initiate drag
- "Save the Cat" framework: columns scroll horizontally across the full 15 beats

---

## Error States

**E1 — AI Beat Mapping No Scene Intelligence**  
If fewer than 50% of scenes have Scene Intelligence data: show warning "AI Beat Mapping requires Scene Intelligence for most scenes. Run Scene Intelligence first." with a "Run Scene Intelligence" button.

**E2 — Beat Mapping Job Failure**  
MCP service fails: sidebar shows "Beat mapping temporarily unavailable." Manual assignment still works.

---

## Security Considerations

- `beat_board_assignment` column on `scenes` follows same RLS as the scene itself
- AI Beat Mapping suggestions are advisory only and are never auto-applied

---

## Human Authorship Considerations

- Manual assignments are `human_written`; AI suggestions are `ai_generated` until confirmed

---

## DB Tables Touched

- `scenes` (UPDATE `beat_board_assignment`, `act`)
- `scene_intelligence` (SELECT summaries and emotional arcs)
- `beat_notes` (INSERT/UPDATE beat notes per scene)

---

## API Endpoints Used

- `GET /scripts/:id/scenes` — load scenes with beat assignments
- `PATCH /scenes/:id` — update beat assignment
- `POST /ai/beat-mapping` — enqueue AI beat mapping job

---

## Test Cases

**TC-001:** Open Beat Board → assert all scenes appear in their assigned columns.  
**TC-002:** Drag scene from Act One to Act Two → assert `PATCH` called; scene navigator shows updated position.  
**TC-003:** Select "Save the Cat" → assert 15 columns render correctly.  
**TC-004:** AI Beat Mapping run → assert suggestions appear as ghost cards.  
**TC-005:** Accept AI suggestion → assert card becomes solid; `human_confirmed` source.  
**TC-006:** Free user opens Beat Board → assert AI toggle locked with upgrade prompt; manual drag works.

---

## Out of Scope

- Hero's Journey framework (Phase 2)
- Custom beat framework builder (Phase 3)
- Beat Board export as PDF outline (Phase 2)
