# story-034 — AI Continue Scene (Writer's Block)

| Field | Value |
|---|---|
| **Story ID** | story-034 |
| **Title** | AI Continue Scene (Writer's Block) |
| **Epic** | E03 — AI Feature Suite |
| **Phase** | 2 |
| **Sprint** | Sprint 4 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-007 |

---

## Problem Statement

Writer's block at the scene level — knowing what the scene needs to achieve but not how to write the next 3–5 lines — is the most common productivity blocker for screenwriters. AI Continue Scene provides three continuation modes: Organic (continue the scene's established tone), Provocative (introduce an unexpected complication), and Minimalist (the least words to advance the scene goal). Each mode generates a draft that the writer must review and approve.

---

## User Story

As a screenwriter stuck on how to continue a scene, I want AI to generate 3 possible continuations in different styles — so that I can choose a direction that unsticks me and then edit it into my own voice.

---

## Acceptance Criteria

**GIVEN** the cursor is at the end of a scene (last block in the scene) or anywhere within a scene,  
**WHEN** the writer triggers "Continue Scene" (Cmd+Shift+C or the ✦ icon → "Continue Scene"),  
**THEN** a modal opens showing 3 mode cards: Organic, Provocative, and Minimalist; each card shows a brief description of what each mode generates; the writer selects a mode and clicks "Generate."

**GIVEN** the writer selects "Organic" and clicks "Generate",  
**WHEN** the job is dispatched via `POST /ai/continue-scene`,  
**THEN** the MCP service generates a continuation of 5–10 blocks (mix of action and dialogue) that maintains the scene's established tone, the characters' voices, and the emotional arc from Scene Intelligence; model: `claude-sonnet-4-5`.

**GIVEN** the writer selects "Provocative",  
**WHEN** the job runs,  
**THEN** the continuation introduces a new complication, an unexpected character reaction, or an ironic reversal — designed to raise the scene's conflict level; 5–10 blocks; same model.

**GIVEN** the writer selects "Minimalist",  
**WHEN** the job runs,  
**THEN** the continuation uses the minimum number of blocks to advance the scene's stated goal (from Scene Intelligence's story structure pillar); typically 3–5 blocks with short, punchy dialogue; same model.

**GIVEN** the continuation is generated (any mode),  
**WHEN** the result arrives,  
**THEN** a "Review Continuation" modal opens showing the generated blocks formatted in screenplay format (same visual as the editor); all blocks are marked with ✦ and `content_source = 'ai_generated'`; there are three actions at the bottom: "Accept All", "Insert as Draft", "Discard."

**GIVEN** the writer clicks "Accept All",  
**WHEN** the acceptance fires,  
**THEN** all generated blocks are inserted into the script after the last block of the current scene; `content_source = 'ai_generated'` on all inserted blocks; they appear in the editor with ✦ badges; the approval event is logged to `agent_approvals`.

**GIVEN** the writer clicks "Insert as Draft",  
**WHEN** the action fires,  
**THEN** a named draft is auto-saved with name "Before AI Continuation — [timestamp]"; the continuation blocks are inserted with `content_source = 'ai_generated'`; the writer can review, edit, and confirm each block individually; if they press "Undo", all inserted blocks are removed and the draft is preserved.

**GIVEN** the writer clicks "Discard",  
**WHEN** the dismissal fires,  
**THEN** no changes are made to the script; the modal closes; `agent_approvals` records `action = 'discarded'`.

**GIVEN** the Review Continuation modal is open,  
**WHEN** the writer wants to modify the continuation before inserting,  
**THEN** each generated block in the modal is individually editable; edits in the modal do not affect the script until "Accept All" or "Insert as Draft" is clicked.

**GIVEN** a Pro user triggers Continue Scene and they have used 500/500 AI requests today,  
**WHEN** the request is made,  
**THEN** the job is queued; the modal shows "You've reached your daily AI limit. This continuation will be generated after midnight UTC."

---

## Continuation Modes — MCP Prompt Strategy

| Mode | Prompt Directive | Max Blocks | Temperature |
|---|---|---|---|
| Organic | "Continue naturally, matching the scene's established tone and character voices. Do not introduce new characters or dramatic reversals." | 10 | 0.7 |
| Provocative | "Introduce an unexpected complication, ironic reversal, or new dramatic tension. Raise the stakes." | 10 | 0.9 |
| Minimalist | "Use the fewest possible words to advance the scene's narrative goal. Short dialogue, spare action." | 6 | 0.5 |

---

## Performance Criteria

- Mode selection modal open: < 100ms
- Continuation generation (claude-sonnet-4-5, 10 blocks): < 10 seconds
- Review modal render: < 300ms
- Block insertion ("Accept All"): < 500ms for 10 blocks

---

## Offline Behaviour

- Continue Scene requires internet
- Trigger while offline: "AI features require an internet connection."

---

## Mobile Behaviour (375px)

- Triggered by long-pressing at end of scene → "Continue Scene" in context menu
- Mode selection: a bottom sheet with 3 option cards in a vertical list
- Review modal: full-screen with scrollable preview of generated blocks

---

## Error States

**E1 — Generation Too Short**  
If the AI generates fewer than 3 blocks: retry once with a "minimum 5 blocks" instruction in the prompt; if still short, show the short result with a note: "Shorter continuation generated. You can request another."

**E2 — Generation Incoherent**  
If generated blocks contain a character name not in the story bible and not in the scene: highlight the unknown character name in red in the review modal with a warning: "Unknown character: [name]. Not in your Story Bible."

**E3 — AI Rate Limit**  
Handled same as story-033 (429 with queuing message).

---

## Security Considerations

- Same as story-033 — scene text sent to MCP stripped of PII
- Continuation is only inserted when the writer explicitly accepts it

---

## Human Authorship Considerations

- No continuation is ever auto-applied; always requires explicit "Accept All" or "Insert as Draft"
- All accepted blocks are `ai_generated` until individually edited (then `human_edited`)
- The draft auto-saved before insertion provides a recovery point
- The Human Authorship Principle mandates that the writer is always the one who decides whether generated content enters the script

---

## DB Tables Touched

- `blocks` (INSERT continuation blocks on acceptance)
- `drafts` (INSERT auto-save before "Insert as Draft")
- `agent_approvals` (INSERT for every accept/discard)
- `ai_usage` (SELECT + UPDATE)
- `scene_intelligence` (SELECT for scene context)

---

## API Endpoints Used

- `POST /ai/continue-scene` — trigger continuation job
- `POST /internal/ai/continue-scene/complete` — MCP callback

---

## Test Cases

**TC-001:** Trigger Continue Scene → select "Organic" → assert 10 blocks generated in < 10s.  
**TC-002:** "Accept All" → assert 10 blocks inserted with `content_source = 'ai_generated'` and ✦ badges.  
**TC-003:** "Insert as Draft" → assert auto-save created before insertion; blocks inserted.  
**TC-004:** "Discard" → assert script unchanged; `agent_approvals` records discard.  
**TC-005:** Edit block in review modal → accept → assert edited content inserted (not original AI output).  
**TC-006:** Undo after "Insert as Draft" → assert all inserted blocks removed.  
**TC-007:** AI returns unknown character → assert warning shown in review modal.

---

## Out of Scope

- AI-generated entire scenes from scratch (Phase 2)
- Multi-scene continuation (one call generates 3+ scenes) — Phase 2
