# story-038 — AI Character Voice Guardian

| Field | Value |
|---|---|
| **Story ID** | story-038 |
| **Title** | AI Character Voice Guardian |
| **Epic** | E03 — AI Feature Suite |
| **Phase** | 2 |
| **Sprint** | Sprint 5 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Priya (the TV staff writer) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-007, story-016 |

---

## Problem Statement

In TV writers' rooms, multiple writers often write different episodes — but all characters must sound consistent across a season. The Voice Guardian passively monitors dialogue blocks as they are written and flags lines that sound "off-character" compared to the character's established voice in the Story Bible and their dialogue history. Crucially, the Voice Guardian never rewrites dialogue — it only flags and suggests.

---

## User Story

As a TV staff writer, I want a passive Character Voice Guardian that flags when a character's dialogue sounds inconsistent with their established voice — so that all characters sound consistent whether written by me or my co-writers.

---

## Acceptance Criteria

**GIVEN** a writer finishes a Dialogue block (cursor leaves the block) and the block has > 5 words,  
**WHEN** the voice check triggers (debounced 2 seconds after cursor leaves the block),  
**THEN** the Rust API enqueues a lightweight voice-check job via `POST /ai/voice-check`; model: `claude-haiku-3-5` (fast model for passive check); the check uses: character's Story Bible profile, their 20 most recent dialogue lines in the script.

**GIVEN** the voice check returns a flag (confidence > 0.7 that the line is off-character),  
**WHEN** the flag is received,  
**THEN** a subtle amber underline appears below the dialogue block (different from the comment amber underline); hovering the underline (desktop) or long-pressing (mobile) shows a tooltip: "This line may not match [CHARACTER]'s voice. [Brief explanation]."

**GIVEN** the voice flag tooltip is visible,  
**WHEN** the writer taps "View Details",  
**THEN** a Voice Guardian card opens in the AI sidebar showing: the specific line, the character's established voice traits (from Story Bible), why this line seems inconsistent (e.g., "This character typically uses short, clipped sentences. This line is unusually verbose."), and a "Dismiss" button; NO rewrite suggestion is offered (per Human Authorship Principle).

**GIVEN** the writer reviews the voice flag and decides the line is intentional,  
**WHEN** they click "Dismiss",  
**THEN** the amber underline disappears; `content_source` remains `human_written`; the dismiss is logged; future voice checks for this specific block are suppressed for 7 days.

**GIVEN** the writer reviews the flag and rewrites the dialogue themselves,  
**WHEN** the block content changes,  
**THEN** the amber underline disappears automatically (the block's content hash changed); a new voice check will run when the cursor leaves the block again.

**GIVEN** a character has fewer than 10 dialogue lines in the entire script,  
**WHEN** a voice check is triggered for that character,  
**THEN** no flag is raised (insufficient data for voice profiling); the check passes silently.

**GIVEN** the Voice Guardian is enabled (Pro/Studio only),  
**WHEN** a Free user finishes a Dialogue block,  
**THEN** no voice check is triggered; no amber underline appears.

**GIVEN** the writer is in Focus Mode (story-006),  
**WHEN** a voice flag would normally appear,  
**THEN** the amber underline is suppressed during Focus Mode; the flag is queued and all flags are shown when Focus Mode is exited.

**GIVEN** the Voice Guardian panel is open (from editor sidebar → "Voice" tab),  
**WHEN** the panel loads,  
**THEN** all active voice flags across the script are listed with: character name, scene reference, flagged line (truncated to 80 chars), confidence score, and "Jump" button.

---

## Performance Criteria

- Voice check job enqueue: < 100ms
- Voice check (claude-haiku-3-5): < 2 seconds
- Amber underline appearance: < 100ms after flag received
- Voice Guardian panel load: < 200ms

---

## Offline Behaviour

- Voice checks do not run offline
- Existing flags (cached) are visible in the Voice Guardian panel offline
- No new amber underlines appear offline

---

## Mobile Behaviour (375px)

- Voice flag: amber underline visible; long-press shows tooltip
- "View Details" opens a bottom sheet with the Voice Guardian card
- Voice Guardian panel accessible from AI tools bottom sheet

---

## Error States

**E1 — Voice Check Rate Limit**  
If the Voice Guardian is running too many checks per minute (> 30 checks per user per minute): queue excess checks; process them at 30/minute; no user-visible error.

**E2 — No Story Bible Profile**  
Character has a Story Bible entry but no voice traits filled in: voice check runs against dialogue history only; tooltip notes: "Voice analysis based on dialogue history only (no profile)."

---

## Security Considerations

- Voice check job sends the single dialogue line and character profile — minimal data
- Voice Guardian flags do NOT modify script content ever — they are purely advisory
- Pro/Studio-only feature; enforced server-side

---

## Human Authorship Considerations

- The Voice Guardian NEVER suggests rewrites or alternative lines
- It only flags and explains — the rewrite decision is 100% the writer's
- This is a core compliance with the Human Authorship First Principle (§0 of constitution.md)
- All flags are `content_source = 'ai_generated'` but they are annotations, not script content

---

## DB Tables Touched

- Voice flags stored in `continuity_flags` table with `flag_type = 'voice'`
- `blocks` (SELECT dialogue content and character name)
- `story_bible_entries` (SELECT character voice traits)
- `ai_usage` (UPDATE — counts as 0.2 AI requests per check — 5 voice checks = 1 AI request)

---

## API Endpoints Used

- `POST /ai/voice-check` — trigger voice check for a single block
- `GET /scripts/:id/voice-flags` — list all active voice flags
- `PATCH /voice-flags/:id/dismiss` — dismiss a flag

---

## Test Cases

**TC-001:** Write out-of-character dialogue → wait 2s → assert amber underline appears.  
**TC-002:** Hover amber underline → assert tooltip with character name and brief explanation.  
**TC-003:** "View Details" → assert Voice Guardian card shows voice traits; no rewrite suggestion shown.  
**TC-004:** Dismiss flag → assert underline disappears; same block not re-flagged for 7 days.  
**TC-005:** Rewrite flagged dialogue → assert underline disappears automatically.  
**TC-006:** Character with < 10 lines → write dialogue → assert no flag raised.  
**TC-007:** Free user → write dialogue → assert no voice check triggered.  
**TC-008:** Focus Mode active → voice flag triggered → assert underline suppressed; appears on Focus Mode exit.

---

## Out of Scope

- Automated voice rewriting (explicitly out of scope — Human Authorship Principle)
- Cross-episode voice consistency (Phase 2)
