# story-032 — Inspiration & Research Mode

| Field | Value |
|---|---|
| **Story ID** | story-032 |
| **Title** | Inspiration & Research Mode |
| **Epic** | E06 — Story Tools |
| **Phase** | 3 |
| **Sprint** | Sprint 4 |
| **Priority** | P2 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Writers gather research — newspaper articles, images, reference films, character notes — from across the internet while writing. Without an integrated research panel, they switch between the script and dozens of browser tabs, losing flow. The Research Mode is a distraction-contained panel inside the editor where writers can clip, annotate, and organise research material tied to specific scenes.

---

## User Story

As a screenwriter doing research while writing, I want a Research Panel inside the editor where I can clip articles, save links, write notes, and pin images — so that all my research is attached to the relevant scene without leaving the editor.

---

## Acceptance Criteria

**GIVEN** the writer opens the Research Panel (editor right sidebar → "Research" tab or Cmd+Shift+R),  
**WHEN** the panel opens,  
**THEN** it shows a search bar, an "Add Note" button, an "Add Link" button, a list of existing research notes for the current scene, and a toggle to "Show All Research" (across all scenes).

**GIVEN** the writer clicks "Add Note",  
**WHEN** the note editor opens,  
**THEN** they can type a free-text note (Markdown supported); attach it to the current scene (default) or select a different scene from a dropdown; add tags (free text, comma-separated); save the note via `POST /research-notes`.

**GIVEN** the writer pastes a URL and clicks "Add Link",  
**WHEN** the link is saved,  
**THEN** the Rust API fetches the page title and a preview description (OG:title, OG:description) via a server-side fetch (not the client, to avoid CORS); a research note is created with the URL, title, and description; a thumbnail is shown if OG:image is available.

**GIVEN** a research note has been added,  
**WHEN** the writer views the Research Panel,  
**THEN** notes are displayed as cards: title/excerpt, tags, the scene they're attached to, and a "View" button (opens the full note in a modal).

**GIVEN** the writer selects text in any block and right-clicks,  
**WHEN** they choose "Add to Research",  
**THEN** the selected text is saved as a research note with `source_block_id` pointing to the block it came from; the note appears in the Research Panel for the current scene.

**GIVEN** the research panel has more than 20 notes,  
**WHEN** the panel renders,  
**THEN** notes are virtualised; a search/filter bar is visible; notes can be filtered by tag.

**GIVEN** the writer deletes a research note,  
**WHEN** "Delete" is confirmed,  
**THEN** `DELETE /research-notes/:id` is called; the note is permanently deleted (no soft-delete for research notes — they are considered ephemeral).

**GIVEN** a research note has an external link,  
**WHEN** the writer taps "Open Link",  
**THEN** the link opens in the device's default browser (not in-app) to prevent data exfiltration risks; the editor is not navigated away.

**GIVEN** the script is exported to PDF or Fountain,  
**WHEN** the export runs,  
**THEN** research notes are NOT included in the export (they are writer's private notes, not script content).

---

## Performance Criteria

- Research panel open: < 150ms
- URL fetch (OG metadata): < 3 seconds (timeout after 3s)
- Note save: < 300ms
- 50 notes virtualised list: < 200ms render

---

## Offline Behaviour

- Existing research notes viewable offline from Drift cache
- Adding new notes (text only) works offline; queued in `OfflineWriteQueue`
- URL link fetch requires internet; disabled with tooltip when offline

---

## Mobile Behaviour (375px)

- Research Panel accessed from editor toolbar "..." → "Research"
- Opens as a full-screen bottom sheet
- Add Note: opens a full-screen text editor
- Add Link: URL input field at top of panel

---

## Error States

**E1 — OG Fetch Failure**  
URL's OG metadata cannot be fetched (CORS, 404, timeout): save the raw URL as the note title; no preview image.

**E2 — Malicious URL**  
URL contains a known malicious domain (checked against a basic blocklist): show error "This URL appears to be unsafe and cannot be added."

---

## Security Considerations

- URL fetch is performed server-side only (Rust API) to prevent client-side SSRF
- Research notes have RLS: only the script owner and collaborators can read; only the creator can delete
- External links always open in external browser, never in an in-app WebView

---

## Human Authorship Considerations

Research notes are entirely human-written. No AI interaction in Research Mode for v1.

---

## DB Tables Touched

- `research_notes` (SELECT, INSERT, DELETE)

---

## API Endpoints Used

- `GET /scripts/:id/research-notes` — list notes
- `POST /research-notes` — add note or link
- `DELETE /research-notes/:id` — delete note
- `POST /research-notes/fetch-preview` — server-side URL OG metadata fetch

---

## Test Cases

**TC-001:** Add text note → assert research note created and appears in panel.  
**TC-002:** Add URL → assert OG title/description fetched and shown as preview card.  
**TC-003:** Add note offline → reconnect → assert note synced.  
**TC-004:** 50 notes in panel → assert virtualised; search filters correctly.  
**TC-005:** Export script to PDF → assert research notes NOT included.  
**TC-006:** Open external link → assert opens in system browser, not in-app.

---

## Out of Scope

- AI-assisted research ("find articles about [topic]") — Phase 2
- Image clip/upload to research notes — Phase 2
- Research note sharing with collaborators — Phase 2
