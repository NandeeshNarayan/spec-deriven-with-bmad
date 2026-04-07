# story-026 — Find & Replace

| Field | Value |
|---|---|
| **Story ID** | story-026 |
| **Title** | Find & Replace |
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

Professional script rewrites often require changing a character name across 80 occurrences, fixing a typo that appears 30 times, or updating a location name throughout. The global Find & Replace panel must work at the block level (not just text-matching), support regex for power users, and respect block type scope filtering (e.g., replace only in Dialogue blocks).

---

## User Story

As a screenwriter doing a rewrite, I want a Find & Replace panel that searches and replaces text across the entire script — with options for case-sensitive matching, whole word, regex, and block type filtering.

---

## Acceptance Criteria

**GIVEN** the user presses Cmd+H (or Ctrl+H on Windows) in the editor,  
**WHEN** the Find & Replace panel opens,  
**THEN** a panel slides in from the right (or drops down from the top on mobile) with: a "Find" input field, a "Replace With" input field, checkboxes for "Case Sensitive", "Whole Word", "Regular Expression", a "Block Types" multi-select filter (Scene Heading / Action / Character / Parenthetical / Dialogue / Transition / All), and buttons: "Find Next" (↓), "Find Previous" (↑), "Replace", "Replace All."

**GIVEN** the user types a search term in the "Find" field,  
**WHEN** they press Enter or click "Find Next",  
**THEN** the first matching occurrence is highlighted in amber in the editor; the status bar shows "1 of [N] matches"; the editor scrolls to the first match.

**GIVEN** the user clicks "Find Next" repeatedly,  
**WHEN** each click fires,  
**THEN** the next match in the script is highlighted; the status bar updates ("2 of N", "3 of N"…); when the last match is passed, it wraps around to the first match.

**GIVEN** the user enters a replacement term and clicks "Replace",  
**WHEN** the replacement fires,  
**THEN** the currently highlighted match is replaced with the replacement text; the Y.Doc is updated via the standard block edit path (same as typing); the content_source of the modified block is updated to `human_edited` if the block was previously `ai_generated`, otherwise stays `human_written`; the next match is automatically highlighted.

**GIVEN** the user clicks "Replace All",  
**WHEN** the replacement fires,  
**THEN** all matching occurrences across all in-scope blocks are replaced; a confirmation dialog first: "Replace all [N] occurrences of '[term]' with '[replacement]'? This action can be undone." On confirm, all replacements are applied atomically as a single Y.Doc transaction; a toast: "Replaced [N] occurrences."

**GIVEN** the "Block Types" filter is set to "Dialogue" only,  
**WHEN** the search runs,  
**THEN** only blocks with `block_type = 'dialogue'` are searched; matches in other block types are ignored; the status bar shows "1 of N (in Dialogue blocks)".

**GIVEN** the "Regular Expression" checkbox is enabled,  
**WHEN** the user types a valid regex pattern,  
**THEN** the search uses the regex for matching; invalid regex patterns show an inline error "Invalid regular expression" and the search is not triggered.

**GIVEN** the user clicks "Replace All" and then immediately presses Cmd+Z,  
**WHEN** the undo fires,  
**THEN** all replacements from the Replace All are reversed in a single undo step (single Y.Doc transaction is reverted); the script returns to its pre-replace state.

**GIVEN** there are 0 matches for a search term,  
**WHEN** the search runs,  
**THEN** the "Find" input border turns red; the status bar shows "No results"; the Find Next / Replace buttons are disabled.

**GIVEN** the Find & Replace panel is open and a collaborator is editing the same block,  
**WHEN** both actions occur simultaneously,  
**THEN** the Replace operation is applied to the current Y.Doc state (which includes the collaborator's edits); the CRDT merges correctly.

---

## Performance Criteria

- Find first match: < 100ms for a 120-page script
- Replace All (100 occurrences): < 500ms
- Find Next navigation: < 50ms per step
- Regex search: < 200ms

---

## Offline Behaviour

- Find & Replace works fully offline (all operations on in-memory Y.Doc and Drift `CachedBlocks`)
- Changes queue in `OfflineWriteQueue` for sync on reconnect

---

## Mobile Behaviour (375px)

- Find & Replace opens as a sheet at the top of the screen (below the status bar)
- "Find" and "Replace" fields in a horizontal pair; options in a scrollable row of chips below
- "Find Next / Previous" as arrow buttons to the right of the Find field
- "Replace All" as a full-width button at the bottom of the sheet

---

## Error States

**E1 — Replace in Locked Block**  
If a Replace All includes a locked block (story-010): the locked block is skipped; a warning toast: "2 locked blocks were skipped."

**E2 — Regex Catastrophic Backtracking**  
Regex with pathological backtracking (e.g., `(a+)+$`) times out after 500ms: show inline error "Regex timed out. Please simplify your expression."

---

## Security Considerations

- Find & Replace operates on the user's own blocks; RLS enforced on all PATCH calls
- Regex input is evaluated client-side with a timeout; no server-side regex execution

---

## Human Authorship Considerations

- All blocks modified by Replace are updated to `human_written` or `human_edited` as appropriate
- Replace does not alter `content_source` of `human_confirmed` blocks — they remain confirmed even after a replacement

---

## DB Tables Touched

- `blocks` (UPDATE content on replace — via standard block edit path)

---

## API Endpoints Used

- `PATCH /blocks/:id` — same as standard block edit (called per block for Replace)
- `PATCH /blocks/batch` — batch update for Replace All (single transaction)

---

## Test Cases

**TC-001:** Type "JAMES" in Find → assert all occurrences highlighted; status shows "N of M".  
**TC-002:** Replace single occurrence → assert only that one changes; content_source updates correctly.  
**TC-003:** Replace All 5 occurrences → confirm dialog → assert all 5 replaced; toast shows "5 replacements".  
**TC-004:** Cmd+Z after Replace All → assert all 5 reversed in single undo step.  
**TC-005:** Filter to "Dialogue" only → assert matches in Action blocks are not highlighted.  
**TC-006:** Enable Regex → type `[A-Z]+` → assert regex matching works.  
**TC-007:** Invalid regex → assert inline error shown.  
**TC-008:** No matches → assert input border red and status shows "No results".

---

## Out of Scope

- Cross-script Find & Replace (story-023 global search handles cross-script discovery)
- Regex replace groups (e.g., `$1` backreferences) — Phase 2
