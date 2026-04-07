# story-027 — Script Formatting Preferences

| Field | Value |
|---|---|
| **Story ID** | story-027 |
| **Title** | Script Formatting Preferences |
| **Epic** | E09 — Editor Utilities |
| **Phase** | 2 |
| **Sprint** | Sprint 4 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Marcus (the Hollywood rewrite specialist) |
| **Estimated Effort** | 2 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Different markets have slightly different formatting conventions. UK television uses different dialogue indentation than Hollywood features. Some international formats prefer different page sizes (A4 vs US Letter). Some writers with dyslexia benefit from increased line spacing. Formatting preferences let writers adjust these parameters per-script without breaking WGA compliance for US submissions.

---

## User Story

As a professional screenwriter, I want to customise formatting preferences per script — so that I can match regional formatting conventions or personal readability needs without manually adjusting every block.

---

## Acceptance Criteria

**GIVEN** the writer opens "Script Settings" (from the editor header or script card context menu),  
**WHEN** the Formatting Preferences panel opens,  
**THEN** it shows the following adjustable parameters with their default values:
- Page Size: US Letter (8.5×11) / A4 (default: US Letter)
- Font: Courier Prime 12pt / Courier New 12pt / Courier Final Draft 12pt (default: Courier Prime)
- Line Spacing: Normal (1.0) / 1.15 / 1.5 (default: 1.0)
- Scene Heading Left Margin: 1.5" (adjustable: 1.0" – 2.0")
- Dialogue Left Margin: 2.5" (adjustable: 2.0" – 3.0")
- Dialogue Width: 3.5" (adjustable: 3.0" – 4.5")
- Character Centre Position: 3.7" (adjustable: 3.0" – 4.5")
- Smart Flow Behaviour: Standard (WGA) / Minimal (Tab-only, no auto block insert)

**GIVEN** the writer changes any parameter,  
**WHEN** the change is made,  
**THEN** the editor preview updates in real time to show the adjusted formatting; changes are saved to `formatting_preferences` table via `PATCH /scripts/:id/formatting` on each change.

**GIVEN** a writer changes the page size to A4,  
**WHEN** the PDF export runs,  
**THEN** the exported PDF uses A4 page dimensions (297mm × 210mm); all margins are recalculated proportionally; the WGA-compliant layout is maintained relative to the page size.

**GIVEN** a writer selects "Minimal" Smart Flow Behaviour,  
**WHEN** they are typing in the editor,  
**THEN** pressing Enter does NOT auto-insert a new block type (it inserts a line break within the current block); pressing Tab cycles manually through block types (same as Vim's block mode); no automatic transitions occur.

**GIVEN** a writer increases line spacing to 1.5,  
**WHEN** the editor re-renders,  
**THEN** all blocks render with 1.5× line spacing; the page count estimate updates (more lines per word count); the PDF export respects the line spacing.

**GIVEN** the writer opens a script and no `formatting_preferences` row exists,  
**WHEN** the editor loads,  
**THEN** all defaults are used; a `formatting_preferences` row is created on first explicit save.

**GIVEN** a collaborator changes formatting preferences (editor role),  
**WHEN** the change is saved,  
**THEN** it applies to the script for ALL editors (formatting is per-script, not per-user); the script owner is shown a notification: "[Name] changed the script formatting preferences."

**GIVEN** a writer wants to reset all formatting to WGA defaults,  
**WHEN** they click "Reset to WGA Defaults",  
**THEN** all parameters are reset to their default values; the editor re-renders.

---

## Performance Criteria

- Formatting Preferences panel open: < 100ms
- Live preview update per parameter change: < 100ms
- PDF export with custom formatting: same < 30s SLA as story-019

---

## Offline Behaviour

- Formatting preferences are stored in Drift alongside the script
- Changes are queued in `OfflineWriteQueue` offline
- The editor applies local preferences even offline

---

## Mobile Behaviour (375px)

- Script Settings accessed from the "..." menu in the editor toolbar
- Opens as a full-screen bottom sheet with grouped sections: Page, Typography, Layout, Smart Flow
- Each setting: a row with label + control (toggle, slider, or segmented control)

---

## Error States

**E1 — Conflicting Collaborator Change**  
Two collaborators change different formatting parameters simultaneously: both changes persist (no conflict — last-write-wins per field).

**E2 — Invalid Margin Combination**  
Dialogue width + left margin exceeds page width: client validates and shows inline error "These margins exceed the page width. Please adjust."

---

## Security Considerations

- `formatting_preferences` has RLS: SELECT for all collaborators; UPDATE for owner and editor-role only

---

## Human Authorship Considerations

Not applicable.

---

## DB Tables Touched

- `formatting_preferences` (SELECT, INSERT defaults, UPDATE)

---

## API Endpoints Used

- `GET /scripts/:id/formatting` — load formatting preferences
- `PATCH /scripts/:id/formatting` — update formatting preferences

---

## Test Cases

**TC-001:** Change page size to A4 → export PDF → assert A4 dimensions in PDF.  
**TC-002:** Change line spacing to 1.5 → assert editor re-renders with correct spacing.  
**TC-003:** Select "Minimal" Smart Flow → press Enter in editor → assert no auto block insertion.  
**TC-004:** Reset to WGA Defaults → assert all parameters return to default values.  
**TC-005:** Collaborator changes formatting → assert script owner notified.  
**TC-006:** No formatting_preferences row → open editor → assert defaults applied.

---

## Out of Scope

- Per-user formatting preferences (currently per-script only)
- Custom font upload (Phase 2)
- Custom margin presets / templates (Phase 2)
