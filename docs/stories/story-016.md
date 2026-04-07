# story-016 — Character & Location Manager

| Field | Value |
|---|---|
| **Story ID** | story-016 |
| **Title** | Character & Location Manager |
| **Epic** | E06 — Story Tools |
| **Phase** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-007, story-008 |

---

## Problem Statement

Scripts with large casts and multiple locations require disciplined tracking. A character that enters on page 5 and reappears on page 87 must have consistent spelling, consistent traits, and consistent backstory. LIPILY's Character & Location Manager extends the Story Bible with detailed profile cards for each character and location, including scene appearance lists, note fields, and consistency flags.

---

## User Story

As a screenwriter, I want a dedicated Character & Location Manager where I can edit detailed profiles for each character and location in my script — so that I maintain consistency across a 120-page script without losing track of who's who.

---

## Acceptance Criteria

**GIVEN** the writer opens the Character tab in the Story Bible panel (story-008) and selects a character,  
**WHEN** the character detail view opens,  
**THEN** it shows: Name (editable), Archetype (editable dropdown: Hero / Mentor / Trickster / Shadow / Herald / Shapeshifter / Guardian), Description (editable multi-line text), Age (editable number), Motivation (editable text), Core Flaw (editable text), Scene Appearances (auto-populated list of scenes with this character), and a "Notes" free-text field.

**GIVEN** the writer edits any field in the character detail view,  
**WHEN** they leave the field (blur event),  
**THEN** `PATCH /story-bible/:entry_id` is called with the updated fields; the `content_source` updates to `human_written` for edited fields; success is silent (no toast for individual field saves — only a sync indicator).

**GIVEN** the writer clicks "Add Character" and enters a name that already exists (case-insensitive match),  
**WHEN** the save is attempted,  
**THEN** an inline warning appears: "A character named [name] already exists. Add anyway?" with Yes / Cancel options; if Yes is selected, both entries coexist (the writer can later merge them).

**GIVEN** a character appears in more than 5 scenes,  
**WHEN** the "Scene Appearances" section is viewed,  
**THEN** it shows a virtualised list of scenes with scene heading, scene number, and a "Jump to Scene" button; scenes are sorted by scene position.

**GIVEN** the writer opens the Locations tab,  
**WHEN** a location entry is selected,  
**THEN** the detail view shows: Name (editable), INT/EXT badge (toggle), Time-of-Day Pattern (editable: DAY / NIGHT / CONTINUOUS / LATER / MORNING), Description (editable), Scene Appearances (auto-populated), and Notes.

**GIVEN** the AI Continuity Checker (story-035) runs and detects a consistency issue for a character (e.g., same character referred to as "SARAH" in 10 scenes and "SARA" in 2 scenes),  
**WHEN** the issue is flagged,  
**THEN** a `continuity_flags` row is created; the character entry in the Character Manager shows a red "Inconsistency" badge; the detail view shows the flag: "Found 2 scenes with 'SARA' — possible name inconsistency."

**GIVEN** the writer wants to rename a character across the entire script,  
**WHEN** they type a new name in the character detail view and click "Rename in Script",  
**THEN** `POST /scripts/:id/characters/:char_id/rename` is called; all Character block occurrences of the old name in the script are updated to the new name via a batch `PATCH /blocks` call; the Story Bible entry is updated; a toast: "Renamed [old] → [new] in 34 locations."

**GIVEN** the writer wants to delete a character entry from the Story Bible,  
**WHEN** they confirm deletion,  
**THEN** the `story_bible_entries` row is deleted; the character's name is NOT removed from the script blocks (it remains in the script text); only the Story Bible card is removed.

**GIVEN** two characters are suspected duplicates (flagged by continuity checker),  
**WHEN** the writer clicks "Merge Characters" on the flagged entry,  
**THEN** a merge modal shows both entries' details side-by-side; the writer selects which name to keep; on confirm, all occurrences of the secondary name in Character blocks are renamed to the primary; the secondary Story Bible entry is deleted.

**GIVEN** a location has never been typed in the script (manually added),  
**WHEN** it is viewed in the Location Manager,  
**THEN** the "Scene Appearances" section shows "0 scenes" with a message: "This location hasn't appeared in the script yet."

---

## Performance Criteria

- Character detail view open: < 150ms
- Scene Appearances list load (50 scenes): < 200ms (virtualised)
- Rename across script (100 occurrences): < 2 seconds
- Field save: < 300ms (API round-trip)
- Merge characters: < 1 second (batch update)

---

## Offline Behaviour

- Character Manager reads from Drift cache offline
- Field edits are queued in `OfflineWriteQueue`
- "Rename in Script" and "Merge Characters" require internet; disabled with tooltip when offline

---

## Mobile Behaviour (375px)

- Character list: full-screen, one row per character (name + archetype + scene count badge)
- Tapping a character opens a detail modal (full-screen)
- All fields editable inline with standard text inputs
- Rename and Merge in "..." menu in the detail modal header

---

## Error States

**E1 — Rename Conflict**  
Two collaborators simultaneously rename the same character to different names: CRDT last-write-wins; a blue banner: "Character name was updated by [collaborator]. Check your recent changes."

**E2 — Merge with Active Editor Cursor in Character Block**  
If a collaborator's cursor is inside a character block being renamed in the merge: the rename applies; the collaborator's cursor jumps to the start of the renamed block; a toast: "[Name] renamed a character."

---

## Security Considerations

- Story Bible RLS applies (same as story-008)
- "Rename in Script" is a batch operation that only touches Character-type blocks; it cannot modify Action, Dialogue, or other block types
- The rename endpoint is rate-limited to 10 per minute per user

---

## Human Authorship Considerations

- Manually added fields are `human_written`; AI-extracted fields from Scene Intelligence are `ai_generated`; edited AI fields are `human_edited`

---

## DB Tables Touched

- `story_bible_entries` (SELECT, UPDATE fields, DELETE)
- `blocks` (UPDATE character name on rename)
- `scenes` (SELECT for Scene Appearances list)
- `continuity_flags` (SELECT for consistency badges)

---

## API Endpoints Used

- `GET /story-bible/:script_id` — load all entries (same as story-008)
- `PATCH /story-bible/:entry_id` — update character/location fields
- `POST /scripts/:id/characters/:char_id/rename` — bulk rename
- `POST /story-bible/:entry_id/merge` — merge duplicate entries
- `DELETE /story-bible/:entry_id` — delete entry

---

## Test Cases

**TC-001:** Open character detail → edit description → assert `PATCH` called with `human_written` source.  
**TC-002:** Add duplicate character name → assert inline warning shown.  
**TC-003:** Rename "JAMES" → "JIMMY" → assert all Character blocks updated; toast confirms count.  
**TC-004:** Merge two characters → assert secondary name replaced throughout script; secondary entry deleted.  
**TC-005:** Continuity flag for "SARA/SARAH" → assert red inconsistency badge on character entry.  
**TC-006:** Jump to scene from Scene Appearances → assert editor scrolls to correct scene.  
**TC-007:** Delete character entry → assert Story Bible card removed; script blocks unchanged.

---

## Out of Scope

- Character relationship graph (Phase 2)
- Character arc visualisation (Phase 3)
- Location map (Phase 2)
