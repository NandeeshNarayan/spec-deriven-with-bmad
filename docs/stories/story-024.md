# story-024 — Inline Scene Tagging

| Field | Value |
|---|---|
| **Story ID** | story-024 |
| **Title** | Inline Scene Tagging |
| **Epic** | E09 — Editor Utilities |
| **Phase** | 2 |
| **Sprint** | Sprint 3 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Marcus (the Hollywood rewrite specialist) |
| **Estimated Effort** | 2 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Production breakdowns require scenes to be tagged by department need: props, costumes, special effects, stunts, locations, cast. Without in-script tagging, production managers build breakdowns manually from scratch. Inline tags let writers (or line producers with editor access) mark specific lines directly in the script — making automatic production breakdowns possible (story-031).

---

## User Story

As a screenwriter or line producer, I want to tag specific lines in the script with production categories (PROP, SFX, COSTUME, STUNT, etc.) — so that production breakdowns can be generated automatically from my tags.

---

## Acceptance Criteria

**GIVEN** a user selects text in any block or right-clicks a block,  
**WHEN** they select "Tag" from the context menu (or use Cmd+Shift+T),  
**THEN** a tag picker dropdown appears showing all available tag categories: PROP, SFX, COSTUME, STUNT, LOCATION, CAST, VFX, ANIMAL, VEHICLE, MUSIC; the user can select one or more tags.

**GIVEN** the user selects "PROP" from the tag picker,  
**WHEN** the tag is applied,  
**THEN** a `script_tags` row is inserted with `block_id`, `tag_type = 'PROP'`, `tagged_text` (the selected text or the full block content if no selection), `scene_id`, `created_by`; the tagged text is underlined with a colour-coded underline (each tag type has a distinct colour from a fixed palette).

**GIVEN** a tagged block is rendered in the editor,  
**WHEN** the tag underline is visible,  
**THEN** hovering over (desktop) or long-pressing (mobile) the underline shows a tooltip with the tag type and the username who applied it; multiple tags on the same text show a multi-colour underline stack.

**GIVEN** the user opens the Tags panel (accessible from the editor right sidebar or Cmd+Shift+T),  
**WHEN** the panel loads,  
**THEN** all tags across the script are listed, grouped by tag type; each entry shows: tagged text, scene heading, scene number, created by; there are filter buttons for each tag type.

**GIVEN** a viewer-role collaborator is in the editor,  
**WHEN** they try to add a tag,  
**THEN** the Tag option is disabled in the context menu with tooltip "You need editor access to tag."

**GIVEN** the tag panel is open and the user clicks a tag entry,  
**WHEN** the click registers,  
**THEN** the editor scrolls to the scene and block containing that tag; the tagged text is briefly highlighted in amber.

**GIVEN** a user wants to remove a tag,  
**WHEN** they right-click the underlined text and select "Remove Tag",  
**THEN** the `script_tags` row is deleted; the underline disappears.

**GIVEN** the script is exported to PDF (story-019),  
**WHEN** the PDF renders,  
**THEN** by default, tag underlines are NOT shown in the PDF (the PDF is the clean submission copy); if the writer enables "Show Production Tags" in the export dialog, tags appear as underlines in the PDF.

**GIVEN** the Tags panel shows more than 100 tags,  
**WHEN** the panel renders,  
**THEN** the list is virtualised; a count badge shows the total; the filter bar reduces the list to the selected type.

**GIVEN** the Script Breakdown is generated (story-031),  
**WHEN** it is generated,  
**THEN** all `script_tags` entries for the script are grouped by scene and by tag type; the tags from this story are the data source for the breakdown story.

---

## Performance Criteria

- Tag picker dropdown open: < 100ms
- Tag apply (API + optimistic UI): < 300ms
- Tags panel load (100 tags): < 200ms (virtualised)
- Underline render: no jank; all tags rendered in < 16ms per frame

---

## Offline Behaviour

- Tags are applied optimistically and queued in `OfflineWriteQueue` offline
- Tags panel reads from Drift cache offline
- Tag removal is also queued offline

---

## Mobile Behaviour (375px)

- Tag triggered by long-pressing a block → "Tag" in context menu
- Tag picker opens as a bottom sheet with a grid of tag type buttons (4-column grid, icon + label)
- Tags panel accessible from the editor toolbar via tag icon

---

## Error States

**E1 — Tag Apply Conflict**  
Two collaborators tag the same text simultaneously with different tags: both tags are applied (tags are additive — multiple tags on same text are supported).

**E2 — Block Deleted While Tagged**  
If a tagged block is deleted: the `script_tags` row is soft-deleted along with the block; it no longer appears in the Tags panel.

---

## Security Considerations

- `script_tags` RLS: SELECT for all collaborators; INSERT/DELETE for editor/owner only
- Viewer cannot tag; enforced server-side

---

## Human Authorship Considerations

- Tags are always `human_written` (no AI tagging in v1)
- AI-assisted auto-tagging from Scene Intelligence data is a Phase 2 feature

---

## DB Tables Touched

- `script_tags` (SELECT, INSERT, DELETE)
- `blocks` (SELECT — for context)
- `scenes` (SELECT — for scene reference in tags panel)

---

## API Endpoints Used

- `POST /tags` — apply tag to block/text
- `DELETE /tags/:id` — remove tag
- `GET /scripts/:id/tags` — list all tags for a script

---

## Test Cases

**TC-001:** Select text → Tag → PROP → assert `script_tags` row created; underline appears with PROP colour.  
**TC-002:** Hover tag underline → assert tooltip shows tag type and tagger username.  
**TC-003:** Open Tags panel → filter by SFX → assert only SFX tags shown.  
**TC-004:** Click tag in Tags panel → assert editor scrolls to the tagged block.  
**TC-005:** Remove tag → assert underline disappears and `script_tags` row deleted.  
**TC-006:** Export PDF without "Show Production Tags" → assert no underlines in PDF.  
**TC-007:** Viewer tries to tag → assert option disabled.  
**TC-008:** Apply tag offline → reconnect → assert tag synced to server.

---

## Out of Scope

- AI-assisted auto-tagging (Phase 2)
- Custom tag types beyond the 10 predefined (Phase 2)
- Script Breakdown generation (story-031)
