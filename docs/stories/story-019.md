# story-019 — Export — Production PDF

| Field | Value |
|---|---|
| **Story ID** | story-019 |
| **Title** | Export — Production PDF |
| **Epic** | E08 — Export & Import |
| **Phase** | 2 |
| **Sprint** | Sprint 3 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Marcus (the Hollywood rewrite specialist) |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-004, story-025 |

---

## Problem Statement

A screenplay that cannot be submitted to agents, producers, and studios as a correctly formatted PDF has no professional value. The production PDF export must produce output that is visually indistinguishable from Final Draft's PDF output: correct margins, correct fonts (Courier Prime 12pt), correct page breaks, correct page numbering, scene numbers in the margin (optional), and WGA-compliant header/footer.

---

## User Story

As a professional screenwriter, I want to export my script as a production-quality PDF that looks exactly like an industry-standard screenplay — so that I can submit it to agents and studios with confidence.

---

## Acceptance Criteria

**GIVEN** a user clicks "Export" in the editor menu and selects "Production PDF",  
**WHEN** the export dialog opens,  
**THEN** it shows: title (pre-filled from script title), author name (pre-filled from profile), include title page toggle (default ON), scene numbers in margin toggle (default OFF), revision colour (None / Blue / Pink / Yellow / Green), and an "Export PDF" button.

**GIVEN** the user clicks "Export PDF",  
**WHEN** the export job is dispatched via `POST /scripts/:id/export/pdf`,  
**THEN** the Rust API enqueues a PDF generation job; the export dialog shows a progress indicator "Generating your PDF…"; the job runs server-side (not client-side) using a headless PDF renderer (Puppeteer or a Rust PDF library such as `printpdf`).

**GIVEN** the PDF generation completes,  
**WHEN** the job result is ready,  
**THEN** the Rust API stores the PDF in S3 and returns a pre-signed URL valid for 1 hour; the Flutter client downloads the PDF and triggers the platform's native share sheet (iOS/Android) or a download dialog (Web); a success toast: "PDF exported — 120 pages."

**GIVEN** the PDF is generated,  
**WHEN** it is inspected for format compliance,  
**THEN** it must satisfy all WGA formatting rules:
- Page size: 8.5" × 11"
- Font: Courier Prime 12pt
- Scene Heading: bold, ALL CAPS, left margin 1.5"
- Action: left margin 1.5", right margin 1.0"
- Character: centred at 3.7" from left
- Parenthetical: 3.0"–4.2"
- Dialogue: 2.5"–6.0"
- Transition: right-aligned 6.0"
- Page number: top-right, 0.5" from top, 7.0" from left
- Header: script title (shortened) top-left of pages 2+
- Bottom margin: 1.0"; top margin: 0.75"
- One page ≈ one minute of screen time (approximately 55 lines per page)

