# story-020 — Export — Fountain & FDX

| Field | Value |
|---|---|
| **Story ID** | story-020 |
| **Title** | Export — Fountain & FDX |
| **Epic** | E08 — Export & Import |
| **Phase** | 1 |
| **Sprint** | Sprint 3 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Aisha (the independent short-film writer) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Fountain (.fountain) is the open-source plain-text screenplay format used by Highland 2, Fade In, and many open-source tools. FDX (.fdx) is the Final Draft XML format — the industry standard for professional productions. Writers need to export in both formats to collaborate with writers using other tools, to submit to competitions accepting Fountain, and to hand off to production companies that use Final Draft.

---

## User Story

As a writer, I want to export my script as a Fountain file (for open-source tools) and as an FDX file (for Final Draft) — so that I can collaborate with writers on other platforms and submit to competitions and productions.

---

## Acceptance Criteria

**GIVEN** the user selects "Export → Fountain (.fountain)" from the editor export menu,  
**WHEN** the export is triggered via `POST /scripts/:id/export/fountain`,  
**THEN** the Rust API converts all blocks to Fountain format syntax and returns the `.fountain` file directly (not a job — synchronous for small scripts < 200 pages); on the Flutter client, the platform share sheet opens with the `.fountain` file; the filename is `[script_title].fountain`.

**GIVEN** the Fountain file is exported,  
**WHEN** it is inspected for format compliance,  
**THEN** it must satisfy all Fountain spec rules:
- Scene Heading: `INT. LOCATION - DAY` (no prefix needed — detected by location pattern)
- Action: plain paragraph text
- Character: ALL CAPS name on its own line
- Parenthetical: `(text in parentheses)` on own line
- Dialogue: indented paragraph following a character cue
- Transition: `> CUT TO:` (right-chevron syntax) or detected ALL CAPS with `TO:`
- Page Break: `===`
- Title Page: `Title:`, `Author:`, `Contact:` key-value pairs at top of file

**GIVEN** the user selects "Export → Final Draft (.fdx)" from the editor export menu,  
**WHEN** the export is triggered via `POST /scripts/:id/export/fdx`,  
**THEN** the Rust API generates a valid FDX XML file conforming to the Final Draft 11 FDX schema; the file is returned to the Flutter client; the filename is `[script_title].fdx`.

**GIVEN** the FDX file is exported and opened in Final Draft 11,  
**WHEN** it renders,  
**THEN** all block types are preserved correctly as FDX paragraph types (`Scene Heading`, `Action`, `Character`, `Parenthetical`, `Dialogue`, `Transition`, `General`); the title page is included; no formatting is lost.

**GIVEN** a script with Dialogue Variants (story-015) is exported,  
**WHEN** the Fountain/FDX export runs,  
**THEN** only the `is_active = true` variant is included; all inactive variants are ignored (same as PDF export).

**GIVEN** a script with Text Note blocks is exported,  
**WHEN** the Fountain export runs,  
**THEN** Text Note blocks are exported as Fountain `[[Note text]]` (double-bracket note syntax); in FDX export, they are exported as `<Paragraph Type="General">` with a custom `Note="true"` attribute.

**GIVEN** the user is on the Free plan and exports to Fountain,  
**WHEN** the export is triggered,  
**THEN** the Fountain export proceeds without any paywall — Fountain export is available to all tiers.

**GIVEN** the user is on the Free plan and tries to export to FDX,  
**WHEN** the export is triggered,  
**THEN** a paywall modal appears: "FDX export is available on Pro. Upgrade to unlock Final Draft compatibility."

**GIVEN** a very large script (200+ pages) is exported to Fountain,  
**WHEN** the export is triggered,  
**THEN** the export is still synchronous if < 500KB; if the estimated Fountain file size exceeds 500KB, it falls back to an async job (same pattern as PDF export with pre-signed S3 URL).

**GIVEN** the exported Fountain file is imported back into LIPILY (story-021 round-trip test),  
**WHEN** the import completes,  
**THEN** all block types are preserved; scene structure is identical; no content is lost or duplicated; this round-trip must be 100% lossless for the 7 supported block types.

