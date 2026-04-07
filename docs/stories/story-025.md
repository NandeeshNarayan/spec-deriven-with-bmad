# story-025 — Title Page Editor

| Field | Value |
|---|---|
| **Story ID** | story-025 |
| **Title** | Title Page Editor |
| **Epic** | E09 — Editor Utilities |
| **Phase** | 2 |
| **Sprint** | Sprint 3 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 2 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Every professional screenplay submission requires a properly formatted title page: title, writer credit, contact information, WGA registration number (optional), draft date. Without a title page editor, writers must add this information manually in Final Draft or Word before submission — defeating the purpose of a complete professional tool.

---

## User Story

As a screenwriter, I want to edit and preview my script's title page — so that my submission PDFs include a professional, properly formatted title page with my contact information and credits.

---

## Acceptance Criteria

**GIVEN** the writer is in the editor and opens "Title Page" (from the editor header menu or the script's context menu),  
**WHEN** the Title Page editor opens,  
**THEN** it shows a WYSIWYG preview of the title page on the left (rendered as it will appear in the PDF) and an edit panel on the right with fields: Title (pre-filled), Written By (single author or "Written by [A] & [B]"), Contact Name, Contact Address, Contact Email, Contact Phone, WGA Registration Number (optional), Draft Date (date picker), Based On (optional — "Based on [original work] by [author]").

**GIVEN** the writer edits any field in the title page editor,  
**WHEN** they type,  
**THEN** the WYSIWYG preview updates in real time (< 100ms debounce); the changes are saved to `scripts.title_page_data` (JSON) via `PATCH /scripts/:id` on field blur.

**GIVEN** the title page is saved,  
**WHEN** the PDF export (story-019) is run with "Include Title Page" toggle ON,  
**THEN** the generated PDF's page 1 renders the title page with exactly the layout specified:
- Script title: centred, 3.5" from top, 14pt Courier Prime CAPS
- "Written by" credit: centred, 4.0" from top, 12pt
- Author names: centred, 4.25" from top, 12pt
- Contact block: bottom-left corner, 10pt, lines: name / email / phone / address
- WGA number: bottom-right corner, 10pt: "WGA Reg. #[number]"
- Draft date: bottom-right below WGA, 10pt
- No page number on the title page

**GIVEN** the writer has set "Based On" text,  
**WHEN** the title page renders,  
**THEN** the "Based on [work] by [author]" line appears below the "Written by" credit, centred, in 12pt Courier Prime.

**GIVEN** the script was imported from Fountain with a title page (story-021),  
**WHEN** the Title Page editor opens,  
**THEN** the fields are pre-populated from the imported Fountain title page key-value pairs (Title, Author, Contact).

**GIVEN** the Title Page editor is open on a 375px mobile screen,  
**WHEN** it renders,  
**THEN** the WYSIWYG preview is shown as a thumbnail card at the top; the edit fields are below it; the preview updates when the user taps "Preview" after editing.

**GIVEN** the writer saves contact information in the title page,  
**WHEN** they create a new script in the future,  
**THEN** the Contact Name, Email, and Phone fields are pre-populated from the last used values (stored in `profiles.default_contact_info` JSON field).

**GIVEN** a collaborator on an editor-role opens the title page editor,  
**WHEN** they make changes,  
**THEN** their changes are saved to `title_page_data` (last-write-wins, no CRDT needed here); the script owner sees the updated title page.

**GIVEN** a viewer-role collaborator opens the title page,  
**WHEN** they view it,  
**THEN** all fields are read-only; no edit controls are shown.

---

## Performance Criteria

- Title Page editor open: < 200ms
- WYSIWYG preview update per keystroke: < 100ms
- Save (PATCH /scripts/:id): < 300ms
- PDF rendering with title page: included in the < 30s PDF generation SLA (story-019)

---

## Offline Behaviour

- Title page fields are cached in Drift as part of `CachedScripts.title_page_data`
- Edits are queued in `OfflineWriteQueue` offline
- WYSIWYG preview works offline from local cache

---

## Mobile Behaviour (375px)

- Title page accessed from the "Script Info" icon in the editor toolbar
- Opens as a full-screen bottom sheet
- "Preview" tab + "Edit" tab at top
- Preview tab: renders the title page as a scaled-down A4 card (50% scale)
- Edit tab: scrollable list of input fields

---

## Error States

**E1 — Title Too Long**  
Title > 80 characters: show warning below the title field "Long titles may be truncated in PDF. Consider a shorter title." The full title is still saved.

**E2 — Missing Author Name**  
If "Written By" is left blank when exporting: the export proceeds with "Written by [Username from profile]" as fallback; no error.

---

## Security Considerations

- `title_page_data` is part of the `scripts` row; it has the same RLS protection
- Contact information (email, phone) stored in `title_page_data` is not publicly exposed — it only appears in the writer's exported PDF

---

## Human Authorship Considerations

Not applicable.

---

## DB Tables Touched

- `scripts` (UPDATE `title_page_data` JSON column)
- `profiles` (SELECT/UPDATE `default_contact_info`)

---

## API Endpoints Used

- `GET /scripts/:id` — load title page data
- `PATCH /scripts/:id` — save title page data

---

## Test Cases

**TC-001:** Open Title Page editor → assert fields pre-filled with script title and profile username.  
**TC-002:** Edit "Written By" → assert WYSIWYG preview updates in real time.  
**TC-003:** Save → export PDF → assert title page rendered correctly with all fields.  
**TC-004:** Import Fountain with title page → assert fields pre-populated from import.  
**TC-005:** Open Title Page editor offline → assert fields loaded from Drift cache.  
**TC-006:** Viewer opens Title Page → assert all fields are read-only.  
**TC-007:** Save contact info → create new script → assert contact fields pre-populated.

---

## Out of Scope

- Co-writer royalty split display on title page (Phase 2)
- Copyright notice field (Phase 2 — beyond WGA number)
- Title page templates beyond the single WGA-standard layout (Phase 2)