**GIVEN** the title page toggle is ON,  
**WHEN** the PDF is generated,  
**THEN** page 1 is the title page containing: title (centred, 3.5" from top, 14pt CAPS), "Written by" (centred, below title, 12pt), author name (centred, 12pt), contact info from profile (bottom-left if provided), WGA/copyright notice (bottom-right if provided); page 1 has no page number.

**GIVEN** the scene numbers toggle is ON,  
**WHEN** the PDF is generated,  
**THEN** each scene heading has its scene number printed both to the left (in the left margin, 1.0") and to the right (7.5" from left) of the heading text.

**GIVEN** a revision colour is selected (e.g., "Blue Revision"),  
**WHEN** the PDF is generated,  
**THEN** all blocks with `revision_marks` set (from story-029) are printed on blue paper–style pages; the revision marker asterisk (*) appears in the right margin of revised lines (1.0" from right edge); non-revised pages are white.

**GIVEN** a 120-page script export is initiated,  
**WHEN** the generation completes,  
**THEN** the total processing time is < 30 seconds; the PDF file size is < 5MB for a standard 120-page script (Courier Prime is embedded as a subset).

**GIVEN** a Free plan user tries to export a PDF,  
**WHEN** they click "Export PDF",  
**THEN** a paywall prompt appears: "PDF export is available on Pro. Upgrade to unlock professional PDF exports." Free users can only export Fountain (story-020).

**GIVEN** the PDF export is downloaded by the user,  
**WHEN** they open it in any PDF viewer,  
**THEN** the text is searchable (not rasterised); the file is accessible (tagged PDF for screen readers where supported by the renderer).

---

## Performance Criteria

- Export dialog open: < 100ms
- PDF generation time: < 30 seconds for 120-page script
- Pre-signed S3 URL generation: < 200ms
- PDF file download to device: limited by network, but S3 provides globally distributed download
- PDF file size: < 5MB for 120 pages (Courier Prime subsetted)

---

## Offline Behaviour

- PDF export requires internet (generation happens server-side)
- The Export button shows "Export requires an internet connection" when offline
- Previously exported PDFs cached by the OS share sheet are accessible offline (OS-level caching)

---

## Mobile Behaviour (375px)

- Export dialog is a bottom sheet on mobile
- After export, the platform native share sheet is triggered automatically (iOS: UIActivityViewController; Android: ACTION_SEND intent)
- Progress indicator is a full-screen modal with a pulsing circular progress and "Generating PDF…" text

---

## Error States

**E1 — PDF Generation Timeout**  
If the server-side PDF generation exceeds 60 seconds: the job is killed; return error: "PDF generation timed out. This may happen with very large scripts. Please try again." The pre-signed URL is not generated.

**E2 — S3 Upload Failure**  
If the S3 upload fails: retry up to 3 times; if all fail, return error to client: "Export failed. Please try again." Log to Sentry.

**E3 — Corrupted Block Content**  
If any block in the script has malformed content that breaks the PDF renderer: skip the block and insert `[CORRUPTED BLOCK SKIPPED]` placeholder text in the PDF; log the block ID to Sentry; export completes with a warning toast.

**E4 — Pre-Signed URL Expired**  
If the user doesn't download within 1 hour: the URL expires; show "Your export link expired. Click here to re-export." The PDF is deleted from S3 after expiry.

---

## Security Considerations

- Pre-signed S3 URLs are generated with 1-hour expiry
- The export endpoint verifies the user is the script owner or an editor-role collaborator
- PDFs are stored in a private S3 bucket (`s3://lipily-exports`) with no public access; access is only via pre-signed URLs
- Exported PDFs are deleted from S3 after 24 hours (S3 lifecycle policy)
- The PDF does not include any internal IDs, UUIDs, or system metadata in the rendered output

---

## Human Authorship Considerations

The PDF export reflects the writer's complete creative work. AI-generated blocks that have not been confirmed by the writer are included in the export but carry no visible AI marking in the PDF (the PDF is the submission-ready document). The AI badge is only visible in the editor.

---

## DB Tables Touched

- `scripts` (SELECT — title, format, author)
- `scenes` (SELECT — all scenes ordered by position)
- `blocks` (SELECT — all blocks in the script ordered by scene+position)
- `formatting_preferences` (SELECT — any custom formatting overrides from story-027)

---

## API Endpoints Used

- `POST /scripts/:id/export/pdf` — initiate export job
- `GET /scripts/:id/export/:job_id/status` — poll job status (or WebSocket push)
- `GET /scripts/:id/export/:job_id/download` — get pre-signed S3 URL

---

## Test Cases

**TC-001:** Export 120-page script as PDF → assert file generated in < 30s.  
**TC-002:** Open generated PDF → assert all WGA formatting rules met (margins, font, page numbers).  
**TC-003:** Export with title page ON → assert page 1 is title page with title, author, no page number.  
**TC-004:** Export with scene numbers ON → assert scene numbers in left and right margins.  
**TC-005:** Free user clicks Export PDF → assert paywall shown.  
**TC-006:** PDF text is searchable → assert PDF is not rasterised.  
**TC-007:** Pre-signed URL accessed after 1 hour → assert 403 from S3.  
**TC-008:** Export with Blue Revision → assert revised lines have asterisk in right margin.

---

## Out of Scope

- Fountain export (story-020)
- FDX export (story-020)
- Batch export of multiple scripts (Phase 2)
- Interactive PDF with hyperlinks (Phase 2)
