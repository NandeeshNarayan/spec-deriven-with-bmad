# story-029 — Revision Mode (Production Standard)

| Field | Value |
|---|---|
| **Story ID** | story-029 |
| **Title** | Revision Mode (Production Standard) |
| **Epic** | E09 — Editor Utilities |
| **Phase** | 3 |
| **Sprint** | Sprint 4 |
| **Priority** | P2 |
| **Status** | DRAFT |
| **Persona** | Marcus (the Hollywood rewrite specialist) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-012 |

---

## Problem Statement

In professional film and television production, revised pages are issued on coloured paper with asterisks in the right margin marking every changed line. This is the WGA-standard revision marking system (White / Blue / Pink / Yellow / Green / Goldenrod / Buff / Salmon / Cherry / Tan). LIPILY's Revision Mode must replicate this standard digitally so that productions using LIPILY can issue production-standard revised pages.

---

## User Story

As a rewrite specialist working on a production draft, I want Revision Mode that marks changed lines with asterisks and assigns revision colours — so that I can issue production-standard revised pages that any production office can use.

---

## Acceptance Criteria

**GIVEN** the writer is in the editor and enables Revision Mode (from the editor header menu, or Cmd+Shift+R),  
**WHEN** Revision Mode is activated,  
**THEN** a revision colour selector appears in the top bar showing the current revision colour (default: Blue First Revision); the editor header shows "REVISION MODE — [Colour] Revision"; a right-margin ruler with asterisk positions becomes visible.

**GIVEN** Revision Mode is active and the writer edits any block,  
**WHEN** the block content changes,  
**THEN** an asterisk (*) is automatically added to the right margin of the changed line at 7.2" from the left (standard WGA position); `blocks.revision_mark = true`; `blocks.revision_colour = 'blue'`; unchanged blocks show no asterisk.

**GIVEN** Revision Mode is active and the writer adds a completely new block (new scene or new lines),  
**WHEN** the new block is saved,  
**THEN** the new block is marked with `revision_mark = true`; asterisks appear on every line of the new block.

**GIVEN** Revision Mode is active and the writer deletes a block,  
**WHEN** the block is soft-deleted,  
**THEN** the block is NOT immediately hidden; instead it is shown with a strikethrough and `revision_mark = true`; it will only be omitted from future prints/exports once the revision is "locked" or when the lock is set for this revision cycle.

**GIVEN** the writer selects a revision colour (e.g., Pink — Second Revision),  
**WHEN** the colour is selected,  
**THEN** all subsequent edits in this session are marked with `revision_colour = 'pink'`; previously Blue revision marks remain blue; the colour changes are cumulative.

**GIVEN** the writer exits Revision Mode,  
**WHEN** Revision Mode is deactivated,  
**THEN** all revision asterisks remain visible in the editor as small grey markers; the editor returns to normal editing mode; new edits are NOT marked with revision marks (only edits made IN revision mode are marked).

**GIVEN** the PDF is exported while revision marks exist (story-019),  
**WHEN** the PDF is generated with a "Revision Colour" selected in the export dialog,  
**THEN** all pages with at least one changed block show the revision colour in the page header (e.g., "(Blue Revision)"); asterisks appear in the right margin; pages without changes are marked "(A)" for the standard; the revision date is shown in the page header.

**GIVEN** the writer wants to lock the current revision and start a new revision cycle,  
**WHEN** they click "Lock Revision" in the Revision Mode toolbar,  
**THEN** all current revision marks are converted to "baseline" (treated as non-revised going forward); a new revision cycle begins; the next colour in the sequence is automatically selected; a draft is auto-saved.

**GIVEN** a user is on the Free or Pro plan and attempts to enable Revision Mode,  
**WHEN** the toggle is tapped,  
**THEN** a paywall modal appears: "Revision Mode is available on Studio plan. Upgrade to unlock production-standard revision tracking."

**GIVEN** the revision cycle colour order is shown in the colour selector,  
**WHEN** the writer views it,  
**THEN** the colours are shown in WGA production order: White (original) → Blue → Pink → Yellow → Green → Goldenrod → Buff → Salmon → Cherry → Tan.

---

## Performance Criteria

- Revision Mode activation: < 200ms
- Asterisk render per edited line: < 16ms (same frame as edit)
- PDF generation with revision marks: within the < 30s PDF SLA
- "Lock Revision" operation: < 2 seconds (batch update on all marked blocks)

---

## Offline Behaviour

- Revision Mode works fully offline (same as standard editor)
- Revision marks are stored in Drift `CachedBlocks` (`revision_mark` and `revision_colour` columns)
- Revision lock operation is queued in `OfflineWriteQueue` offline

---

## Mobile Behaviour (375px)

- Revision Mode toggle in the editor "..." menu
- Revision colour shown as a coloured dot next to the block gutter
- Asterisk in right margin is displayed as a small red dot on mobile (full asterisk is too small to tap)

---

## Error States

**E1 — Lock Revision Timeout**  
Batch update times out (> 10s for very large scripts): show "Locking revision… this may take a moment" spinner; retry up to 3 times.

**E2 — Colour Exhausted**  
All 10 WGA revision colours have been used: the selector wraps back to Blue with a "(Second Cycle)" label.

---

## Security Considerations

- Revision Mode is a Studio-only feature; enforced server-side on the `PATCH /blocks/:id` endpoint (checks `subscription.tier = 'studio'`)
- Revision marks (`revision_mark`, `revision_colour`) inherit block RLS

---

## Human Authorship Considerations

Not applicable — revision mode marks changes made by human writers.

---

## DB Tables Touched

- `blocks` (UPDATE `revision_mark`, `revision_colour`)
- `scripts` (UPDATE `current_revision_colour`)

---

## API Endpoints Used

- `PATCH /blocks/:id` — mark block as revised (standard block update with revision fields)
- `POST /scripts/:id/revisions/lock` — lock current revision cycle

---

## Test Cases

**TC-001:** Enable Revision Mode → edit a block → assert asterisk appears in right margin.  
**TC-002:** Add new block in Revision Mode → assert all lines marked with asterisk.  
**TC-003:** Change revision colour to Pink → edit blocks → assert new marks are pink; old blue marks remain blue.  
**TC-004:** Export PDF with Blue Revision → assert revision colour in page header and asterisks in margin.  
**TC-005:** Lock Revision → assert all current marks become baseline; new colour auto-selected.  
**TC-006:** Free user enables Revision Mode → assert Studio paywall shown.  
**TC-007:** Disable Revision Mode → assert existing marks remain visible; new edits not marked.

---

## Out of Scope

- Multi-revision comparison view (Phase 2)
- Digital revision distribution to cast/crew (Phase 2)