---

## Fountain Format Mapping Table

| LIPILY Block Type | Fountain Syntax |
|---|---|
| Scene Heading | `INT. LOCATION - DAY` (auto-detected) |
| Action | Plain paragraph |
| Character | ALL CAPS line |
| Parenthetical | `(text)` on own line |
| Dialogue | Paragraph after character cue |
| Transition | `> RIGHT ALIGNED` or `CUT TO:` |
| Text Note | `[[note text]]` |

---

## FDX Paragraph Type Mapping

| LIPILY Block Type | FDX ParagraphType |
|---|---|
| Scene Heading | `Scene Heading` |
| Action | `Action` |
| Character | `Character` |
| Parenthetical | `Parenthetical` |
| Dialogue | `Dialogue` |
| Transition | `Transition` |
| Text Note | `General` |

---

## Performance Criteria

- Fountain export (< 200 pages): < 1 second (synchronous)
- FDX export (< 200 pages): < 2 seconds (synchronous)
- Large script async export (> 500KB Fountain): < 15 seconds
- Round-trip fidelity test (export + import): 100% block-for-block match

---

## Offline Behaviour

- Fountain export can run entirely client-side from Drift cache (no network needed) — generate locally for < 120-page scripts
- FDX export requires the Rust API; disabled offline with tooltip
- When offline, "Export Fountain" uses a client-side Dart Fountain serialiser built from Drift `CachedBlocks`

---

## Mobile Behaviour (375px)

- Export options in editor "..." menu → "Export" sub-menu
- After export, native share sheet opens (same as PDF export)
- Fountain and FDX file icons are shown in the share sheet

---

## Error States

**E1 — FDX Schema Validation Failure**  
If the generated FDX XML fails XSD validation: log to Sentry with the offending block IDs; return error to client: "FDX export failed. Our team has been notified."

**E2 — Unsupported Character in Fountain**  
If any block contains characters that violate the Fountain spec (e.g., unescaped tildes used as comments): the offending characters are escaped or stripped; a warning is added to the export: "1 formatting issue was auto-corrected."

**E3 — Round-Trip Loss Detection**  
If the import of an exported Fountain file produces a different block count than the original script: log a discrepancy to Sentry for analysis; do not show an error to the user.

---

## Security Considerations

- Export endpoint verifies user is owner or editor-role collaborator
- Fountain export is free for all tiers but rate-limited (10 exports per hour per user)
- FDX export pre-signed URLs expire in 1 hour; file deleted from S3 after 24 hours

---

## Human Authorship Considerations

The exported files contain the writer's complete creative work. No AI attribution metadata is embedded in Fountain or FDX files.

---

## DB Tables Touched

- `scripts` (SELECT metadata)
- `scenes` (SELECT all)
- `blocks` (SELECT all, `is_active = true` variants only)

---

## API Endpoints Used

- `POST /scripts/:id/export/fountain` — generate Fountain file
- `POST /scripts/:id/export/fdx` — generate FDX file

---

## Test Cases

**TC-001:** Export 120-page script as Fountain → assert file generated in < 1s; syntax validates against Fountain spec.  
**TC-002:** Export as FDX → open in Final Draft 11 → assert all blocks render correctly.  
**TC-003:** Free user exports Fountain → assert no paywall shown.  
**TC-004:** Free user exports FDX → assert paywall shown.  
**TC-005:** Export script with variants → assert only active variants in output.  
**TC-006:** Export Text Note block → assert Fountain `[[note]]` syntax and FDX `General` type.  
**TC-007:** Export Fountain offline → assert client-side generation succeeds from Drift cache.  
**TC-008:** Round-trip: export Fountain → import Fountain → assert 100% block-for-block match.

---

## Out of Scope

- Celtx (.celtx) export (Phase 2)
- Highland (.highland) export (Phase 2)
- PDF export (story-019)
- RTF export (Phase 2)
