# story-021 — Script Import

| Field | Value |
|---|---|
| **Story ID** | story-021 |
| **Title** | Script Import |
| **Epic** | E08 — Export & Import |
| **Phase** | 1 |
| **Sprint** | Sprint 3 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Marcus (the Hollywood rewrite specialist) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Professional writers come to LIPILY from Final Draft, Highland 2, or Celtx with years of work in existing files. Without import support, LIPILY is inaccessible to established writers. Import must handle Fountain (.fountain), FDX (.fdx), and plain-text screenplay format (.txt) with high fidelity — preserving scene structure, all block types, and character names.

---

## User Story

As a professional screenwriter switching to LIPILY, I want to import my existing scripts from Final Draft (.fdx), Fountain (.fountain), or plain text — so that all my previous work is available in LIPILY without manual re-entry.

---

## Acceptance Criteria

**GIVEN** a user is on the dashboard and clicks "Import Script",  
**WHEN** the import dialog opens,  
**THEN** they can drag-and-drop or tap "Choose File" to select a `.fountain`, `.fdx`, or `.txt` file from their device; the supported formats are listed clearly; maximum file size is 10MB.

**GIVEN** the user selects a `.fountain` file,  
**WHEN** `POST /scripts/import` is called with the file,  
**THEN** the Rust API parses the Fountain file using a Fountain parser; each Fountain paragraph type is mapped to the corresponding LIPILY block type; a new `scripts` row is created; all `scenes` and `blocks` are inserted in correct order; positions use gap strategy (1000, 2000, 3000…); the title is extracted from the Fountain title page if present.

**GIVEN** the user selects a `.fdx` file (Final Draft XML),  
**WHEN** the import runs,  
**THEN** the Rust API parses the FDX XML (using `quick-xml` Rust crate); each `<Paragraph Type="...">` element is mapped to the corresponding LIPILY block type; scene structure is inferred from `Scene Heading` paragraph types; all content is inserted into `scenes` and `blocks` tables.

**GIVEN** the user selects a plain-text `.txt` file,  
**WHEN** the import runs,  
**THEN** the Rust API applies heuristic block-type detection: lines that are ALL CAPS and start with "INT." or "EXT." are `scene_heading`; ALL CAPS lines in the centre of the page are `character`; lines in parentheses following a character cue are `parenthetical`; following-lines are `dialogue`; all other lines are `action`.

**GIVEN** the import completes successfully,  
**WHEN** the result is returned,  
**THEN** the user is navigated to the newly created script in the editor; a success toast: "Imported '[title]' — [N] scenes, [M] blocks."; Scene Intelligence is not run automatically on import (to preserve AI quota); the writer can trigger it manually.

**GIVEN** the imported file has a title page,  
**WHEN** the import runs,  
**THEN** the title page content is stored in `scripts.title_page_data` (JSON) and rendered correctly in the title page editor (story-025); it does not appear as script blocks.

**GIVEN** the imported script has dialogue variants (Fountain: `[[variant]]` syntax or FDX alt dialogue),  
**WHEN** the import runs,  
**THEN** variants are imported as variant blocks with correct `parent_block_id` and `variant_index`; the first variant is set as `is_active = true`.

**GIVEN** the import file contains unsupported elements (e.g., embedded images in FDX, dual-dialogue format),  
**WHEN** the import runs,  
**THEN** unsupported elements are skipped; after the import, a summary is shown: "2 elements were skipped during import: dual-dialogue (1), embedded image (1). These will need to be re-added manually."

**GIVEN** a Free-tier user imports a script and they already have 3 scripts,  
**WHEN** the import is attempted,  
**THEN** the import is blocked with a paywall modal: "You've reached your script limit (3/3 on Free). Upgrade to Pro to import unlimited scripts."

**GIVEN** a large `.fdx` file (200 pages / 10MB),  
**WHEN** the import is submitted,  
**THEN** the import processes as an async job; a progress indicator shows "Importing your script…"; the user is notified via in-app notification when the import completes (they can navigate away and return).

