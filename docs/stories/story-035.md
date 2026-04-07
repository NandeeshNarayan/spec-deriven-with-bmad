# story-035 — AI Continuity & Logic Checker

| Field | Value |
|---|---|
| **Story ID** | story-035 |
| **Title** | AI Continuity & Logic Checker |
| **Epic** | E03 — AI Feature Suite |
| **Phase** | 2 |
| **Sprint** | Sprint 5 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Priya (the TV staff writer) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-007 |

---

## Problem Statement

TV scripts span many episodes, and feature scripts develop over multiple drafts. Continuity errors — a character whose hair changes between scenes, a prop mentioned in Act 1 that contradicts a scene in Act 3, a character who is supposedly in two places at once — are easy to make and hard to catch manually across 120 pages. The AI Continuity Checker runs as a background job and surfaces all detected continuity flags with specific scene and line references.

---

## User Story

As a TV staff writer, I want an AI Continuity Checker that scans my entire script for logic errors, timeline contradictions, and character inconsistencies — so that I catch these errors before the script reaches production.

---

## Acceptance Criteria

**GIVEN** a Pro or Studio user clicks "Run Continuity Check" (from the AI tools menu or the editor header),  
**WHEN** the check is triggered via `POST /ai/continuity-check`,  
**THEN** a BullMQ job is enqueued in the MCP service; the checker runs on the full script text (all scenes and blocks); model: `claude-sonnet-4-5`; the check runs in the background and does not block the editor.

**GIVEN** the continuity check completes,  
**WHEN** the user opens the "Continuity" report panel,  
**THEN** all detected issues are listed, grouped by category: Timeline Contradictions, Character Inconsistencies, Object Continuity (props/costumes appearing/disappearing), Location Logic, Physical Impossibilities; each issue shows: severity (Critical / Warning / Note), a description, and the specific scene(s) / block(s) involved.

**GIVEN** a continuity issue is listed,  
**WHEN** the user clicks the issue,  
**THEN** the editor scrolls to the relevant scene and highlights the specific block(s); the issue remains visible in the panel.

**GIVEN** the user reviews an issue and determines it is intentional (e.g., a continuity break is a narrative device),  
**WHEN** they click "Dismiss" on the issue,  
**THEN** the issue is marked as `dismissed = true` in the `continuity_flags` table; it moves to a "Dismissed" section; future continuity checks do not re-flag it unless the relevant blocks change.

**GIVEN** the user fixes the issue in the editor,  
**WHEN** the fix is saved,  
**THEN** the issue remains in the report (as it was flagged for the previous state); a "Re-check" button appears on the issue card; clicking it re-analyses just the relevant blocks to verify the fix.

**GIVEN** the continuity check detects a character appearing in two locations simultaneously,  
**WHEN** the flag is shown,  
**THEN** it shows as "Critical" severity; the flag reads: "JAMES appears in Scene 14 (INT. OFFICE) and Scene 15 (EXT. PARK) with no travel time. If these scenes are concurrent, this is a logic error."

**GIVEN** a Free user tries to run the Continuity Checker,  
**WHEN** the request is made,  
**THEN** a paywall modal: "Continuity Checker is available on Pro. Upgrade to unlock full-script analysis."

**GIVEN** the continuity check is re-run after fixes are made,  
**WHEN** a new check is triggered,  
**THEN** previously dismissed issues are preserved as dismissed; only new, unresolved, and undismissed issues are shown in the Active Issues section.

**GIVEN** a script has no continuity issues,  
**WHEN** the check completes,  
**THEN** the report shows "✓ No continuity issues found." with a green checkmark; the check date is shown.

---

## Performance Criteria

- Job enqueue to start: < 500ms
- Full 120-page script continuity check: < 30 seconds
- Continuity report panel load (50 flags): < 300ms
- Editor scroll to flagged block: < 100ms

---

## Offline Behaviour

- Previously generated reports viewable offline (cached in Drift)
- Running new checks requires internet

---

## Mobile Behaviour (375px)

- Continuity report opens as a full-screen panel from the AI tools bottom sheet
- Issues grouped in accordion sections (collapsed by default)
- Each issue: severity badge + one-line description + "Jump" button

---

## Error States

**E1 — Check Times Out**  
MCP job exceeds 60s for a very large script: partial results returned for completed scenes; remaining scenes show "Analysis incomplete — script too large for a single pass. Run scene-by-scene analysis."

**E2 — Check Produces Zero Output**  
Model returns an empty analysis: retry once; if still empty, show "Could not complete analysis. Please try again."

---

## Security Considerations

- Full script text sent to MCP service; PII-stripped as per standard AI security rules
- Continuity flags stored with RLS — only collaborators can view
- Studio plan only allows `check_count` per day (same `ai_usage` counter applies — this counts as 5 AI requests regardless of actual model calls)

---

## Human Authorship Considerations

- Continuity flags are advisory only — they never change script content
- The writer always decides whether to fix or dismiss
- All flags logged to `continuity_flags` with `content_source = 'ai_generated'`

---

## DB Tables Touched

- `continuity_flags` (INSERT new flags, UPDATE dismissed status, SELECT for report)
- `blocks` (SELECT — full script text)
- `scenes` (SELECT — for scene reference in flags)
- `ai_usage` (SELECT + UPDATE — counts as 5 AI requests)

---

## API Endpoints Used

- `POST /ai/continuity-check` — trigger full-script check
- `GET /scripts/:id/continuity-flags` — load flag report
- `PATCH /continuity-flags/:id/dismiss` — dismiss a flag
- `POST /continuity-flags/:id/recheck` — re-analyse specific blocks for a flag

---

## Test Cases

**TC-001:** Script with known character location conflict → run check → assert Critical flag raised.  
**TC-002:** Dismiss a flag → assert it moves to Dismissed section.  
**TC-003:** Fix an issue → re-check that flag → assert it clears if fix is confirmed.  
**TC-004:** Click issue in report → assert editor scrolls to relevant scene.  
**TC-005:** Free user tries to run check → assert Pro paywall shown.  
**TC-006:** Script with no issues → assert "No continuity issues found" with checkmark.  
**TC-007:** Re-run check after fixes → assert dismissed items stay dismissed.

---

## Out of Scope

- Episodic continuity across multiple episode scripts (Phase 2)
- Real-time continuity checking as the writer types (Phase 2)
