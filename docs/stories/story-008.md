# story-008 — Story Bible Panel

| Field | Value |
|---|---|
| **Story ID** | story-008 |
| **Title** | Story Bible Panel |
| **Epic** | E06 — Story Tools |
| **Phase** | 1 |
| **Sprint** | Sprint 1 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-007 |

---

## Problem Statement

Professional screenwriters maintain a "story bible" — a living document tracking all characters, locations, themes, and the story's core premise. Without it, scripts grow inconsistent: characters change motivation mid-story, locations are described differently each time, and themes drift. LIPILY's Story Bible auto-populates from Scene Intelligence outputs, saving writers from manual bookkeeping while keeping the creative intelligence in the writer's hands.

---

## User Story

As a screenwriter, I want a Story Bible panel that automatically extracts and tracks characters, locations, themes, and the logline from my script — so that I have a living reference document that keeps my story consistent without manually maintaining notes.

---

## Acceptance Criteria

**GIVEN** the Scene Intelligence job for any scene completes with identified characters or locations,  
**WHEN** the MCP service posts the results back to the Rust API,  
**THEN** the Rust API's internal story bible aggregation logic runs: new character names are inserted into `story_bible_entries` with `entry_type = 'character'`; new location names are inserted with `entry_type = 'location'`; duplicate entries are not created (upsert by `entry_name + script_id`).

**GIVEN** the writer opens the Story Bible panel (accessible from the editor right sidebar),  
**WHEN** the panel loads,  
**THEN** it displays four tabs: Characters, Locations, Themes, and Premise; each tab shows the relevant entries sorted alphabetically; each entry card shows the entry name, a brief AI-extracted description, and the scenes where it appears.

**GIVEN** the Characters tab is open,  
**WHEN** the writer views a character entry,  
**THEN** it shows: character name (ALL CAPS), archetype (AI-extracted, e.g., "The Mentor"), first appearance scene, number of scenes featured in, and a description field that is editable by the writer.

**GIVEN** the writer taps the description field on any Story Bible entry and types a new description,  
**WHEN** they finish editing and the field loses focus,  
**THEN** the description is saved via `PATCH /story-bible/:entry_id`; the `content_source` of the entry is updated to `human_written`; the ✦ AI badge is removed from the description field; the update is reflected immediately across all collaborators' Story Bible panels.

**GIVEN** the writer taps the "Add Character" button in the Characters tab,  
**WHEN** the modal opens,  
**THEN** it shows fields for: Name, Archetype (free text), Description, Notes (free text); on save, a new `story_bible_entries` row is created with `content_source = 'human_written'` and is immediately visible in the panel.

**GIVEN** the writer right-clicks (desktop) or long-presses (mobile) a story bible entry,  
**WHEN** the context menu opens,  
**THEN** it shows: Edit, Delete, "Find in Script" (jumps to first scene featuring this character/location).

**GIVEN** a writer has the Story Bible panel open and a collaborator adds a new character in the editor,  
**WHEN** Scene Intelligence re-runs for that scene and detects the new character,  
**THEN** the new character entry appears in the Characters tab in real time (via Yjs sync), highlighted briefly in amber to indicate it was just added.

**GIVEN** the Themes tab is open,  
**WHEN** the writer views it,  
**THEN** it shows a list of AI-extracted thematic keywords (e.g., "Redemption", "Family", "Power") each with the scenes where the theme appears and a confidence score (the average `thematic_resonance.score` across scenes featuring this theme).

**GIVEN** the Premise tab is open,  
**WHEN** the writer views it,  
**THEN** it shows three editable fields: Logline (1–2 sentence pitch), Central Conflict (1 sentence), and Core Theme (1 sentence); these fields default to AI-extracted values from Scene Intelligence; the writer can edit each field individually.

**GIVEN** the Story Bible panel is open on a 375px mobile screen,  
**WHEN** it renders,  
**THEN** the panel is a full-screen bottom sheet with a tab bar at the top (Characters / Locations / Themes / Premise); each entry is a list tile 72px tall with name and description preview; tapping opens a detail modal.