---

## Block Type Detection Table (Fountain Heuristics)

| Fountain Pattern | LIPILY Block Type |
|---|---|
| `INT.`/`EXT.`/`I/E.` prefix | scene_heading |
| ALL CAPS, centre-ish, not a transition | character |
| `(text)` after character | parenthetical |
| Line following parenthetical/character | dialogue |
| Ends with `TO:` or `SMASH CUT TO:` | transition |
| `[[text]]` double bracket | text_note |
| All other lines | action |

---

## Performance Criteria

- Import dialog open: < 100ms
- Fountain import (120-page file): < 5 seconds
- FDX import (120-page file): < 10 seconds (XML parsing is slower)
- Large async import (200-page FDX): < 30 seconds
- Blocks inserted in correct positions: verified in < 500ms post-import

---

## Offline Behaviour

- Import requires internet (server-side parsing)
- "Import Script" button is disabled offline with tooltip: "Import requires an internet connection"

---

## Mobile Behaviour (375px)

- "Import Script" button in the dashboard FAB menu (alongside "New Script")
- File picker uses `file_picker` Flutter package — opens native file picker
- Progress modal is full-screen with animation and "Importing…" text
- Skipped elements summary shown as a scrollable modal after import

---

## Error States

**E1 — File Too Large**  
File exceeds 10MB: client-side validation before upload shows: "File is too large (12MB). Maximum import size is 10MB."

**E2 — Unsupported File Format**  
User selects a `.pdf` or `.docx` file: inline error: "Unsupported file format. LIPILY supports .fountain, .fdx, and .txt files."

**E3 — Corrupt FDX XML**  
FDX file has malformed XML: return error: "This Final Draft file appears to be corrupted. Please try exporting it again from Final Draft and re-importing."

**E4 — Duplicate Import**  
User imports the same file twice (same title and block count): detect potential duplicate and show warning: "A script with the title '[name]' already exists. Import anyway?" with Import and Cancel buttons.

---

## Security Considerations

- Imported file is not stored permanently — it is parsed and discarded; only the extracted blocks are stored
- File upload is limited to 10MB and only accepted MIME types (`.fountain`, `.fdx`, `.txt`)
- The import endpoint is rate-limited to 5 imports per hour per user
- All inserted blocks inherit the same RLS as manually-created blocks

---

## Human Authorship Considerations

- All imported blocks are set to `content_source = 'human_written'` (the writer created them before LIPILY)
- No AI analysis runs automatically on import

---

## DB Tables Touched

- `scripts` (INSERT new script)
- `scenes` (INSERT all scenes)
- `blocks` (INSERT all blocks)
- `profiles` (UPDATE `script_count`)

---

## API Endpoints Used

- `POST /scripts/import` — upload and parse file; create script/scenes/blocks
- `GET /scripts/import/:job_id/status` — poll async import job status (for large files)

---

## Test Cases

**TC-001:** Import a valid 120-page Fountain file → assert correct scene count, block count, and block types.  
**TC-002:** Import a valid FDX file → open in editor → assert visually identical to Final Draft rendering.  
**TC-003:** Import plain .txt file → assert heuristic block type detection is correct for > 90% of lines.  
**TC-004:** Import Fountain file with title page → assert title page stored in `scripts.title_page_data`.  
**TC-005:** Import file with dual-dialogue → assert skipped elements summary shown.  
**TC-006:** Free user at 3-script limit tries to import → assert paywall shown.  
**TC-007:** Import 200-page FDX as async job → assert notification fires on completion.  
**TC-008:** Round-trip: export Fountain (story-020) → import → assert 100% block-for-block match.

---

## Out of Scope

- PDF-to-script import (optical character recognition — Phase 2)
- Google Docs import (Phase 2)
- Word (.docx) import (Phase 2)
- Import from Celtx or WriterDuet (Phase 2)
