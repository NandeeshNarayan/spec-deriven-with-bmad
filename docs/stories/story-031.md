# story-031 — Script Breakdown Auto-Generation

| Field | Value |
|---|---|
| **Story ID** | story-031 |
| **Title** | Script Breakdown Auto-Generation |
| **Epic** | E06 — Story Tools |
| **Phase** | 3 |
| **Sprint** | Sprint 4 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Dev (the director-writer) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-007, story-024 |

---

## Problem Statement

Script breakdowns — cataloguing every prop, costume, SFX, stunt, location, and cast member per scene — are required by every production before filming begins. Production coordinators currently do this manually, scene-by-scene. With inline tags from story-024 and Scene Intelligence data from story-007, LIPILY can auto-generate a professional breakdown that saves 20–40 hours of production coordinator work.

---

## User Story

As a director-writer preparing a production, I want LIPILY to auto-generate a script breakdown from my inline tags and Scene Intelligence data — so that my production team has a complete department-by-department breakdown without manual cataloguing.

---

## Acceptance Criteria

**GIVEN** a script has inline tags (story-024) and/or Scene Intelligence results (story-007),  
**WHEN** the user clicks "Generate Breakdown" from the script's context menu or the editor toolbar,  
**THEN** a `POST /scripts/:id/breakdown` job is dispatched to the MCP service; the breakdown is generated in the background; a "Generating breakdown…" progress indicator is shown; the user can navigate away and will be notified when complete.

**GIVEN** the breakdown job completes,  
**WHEN** the user opens the Breakdown view,  
**THEN** the breakdown is organised as a table with rows = scenes (ordered by scene number) and columns = department categories (PROPS, SFX, COSTUMES, STUNTS, LOCATIONS, CAST, VFX, ANIMALS, VEHICLES, MUSIC); each cell lists the tagged items from that scene for that category.

**GIVEN** a scene has no explicit inline tags but has Scene Intelligence data,  
**WHEN** the breakdown is generated,  
**THEN** the MCP service uses Claude to extract implied production needs from the action lines (e.g., "A gun on the table" → PROP: gun); these AI-extracted items are marked with ✦ in the breakdown.

**GIVEN** the breakdown is displayed,  
**WHEN** the user taps any cell,  
**THEN** they can edit the items in that cell (add/remove items); edits update the underlying `script_tags` table.

**GIVEN** the user clicks "Export Breakdown",  
**WHEN** the export dialog opens,  
**THEN** they can export as: PDF (formatted breakdown table), CSV (for import into production management software), or HTML; the export is generated server-side and returned as a download.

**GIVEN** the breakdown includes AI-extracted items (not from explicit tags),  
**WHEN** the breakdown is displayed,  
**THEN** AI items are shown with a ✦ icon; the writer can click ✓ to confirm an AI item (moves it to `human_confirmed`) or ✗ to remove it.

**GIVEN** a user is on the Free or Pro plan,  
**WHEN** they try to generate a breakdown,  
**THEN** a paywall modal: "Script Breakdown is available on Studio plan."

**GIVEN** the breakdown is regenerated after the writer adds more tags,  
**WHEN** "Regenerate Breakdown" is clicked,  
**THEN** the previous breakdown is archived; a new breakdown is generated with the latest tag data; the UI shows the new breakdown with the old one accessible under "Previous Breakdowns."

---

## Performance Criteria

- Breakdown generation (50 scenes, 500 tags): < 30 seconds
- Breakdown view render (50 scenes × 10 columns): < 500ms (virtualised table)
- CSV export: < 2 seconds
- PDF export: < 15 seconds

---

## Offline Behaviour

- Viewing a previously generated breakdown works offline (cached in Drift)
- Regeneration requires internet (MCP job)

---

## Mobile Behaviour (375px)

- Breakdown on mobile: horizontal scroll table; pinned scene column on left; department columns scroll right
- Each cell: truncated to 2 items; tap to expand full list
- Export options in "..." menu

---

## Error States

**E1 — No Tags and No Scene Intelligence**  
Script has neither inline tags nor Scene Intelligence: show message "No production data found. Add inline tags or run Scene Intelligence first to generate a breakdown."

**E2 — Breakdown Job Failure**  
MCP job fails after 3 retries: show "Breakdown generation failed. Please try again."; no partial breakdown shown.

---

## Security Considerations

- Breakdown data is stored per-script with same RLS as the script
- Studio-only feature; enforced server-side

---

## Human Authorship Considerations

- AI-extracted breakdown items are `ai_generated` until confirmed by the writer

---

## DB Tables Touched

- `script_tags` (SELECT — source data; INSERT AI-extracted tags)
- `scene_intelligence` (SELECT — for AI extraction context)
- `scenes` (SELECT — for scene ordering)

---

## API Endpoints Used

- `POST /scripts/:id/breakdown` — trigger breakdown generation
- `GET /scripts/:id/breakdown` — load generated breakdown
- `POST /scripts/:id/breakdown/export` — export PDF/CSV/HTML

---

## Test Cases

**TC-001:** Script with 20 tagged scenes → generate breakdown → assert all tags appear in correct cells.  
**TC-002:** Scene with no tags + Scene Intelligence → assert AI items appear with ✦.  
**TC-003:** Confirm AI item → assert ✦ removed; content_source = 'human_confirmed'.  
**TC-004:** Export as CSV → assert each row is a scene; columns are department categories.  
**TC-005:** Free user → assert Studio paywall shown.  
**TC-006:** Regenerate after adding tags → assert new breakdown created; old accessible.

---

## Out of Scope

- One-liner (one-line-per-scene summary export) — Phase 2
- Scheduling integration (Phase 3)