---

## Performance Criteria

- Story Bible panel open (with up to 50 characters): < 300ms
- Entry description save: < 500ms (API round-trip)
- Real-time new entry appearance (from Scene Intelligence): < 2 seconds from job completion
- "Find in Script" navigation: < 100ms to scroll to scene

---

## Offline Behaviour

- Story Bible entries are cached in the Drift database
- Read-only display works fully offline from cache
- Edits to entry descriptions are queued in `OfflineWriteQueue`
- New manual entries added offline are queued and synced on reconnect

---

## Mobile Behaviour (375px)

- Story Bible accessed via the "Book" icon in the mobile editor bottom toolbar
- Opens as a full-screen bottom sheet
- Tab bar: Characters / Locations / Themes / Premise
- Each character card: name in bold, archetype in grey, 1-line description preview
- Edit: taps into a full-screen edit modal with a save button

---

## Error States

**E1 — Story Bible Load Failure**  
`GET /story-bible/:script_id` returns 5xx: show "Couldn't load Story Bible" with a retry button inside the panel; the editor remains functional.

**E2 — Duplicate Character Auto-Detected**  
Scene Intelligence detects "JAMES" and "JIM" who appear to be the same character (>70% co-occurrence in same scenes): the Rust API merges them into one entry with both names listed as aliases; a blue info banner in the panel: "Merged duplicate character: JAMES / JIM."

**E3 — Entry Delete Conflict**  
Writer deletes a character that another collaborator is currently editing: the delete wins (last-write-wins); the other collaborator's edit panel shows: "This entry was deleted by [name]."

---

## Security Considerations

- Story Bible entries have RLS: only the script owner and accepted collaborators can read and write entries
- Viewer-role collaborators can read the Story Bible but cannot add, edit, or delete entries (all edit controls are disabled for viewers)
- MCP service inserts entries via the Rust internal endpoint only — it cannot directly write to `story_bible_entries`

---

## Human Authorship Considerations

- AI-extracted entries are marked `content_source = 'ai_generated'`; the ✦ badge appears on the description field
- Writer-edited descriptions change to `human_written`; ✦ badge disappears
- Manually added entries are always `human_written`
- The Story Bible is a writer's intellectual property — AI content is advisory only and always clearly labelled

---

## DB Tables Touched

- `story_bible_entries` (SELECT all by script_id, INSERT new entry, UPDATE description, DELETE)
- `scene_intelligence` (SELECT — for theme data and scene appearance counts)
- `scenes` (SELECT — for "scenes featuring this character/location" count)

---

## API Endpoints Used

- `GET /story-bible/:script_id` — load all story bible entries
- `POST /story-bible` — add manual entry
- `PATCH /story-bible/:entry_id` — update entry description
- `DELETE /story-bible/:entry_id` — delete entry

---

## Test Cases

**TC-001:** Run Scene Intelligence on a scene with a new character "SARAH" → assert `story_bible_entries` row created for SARAH.  
**TC-002:** Same character appears in 2 scenes → assert single entry, not duplicate.  
**TC-003:** Edit description in Story Bible → assert `content_source` changes to `human_written` and ✦ disappears.  
**TC-004:** Add manual character → assert new entry appears with `human_written` source and no ✦.  
**TC-005:** "Find in Script" on a character → assert editor scrolls to first scene featuring that character.  
**TC-006:** Open panel offline → assert cached entries are shown with offline indicator.  
**TC-007:** Collaborator adds character → assert it appears in real time in the other writer's Story Bible panel.  
**TC-008:** Viewer-role user opens Story Bible → assert edit/delete controls are disabled.

---

## Out of Scope

- Character relationship graph (Phase 2 / v2)
- Location map or floor plan visualisation (Phase 2 / v2)
- Full story bible export as PDF (Phase 2 / v2)
- Character arc timeline (Phase 3 / v3)
